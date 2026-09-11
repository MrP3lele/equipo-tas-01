# Respaldo y recuperación de VMMaty

Este documento describe el respaldo conocido anterior al cambio de dominio y las condiciones para repetirlo o probar su restauración. **No contiene un respaldo de datos y no acredita una recuperación ejecutada.**

## Respaldo histórico

Se creó el 7 de septiembre de 2026 el directorio privado `/root/respaldo-wp-dominio.dyXU0l` con:

| Archivo | Contenido y comprobación conocida |
| --- | --- |
| `wp-config.php` | Copia de configuración, aproximadamente 3,3 KB; contiene información sensible. |
| `wordpress.sql` | Exportación de la BD, aproximadamente 119 KB; la exportación terminó y el archivo no estaba vacío. |
| `permisos-originales.txt` | Usuario, grupo, permisos y ruta del archivo de configuración original. |

Los archivos del respaldo quedaron con propietario `root:root` y modo `600`, fuera de `/var/www/html`. No se suben al repositorio. El nombre del directorio no sustituye una comprobación actual de existencia, integridad o contenido.

## Crear otro respaldo

Ejecutar solo con autorización, desde una sesión root **en VMMaty**, después de comprobar espacio disponible y que `/var/www/html` identifica la instalación correcta. Este bloque crea archivos privados nuevos; no se ha ejecutado durante la preparación de los commits.

```bash
(
  set -eu
  umask 077
  cd /var/www/html
  tas_config=$(sudo -u www-data /usr/local/bin/wp \
    --path=/var/www/html config path)
  tas_respaldo=$(mktemp -d /root/respaldo-wp-dominio.XXXXXX)

  stat -c '%U %G %a %n' "$tas_config" \
    > "$tas_respaldo/permisos-originales.txt"
  install -o root -g root -m 600 "$tas_config" \
    "$tas_respaldo/wp-config.php"

  sudo -u www-data /usr/local/bin/wp --path=/var/www/html \
    db export - --single-transaction --quick --skip-lock-tables \
    --quiet > "$tas_respaldo/wordpress.sql"

  test -s "$tas_respaldo/wordpress.sql"
  printf 'Directorio del respaldo: %s\n' "$tas_respaldo"
)
```

- `set -eu` detiene esta secuencia ante fallos no manejados o variables sin definir. El subshell limita el efecto de `umask` y las variables.
- `mktemp -d` crea una carpeta nueva; no sobrescribe el respaldo anterior.
- `stat` conserva los permisos originales; `install` copia la configuración al respaldo con permisos restringidos.
- El guion de `db export -` entrega SQL a la salida estándar, redirigida al archivo por la sesión root. No publica credenciales en el comando.
- `--single-transaction` permite coherencia de tablas transaccionales como InnoDB; no cubre automáticamente motores no transaccionales ni cambios de estructura concurrentes.
- `--quick` procesa filas progresivamente; `--skip-lock-tables` evita bloqueos explícitos por tabla; `--quiet` reduce mensajes.
- `test -s` solo comprueba existencia y tamaño mayor que cero. No demuestra que la exportación sea restaurable.

Para una migración hay que coordinar una pausa de escrituras y verificar consistencia de BD y archivos. No basta con repetir este bloque mientras se siguen recibiendo pedidos.

## Límites del respaldo conocido

El respaldo documentado cubre BD y `wp-config.php`, **no una copia completa recuperable de WordPress ni de la VM**. Antes de declarar continuidad operacional faltan:

- Respaldar archivos necesarios del sitio, especialmente contenido subido, plugins y temas propios, junto con su política de reconstrucción.
- Respaldar configuraciones relevantes de Nginx, PHP-FPM y BIND, inventariando propietario, permisos y dependencias.
- Definir retención, copia separada y verificación de integridad.
- Restaurar en un destino de prueba, comprobar funcionamiento y medir RPO/RTO.

## Recuperación controlada pendiente

1. Identificar el respaldo exacto y verificar fecha, integridad, espacio y destino de prueba. No asumir que el backup de septiembre contiene cambios posteriores.
2. Obtener autorización y preservar el estado actual antes de cualquier sustitución.
3. Preparar una instancia aislada compatible y restaurar configuración, datos y archivos necesarios sin sobrescribir la instancia activa.
4. Recuperar `wp-config.php` con permisos adecuados para PHP, usando el registro original. No instalar directamente en el sitio una copia `root:root 600` que el servicio no pueda leer.
5. Validar sintaxis, conexión a BD, raíz, login, administración, contenido y la operación empresarial elegida.
6. Documentar tiempos, resultados y diferencias; solo entonces decidir la recuperación o migración definitiva.

No se proporciona un comando de importación contra la BD activa porque el destino aún debe validarse. Conservar la BD local durante una futura migración hasta aceptar el resultado y acordar el tratamiento de las escrituras posteriores; revertir `DB_HOST` por sí solo no sincroniza esos datos.
