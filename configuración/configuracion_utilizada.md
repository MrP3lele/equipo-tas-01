# Configuración utilizada

Este índice identifica rutas y ajustes observados. Las configuraciones con secretos no se copian al repositorio.

| VM | Archivo o ajuste de origen | Propósito |
| --- | --- | --- |
| VMFabian | `/etc/bind/db.equipo-tas-01.local` | Zona directa y subdominios de monitoreo. |
| VMFabian | `/etc/bind/db.10`, `/etc/bind/named.conf.local` | Zona inversa y declaraciones de zonas. |
| VMFabian | `/etc/haproxy/haproxy.cfg` | HTTPS, rutas de monitoreo y balanceo de WordPress. |
| VMFabian | `/root/equipo-tas-pki/server.ext`, `/etc/haproxy/certs/equipo-tas-01.local.pem` | SAN y certificado. El PEM contiene una clave privada y no se publica. |
| VMFabian | `/home/fabi/monitoreo/docker-compose.yml` | Despliegue existente de Grafana y Prometheus; consultado, sin cambio acreditado en la intervención de subdominios. |
| VMMaty | `/etc/nginx/sites-available/wordpress`, `/var/www/html/health.php` | Sitio PHP y chequeo de HAProxy. |
| VMMaty | `/var/www/html/wp-config.php` | Destino de BD y configuración sensible excluida. |
| VMMaty | `/etc/fstab` y dependencia de montaje de Nginx | Montaje NFS al arranque y espera antes de iniciar el sitio. |
| VMMaty2 | `/etc/nginx/sites-enabled/wordpress`, `/var/www/html/wp-config.php` | Sitio WordPress y destino de BD; confirmar ruta de `sites-available` antes de copiar. |
| VMMaty2 | `/etc/fstab` | Montaje NFS con `x-systemd.automount` observado en la revisión de persistencia. |
| VMOmar | Configuración de MariaDB y cuentas SQL | Escucha en `10.33.199.11:3306`; cuentas limitadas a los dos backends. Ruta exacta del archivo editado por confirmar. |
| VMNFS | `/etc/exports`, `/srv/nfs/wp-uploads` | Exportación NFS a `.12` y `.13`. |

Los registros DNS, ACL y opciones de exportación se resumen en [estado del nodo](../docs/estado-nodo-sur.md). No se incorporan copias completas de configuración sin leerlas y sanearlas en su VM de origen.
