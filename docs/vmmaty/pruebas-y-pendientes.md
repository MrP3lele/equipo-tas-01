# Pruebas y pendientes de VMMaty

**Actualización:** las pruebas y pendientes vigentes para las cinco VM están en [verificación de infraestructura](../verificacion-infraestructura.md) y [estado del nodo](../estado-nodo-sur.md). En particular, `DB_HOST=localhost`, migración de BD pendiente y segundo backend pendiente son datos históricos de septiembre: en octubre se observaron `DB_HOST=10.33.199.11`, dos backends y HTTP 200.

Las observaciones históricas corresponden al trabajo registrado hasta el **7 de septiembre de 2026**. Los comandos siguientes son procedimientos para volver a comprobarlo; no se han ejecutado contra las VM durante esta incorporación al repositorio.

## Comprobaciones en VMMaty

```bash
hostname
ip -br -4 address
sudo nginx -t
systemctl is-active nginx php8.3-fpm named
ls -l /etc/nginx/sites-enabled/wordpress
ls -l /run/php/php8.3-fpm.sock
dig @127.0.0.1 199.33.10.in-addr.arpa SOA +short
dig @127.0.0.1 -x 10.33.199.12 +short
sudo -u www-data /usr/local/bin/wp --path=/var/www/html \
  --skip-plugins --skip-themes option get home
sudo -u www-data /usr/local/bin/wp --path=/var/www/html \
  --skip-plugins --skip-themes option get siteurl
sudo -u www-data /usr/local/bin/wp --path=/var/www/html \
  config get DB_HOST --type=constant
```

Estas consultas no editan la configuración. WP-CLI puede cargar partes de la aplicación; `--skip-plugins` no omite necesariamente los mu-plugins. Revisar la salida antes de compartirla y no sustituir las consultas selectivas por un volcado de todo `wp-config.php`.

El serial histórico fue `2026090501`; si la zona cambió posteriormente se debe comparar el serial vigente del primario y el secundario, no forzar el valor antiguo.

## Comprobar el backend desde el proxy autorizado

La siguiente prueba se ejecuta desde la VM que alcanza Maty con origen privado `10.33.199.10`. No requiere cambiar la configuración del proxy.

```bash
curl --noproxy '*' --connect-timeout 3 --max-time 10 \
  -sS -D - -o /dev/null \
  -H 'Host: equipo-tas-01.local' \
  -H 'X-Forwarded-Proto: https' \
  http://10.33.199.12/health.php
```

Esperado: HTTP 200. Un 404 indica que la solicitud no encuentra la ruta esperada; un 503 puede indicar fallo de la comprobación. Desde orígenes no autorizados se espera rechazo de acceso al endpoint. No se acredita aún una prueba negativa real para todos esos orígenes.

`-H` selecciona el sitio y comunica el esquema externo. `--noproxy` evita proxies de entorno para esta consulta, no desactiva HAProxy. `-D -` muestra cabeceras y `-o /dev/null` descarta el cuerpo. Sin `--fail`, un código de salida cero de curl no garantiza HTTP 200: revisar el estado HTTP.

## Prueba integrada por HTTPS

Desde el proxy, con el certificado público de la CA ya disponible en la ruta indicada:

```bash
for tas_ruta in / /wp-login.php /wp-admin/; do
  curl --noproxy '*' \
    --cacert /root/equipo-tas-pki/ca.crt \
    --resolve equipo-tas-01.local:443:127.0.0.1 \
    --connect-timeout 3 --max-time 15 \
    --silent --show-error --output /dev/null \
    --write-out 'URL: %{url_effective}\nHTTP: %{http_code}\nRedireccion: %{redirect_url}\n\n' \
    "https://equipo-tas-01.local${tas_ruta}"
done
```

`--cacert` verifica la confianza explícitamente; `--resolve` solo modifica la resolución de esa ejecución de curl. No se usa `-k` para omitir el certificado ni `-L` para ocultar la primera redirección siguiéndola automáticamente. Esta prueba local no sustituye una prueba DNS y HTTPS desde un cliente por VPN.

## Matriz de resultados conocidos

Los identificadores siguientes organizan el registro documental; no afirman que existan archivos de evidencia originales dentro del repositorio.

| ID | Objetivo | Resultado histórico | Límite de la evidencia |
| --- | --- | --- | --- |
| MATY-01 | Identificar red | `mati`, `ens18 10.33.195.162/25`, `ens19 10.33.199.12/29` | No prueba persistencia de red tras reinicio. |
| MATY-02 | Validar Nginx | `nginx -t` finalizó correctamente | No valida por sí solo la BD ni una compra. |
| MATY-03 | Verificar DNS inverso | PTR `backend1.equipo-tas-01.local.` y SOA `2026090501` | No prueba que Maty sirva la zona directa nueva. |
| MATY-04 | Verificar salud del backend | El registro compartido informó UP y HTTP 200 | Salud mínima de PHP/BD, no todos los plugins. |
| MATY-05 | Verificar URL y destino de BD | `home` y `siteurl` nuevos; `DB_HOST=localhost` | No acredita una migración a otra VM. |
| MATY-06 | Publicación HTTPS | `/` y `/wp-login.php`: 200; `/wp-admin/`: 302 al login HTTPS | Administración sin sesión; no prueba acceso autenticado. |
| MATY-07 | Experiencia en navegador | Respuesta 200 mediante `10.33.195.224:443` | El navegador tenía excepción de certificado; confianza de CA pendiente. |
| MATY-08 | Respaldo previo | Exportación finalizada y SQL no vacío | Restauración y cobertura completa no probadas. |

Para nuevas evidencias anotar fecha/zona horaria, VM, comando, esperado, observado e interpretación. Conservar solo extractos relevantes sin contraseñas, cookies, datos de clientes ni tokens. Las capturas complementan, pero no reemplazan, configuraciones saneadas.

## Pendientes priorizados al 10 de septiembre de 2026

1. Contrastar las rutas documentadas con las configuraciones actuales y completar recursos/versiones del inventario.
2. Preparar y validar la futura BD separada antes de cambiar `DB_HOST`; coordinar respaldo nuevo, pausa de escrituras y reversión.
3. Verificar WooCommerce y un flujo empresarial completo con datos de prueba no personales.
4. Validar CA y acceso por nombre desde el cliente real del evaluador, sin excepciones de certificado.
5. Revisar mínimo privilegio, firewall y acceso directo a Maty. No se activó UFW en este trabajo; falta comprobar y documentar el estado actual específico de Maty.
6. Si se retoma la mejora, implementar y verificar TLS entre proxy y backend. La confianza de CA en Windows no cifra ese tramo.
7. Probar restauración, arranque de servicios, red persistente y reinicio controlado con autorización.
8. Completar el segundo backend y la coherencia de estado cuando corresponda a la etapa del proyecto. Un único backend no proporciona tolerancia a su caída.

No se declara completada la plataforma ni la rúbrica solo por tener WordPress accesible. Las dependencias externas se mencionan como requisitos de VMMaty, sin afirmar que ya se hayan configurado.
