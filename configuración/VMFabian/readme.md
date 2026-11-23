# VMFabian — DNS, proxy y monitoreo

- IP de acceso observada: `10.33.195.224/25`; IP privada: `10.33.199.10/29`.
- BIND sirve `equipo-tas-01.local` y la zona inversa `199.33.10.in-addr.arpa`. El serial de la zona directa después de añadir Grafana y Prometheus fue `2026112301`.
- `/etc/bind/db.equipo-tas-01.local` incluye `grafana` y `prometheus` como registros A a `10.33.195.224`.
- `/etc/haproxy/haproxy.cfg` usa `roundrobin` para `10.33.199.12:80` y `10.33.199.13:80`, con GET `/health.php` y estado esperado 200. Las ACL de los subdominios envían HTTPS a `127.0.0.1:3000` y `127.0.0.1:9090`.
- `/root/equipo-tas-pki/server.ext` recibió los SAN nuevos. Se instaló un certificado renovado en `/etc/haproxy/certs/equipo-tas-01.local.pem`; combina certificado y clave privada y no debe versionarse.
- `/home/fabi/monitoreo/docker-compose.yml` se consultó, pero no se modificó en la intervención de subdominios.
- Copia previa de los archivos cambiados por los subdominios: `/root/subdominios-20261123.5hsNu4/`, solo en la VM.

Se validaron sintaxis de BIND y HAProxy, resolución local de los nombres, certificado con la CA interna y GET HTTPS de Grafana, Prometheus y WordPress. La API de Prometheus informó cinco máquinas y tres servicios `up` en ese momento. Ver [registro detallado](../../docs/documentacion_subdominios_monitoreo_2026-11-23.docx) y [límites](../../docs/estado-nodo-sur.md).

Cada cliente debe confiar en la CA interna para evitar la advertencia del navegador. El tramo HAProxy → WordPress registrado sigue siendo HTTP por la red privada.

## Fragmento comprobado del backend WordPress

Este extracto proviene de la salida compartida de `/etc/haproxy/haproxy.cfg`; no reemplaza el archivo completo ni sus ACL de frontend:

```haproxy
backend equipo_wp
    balance roundrobin
    option httpchk
    http-check connect
    http-check send meth GET uri /health.php ver HTTP/1.1 hdr Host equipo-tas-01.local hdr X-Forwarded-Proto https
    http-check expect status 200
    server vmmaty 10.33.199.12:80 check inter 5s fall 3 rise 2
    server vmmaty2 10.33.199.13:80 check inter 5s fall 3 rise 2
```

Los registros A añadidos a la zona del equipo fueron `grafana` y `prometheus`, ambos hacia `10.33.195.224`. Publicar la zona completa requiere contrastarla y sanearla en la VM.
