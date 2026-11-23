# VMMaty — backend WordPress 1

VMMaty (`10.33.195.162`, `10.33.199.12`) ejecuta Nginx, PHP 8.3, WordPress y WooCommerce en `/var/www/html`. El sitio se configuró en `/etc/nginx/sites-available/wordpress`; `/var/www/html/health.php` comprueba la salud mínima para HAProxy.

La configuración sensible `/var/www/html/wp-config.php` pasó de `DB_HOST=localhost` en septiembre a `DB_HOST=10.33.199.11` en octubre. El nombre observado fue `wordpress` y el usuario `wp_user`. No se publica el archivo porque contiene contraseña y claves de WordPress.

Los archivos subidos se comparten por NFS desde `10.33.199.14:/srv/nfs/wp-uploads` en `/var/www/html/wp-content/uploads`. En la revisión de persistencia se observó el montaje activo, una entrada NFS en `/etc/fstab` y una dependencia de montaje para Nginx; Nginx y `php8.3-fpm` estaban habilitados. No consta un reinicio posterior en la evidencia conservada.

El 23 de octubre se observó TCP a `.11:3306` y HTTP 200 del sitio local. La consola de Proxmox mostró el montaje NFS y HTTP 200 en noviembre. La etapa inicial y los procedimientos están en [docs/vmmaty](../../docs/vmmaty/README.md).
