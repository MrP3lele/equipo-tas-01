# VMOmar: la base de datos compartida

VMOmar guarda la base `wordpress` que consultan VMMaty y VMMaty2. Su dirección privada es `10.33.199.11`; los backends llegan a MariaDB por el puerto 3306. Mantener una sola base evita que cada backend muestre contenidos o pedidos distintos.

[Volver al inicio](../../README.md) · [Ver el recorrido completo](../estado-nodo-sur.md)

## Qué se cambió

En un primer momento MariaDB escuchaba en `0.0.0.0:3306`, es decir, en todas las interfaces. Después la escucha se restringió a `10.33.199.11:3306`. La salida de cuentas mostró `wp_user` autorizado desde `10.33.199.12` y `10.33.199.13`. A su vez, los dos `wp-config.php` pasaron a usar `DB_HOST=10.33.199.11`.

El 23 de octubre se listó `/root/wordpress-pre-cutover.sql`, un respaldo privado de 2,5 MB creado antes del cambio. Debe permanecer en la VM y fuera de Git. Saber que existe y que tiene datos no demuestra que pueda restaurarse: todavía falta una recuperación en un destino de prueba.

## Dónde mirar

| Lugar | Qué comprobar |
| --- | --- |
| Servicio `mariadb` | Que esté activo y habilitado para arrancar con la VM. |
| Escucha TCP | Que el puerto 3306 aparezca en `10.33.199.11`, no en `0.0.0.0`. |
| Cuentas SQL | Que `wp_user` tenga los orígenes `.12` y `.13` necesarios. No publicar contraseñas ni grants con secretos. |
| Configuración de MariaDB | Falta contrastar el nombre exacto del archivo que se editó; no se incluye una copia supuesta. |
| `/root/wordpress-pre-cutover.sql` | Respaldo histórico, no una política de recuperación. |

En la consola de Proxmox del 23 de noviembre se veía `mariadb` habilitado y escuchando en `.11:3306`. Las pruebas desde ambos backends habían dado conexión TCP y HTTP 200. Eso confirma que WordPress podía consultar la base en ese momento; aún no confirma restauración, cifrado del enlace a MariaDB ni una compra completa. Los [comandos de revisión](../verificacion-infraestructura.md#vmomar-y-vmnfs) ayudan a repetir la comprobación.
