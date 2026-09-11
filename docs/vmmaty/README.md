# Documentación de VMMaty

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

## Estado documentado

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
