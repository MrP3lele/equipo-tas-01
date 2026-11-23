# Cómo fuimos armando el Nodo Sur

Esta bitácora reúne los cambios que conectaron las cinco VM. Conservamos los errores porque ayudan a explicar por qué existe cada ajuste. Para los primeros pasos de VMMaty está la [bitácora de septiembre](vmmaty/bitacora.md); aquí seguimos desde la integración con el resto del equipo.

[Volver al inicio](../README.md) · [Ver el sistema completo](estado-nodo-sur.md)

## Septiembre: dejar listo el primer backend

La instalación empezó con VMMaty y el nombre de prueba `mati-tas-01.local`. El PTR de `10.33.199.12` llegó a devolver `NXDOMAIN`; después de corregir la zona inversa y su copia secundaria, la consulta devolvió `backend1.equipo-tas-01.local.`.

Antes de pasar WordPress a `https://equipo-tas-01.local`, se guardó una copia privada de `wp-config.php` y de la base. La simulación de `search-replace` anunció cuatro reemplazos. Las consultas posteriores a `home` y `siteurl` mostraron el nombre nuevo. HAProxy necesitaba `/health.php`, pero el archivo no existía y el chequeo respondía 404. Al añadirlo y configurar Nginx, el backend pasó a responder 200. El [manual de VMMaty](vmmaty/README.md) conserva los comandos de esa etapa.

## Octubre: pasar de una VM web a dos

VMMaty2 empezó con la página de bienvenida de Nginx, sin WordPress. Se preparó una copia del sitio de VMMaty. El primer `wp-site.tgz` se escribió dentro de `uploads`, que ya era NFS, y quedó truncado: `tar` devolvió `Unexpected EOF`. La solución fue crear el paquete en `/tmp`, transferirlo y comprobar en VMMaty2 que estaban `wp-config.php` y `wp-load.php`.

Después llegaron dos fallos distintos. WordPress respondió HTTP 500 porque a PHP 8.1 le faltaba la extensión `mysqli`. Al resolver eso, la página seguía esperando la BD: la prueba TCP a `10.33.199.11:3306` fallaba. Se corrigió la conectividad y, en las salidas del **23 de octubre**, los dos backends alcanzaron MariaDB y devolvieron HTTP 200. El servicio PHP de la segunda VM es `php8.1-fpm.service`; instalar el metapaquete `php-fpm` no creó un servicio llamado `php-fpm.service`.

En VMOmar, MariaDB escuchaba inicialmente en `0.0.0.0:3306`. La escucha observada después quedó limitada a `10.33.199.11:3306`. El usuario `wp_user` figuraba para los dos orígenes privados, `.12` y `.13`. También se listó un SQL privado previo al cambio. Sigue faltando probar que ese respaldo se puede restaurar.

HAProxy marcó VMMaty2 como DOWN mientras su chequeo devolvía 500. Cuando la página volvió a responder 200, el registro del proxy informó el backend UP y dos servidores activos. La causa estaba en el backend; cambiar el balanceador no habría reparado `mysqli` ni la conexión a la BD.

## Noviembre: comprobar el NFS y el monitoreo

Una revisión de VMMaty2 había mostrado `/` en `findmnt` para `uploads` y no había entrada NFS en fstab. Eso significaba que el punto de montaje no estaba activo en ese momento. En las consolas posteriores se vio el NFS montado en **ambos** backends, entradas para el arranque, Nginx dependiente del montaje y la raíz del sitio respondiendo 200. VMMaty también mostró una prueba de escritura y lectura en el recurso. Esta evidencia es buena para revisar la configuración, pero no sustituye un reinicio completo de prueba.

El **23 de noviembre** se dieron nombres propios a Grafana y Prometheus. En VMFabian se añadieron registros DNS, rutas de HAProxy y nombres al certificado. Las pruebas locales con la CA interna devolvieron 200 para WordPress, Grafana y la API de Prometheus. Una prueba con HEAD en `/targets` dio 405 porque esa ruta aceptaba GET y OPTIONS; al usar GET en `/api/v1/targets`, la respuesta fue 200. El [documento de subdominios](documentacion_subdominios_monitoreo_2026-11-23.docx) guarda rutas, comandos y respaldo de esa intervención.

En el computador Windows, los nombres no resolvían con el DNS predeterminado aunque BIND sí respondía a una consulta directa. Se probó una regla DNS específica para `equipo-tas-01.local` y sus subdominios: los tres nombres empezaron a resolver y los sitios respondieron. Luego el propietario pidió revertirla. Se eliminó la regla, se limpió la caché y se verificó que no quedara instalada. La advertencia del certificado era un asunto aparte: Windows no confiaba en la CA interna y no se instaló esa CA.

## Qué sigue abierto

No hemos dejado registrada una restauración, una caída controlada de cada backend observada desde el navegador ni una compra completa de WooCommerce. Tampoco hay una prueba de reinicio de todas las VM después de configurar la persistencia. La [guía de verificación](verificacion-infraestructura.md) indica cómo repetir las consultas antes de cerrar esos puntos.
