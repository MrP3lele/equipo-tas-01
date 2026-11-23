# VMMaty2: el segundo backend

VMMaty2 (`10.33.199.13`) sirve el mismo WordPress que VMMaty. Tiene su propia copia del código, plugins y temas, pero usa la misma base de datos en VMOmar y el mismo directorio de archivos subidos en VMNFS. Esa combinación permite que HAProxy elija cualquiera de los dos backends.

[Volver al inicio](../../README.md) · [Comparar con VMMaty](../vmmaty/README.md)

## Cómo llegó a funcionar

Al principio solo aparecía la página «Welcome to nginx!»: Nginx estaba activo, pero WordPress aún no estaba instalado en esta VM. Se copió el sitio desde VMMaty. El primer paquete `wp-site.tgz` se creó dentro del directorio NFS y quedó incompleto; `tar` avisó `Unexpected EOF`. Se rehizo el paquete en `/tmp`, se transfirió y después se comprobó la presencia de `wp-config.php` y `wp-load.php`.

La primera respuesta de WordPress fue HTTP 500. El propio sitio indicó que faltaba `mysqli`, la extensión de PHP que necesita para hablar con MariaDB. En esta VM el servicio es `php8.1-fpm`, no `php8.3-fpm` ni un servicio llamado `php-fpm`. Después apareció otro problema distinto: el puerto 3306 de VMOmar no respondía desde `.13`. Una vez corregida esa conectividad, la prueba TCP y la página local devolvieron el resultado esperado: HTTP 200.

## Dónde mirar la configuración

| Ruta o servicio | Qué revisar |
| --- | --- |
| `/etc/nginx/sites-enabled/wordpress` | Nombre del dominio, raíz `/var/www/html` y socket `/run/php/php8.1-fpm.sock`. La consola mostró este sitio activo. |
| `/var/www/html/wp-config.php` | `DB_HOST=10.33.199.11`, base `wordpress` y usuario `wp_user`. No copiar el archivo completo: contiene secretos. |
| `/var/www/html/health.php` | Chequeo usado por HAProxy. Desde localhost devolvió 403 en una prueba; hay que consultarlo desde VMFabian para esperar 200. |
| `/etc/fstab` | Montaje de `10.33.199.14:/srv/nfs/wp-uploads` en `/var/www/html/wp-content/uploads`. En noviembre se vio `x-systemd.automount`. |
| `nginx` y `php8.1-fpm` | Ambos servicios aparecieron activos; Nginx mostró una dependencia hacia el montaje NFS. |

La consola de noviembre mostró NFS montado, `nginx -t` correcto y HTTP 200 de la raíz. Son pruebas útiles, aunque todavía no registran un reinicio completo. Los [fragmentos de configuración](../../configuración/VMMaty2/readme.md) permiten comparar el sitio Nginx sin publicar la contraseña de WordPress.

## Qué no sincroniza el NFS

El NFS comparte `wp-content/uploads`: imágenes, medios y archivos que WordPress guarda allí. Si se actualiza un plugin, un tema o el código en un backend, el otro no recibe ese cambio automáticamente. Antes de activar ambos en el balanceador conviene comprobar que sus versiones y archivos de aplicación coincidan. Para revisar esta VM, usa la [guía de verificación](../verificacion-infraestructura.md#ambos-backends-web-bd-y-nfs).
