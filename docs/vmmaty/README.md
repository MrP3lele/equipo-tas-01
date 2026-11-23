# Documentación de VMMaty

## Actualización de octubre y noviembre de 2026

VMMaty ya no usa la BD local como destino de WordPress. Las salidas compartidas del 23 de octubre mostraron `DB_HOST=10.33.199.11`, conectividad TCP y HTTP 200. Su directorio `/var/www/html/wp-content/uploads` quedó montado desde `10.33.199.14:/srv/nfs/wp-uploads`. En la consola de noviembre se observó la entrada NFS de arranque, una dependencia de montaje para Nginx, escritura/lectura en el NFS y HTTP 200. También se integró VMMaty2 como segundo backend en HAProxy. Ver [estado actual del nodo](../estado-nodo-sur.md) y [bitácora reciente](../bitacora-infraestructura.md).

**Las secciones siguientes conservan el estado histórico hasta el 7 de septiembre de 2026.** Sus valores `DB_HOST=localhost`, BD local y segundo backend pendiente no describen la configuración posterior.

VMMaty ejecuta WordPress con Nginx y PHP-FPM como backend del equipo TAS 01. Este directorio respalda documentalmente su configuración, los cambios realizados y las pruebas conocidas; no contiene una copia de la máquina ni de su base de datos.

**Corte técnico: 7 de septiembre de 2026. Incorporación al repositorio: 10 de septiembre de 2026.** No se ha realizado una nueva inspección del servidor para esta incorporación. Antes de aplicar procedimientos hay que contrastar el estado actual.

## Documentos

| Documento | Contenido |
| --- | --- |
| [Inventario y configuración](inventario-y-configuracion.md) | Interfaces, servicios, rutas y relación con las dependencias externas. |
| [Bitácora](bitacora.md) | Problemas encontrados, cambios y resultados disponibles. |
| [Respaldo y recuperación](respaldo-y-recuperacion.md) | Respaldo previo al cambio de dominio, precauciones y recuperación pendiente de probar. |
| [Pruebas y pendientes](pruebas-y-pendientes.md) | Comandos de comprobación, criterios de aceptación y deuda técnica. |
| [Manual detallado](manual_vmmaty_detallado_v2.pdf) | Manual de 20 páginas con comandos explicados, solicitudes, correcciones y guía de acceso por VPN. |

## Estado documentado en septiembre

- Dominio de WordPress: `https://equipo-tas-01.local`.
- Backend: `10.33.199.12:80`; interfaz de acceso: `10.33.195.162/25`.
- Raíz del sitio: `/var/www/html`; PHP-FPM: `php8.3-fpm`.
- `home` y `siteurl` apuntan al dominio nuevo; `DB_HOST` todavía es `localhost`.
- La publicación HTTPS depende del proxy externo. El enlace de ese proxy a Maty permanece en HTTP.
- La migración de la BD, la comprobación funcional de WooCommerce y la restauración de respaldos siguen pendientes.

Las referencias a otras VM describen exclusivamente las dependencias de VMMaty. No son manuales ni configuraciones de esas máquinas.

## Alcance de seguridad

No se incluyen `wp-config.php` real, claves o salts de WordPress, contraseñas, certificados con claves privadas, perfiles VPN ni volcados SQL. Las rutas, nombres e IP documentados identifican el laboratorio, no credenciales.

Los fragmentos del manual son referencias históricas, no una exportación nueva de los archivos actuales. Las configuraciones reales saneadas se incorporarán únicamente después de obtenerlas y contrastarlas. No sustituir archivos completos a partir de un fragmento ni ejecutar cambios sin respaldo y autorización.

El PDF conserva el registro histórico de solicitudes y validaciones. Las pruebas descritas no deben presentarse como ejecutadas nuevamente el día del commit.
