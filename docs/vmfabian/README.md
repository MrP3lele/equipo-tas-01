# VMFabian: la entrada al sitio

Cuando alguien escribe `equipo-tas-01.local`, esta es la primera VM con la que se encuentra. Tiene la IP `10.33.195.224` hacia los clientes y `10.33.199.10` en la red privada. Aquí trabajan BIND, HAProxy, Prometheus y Grafana.

[Volver al inicio](../../README.md) · [Ver el recorrido completo](../estado-nodo-sur.md)

## Qué hace cada servicio

**BIND responde por los nombres.** El dominio principal y los subdominios `grafana` y `prometheus` apuntan a `10.33.195.224`. También existe una zona inversa para la red privada: por ejemplo, el PTR de `10.33.199.12` devolvió `backend1.equipo-tas-01.local`. Al agregar los dos subdominios de monitoreo, el serial observado de la zona directa quedó en `2026112301`.

**HAProxy recibe la visita.** Redirige HTTP a HTTPS y, para WordPress, reparte las peticiones entre VMMaty (`10.33.199.12:80`) y VMMaty2 (`10.33.199.13:80`) con `roundrobin`. Antes de usar cada backend consulta `/health.php` y espera un 200. Los nombres de Grafana y Prometheus se envían a servicios locales de esta misma VM, en los puertos 3000 y 9090.

**El certificado protege la conexión del navegador al proxy.** La CA es interna del equipo. En noviembre se amplió el certificado para que también cubriera `grafana.equipo-tas-01.local` y `prometheus.equipo-tas-01.local`. El cliente debe confiar en esa CA para que el navegador no muestre una advertencia. El tráfico que HAProxy envía a los backends WordPress seguía usando HTTP por la red privada en la configuración documentada.

## Dónde se trabajó

| Ruta en VMFabian | Para qué se usó |
| --- | --- |
| `/etc/bind/named.conf.local` | Declara las zonas DNS directa e inversa. |
| `/etc/bind/db.equipo-tas-01.local` | Contiene los registros del equipo; se añadieron `grafana` y `prometheus`. |
| `/etc/bind/db.10` | Mantiene los PTR de las IP privadas. |
| `/etc/haproxy/haproxy.cfg` | Decide a qué servicio enviar cada nombre y cómo revisar los backends. |
| `/root/equipo-tas-pki/server.ext` | Enumera los nombres válidos del certificado. |
| `/etc/haproxy/certs/equipo-tas-01.local.pem` | Certificado instalado; incluye una clave privada y no se sube al repositorio. |
| `/home/fabi/monitoreo/docker-compose.yml` | Se leyó para conocer el despliegue de monitoreo; no se cambió al crear los subdominios. |

Antes del cambio de subdominios se guardaron copias en `/root/subdominios-20261123.5hsNu4/`, dentro de la VM. Los [fragmentos seguros de configuración](../../configuración/VMFabian/readme.md) y el [documento de esa intervención](../documentacion_subdominios_monitoreo_2026-11-23.docx) dan más detalle.

## Qué vimos funcionar y qué conviene repetir

En la intervención del 23 de noviembre pasaron `named-checkconf -z`, `haproxy -c` y la verificación del certificado con la CA interna. Las consultas DNS devolvieron `.224` para los dos subdominios. Con GET por HTTPS respondieron WordPress, Grafana y la API de Prometheus. Un intento con HEAD a `/targets` dio 405; al comprobar con GET, la API respondió 200.

Antes de presentar, conviene repetir las consultas desde el computador del profesor y revisar que ambos backends sigan sanos en HAProxy. El nombre `.local` solo resuelve si el cliente consulta el DNS adecuado; además, resolver el nombre no instala la CA. La regla DNS temporal que se probó en Windows el 23 de noviembre fue retirada después de la prueba.

Usa los [comandos de verificación](../verificacion-infraestructura.md#vmfabian-dns-certificado-y-proxy) para hacer esa revisión sin abrir archivos con claves.
