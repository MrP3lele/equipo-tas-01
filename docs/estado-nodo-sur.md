# Así funciona el Nodo Sur

El sitio parece una sola página para quien lo visita, pero detrás trabajan cinco VM. Esta guía sigue una petición desde el navegador hasta WordPress y señala qué piezas comparten los dos backends. El estado descrito llega hasta el **23 de noviembre de 2026** y combina salidas de implementación con lo que quedó visible en las consolas de Proxmox.

[Volver al inicio](../README.md) · [Ir a los comandos de revisión](verificacion-infraestructura.md)

## Flujo de una solicitud

1. **El navegador busca el nombre.** El DNS de [VMFabian](vmfabian/README.md) responde que `equipo-tas-01.local` está en `10.33.195.224`. Ese nombre solo funciona en clientes que llegan a la red del proyecto y consultan el DNS adecuado; desde fuera puede hacer falta la VPN.
2. **HAProxy recibe la conexión HTTPS.** Está en VMFabian y usa un certificado firmado por la CA interna del equipo. Si la petición es para WordPress, revisa `/health.php` y elige entre [VMMaty](vmmaty/README.md) (`10.33.199.12:80`) y [VMMaty2](vmmaty2/README.md) (`10.33.199.13:80`) con `roundrobin`.
3. **El backend prepara la página.** Cada VM web tiene Nginx, PHP-FPM y una copia del código de WordPress. Las dos usan la misma base `wordpress` en [VMOmar](vmomar/README.md), `10.33.199.11:3306`.
4. **Las imágenes salen del mismo lugar.** Ambos backends montan `10.33.199.14:/srv/nfs/wp-uploads` desde [VMNFS](vmnfs/README.md) en `/var/www/html/wp-content/uploads`.

Los nombres `grafana.equipo-tas-01.local` y `prometheus.equipo-tas-01.local` también llegan a VMFabian. HAProxy los envía a Grafana (`127.0.0.1:3000`) y Prometheus (`127.0.0.1:9090`) en esa VM.

## Por qué hacen falta la BD y el NFS compartidos

Si cada backend tuviera su propia base, un pedido o cambio hecho en uno podría no aparecer en el otro. Por eso ambos apuntan a VMOmar. Si cada uno guardara las imágenes en su disco, una foto subida a VMMaty podría faltar cuando HAProxy enviara la siguiente visita a VMMaty2. Por eso `uploads` se monta desde VMNFS.

El NFS **no copia plugins, temas ni el código**. Esos archivos viven por separado en los dos backends y deben actualizarse de forma coordinada. Tampoco reemplaza los respaldos: todavía no hay una restauración de prueba documentada.

## Qué comprobamos

| Pieza | Resultado visto | Qué demuestra |
| --- | --- | --- |
| DNS y HAProxy | Configuración válida; los tres nombres respondieron hacia `.224`. | El proxy pudo recibir WordPress, Grafana y Prometheus en las pruebas de noviembre. |
| VMMaty y VMMaty2 | Nginx y PHP activos, `DB_HOST=10.33.199.11`, conexión TCP a la BD, NFS montado y HTTP 200. | Ambos podían servir el sitio al probarlos. |
| VMOmar | MariaDB escuchaba en `10.33.199.11:3306`; `wp_user` figuraba para `.12` y `.13`. | Los dos backends tenían un destino de BD común. |
| VMNFS | `/etc/exports` autorizaba `.12` y `.13`; el servicio estaba habilitado. | Los backends podían compartir el directorio de archivos subidos. |
| Monitoreo | La consulta registrada a Prometheus informó cinco máquinas y tres servicios `up`. | Llegaban métricas en ese momento; no es una garantía permanente. |

En noviembre también quedaron visibles entradas NFS de arranque y dependencias de montaje de Nginx en los dos backends. Eso respalda que se preparó la persistencia. Todavía falta registrar un reinicio controlado de las VM para comprobar todo el recorrido después de arrancar.

## DNS, VPN y certificado en el computador de pruebas

La IP `10.33.195.224` debe ser alcanzable desde el cliente. Cuando la VPN estaba desconectada, fallaban las pruebas a DNS y HTTPS; con VPN se pudo consultar BIND directamente y alcanzar los puertos 53 y 443. El dominio `.local` no está publicado en el DNS general de la universidad.

Durante la prueba del 23 de noviembre se añadió en Windows una regla DNS temporal para el dominio y sus subdominios. Funcionó, pero después se **eliminó a petición del propietario** y se limpió la caché. No quedó esa regla instalada. El certificado sigue firmado por una CA interna que Windows no tenía como confiable, por lo que el navegador puede mostrar una advertencia aunque el DNS resuelva bien. No se instaló la CA en Windows.

El navegador habla con HAProxy por HTTPS. En la configuración documentada, HAProxy habla con WordPress por HTTP dentro de la red privada; tampoco se verificó TLS hacia MariaDB. Conviene tener claro ese límite al presentar la seguridad del sistema.

## Lo que todavía nos falta demostrar

- Apagar por separado cada backend y observar desde la web que el sitio continúa funcionando.
- Reiniciar las VM y repetir las pruebas de NFS, BD, proxy y monitoreo después del arranque.
- Restaurar un respaldo en un destino de prueba; un archivo SQL de 2,5 MB por sí solo no prueba recuperación.
- Completar una compra de prueba en WooCommerce desde ambos backends y comprobar que código, plugins y temas coinciden.
- Revisar acceso a Grafana y Prometheus, confianza de la CA en los clientes y el cifrado de los enlaces internos.

Las rutas concretas están en la página de cada VM y en el [índice de configuración](../configuración/configuracion_utilizada.md). La [bitácora](bitacora-infraestructura.md) cuenta los errores que encontramos por el camino y cómo se corrigieron. No se publican contraseñas, claves privadas, SQL ni datos de clientes.
