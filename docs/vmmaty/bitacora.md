# Bitácora de VMMaty

La continuación de octubre y noviembre, incluida la migración a MariaDB, NFS y el segundo backend, está en la [bitácora de infraestructura](../bitacora-infraestructura.md). Las entradas de este archivo son históricas y terminan en septiembre.

Registro documental incorporado el **10 de septiembre de 2026**. Se basa en las salidas y capturas conservadas del trabajo de Maty; no recrea commits pasados ni inventa fechas de ejecución. Cuando no hay fecha verificable se indica el límite del registro.

| Fecha o período acreditado | Problema o actividad | Cambio y resultado conocido |
| --- | --- | --- |
| Hasta 2026-09-07; fecha exacta no fijada | Identificación de VM y servicios | `hostname`, `ip -br -4 address`, `nginx -t` y listado del socket identificaron `mati`, ambas interfaces, sitio `wordpress` y PHP 8.3. |
| Hasta 2026-09-07; fecha exacta no fijada | PTR local devolvía `NXDOMAIN` | La zona inversa quedó como secundaria. Las consultas locales posteriores devolvieron `backend1.equipo-tas-01.local.` y serial `2026090501`. |
| Hasta 2026-09-07; registro del proxy del 2026-09-05 | Chequeo HTTP devolvía 404 | Se detectó que `/var/www/html/health.php` no existía. Se añadió el endpoint y se adaptó el sitio Nginx, limitando su acceso al proxy privado. |
| 2026-09-07, 18:09:22 según el registro de HAProxy | Recuperación del chequeo | El registro compartido informó backend UP y comprobación HTTP 200. No es una prueba de tolerancia a fallas con varios backends. |
| Hasta 2026-09-07; fecha exacta no fijada | WP-CLI no se encontraba mediante `command -v wp` | Se incorporó `/usr/local/bin/wp`; posteriormente ejecutó consultas como `www-data`. |
| Hasta 2026-09-07; fecha exacta no fijada | `config list` devolvía `Invalid field: key` | Se corrigió la selección de columnas a `--fields=name,value`; se obtuvieron `DB_NAME=wordpress` y `table_prefix=wp_`. No se modificó la BD con esa consulta. |
| 2026-09-07; listado del backup a las 18:41, sin zona horaria acreditada en esa salida | Respaldo anterior al cambio de URL | Se creó `/root/respaldo-wp-dominio.dyXU0l` con configuración, SQL y permisos originales. El SQL tenía contenido; no se realizó una restauración de prueba. |
| 2026-09-07 | Simulación y cambio de dominio | El `search-replace --dry-run` detectó cuatro reemplazos. Las lecturas finales de `home` y `siteurl` mostraron `https://equipo-tas-01.local`. `DB_HOST` continuó en `localhost`. |
| 2026-09-07 | Reconocimiento de HTTPS y pruebas integradas | Se indicó el ajuste de confianza en el proxy en `wp-config.php`. PHP-FPM respondió `active`; raíz y login respondieron 200 y administración redirigió al login por HTTPS. |
| 2026-09-10 | Incorporación documental | Se organizó la documentación de VMMaty para el repositorio sin modificar las VM ni aportar copias de archivos sensibles. |

## Qué se cambió y qué se conservó

- Se trabajó en la declaración DNS inversa, el sitio Nginx, la comprobación de salud y las opciones URL de WordPress.
- El enlace del sitio habilitado y el directorio raíz de WordPress se conservaron.
- Se documentó el ajuste HTTPS propuesto en la configuración sensible y su resultado funcional; falta contrastar una copia saneada actual.
- Se mantuvo la BD local. El cambio de dominio no fue una migración de datos a otra VM.
- No se acredita aquí una eliminación concreta de PrestaShop ni de otros datos de la VM.

## Registro para cambios posteriores

Cada nueva entrada debe incluir fecha y zona horaria, responsable, motivo, rutas afectadas, respaldo, comandos, resultado esperado, resultado observado y reversión. Vincular el commit y la evidencia de esa ejecución real; no utilizar resultados históricos como prueba de una ejecución nueva.

Antes de publicar salidas, eliminar secretos y datos personales. No registrar contraseñas ni pegar el contenido completo de `wp-config.php`.
