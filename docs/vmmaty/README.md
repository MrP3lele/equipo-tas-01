# VMMaty: el primer backend

VMMaty fue el primer servidor que puso en marcha WordPress y WooCommerce. Hoy comparte el trabajo con VMMaty2: HAProxy puede enviarle visitas a cualquiera de los dos. VMMaty conserva su propia copia del sitio, consulta la base de datos de VMOmar y usa los archivos subidos que entrega VMNFS.

[Volver al inicio](../../README.md) · [Conocer VMMaty2](../vmmaty2/README.md)

## Qué hay en esta VM

La dirección privada es `10.33.199.12` y la IP de acceso observada fue `10.33.195.162`. Nginx sirve el sitio desde `/var/www/html`; PHP usa `php8.3-fpm` y el socket `/run/php/php8.3-fpm.sock`. La página pública usa `https://equipo-tas-01.local`, que llega primero a VMFabian.

En septiembre WordPress todavía tenía la URL de prueba `http://www.mati-tas-01.local` y una base local. Antes de cambiar el dominio se hizo un respaldo privado de `wp-config.php` y SQL; luego se actualizaron `home` y `siteurl`. También se agregó `/health.php` porque HAProxy recibía 404 al consultar un archivo que aún no existía. Esa ruta ofrece un chequeo mínimo de PHP y BD, no una prueba de compra en WooCommerce.

En octubre, `DB_HOST` pasó de `localhost` a `10.33.199.11` para usar MariaDB en VMOmar. `wp-content/uploads` se montó desde VMNFS (`10.33.199.14`). Las salidas compartidas mostraron conexión TCP a la BD y HTTP 200 del sitio. En noviembre la consola mostró el montaje NFS activo, una entrada de arranque y una dependencia de montaje para Nginx; también quedó visible una prueba de escritura y lectura en el recurso.

## Archivos que conviene conocer

| Ruta en VMMaty | Para qué sirve |
| --- | --- |
| `/etc/nginx/sites-available/wordpress` | Dominio, raíz del sitio y paso de PHP al socket 8.3. |
| `/etc/nginx/sites-enabled/wordpress` | Enlace que activa el sitio anterior. |
| `/var/www/html/health.php` | Chequeo que consulta HAProxy desde la red privada. |
| `/var/www/html/wp-config.php` | Nombre, usuario y destino de la BD, junto con secretos. No subir el archivo real. |
| `/etc/fstab` | Entrada para montar `uploads` desde VMNFS al arrancar. |
| `/var/www/html/wp-content/uploads` | Punto de montaje compartido con VMMaty2. |
| `/etc/bind/named.conf.local` | Declaración histórica de una zona inversa secundaria; comprobar si aún se usa antes de cambiarla. |

El [resumen seguro de configuración](../../configuración/VMMaty/readme.md) señala los valores que se pueden publicar sin revelar contraseñas.

## Cómo comprobar que sigue bien

Revisa Nginx y PHP, confirma que `findmnt` muestre `10.33.199.14` en `uploads`, comprueba el puerto 3306 de VMOmar y pide la raíz del sitio con `curl`. Los comandos exactos están en la [guía de verificación](../verificacion-infraestructura.md#ambos-backends-web-bd-y-nfs). La respuesta 200 y el montaje activo se observaron; todavía no hay una prueba documentada de reinicio completo ni de restauración.

## Si necesitas reconstruir la historia

Estos documentos conservan el estado de **septiembre de 2026**. Cuando mencionan `DB_HOST=localhost` o un segundo backend pendiente, describen esa etapa y no la configuración posterior:

- [Inventario y rutas de septiembre](inventario-y-configuracion.md)
- [Bitácora de septiembre](bitacora.md)
- [Respaldo inicial y recuperación pendiente](respaldo-y-recuperacion.md)
- [Pruebas y pendientes de esa etapa](pruebas-y-pendientes.md)
- [Manual detallado en PDF](manual_vmmaty_detallado_v2.pdf)

Para los cambios de octubre y noviembre, continúa con la [bitácora del nodo](../bitacora-infraestructura.md).
