# VMNFS: los archivos que comparten los backends

Cuando alguien sube una imagen a WordPress, queremos que aparezca igual aunque HAProxy envíe la siguiente visita al otro backend. VMNFS resuelve esa parte: exporta `/srv/nfs/wp-uploads` desde `10.33.199.14` y ambos servidores web lo montan como `/var/www/html/wp-content/uploads`.

[Volver al inicio](../../README.md) · [Ver ambos backends](../estado-nodo-sur.md#flujo-de-una-solicitud)

## Quién puede entrar al recurso

En `/etc/exports` solo están autorizados `10.33.199.12` y `10.33.199.13`. La línea observada usa `rw,sync,no_subtree_check,all_squash,anonuid=33,anongid=33`. En la práctica, `all_squash` hace que las operaciones de los clientes se traten como `www-data` (UID y GID 33). El servicio `nfs-server` apareció activo y habilitado.

La carpeta exportada estaba en el sistema de archivos raíz de VMNFS. No quedó acreditado un disco separado. Tampoco hay una restauración de los archivos subidos registrada.

## Cómo queda disponible después de arrancar

Los clientes tienen una entrada NFS en `/etc/fstab`. En VMMaty2 se vio además `x-systemd.automount`; en ambos backends Nginx mostró una dependencia hacia el montaje. Durante la revisión de noviembre, `findmnt` mostró la exportación activa y VMMaty pudo escribir y leer un archivo de prueba en ella.

Esto documenta la configuración de arranque, pero la prueba que falta es reiniciar las VM de manera controlada y comprobar que NFS se monte antes de servir el sitio. Si `findmnt -T /var/www/html/wp-content/uploads` muestra `/` como origen, se está viendo la carpeta local y no el recurso compartido.

## El límite importante

NFS comparte `uploads`; no sincroniza plugins, temas, código ni la base de datos. También conviene crear los paquetes de despliegue fuera de `uploads`: el primer `wp-site.tgz` creado ahí quedó truncado. Puedes ver la [línea de exportación documentada](../../configuración/VMNFS/readme.md) y repetir las [pruebas de NFS](../verificacion-infraestructura.md#vmomar-y-vmnfs).
