# VMMaty2 — backend WordPress 2

VMMaty2 (VM 311) usa `10.33.195.228/25` y `10.33.199.13/29`. Su WordPress está en `/var/www/html`; `/etc/nginx/sites-enabled/wordpress` usa `equipo-tas-01.local`, `/var/www/html` y el socket `/run/php/php8.1-fpm.sock`. Nginx y `php8.1-fpm` estaban activos; el sitio local devolvió HTTP 200 tras corregir la extensión `mysqli` y el acceso a MariaDB.

`/var/www/html/wp-config.php` apunta a `10.33.199.11` con la base `wordpress` y el usuario `wp_user`. No subirlo: contiene secretos. El directorio `/var/www/html/wp-content/uploads` monta `10.33.199.14:/srv/nfs/wp-uploads`.

La revisión de noviembre mostró la entrada NFS en `/etc/fstab` con `_netdev`, `nofail` y `x-systemd.automount`, además de una dependencia de Nginx hacia el montaje. `findmnt` mostró NFS activo y la raíz del sitio devolvió 200. El chequeo `/health.php` devolvió 403 al consultarlo desde localhost; corresponde verificar 200 desde el proxy autorizado `10.33.199.10`.

La copia inicial del sitio mediante `wp-site.tgz` quedó incompleta al escribirla en el mismo NFS. Se rehizo desde `/tmp` y después se transfirió. El paquete temporal no debe permanecer en `uploads`. La copia de código, plugins y temas entre backends no queda automatizada por el montaje NFS.

Ver [estado del nodo](../../docs/estado-nodo-sur.md) y [verificación](../../docs/verificacion-infraestructura.md).

## Fragmento visible de Nginx

La consola mostró estas líneas de `/etc/nginx/sites-enabled/wordpress`; el archivo enlazado de `sites-available` debe comprobarse antes de una restauración:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name equipo-tas-01.local www.equipo-tas-01.local;
    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }
}
```
