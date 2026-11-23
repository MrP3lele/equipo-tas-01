# VMNFS — archivos subidos compartidos

VMNFS (VM 312) usa `10.33.199.14/29` en la red privada; su IP de acceso observada el 23 de noviembre fue `10.33.195.166/25`.

`/etc/exports` publica `/srv/nfs/wp-uploads` únicamente a `10.33.199.12` y `10.33.199.13`. Las opciones observadas son `rw,sync,no_subtree_check,all_squash,anonuid=33,anongid=33`. El servicio `nfs-server` apareció activo y habilitado. Los dos backends montan esa exportación en `/var/www/html/wp-content/uploads`.

La exportación conserva archivos subidos; no sincroniza la base de datos, plugins, temas ni código de WordPress. El directorio exportado aparecía sobre el sistema de archivos raíz de VMNFS; no se acreditó un volumen independiente ni una restauración de prueba.

Ver [arquitectura y pendientes](../../docs/estado-nodo-sur.md).

## Línea observada en `/etc/exports`

```exports
/srv/nfs/wp-uploads 10.33.199.12(rw,sync,no_subtree_check,all_squash,anonuid=33,anongid=33) 10.33.199.13(rw,sync,no_subtree_check,all_squash,anonuid=33,anongid=33)
```
