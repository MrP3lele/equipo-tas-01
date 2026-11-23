# Equipo TAS 01 — Nodo Sur

Este repositorio cuenta cómo quedó montado nuestro sitio de WordPress y WooCommerce. La idea es que cualquier integrante del equipo pueda entender qué hace cada VM, dónde revisar su configuración y qué pruebas faltan antes de dar el proyecto por terminado.

## Máquinas, direcciones y servicios

| Máquina | IP de acceso (`ens18`) | IP privada (`ens19`) | Servicios del proyecto | Documentación |
| --- | --- | --- | --- | --- |
| VMFabian | `10.33.195.224/25` | `10.33.199.10/29` | BIND (DNS), HAProxy (HTTPS y balanceo), Prometheus y Grafana. | [Leer VMFabian](docs/vmfabian/README.md) |
| VMOmar | `10.33.195.225/25` | `10.33.199.11/29` | MariaDB: base de datos `wordpress` para ambos backends. | [Leer VMOmar](docs/vmomar/README.md) |
| VMMaty | `10.33.195.162/25` | `10.33.199.12/29` | Nginx, PHP 8.3-FPM, WordPress, WooCommerce y cliente NFS. | [Leer VMMaty](docs/vmmaty/README.md) |
| VMMaty2 | `10.33.195.228/25` | `10.33.199.13/29` | Nginx, PHP 8.1-FPM, WordPress, WooCommerce y cliente NFS. | [Leer VMMaty2](docs/vmmaty2/README.md) |
| VMNFS | `10.33.195.166/25` | `10.33.199.14/29` | Servidor NFS para `wp-content/uploads`. | [Leer VMNFS](docs/vmnfs/README.md) |

Llamamos «IP de acceso» a la interfaz que se alcanza desde la red universitaria o por VPN. Son direcciones `10.*` de uso privado; no son IP públicas de Internet. Las direcciones reflejan las salidas observadas en las VM, así que conviene confirmarlas antes de una presentación.

## ¿Qué pasa cuando alguien abre la página?

El computador pregunta al DNS de VMFabian por `equipo-tas-01.local` y llega a `10.33.195.224`. HAProxy recibe la conexión HTTPS y elige uno de los dos backends sanos. El backend consulta la base de datos de VMOmar y lee los archivos subidos desde VMNFS. Para mirar métricas, VMFabian también publica `grafana.equipo-tas-01.local` y `prometheus.equipo-tas-01.local`.

La [vista completa del recorrido](docs/estado-nodo-sur.md) explica las conexiones y sus límites. La [bitácora](docs/bitacora-infraestructura.md) cuenta qué problemas aparecieron y cómo se resolvieron. Antes de una demostración, usa la [guía de verificación](docs/verificacion-infraestructura.md).

## Configuraciones y evidencias

Los [archivos de configuración documentados](configuración/configuracion_utilizada.md) indican las rutas en cada servidor y muestran solo fragmentos seguros. El [documento de los subdominios de monitoreo](docs/documentacion_subdominios_monitoreo_2026-11-23.docx) recoge el cambio de DNS, certificado y proxy.

**Antes de subir archivos:** no incluir `wp-config.php` real, contraseñas, claves privadas, tokens, respaldos SQL, perfiles VPN ni datos de clientes. Este repositorio ayuda a entender y revisar el sistema; no reemplaza un respaldo de las VM.
