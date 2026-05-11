# SQL Diagnostic Queries — osTicket

Estas queries son de solo lectura y seguras para correr en producción.

## Tickets recientes y estado

```sql
-- Últimos 20 tickets abiertos
SELECT t.number, t.created, ts.name AS status, u.name AS user_email,
       s.name AS assigned_to, t.lastmessage
FROM ost_ticket t
JOIN ost_ticket_status ts ON t.status_id = ts.id
LEFT JOIN ost_user u ON t.user_id = u.id
LEFT JOIN ost_staff s ON t.staff_id = s.staff_id
WHERE ts.state = 'open'
ORDER BY t.lastmessage DESC LIMIT 20;

-- Tickets por estado
SELECT ts.name, ts.state, COUNT(*) as total
FROM ost_ticket t
JOIN ost_ticket_status ts ON t.status_id = ts.id
GROUP BY ts.id ORDER BY total DESC;
```

## Correo y supresiones Postmark

```sql
-- Estado actual de emails suprimidos
SELECT email, is_suppressed, suppression_reason, origin, last_event_at, updated_at
FROM ost_postmark_email_status
WHERE is_suppressed = 1
ORDER BY updated_at DESC LIMIT 20;

-- Últimos eventos recibidos
SELECT id, email, event_type, suppression_reason, suppress_sending, event_at
FROM ost_postmark_email_events
ORDER BY id DESC LIMIT 10;
```

## SLA / ANS

```sql
-- Tickets que vencen pronto o ya vencidos
SELECT t.number, t.created, sla.name AS sla_plan,
       t.duedate, TIMESTAMPDIFF(HOUR, NOW(), t.duedate) AS horas_restantes
FROM ost_ticket t
JOIN ost_sla sla ON t.sla_id = sla.id
JOIN ost_ticket_status ts ON t.status_id = ts.id
WHERE ts.state = 'open' AND t.duedate IS NOT NULL
ORDER BY t.duedate ASC LIMIT 20;
```

## Staff y departamentos

```sql
-- Agentes activos y su departamento principal
SELECT s.firstname, s.lastname, s.email, d.name AS department, s.isactive
FROM ost_staff s
LEFT JOIN ost_department d ON s.dept_id = d.id
WHERE s.isactive = 1 ORDER BY d.name, s.firstname;
```

## Configuración

```sql
-- Config general del sistema
SELECT namespace, `key`, LEFT(value, 100) AS value
FROM ost_config
WHERE namespace = 'core'
ORDER BY `key`;

-- Colas configuradas
SELECT id, title, sort, flags FROM ost_queue ORDER BY sort;
```

## Email / cuentas de correo

```sql
SELECT e.name, e.email, ea.host, ea.port, ea.protocol, ea.active
FROM ost_email e
LEFT JOIN ost_email_account ea ON e.id = ea.email_id
ORDER BY e.email;
```
