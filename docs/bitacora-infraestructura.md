# Bitácora de integración reciente

Este registro complementa la [bitácora histórica de VMMaty](vmmaty/bitacora.md). Se basa en salidas compartidas, el documento de subdominios y las consolas visibles de Proxmox; no recrea cambios sin evidencia.

| Período | Problema o cambio | Resultado acreditado |
| --- | --- | --- |
| Septiembre de 2026 | El PTR de `.12` devolvía `NXDOMAIN`; luego la zona inversa se corrigió y VMMaty recibió una copia secundaria. | `dig -x 10.33.199.12` devolvió `backend1.equipo-tas-01.local.`. |
| 7 de septiembre | WordPress todavía usaba `http://www.mati-tas-01.local`. Se respaldaron configuración y SQL y se hizo `search-replace` de URL. | `home` y `siteurl` devolvieron `https://equipo-tas-01.local`; HTTPS en el proxy respondió 200 para raíz y login. |
| Octubre de 2026 | Se introdujeron VMMaty2, VMOmar y VMNFS. VMMaty2 empezó solo con la página predeterminada de Nginx y sin WordPress. | El sitio se copió; Nginx pasó a servir PHP con socket 8.1. |
| 23 de octubre | El primer `wp-site.tgz` escrito en `uploads` quedó truncado; al extraerlo apareció `Unexpected EOF`. | Se eliminó la copia incompleta, se creó el paquete en `/tmp` y luego se transfirió. La presencia de `wp-config.php` y `wp-load.php` se verificó en VMMaty2. |
| 23 de octubre | VMMaty2 devolvió HTTP 500 por falta de `mysqli`. Después hubo timeout hacia `10.33.199.11:3306`. | Se instaló/habilitó la extensión necesaria y se resolvió la conectividad. Ambos backends mostraron TCP a `.11` y HTTP 200. |
| 23 de octubre | MariaDB escuchaba inicialmente en todas las interfaces. | La escucha observada se restringió a `10.33.199.11:3306`; `wp_user` se listó para `.12` y `.13`. |
| 23 de octubre | HAProxy inicialmente marcó VMMaty2 como DOWN por HTTP 500. | El registro posterior informó VMMaty2 UP con código 200 y dos servidores activos. |
| 21–23 de noviembre | Se revisó persistencia del NFS. VMMaty2 había mostrado previamente el sistema de archivos raíz en `findmnt` y ausencia de entrada NFS en fstab. | Las consolas posteriores mostraron NFS montado en ambos backends, entradas de arranque, Nginx dependiente del montaje y HTTP 200. No se documentó un reinicio completo de prueba. |
| 23 de noviembre | Se añadieron subdominios de Grafana y Prometheus en BIND, HAProxy y el certificado de VMFabian. | Resolución a `.224`, certificado validado con CA interna y GET HTTPS 200 de los servicios y de WordPress. [Detalle](documentacion_subdominios_monitoreo_2026-11-23.docx). |
| 23 de noviembre | Windows no resolvía los tres nombres mediante sus DNS predeterminados. | Se añadió una regla NRPT temporal dirigida a `.224`; las consultas y sitios funcionaron. Luego se eliminó a solicitud del propietario, se limpió la caché y se verificó que no quedara regla. |

## Correcciones que conviene explicar

- `php-fpm` era un metapaquete en VMMaty2; el servicio efectivo era `php8.1-fpm.service`.
- El `HTTP 500` de VMMaty2 no era un problema de HAProxy: WordPress indicó que faltaba la extensión MySQL de PHP.
- La conectividad TCP a MariaDB y un HTTP 200 del sitio son pruebas diferentes; ambas se comprobaron.
- `HEAD /targets` devolvió 405 en Prometheus; `GET /api/v1/targets` devolvió 200.
- El 403 del `/health.php` de VMMaty2 desde localhost es compatible con limitar esa ruta al proxy; debe comprobarse desde VMFabian.
- El DNS interno funciona al consultar VMFabian por VPN. Resolver el nombre no implica confiar en la CA del certificado ni que el resto de Internet conozca el dominio `.local`.

Los respaldos SQL y del sitio permanecen fuera del repositorio. No hay evidencia de restauración, failover desde el navegador ni pruebas completas de WooCommerce.
