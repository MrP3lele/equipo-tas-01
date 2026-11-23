# Estado y arquitectura del Nodo Sur

**Corte documental: 23 de noviembre de 2026.** Se combinaron salidas compartidas durante la implementación, el registro de subdominios de ese día y la lectura visual de las consolas de Proxmox. Una consola abierta muestra el estado visible y comandos anteriores; no equivale a releer todos los archivos de la VM ni a probar un reinicio.

## Flujo de una solicitud

1. Un cliente que alcanza la red del laboratorio o entra por VPN consulta `equipo-tas-01.local` al BIND de VMFabian (`10.33.195.224`). El registro A devuelve esa misma IP. `grafana` y `prometheus` también apuntan a ella.
2. HAProxy recibe HTTP en 80 y lo redirige a HTTPS en 443. Para WordPress termina TLS y usa `roundrobin` hacia `10.33.199.12:80` y `10.33.199.13:80`, según los chequeos GET `/health.php` con estado 200.
3. Cada backend usa Nginx, PHP-FPM y su propia copia de WordPress. `home` y `siteurl` se migraron al dominio HTTPS. Los dos consultan la misma BD `wordpress` en `10.33.199.11:3306` mediante el usuario `wp_user`.
4. Los archivos de `wp-content/uploads` residen en VMNFS (`10.33.199.14:/srv/nfs/wp-uploads`) y se montan en los dos backends. El código, plugins y temas de WordPress permanecen en cada backend y requieren despliegue coordinado; NFS no sincroniza esos archivos.
5. Las ACL de HAProxy envían `grafana.equipo-tas-01.local` a `127.0.0.1:3000` y `prometheus.equipo-tas-01.local` a `127.0.0.1:9090` en VMFabian. Prometheus recopila métricas del conjunto; Grafana las visualiza.

El tramo externo cliente → HAProxy usa HTTPS con una CA interna. El tramo observado HAProxy → backends WordPress usa HTTP en la red privada. La confianza de la CA en Windows no cifra ese tramo. No se verificó TLS entre backends y MariaDB.

## Inventario y evidencia

| VM | Servicio y red observados | Evidencia y límite |
| --- | --- | --- |
| VMFabian, 236 | `10.33.195.224`, `10.33.199.10`; BIND, HAProxy, Grafana, Prometheus | `named` y `haproxy` activos, sintaxis válida, dos backends en configuración, subdominios y HTTPS probados localmente. La consola del 23-11 mostró las pruebas ya ejecutadas. |
| VMOmar, 239 | `10.33.195.225`, `10.33.199.11`; MariaDB | El puerto 3306 escuchaba en `.11` y `mariadb` figuraba habilitado. Las cuentas `wp_user` para `.12` y `.13` fueron verificadas en octubre. |
| VMMaty, 237 | `10.33.195.162`, `10.33.199.12`; Nginx, PHP 8.3, WordPress | `DB_HOST=.11`, NFS montado, TCP a BD y HTTP 200. En noviembre se vieron entrada de montaje, dependencia de Nginx, prueba de escritura NFS y HTTP 200. |
| VMMaty2, 311 | `10.33.195.228`, `10.33.199.13`; Nginx, PHP 8.1, WordPress | `DB_HOST=.11`, NFS montado, TCP a BD y HTTP 200. En noviembre se vieron `x-systemd.automount`, dependencia de montaje de Nginx y HTTP 200. |
| VMNFS, 312 | `10.33.195.166`, `10.33.199.14`; NFS | `/etc/exports` autoriza solo `.12` y `.13`; `nfs-server` activo y habilitado en salidas compartidas. La consola del 23-11 mostró `enabled` y el mismo archivo de exportación. |

Las IP de acceso `10.33.195.*` pueden variar si no existe reserva o configuración fija; las direcciones indicadas son las observadas. Las IP privadas `.10` a `.14` pertenecen a `10.33.199.8/29` y fueron usadas para las dependencias internas.

## Rutas y cambios por VM

### VMFabian

| Ruta | Cambio o uso acreditado |
| --- | --- |
| `/etc/bind/named.conf.local` | Declaraciones de las zonas directa e inversa. La zona antigua `tas-01.local` coexistía en septiembre. |
| `/etc/bind/db.equipo-tas-01.local` | Se añadieron registros A para `grafana` y `prometheus` a `10.33.195.224`; serial observado `2026112301`. |
| `/etc/bind/db.tas-01.local` | En septiembre se corrigió un registro fuera de zona; se conserva como antecedente histórico. |
| `/etc/bind/db.10` | PTR de la red privada; `10.33.199.12` devolvía `backend1.equipo-tas-01.local.`. |
| `/etc/haproxy/haproxy.cfg` | Frontend HTTP/HTTPS, selección por nombre, backends de monitoreo y dos servidores WordPress con `roundrobin` y chequeo `/health.php`. |
| `/root/equipo-tas-pki/server.ext` | Se ampliaron los SAN a Grafana y Prometheus, conservando los nombres web anteriores. |
| `/etc/haproxy/certs/equipo-tas-01.local.pem` | Se instaló el certificado renovado. Contiene clave privada: nunca copiar al repositorio. |
| `/home/fabi/monitoreo/docker-compose.yml` | Despliegue existente de Grafana/Prometheus; leído, sin modificación acreditada en la intervención de subdominios. |

Antes de la intervención de subdominios se copiaron los archivos afectados a `/root/subdominios-20261123.5hsNu4/`. En la verificación local, `named-checkconf -z`, `haproxy -c`, `dig`, `openssl verify` y GET HTTPS con `--cacert` dieron resultados correctos. Prometheus devolvió 405 al usar HEAD en `/targets`; la comprobación GET de `/api/v1/targets` devolvió 200.

### VMMaty y VMMaty2

| Ruta o ajuste | Cambio o uso acreditado |
| --- | --- |
| `/var/www/html/wp-config.php` en ambas | `DB_HOST=10.33.199.11`, `DB_NAME=wordpress`, `DB_USER=wp_user`. Excluir archivo real por contraseña y salts. |
| `/etc/nginx/sites-available/wordpress` en VMMaty | Sitio del dominio `equipo-tas-01.local`, PHP 8.3 y chequeo de salud. |
| `/etc/nginx/sites-enabled/wordpress` en VMMaty2 | Sitio observado con `server_name equipo-tas-01.local www.equipo-tas-01.local`, raíz `/var/www/html` y socket `/run/php/php8.1-fpm.sock`. |
| `/var/www/html/health.php` en ambas | Ruta del chequeo de HAProxy. Desde localhost en VMMaty2 devolvió 403 en la revisión de noviembre, coherente con la restricción de origen; HAProxy debe comprobarla desde `.10`. |
| `/etc/fstab` en ambas | Entrada NFS a `/var/www/html/wp-content/uploads`. En VMMaty2 se observó `x-systemd.automount`; en ambas, la consola mostró montaje activo y Nginx dependiente del punto de montaje. |
| `/var/www/html/wp-content/uploads` | Montaje compartido desde `.14`; no guardar el paquete de despliegue dentro de esta ruta. |

En VMMaty se cambió el dominio de las opciones de WordPress desde `http://www.mati-tas-01.local` a `https://equipo-tas-01.local` tras un respaldo privado. Ese procedimiento y sus resultados históricos se detallan en [docs/vmmaty](vmmaty/README.md).

En VMMaty2 se copió el árbol del sitio desde el backend 1. El primer intento de archivo `wp-site.tgz` en el propio NFS fue incompleto; se volvió a crear en `/tmp` y se transfirió después. Faltó inicialmente la extensión PHP `mysqli`, lo que causó HTTP 500. Tras corregir esa dependencia y el acceso TCP a MariaDB, el sitio local pasó a HTTP 200. La consola de noviembre mostró `/etc/nginx/sites-enabled/wordpress`, HTTP 200 y montaje NFS activo.

### VMOmar y VMNFS

La escucha de MariaDB se redujo de `0.0.0.0:3306` a `10.33.199.11:3306`. Las cuentas `wp_user` vistas en octubre tenían host `.12` y `.13`; el respaldo privado `/root/wordpress-pre-cutover.sql` figuraba con 2,5 MB. No hay restauración de prueba acreditada ni copia saneada del archivo de configuración de MariaDB.

`/etc/exports` de VMNFS contiene `/srv/nfs/wp-uploads` para `10.33.199.12` y `10.33.199.13` con `rw,sync,no_subtree_check,all_squash,anonuid=33,anongid=33`. `all_squash` mapea escrituras al usuario/grupo `www-data` (UID/GID 33). Los dos backends vieron el mismo directorio. Una prueba de escritura y lectura en el montaje de VMMaty quedó registrada en la consola de noviembre.

## Acceso desde Windows y VPN

El cliente debe alcanzar `10.33.195.224` por la red universitaria o VPN. Cuando la VPN estaba desconectada, las consultas a DNS y HTTPS agotaban el tiempo; con la VPN, las pruebas TCP a 53 y 443 y `nslookup ... 10.33.195.224` funcionaron. `equipo-tas-01.local` es un nombre interno y los DNS de la universidad no lo resuelven por sí solos.

El 23-11 se añadió temporalmente en Windows una regla NRPT para `equipo-tas-01.local` y sus subdominios hacia `10.33.195.224`. La resolución predeterminada funcionó y los tres sitios respondieron con 200, 200 y redirección 302, respectivamente, al probar la aplicación por HTTPS sin validar la CA. **A solicitud del propietario se eliminó la regla y se limpió la caché DNS**; se verificó que no quedó ninguna regla para ese dominio. Los servidores DNS de Wi-Fi y VPN conservaron sus valores. Windows todavía no confía en la CA interna: `curl` con validación informó `SEC_E_UNTRUSTED_ROOT`. No se instaló ninguna CA en Windows.

El sufijo `.local` puede interferir con mDNS. Para una presentación hay que comprobar resolución con el cliente real y decidir conscientemente cómo confiar en la CA interna.

## Pendientes que no deben declararse cumplidos

- Restaurar respaldos en un destino de prueba y medir recuperación; la existencia de SQL o copias de archivos no basta.
- Reiniciar de forma controlada las VM y comprobar montaje NFS, dependencias, MariaDB, HAProxy y monitoreo después del arranque.
- Ejecutar y registrar un fallo real de cada backend desde la web; la configuración `roundrobin` y los chequeos `up` no sustituyen esa prueba.
- Verificar compra de prueba y plugins de WooCommerce en ambos backends, además de despliegue coordinado de código y sesiones si la aplicación lo requiere.
- Revisar permisos de acceso a Grafana y Prometheus, confianza de CA de clientes y cifrado de los tramos internos según la rúbrica.

No se confirma aquí la eliminación de PrestaShop, una CA instalada en clientes ni una política de respaldos restaurables. Tampoco se publican archivos de configuración completos sin una copia saneada y revisada del origen.
