# VMOmar — MariaDB compartida

VMOmar usa `10.33.199.11/29` como dirección privada. La última salida compartida mostró MariaDB activo y escuchando solo en `10.33.199.11:3306`; antes escuchaba en `0.0.0.0:3306`. La cuenta `wp_user` figuraba limitada a los orígenes `10.33.199.12` y `10.33.199.13`.

La base `wordpress` ya existía. Ambos backends mostraron `DB_HOST=10.33.199.11`, conectividad TCP y HTTP 200 después del ajuste. El 23 de octubre se listó un respaldo privado `/root/wordpress-pre-cutover.sql` de 2,5 MB; tamaño y presencia no acreditan restauración.

Una consola de Proxmox del 23 de noviembre mostró `mariadb` habilitado y el puerto 3306 limitado a `.11`. No se ha contrastado una copia del archivo de configuración ni el estado actual de TLS o grants específicos. No subir SQL, contraseñas ni datos de clientes. Véase [estado del nodo](../../docs/estado-nodo-sur.md).
