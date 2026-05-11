# osTicket — Mapa de archivos y comportamientos

## ¿Dónde vive cada comportamiento?

| Qué querés cambiar | Archivo |
|---|---|
| Vista principal de un ticket (staff) | `include/staff/ticket-view.inc.php` |
| Header del SCP | `include/staff/header.inc.php` |
| Footer del SCP (carga JS) | `include/staff/footer.inc.php` |
| Login del SCP | `include/staff/login.tpl.php` + `login.header.php` |
| Dropdown de acciones en thread entry | `include/class.thread_actions.php` |
| Modal de tarea (task preview) | `include/staff/templates/task-preview.tmpl.php` |
| Template de entrada de thread | `include/staff/templates/thread-entry.tmpl.php` |
| Envío de correo | `include/class.mailer.php` |
| Modelo de ticket | `include/class.ticket.php` |
| Modelo de tarea | `include/class.task.php` |
| Colas y filtros | `include/class.queue.php` |
| Portal del cliente (header) | `include/client/header.inc.php` |
| Portal del cliente (sidebar) | `include/client/templates/sidebar.tmpl.php` |
| Página de inicio del portal | `upload/index.php` |
| CSS del SCP | `scp/css/scp.css` |
| CSS del formulario de respuesta | `scp/css/ticket-reply-enhancements.css` |
| CSS del login | `scp/css/login.css` |
| CSS del portal cliente | `assets/default/css/theme.css` |
| JS principal SCP | `scp/js/scp.js` |
| JS dropdown | `scp/js/jquery.dropdown.js` |
| JS tooltips/previews | `scp/js/tips.js` |
| JS shortcuts (Ctrl+K etc.) | `scp/js/shortcuts.js` |
| Visor de adjuntos | `scp/file-preview.php`, `scp/doc-preview.php` |
| Plugin de almacenamiento | `include/plugins/storage-fs.phar` |

## Flujo de carga de una página SCP

```
scp/tickets.php
  → require scp/staff.inc.php
    → require main.inc.php
      → require bootstrap.php   (BD, config, autoload)
      → Bootstrap::loadCode()   (carga clases)
      → Bootstrap::connect()    (conecta BD)
  → require include/staff/ticket-view.inc.php  (lógica + HTML)
  → require include/staff/footer.inc.php        (JS al final)
```

## Namespaces y autoload

osTicket usa autoload clásico de PHP 5 — no PSR-4.
Las clases viven directamente en `include/class.*.php`.
`Bootstrap::loadCode()` en `bootstrap.php` hace el require de las clases core.

## Hooks y señales

osTicket usa un sistema de señales propio:
```php
Signal::connect('model.created', function($object) { ... });
```
Ver `include/class.signal.php` para la implementación.

## Templates (tmpl.php)

Los archivos `*.tmpl.php` en `include/staff/templates/` son cargados dinámicamente
vía AJAX y renderizados como modales/popups. Se cargan con `tips.js` o directamente
con `$.ajax()` desde `scp.js`.
