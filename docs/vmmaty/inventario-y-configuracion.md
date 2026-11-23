# Inventario y configuración de VMMaty

**Actualización posterior:** desde octubre WordPress usa MariaDB en `10.33.199.11` y `uploads` por NFS en `10.33.199.14`. La consola de noviembre mostró el montaje activo y su dependencia de Nginx. El cuadro siguiente es una fotografía del **7 de septiembre**, antes de esos cambios. Ver [estado del nodo](../estado-nodo-sur.md).

Estado documentado con salidas y capturas compartidas hasta el **7 de septiembre de 2026**. Responsable técnico de la VM: Maty. CPU, RAM, disco, versión exacta de WordPress y versión del motor de BD requieren una lectura actual antes de completarse.

## Identidad y servicios

| Elemento | Valor conocido |
| --- | --- |
| VM | VMMaty, identificador 237, nodo Proxmox beta |
| Sistema | Ubuntu Server 24.04; revisión de paquetes pendiente |
| Hostname y usuario | `mati` |
| Interfaz de acceso | `ens18`, `10.33.195.162/25` |
| Interfaz privada | `ens19`, `10.33.199.12/29` |
| Red privada | `10.33.199.8/29`; gateway indicado `10.33.199.9`; `.15` es broadcast |
| Web | Nginx 1.24, HTTP en puerto 80; sitio `wordpress` |
| PHP | Servicio `php8.3-fpm`; socket `/run/php/php8.3-fpm.sock` |
| WordPress | Raíz `/var/www/html`; opciones `home` y `siteurl`: `https://equipo-tas-01.local` |
| Base de datos | Nombre `wordpress`, prefijo `wp_`, `DB_HOST=localhost` |
| Administración de WordPress | WP-CLI en `/usr/local/bin/wp`, ejecutado como `www-data` |
| DNS inverso | Secundario de `199.33.10.in-addr.arpa` |

No se acredita una modificación de hostname o netplan en la etapa aquí documentada. Mostrar una IP con `ip address` no prueba por sí solo su persistencia tras reiniciar.

## Ubicación de los archivos

| Ruta en VMMaty | Función y tratamiento |
| --- | --- |
| `/etc/bind/named.conf.local` | Declaración de la zona inversa secundaria y su primario `10.33.199.10`. |
| `/var/cache/bind/db.10.33.199` | Ruta resultante del nombre relativo `db.10.33.199` si `directory` es `/var/cache/bind`; confirmar antes de asumirla. Copia administrada por BIND, no editar a mano. |
| `/etc/nginx/nginx.conf` | Configuración global e inclusión de los sitios; no se acredita un cambio en este archivo. |
| `/etc/nginx/sites-available/wordpress` | Sitio al que se añadieron los nombres del equipo y el tratamiento exacto de `/health.php`. |
| `/etc/nginx/sites-enabled/wordpress` | Enlace preexistente al sitio anterior; no es otra configuración independiente. |
| `/etc/nginx/snippets/fastcgi-php.conf` | Fragmento incluido para PHP; no se acredita una modificación propia. |
| `/var/www/html/health.php` | Archivo añadido para comprobar PHP y la conexión de BD; acceso limitado al proxy privado. |
| `/var/www/html/wp-config.php` | Configuración sensible. Se indicó reconocimiento de HTTPS desde el proxy y `FORCE_SSL_ADMIN`; el comportamiento HTTPS se validó. No se publica el archivo real. |
| `/usr/local/bin/wp` | Ejecutable WP-CLI instalado para consultar y administrar esta instalación. |
| `/run/php/php8.3-fpm.sock` | Socket generado por PHP-FPM; no se versiona ni se edita como archivo. |
| `/root/respaldo-wp-dominio.dyXU0l` | Respaldo histórico privado de configuración y BD. Su contenido no pertenece al repositorio. |

La ruta del backup se documenta para localizarlo en el servidor. El commit no certifica que siga existiendo ni que ya se haya probado su restauración.

## Configuración y dependencias

### DNS inverso

La declaración secundaria de Maty utiliza `primaries { 10.33.199.10; };`. El resultado local conocido fue el PTR `backend1.equipo-tas-01.local.` para `10.33.199.12`, con serial SOA `2026090501`.

La zona directa anterior `mati-tas-01.local` se conservó durante la transición. No se ha demostrado que Maty sirva como secundario de la nueva zona directa del equipo. La consulta del dominio publicado usa el DNS de entrada del proyecto.

### Publicación web

El navegador accede al proxy del equipo en `10.33.195.224:443`. Este envía las solicitudes a `10.33.199.12:80`; Nginx entrega PHP al socket de PHP-FPM. WordPress consulta su BD local.

El sitio mantiene `/var/www/html` y el socket PHP 8.3. Se incorporaron `equipo-tas-01.local` y `www.equipo-tas-01.local`, conservando temporalmente los nombres anteriores. La URL almacenada en la BD también se actualizó: editar únicamente `server_name` no habría realizado ese cambio.

### Salud y esquema HTTPS

`/health.php` se restringió al origen privado `10.33.199.10`. Su comprobación utiliza una carga mínima de WordPress y `SELECT 1`; un resultado 200 no demuestra que funcionen todos los plugins ni una compra completa.

El ajuste indicado en `wp-config.php` acepta `X-Forwarded-Proto=https` únicamente desde esa IP del proxy, comunica HTTPS a WordPress y exige HTTPS para la administración. No se ha obtenido un diff nuevo del archivo completo. Si se cambia el tratamiento de `REMOTE_ADDR`, esta confianza debe revisarse.

**Esta configuración no cifra el tramo HTTP privado.** La BD futura en `10.33.199.11` es una dependencia prevista, no el destino confirmado de la instalación documentada.
