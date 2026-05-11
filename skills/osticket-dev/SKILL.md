---
name: osticket-dev
description: >
  Expert workflow skill for developing, debugging, and operating customized osTicket
  installations running in Docker. Use this skill whenever the user is working with
  osTicket source code, Docker containers running osTicket, PHP errors in osTicket,
  SQL queries against osTicket databases, deploying file changes to a container,
  or debugging email/mailer issues in osTicket. Trigger on any mention of: osTicket,
  tickets/asesorias/requerimientos in a helpdesk context, docker cp to a PHP container,
  thread_actions, ticket-view, class.mailer, bootstrap.php errors, SLA/ANS in a
  support system, or any osTicket-specific file path patterns like scp/, include/staff/,
  include/client/, assets/default/.
---

# osTicket Dev & Ops Skill

You're working with a customized osTicket installation. This skill gives you the
mental model, file structure, deployment patterns, and diagnostic queries you need
to move fast without breaking things.

## Project structure — where things live

```
upload/                         ← webroot (what Apache serves)
├── bootstrap.php               ← entry point: DB connect, config load
├── main.inc.php                ← required at top of every page
├── scp/                        ← Staff Control Panel (agents/admins)
│   ├── tickets.php             ← ticket list
│   ├── css/scp.css             ← main SCP styles
│   ├── css/ticket-reply-enhancements.css  ← custom reply form styles
│   └── js/                     ← jquery.dropdown.js, scp.js, tips.js, shortcuts.js
├── include/
│   ├── staff/                  ← SCP PHP includes
│   │   ├── ticket-view.inc.php ← main ticket view (most UI changes go here)
│   │   ├── header.inc.php      ← SCP page header
│   │   ├── footer.inc.php      ← SCP page footer + JS loading
│   │   └── templates/          ← modal/popup templates (thread-entry, task-preview…)
│   ├── client/                 ← client portal PHP includes
│   │   ├── header.inc.php
│   │   └── templates/
│   ├── class.mailer.php        ← email sending logic
│   ├── class.thread_actions.php ← dropdown actions on thread entries
│   ├── class.ticket.php        ← core ticket model
│   ├── class.queue.php         ← ticket queue/filters
│   └── plugins/                ← plugin .phar files
├── assets/default/css/
│   └── theme.css               ← client portal theme
└── images/                     ← logos, icons, login-frase.svg
```

Read `references/file-map.md` for a deeper map when you need to find where a
specific behavior lives.

---

## Deployment — how to push changes

osTicket Docker containers typically have **no bind mount** — changes to local files
don't appear in the container automatically. You have two deployment paths:

### Quick deploy (single file changed)
```bash
docker cp "SoporteX/upload/path/to/file.php" <container>:/var/www/html/path/to/file.php
```
Fast, no downtime. Use for CSS, JS, PHP includes, templates.

### Full rebuild (Dockerfile or new dependencies changed)
```bash
# from the directory with docker-compose.yml
docker compose build <service>
docker compose up -d <service>
```
Required when: Dockerfile changed, new files added that aren't docker cp'd yet,
or the container was recreated and lost previous docker cp changes.

**After a rebuild, all previous docker cp changes are gone** — the image is the
source of truth. Always commit changes to the repo before rebuilding.

### Verify the deploy landed
```bash
docker exec <container> php -l /var/www/html/path/to/file.php   # syntax check
docker exec <container> sh -c "grep -n 'keyword' /var/www/html/path/to/file.php"
```

---

## Diagnosis — first things to check

When something is broken, check in this order:

```bash
# 1. Container running?
docker ps | grep <container>

# 2. Recent PHP errors
docker logs <container> --tail 50

# 3. Syntax error in a specific file
docker exec <container> php -l /var/www/html/include/file.php

# 4. Apache error log (more detail than docker logs)
docker exec <container> tail -50 /var/log/apache2/error.log

# 5. DB reachable from container?
docker exec <container> php -r "new mysqli(getenv('MYSQL_HOST'), getenv('MYSQL_USER'), getenv('MYSQL_PASSWORD'), getenv('MYSQL_DATABASE')) or die('fail');"
```

Common error patterns and what they mean:
- `Failed opening required 'bootstrap.php'` → wrong webroot or missing core file
- `Call to undefined function _S()` → `class.i18n.php` not loaded; usually a DB connection failure triggered it
- `Failed opening required 'class.model.php'` → file missing from `include/`
- `Cannot serve directory` → Apache DocumentRoot mismatch or missing `index.php`

Read `references/errors.md` for a longer catalog of osTicket error patterns.

---

## SQL diagnostics

Connect to the DB:
```bash
docker exec <db_container> mysql -u<user> -p<pass> <database>
```

Read `references/sql.md` for ready-to-run diagnostic queries covering:
- Recent tickets and status
- Email suppression/bounce tables
- SLA/ANS compliance
- Staff and department setup
- Queue configuration

---

## Key customization patterns

### Adding a warning banner to ticket-view
Check `$warn` variable early in `ticket-view.inc.php` — append HTML to it:
```php
$warn .= ($warn ? '<br>' : '') . 'Your message here';
```
The `$warn` var is rendered in the sticky toolbar at the top of the ticket view.

### Disabling the Post Reply button conditionally
In `ticket-view.inc.php`, find the submit button (~line 998) and add:
```php
<input class="save pending" type="submit" value="<?php echo __('Post Reply');?>"
    <?php if ($yourCondition) { ?>disabled="disabled"<?php } ?>>
```
Also block the tab click in the JS section below (search for `post-response`).

### Adding a new thread action (dropdown item)
In `class.thread_actions.php`, extend `ThreadEntryAction`:
```php
class MyAction extends ThreadEntryAction {
    static $id = 'my_action';
    static $name = /* trans */ 'Mi Acción';
    // implement render() and trigger()
}
```
Register it at the bottom of the file.

### Intercepting email before send
In `class.mailer.php`, find `sendmail()` or `send()` — add your check before
`$mail->send()`. Return early or log if you want to suppress sending.

---

## OpenCode usage

OpenCode reads this skill via the `CLAUDE.md` / project context. The same
deployment commands and file paths apply. Use the terminal workflow:
1. Edit file locally
2. `docker cp` to container
3. Verify with `curl` or browser

---

## When to read the reference files

- **`references/file-map.md`** — need to find where a behavior lives in the codebase
- **`references/errors.md`** — debugging a specific PHP/Apache error
- **`references/sql.md`** — running diagnostic queries against the DB
