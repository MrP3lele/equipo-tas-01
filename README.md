# Equipo TAS 01 — Nodo Sur

Infraestructura de WordPress y WooCommerce con dos backends, base de datos separada, NFS, DNS, HAProxy y monitoreo. Este repositorio contiene documentación y referencias de configuración sin secretos; no es un respaldo de las VM.

## Mapa del entorno

| VM | IP privada | IP de acceso | Función |
| --- | --- | --- | --- |
| VMFabian (236) | `10.33.199.10` | `10.33.195.224` | BIND, HAProxy, TLS, Prometheus y Grafana |
| VMOmar (239) | `10.33.199.11` | `10.33.195.225` | MariaDB de WordPress |
| VMMaty (237) | `10.33.199.12` | `10.33.195.162` | Backend WordPress 1, Nginx y PHP 8.3 |
| VMMaty2 (311) | `10.33.199.13` | `10.33.195.228` | Backend WordPress 2, Nginx y PHP 8.1 |
| VMNFS (312) | `10.33.199.14` | `10.33.195.166` | Directorio compartido de archivos subidos |

El cliente consulta `equipo-tas-01.local` en el DNS de VMFabian. HAProxy recibe HTTPS en `10.33.195.224:443` y reparte WordPress entre `.12:80` y `.13:80`. Ambos backends consultan MariaDB en `.11:3306` y montan `10.33.199.14:/srv/nfs/wp-uploads` en `/var/www/html/wp-content/uploads`.

Grafana y Prometheus se publican como `grafana.equipo-tas-01.local` y `prometheus.equipo-tas-01.local` a través de HAProxy. Las rutas locales observadas son `127.0.0.1:3000` y `127.0.0.1:9090`.

## Documentación

- [Estado y arquitectura](docs/estado-nodo-sur.md): evidencia por VM, flujo, archivos y límites.
- [Bitácora reciente](docs/bitacora-infraestructura.md): problemas y correcciones de octubre y noviembre.
- [Verificación](docs/verificacion-infraestructura.md): comandos de solo lectura y resultados esperados.
- [VMMaty](docs/vmmaty/README.md): etapa inicial y actualización posterior.
- [Subdominios de monitoreo](docs/documentacion_subdominios_monitoreo_2026-11-23.docx): intervención del 23 de noviembre en VMFabian.
- [Índice de configuraciones](configuración/configuracion_utilizada.md): rutas de origen y material saneado.

**Corte de evidencia:** 23 de noviembre de 2026. Las salidas de implementación no implican una inspección nueva de cada archivo de las VM en la fecha del commit.

## Seguridad del repositorio

No subir `wp-config.php` real, claves de CA o TLS, contraseñas, salts, tokens, perfiles VPN, respaldos SQL, datos de clientes ni archivos de `uploads`. Los fragmentos documentados no se deben aplicar como reemplazo de configuraciones completas.
