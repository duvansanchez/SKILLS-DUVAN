# osTicket — Catálogo de errores comunes

## PHP Fatal Errors

### `Failed opening required 'bootstrap.php'`
- **Causa**: Apache sirve desde el directorio equivocado, o `bootstrap.php` no está en el webroot.
- **Fix**: Verificar que `upload/` está montado correctamente en `/var/www/html/`. Confirmar que `bootstrap.php` existe en el webroot.

### `Failed opening required 'class.model.php'`
- **Causa**: Archivo faltante en `include/`. Típico después de clonar un repo incompleto.
- **Fix**: Copiar `class.model.php` desde otra instalación o el repo de referencia.

### `Call to undefined function _S()` en class.mailer.php
- **Causa**: `class.i18n.php` no fue cargado. Casi siempre porque hubo un error de BD antes (Bootstrap::connect() falló y llamó sendmail antes de cargar i18n).
- **Fix**: Resolver el error de conexión a BD primero. Verificar `MYSQL_HOST`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE` en el entorno del contenedor.

### `Call to undefined function osTicket\Mail\_S()`
- **Mismo que arriba** — el namespace prefix lo confirma. La causa raíz siempre es BD.

### `Cannot serve directory /var/www/html/`
- **Causa**: No hay `index.php` en el webroot o Apache tiene DirectoryIndex deshabilitado.
- **Fix**: Verificar que `upload/index.php` existe y está accesible.

---

## Errores de conexión a BD

### `Database error occurred`
- Verificar variables de entorno en docker-compose.yml:
  ```
  MYSQL_HOST, MYSQL_DATABASE, MYSQL_USER, MYSQL_PASSWORD
  ```
- Si usás `localhost` como host dentro de Docker, cambiarlo por el nombre del servicio (ej. `db_piezas`).
- Verificar que el contenedor de BD está corriendo: `docker ps | grep db_`

---

## Errores de permisos

### `Permission denied` al hacer `docker cp`
- El archivo destino en el contenedor probablemente está en un directorio de solo lectura.
- **Fix**: `docker exec <container> chmod -R 755 /var/www/html/include/`

### Archivos PHP actualizados pero la app no los refleja
- ¿Hay un opcode cache (OPcache) activo? Reiniciar Apache dentro del contenedor:
  ```bash
  docker exec <container> service apache2 reload
  ```
- O reiniciar el contenedor: `docker restart <container>`

---

## Errores de Bootstrap / inicialización

### Página en blanco (sin error visible)
- Activar display_errors temporalmente:
  ```bash
  docker exec <container> php -r "error_reporting(E_ALL); ini_set('display_errors','1'); require '/var/www/html/bootstrap.php';"
  ```

### `ost-config.php` faltante o no configurado
- Existe `ost-sampleconfig.php` como plantilla.
- El archivo real con credenciales no se versiona (gitignore).
- Si el contenedor arrancó limpio, `bootstrap.php` redirige al wizard de instalación.
