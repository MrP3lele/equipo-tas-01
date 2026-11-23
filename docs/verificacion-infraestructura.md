# Verificación de infraestructura

Comandos de lectura para volver a comprobar el estado antes de una presentación. Comparar las salidas reales con la columna esperada; estas órdenes no reemplazan las pruebas funcionales ni una restauración. No publicar salidas que revelen contraseñas, claves, tokens o datos de clientes.

## VMFabian: DNS, certificado y proxy

```bash
systemctl is-active named haproxy
sudo named-checkconf -z
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
dig @127.0.0.1 equipo-tas-01.local A +short
dig @127.0.0.1 grafana.equipo-tas-01.local A +short
dig @127.0.0.1 prometheus.equipo-tas-01.local A +short
dig @127.0.0.1 -x 10.33.199.12 +short
curl --noproxy '*' --cacert /root/equipo-tas-pki/ca.crt --resolve equipo-tas-01.local:443:127.0.0.1 -sS -o /dev/null -w '%{http_code}\n' https://equipo-tas-01.local/
curl --noproxy '*' --cacert /root/equipo-tas-pki/ca.crt --resolve grafana.equipo-tas-01.local:443:127.0.0.1 -sS -o /dev/null -w '%{http_code}\n' https://grafana.equipo-tas-01.local/login
curl --noproxy '*' --cacert /root/equipo-tas-pki/ca.crt --resolve prometheus.equipo-tas-01.local:443:127.0.0.1 -sS -o /dev/null -w '%{http_code}\n' https://prometheus.equipo-tas-01.local/api/v1/targets
```

Esperado según la implementación: servicios activos, configuración válida, tres A a `10.33.195.224`, PTR de `.12` al backend 1 y GET HTTPS 200. No usar HEAD para `/targets` como prueba de disponibilidad.

## Ambos backends: web, BD y NFS

```bash
hostname
ip -br -4 address
systemctl is-active nginx php8.3-fpm  # VMMaty
systemctl is-active nginx php8.1-fpm  # VMMaty2
sudo nginx -t
findmnt -T /var/www/html/wp-content/uploads
findmnt --fstab --types nfs,nfs4
systemctl show nginx -p RequiresMountsFor -p After --no-pager
timeout 5 bash -c '</dev/tcp/10.33.199.11/3306' && echo TCP_DB_OK
curl --max-time 10 -sS -o /dev/null -w 'HTTP=%{http_code}\n' -H 'Host: equipo-tas-01.local' -H 'X-Forwarded-Proto: https' http://127.0.0.1/
sudo grep -nE "DB_NAME|DB_USER|DB_HOST" /var/www/html/wp-config.php
```

Esperado: NFS desde `10.33.199.14`, entrada de arranque, `DB_HOST=10.33.199.11`, TCP a MariaDB y HTTP 200. La última orden lee un archivo sensible; compartir solo esas tres líneas después de confirmar que no aparece la contraseña. `findmnt` en `/` en vez de la ruta NFS indica que el montaje no está activo.

Para comprobar el chequeo desde el origen permitido, ejecutar en **VMFabian**:

```bash
curl --noproxy '*' --max-time 10 -sS -o /dev/null -w 'backend1=%{http_code}\n' -H 'Host: equipo-tas-01.local' -H 'X-Forwarded-Proto: https' http://10.33.199.12/health.php
curl --noproxy '*' --max-time 10 -sS -o /dev/null -w 'backend2=%{http_code}\n' -H 'Host: equipo-tas-01.local' -H 'X-Forwarded-Proto: https' http://10.33.199.13/health.php
```

Esperado: 200 de ambos. Desde un origen no autorizado puede responder 403 sin que HAProxy esté averiado.

## VMOmar y VMNFS

```bash
# En VMOmar
systemctl is-active mariadb
systemctl is-enabled mariadb
sudo ss -lntp '( sport = :3306 )'
sudo mariadb -e "SELECT User,Host FROM mysql.user WHERE User='wp_user';"

# En VMNFS
systemctl is-active nfs-server
systemctl is-enabled nfs-server
sudo exportfs -v
findmnt -T /srv/nfs/wp-uploads
```

Esperado: MariaDB en `10.33.199.11:3306`, cuentas para `.12` y `.13`, NFS activo y exportación solo a esos dos clientes. `findmnt` muestra el sistema de archivos que contiene la exportación; no demuestra por sí solo un disco de datos separado.

## Cliente Windows conectado por VPN

```powershell
Test-NetConnection 10.33.195.224 -Port 53
Test-NetConnection 10.33.195.224 -Port 443
Resolve-DnsName equipo-tas-01.local -Server 10.33.195.224
Resolve-DnsName grafana.equipo-tas-01.local -Server 10.33.195.224
Resolve-DnsName prometheus.equipo-tas-01.local -Server 10.33.195.224
Get-DnsClientNrptRule | Where-Object { $_.Namespace -match 'equipo-tas-01\.local' }
```

La última consulta debe estar vacía en el equipo usado para la prueba del 23-11, porque la regla temporal se revirtió. Las consultas con `-Server` prueban BIND directamente; no prueban que el navegador use ese DNS por defecto. El certificado seguirá dando advertencia hasta que el cliente confíe explícitamente en la CA interna.

## Pruebas pendientes de aceptación

Registrar un reinicio controlado y comprobar todos los montajes y servicios; apagar por separado cada backend y observar la continuidad desde el navegador; restaurar un respaldo en un destino de prueba; y completar una operación de WooCommerce con datos ficticios. Anotar fecha, VM, esperado, observado y evidencia para cada prueba.
