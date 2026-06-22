# InvGate Service Management — API v1

Base URL: `https://<instancia>/api/v1/<endpoint>`  
Autenticación: HTTP Basic Auth

---

## Tickets (Requests)

### `incident` — Un ticket

**GET** — Retorna la información de un ticket.

| Parámetro | Tipo | Req | Descripción |
|---|---|---|---|
| `id` | INTEGER | ✓ | ID del ticket |
| `date_format` | STRING | | `epoch` (default) o `iso8601` |
| `comments` | BOOLEAN | | Incluir comentarios |
| `decoded_special_characters` | BOOLEAN | | Devolver mensaje decodificado |

**POST** — Crea un nuevo ticket.

| Parámetro | Tipo | Req | Descripción |
|---|---|---|---|
| `creator_id` | INTEGER | ✓ | |
| `customer_id` | INTEGER | ✓ | |
| `category_id` | INTEGER | ✓ | Ver `/categories` |
| `priority_id` | INTEGER | ✓ | 1=Low, 2=Medium, 3=High, 4=Urgent, 5=Critical |
| `type_id` | INTEGER | ✓ | 1=Incident, 2=Service Request, 3=Question, 4=Problem, 5=Change, 6=Major Incident |
| `title` | STRING | ✓ | |
| `source_id` | INTEGER | | |
| `description` | TEXT | | |
| `date` | STRING | | Epoch timestamp de ocurrencia |
| `attachments` | ARRAY | | |
| `related_to` | ARRAY | | IDs de tickets a vincular |
| `location_id` | INTEGER | | |

**PUT** — Modifica atributos de un ticket.

| Parámetro | Tipo | Req | Descripción |
|---|---|---|---|
| `id` | INTEGER | ✓ | |
| `customer_id`, `category_id`, `priority_id`, `type_id`, `source_id` | INTEGER | | |
| `title` | STRING | | |
| `description` | TEXT | | |
| `location_id` | INTEGER | | |
| `date`, `date_format` | STRING | | |
| `reassignment` | BOOLEAN | | Reasignar según nuevas propiedades |

**Campos de respuesta comunes:**
`id`, `title`, `category_id`, `description`, `priority_id`, `user_id`, `creator_id`, `assigned_id`, `assigned_group_id`, `date_ocurred`, `source_id`, `status_id`, `type_id`, `created_at`, `last_update`, `process_id`, `solved_at`, `closed_at`, `closed_reason` (1=aceptada, 2=expirada, 3=timeout, 4=workflow), `attachments`, `custom_fields`, `sla_incident_resolution`, `sla_incident_first_reply`, `comments`, `rating`, `pretty_id`, `request_customer_sentiment_initial`, `request_customer_sentiment_current`

---

### `incidents` — Múltiples tickets

**GET** — Lista tickets por IDs.

| Parámetro | Tipo | Req | Descripción |
|---|---|---|---|
| `ids` | ARRAY | ✓ | IDs de tickets |
| `date_format` | STRING | | |
| `comments` | BOOLEAN | | |

---

### `incidents.by.status` — Tickets por estado

**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `status_id` | INTEGER | ID de un estado |
| `status_ids` | ARRAY | Múltiples IDs |
| `limit` | INTEGER | Items por página |
| `offset` | INTEGER | Items a saltear |

Respuesta: `{ status, info, requestIds[], limit, offset, total }`

---

### `incidents.by.helpdesk` — Tickets abiertos por help desk

**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `helpdesk_id` | INTEGER | |
| `helpdesk_ids` | ARRAY | |

Respuesta: `{ status, info, requestIds[] }`

---

### `incidents.by.agent` — Tickets por agente

**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `id` / `email` / `username` | | Identificador del agente |
| `comments` | BOOLEAN | |
| `page_key` | STRING | Cursor de paginación |
| `limit` | INTEGER | |

---

### `incidents.by.customer` — Tickets por cliente

**GET** — Mismos parámetros que `incidents.by.agent` pero para clientes. Incluye `next_page_key` en respuesta.

---

### `incidents.by.view` — Tickets por vista

**GET**

| Parámetro | Tipo | Req | Descripción |
|---|---|---|---|
| `view_id` | INTEGER | ✓ | |
| `limit` | INTEGER | | |
| `offset` | INTEGER | | |

---

### `incidents.details.by.view` — Tickets con detalle por vista

**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `view_id` | INTEGER | ✓ |
| `page_key` | STRING | Cursor de paginación |
| `sort_by` | STRING | `id` o `last_update` |
| `order_by` | STRING | `asc` o `desc` |

Respuesta: `{ metadata[], data[], next_page_key }` (1000 tickets por página)

---

### `incidents.by.sentiment` — Tickets por sentimiento

**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `sentiment_id` | STRING | `negative`, `positive`, `neutral` |
| `sentiment_ids` | ARRAY | |
| `limit` / `offset` | INTEGER | |

---

### `incidents.by.asset` — Tickets por asset

**GET** — Parámetro: `asset_id` (STRING, requerido).

---

### `incidents.last.hour` — Tickets creados en la última hora

**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `page_key` | STRING | Cursor de paginación |
| `limit` | INTEGER | |

---

## Acciones sobre Tickets

### `incident.reassign` — Reasignar ticket

**POST**

| Parámetro | Tipo | Req | Descripción |
|---|---|---|---|
| `request_id` | INTEGER | ✓ | |
| `author_id` | INTEGER | ✓ | |
| `group_id` | INTEGER | ✓ | Help desk destino |
| `agent_id` | INTEGER | | |

---

### `incident.reopen` — Reabrir ticket

**PUT**

| Parámetro | Tipo | Req |
|---|---|---|
| `request_id` | INTEGER | ✓ |
| `author_id` | INTEGER | |

---

### `incident.reject` — Rechazar ticket

**POST** — `request_id` (✓), `author_id` (✓)

---

### `incident.cancel` — Cancelar ticket

**POST** — `request_id` (✓), `author_id` (✓), `comment` (opcional)

---

### `incident.promote.to.major.incident` — Promover a incidente mayor

**POST** — `request_id` (✓), `author_id` (✓), `confirm` (BOOLEAN)

Respuesta: `{ success, messages[] }`

---

### `incident.solution.accept` — Aceptar solución

**PUT** — `id` (✓), `rating` 1–5 (✓), `comment` (requerido si rating < 4)

---

### `incident.solution.reject` — Rechazar solución

**PUT** — `id` (✓), `comment` (✓)

---

## Relaciones de Tickets

### `incident.link` — Tickets vinculados

**GET** — `request_id` (✓) → `[{ id, title }]`

**POST** — `request_id` (✓), `request_ids[]` (✓)

---

### `incident.observer` — Observadores

**GET** — `request_id` (✓) → `[{ user_id }]`

**POST** — `request_id` (✓), `author_id` (✓), `user_id` o `users_id[]`

---

### `incident.collaborator` — Colaboradores

**GET** — `request_id` (✓) → `[{ id }]`

**POST** — `request_id` (✓), `author_id` (✓), `user_id` o `users_id[]`

---

### `incident.external_entity` — Entidades externas

**GET** — `request_id` (✓) → `[{ link_id, ref_id, ext_ref_id, type, name, status }]`

**POST** — `request_id` (✓), `external_entity_id` (✓), `external_entity_ref_id`

---

### `incident.tasks` — Tareas del ticket

**GET** — `request_id` (✓) → `[{ task_id, name, description, expiration_date, agent_id, helpdesk_id, status (0=open,1=done,2=deleted,3=skipped), ... }]`

---

## Estados de Espera (Waiting For)

### `incident.waitingfor.incident`
**POST** — `request_id` (✓), `wait_request_id` (✓)

### `incident.waitingfor.external_entity`
**POST** — `request_id` (✓), `entity_link_id` (✓)

### `incident.waitingfor.agent`
**POST** — `request_id` (✓)

### `incident.waitingfor.customer`
**POST** — `request_id` (✓)

### `incident.waitingfor.date`
**POST** — `request_id` (✓), `timestamp` (✓, epoch)

---

## Adjuntos y Comentarios

### `incident.attachment` — Adjuntos

**GET** — `id` (✓) → `{ id, name, url, hash, extension }`

---

### `incident.comment` — Comentarios

**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `request_id` | INTEGER | ✓ |
| `date_format` | STRING | |
| `is_solution` | BOOLEAN | 1=solución, 0=no |
| `decoded_special_characters` | BOOLEAN | |

Respuesta: `[{ id, incident_id, author_id, message, created_at, customer_visible, reference, msg_num, is_solution, attachments[] }]`

**POST** — `request_id` (✓), `comment` (✓), `author_id` (✓), `is_solution`, `customer_visible` (0=interno, 1=público), `attachments[]`, `is_propagation`

---

## Aprobaciones

### `incident.approval` — Instancias de aprobación

**GET** — `request_id` (✓), `date_format`, `only_pending` → `[{ id, author_id, status (-2=cancelada,-1=esperando,0=rechazada,1=aprobada), type (1=predefinida,2=espontánea), created_at, approval_request_id, approval_request_description }]`

---

### `incident.spontaneous_approval` — Aprobación espontánea
**POST** — `request_id` (✓), `author_id` (✓), `approval_user_id` (✓), `description` (✓)

---

### `incident.custom_approval` — Aprobación personalizada
**GET** — `request_id` (✓), `date_format`

**POST** — `request_id` (✓), `author_id` (✓), `approval_id` (✓), `description`

---

### `incident.approval.accept` / `incident.approval.reject` / `incident.approval.cancel`

**PUT**

| Parámetro | Tipo | Req |
|---|---|---|
| `approval_id` | INTEGER | ✓ |
| `user_id` | INTEGER | ✓ |
| `note` | STRING | (no aplica a cancel) |

---

### `incident.approval.add_voter`
**POST** — `approval_id` (✓), `user_id` (✓)

### `incident.approval.possible_voters`
**GET** — `approval_id` (✓), `only_pending`

### `incident.approval.status` / `incident.approval.vote_status` / `incident.approval.type`
**GET** — Sin parámetros. Devuelven catálogos de IDs y nombres.

---

## Atributos de Tickets

### `incident.attributes.status` / `incident.attributes.statuses`
**GET** — `id` (opcional) → `[{ id, name }]`

### `incident.attributes.priority`
**GET** — `id` (opcional) → `[{ id, name }]` (1=Low … 5=Critical)

### `incident.attributes.type`
**GET** — `id` (opcional) → `[{ id, name }]` (1=Incident … 6=Major Incident)

### `incident.attributes.source`
**GET** — `id` (opcional) → `[{ id, name }]`

### `categories` / `incident.attributes.category`
**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `id` | INTEGER | Categoría específica |
| `search` | STRING | Búsqueda por nombre |
| `page` / `page_size` | INTEGER | Paginación (max 500) |

Respuesta: `[{ id, name, parent_category_id }]`

---

## Campos Personalizados (Custom Fields)

### `cf.fields.all` — Todos los CF activos
**GET** → `[{ uid, label, description, type, categories, is_required }]`

### `cf.fields.shared.all` — CF compartidos
**GET** → `[{ uid, label, description, type }]`

### `cf.fields.types` — Tipos de CF
**GET** → `[{ id, name }]`

### `cf.fields.by.category`
**GET** — `category_id` (✓) → `[{ uid }]`

### `cf.starting.fields.by.category` — CF al inicio de creación
**GET** — `category_id` (✓), `language` → `[{ uid, label, type_id, subtype_id, multiple_selection, is_required }]`

---

### `incident.custom_field` — CF de un ticket

**POST** — Crea/sobreescribe valor: `request_id` (✓), `author_id` (✓), `custom_field_uid` (✓), `values[]` (✓)

**PATCH** — Agrega valor (solo tipo CI): mismos parámetros.

**DELETE** — Elimina valor: `request_id` (✓), `author_id` (✓), `custom_field_uid` (✓)

---

### `cf.field.options` / `cf.field.options.list` — Opciones de CF tipo Lista

**GET** — `uid` (✓) → `{ key-value }`

**POST** / **PUT** — `uid` (✓), `type` (✓, `key-value`), `key_values[]` (✓)

**DELETE** — `uid` (✓), `type` (✓), `keys[]` (✓)

### `cf.field.options.list.config`
**GET** — `uid` (✓) → `{ configurations[], count }`

---

### `cf.field.options.tree` — Opciones de CF tipo Árbol

**GET** — `uid` (✓) → `[{ hash, name, key, children[] }]`

**POST** — `uid` (✓), `name` (✓), `key`, `parent_hash`

**PUT** — `uid` (✓), `hash` (✓), `key`, `name`

**DELETE** — `uid` (✓), `hash` (✓) — Elimina nodo y sus hijos

---

## Time Tracking

### `timetracking`

**GET**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `request_id` | INTEGER | Requerido si no se envía `from` |
| `from` | STRING | ISO-8601. Requerido si no se envía `request_id` |
| `to` | STRING | ISO-8601. Default: ahora |
| `date_format` | STRING | `iso8601` o `iso8601noT` (default) |

Respuesta: `[{ timetracking_id, incident, user_id, timetracking_category_id, status, comment, total (segundos), from, to }]`

**POST** — `request_id` (✓), `user_id` (✓), `to` (✓ epoch), `category_id`, `from` (epoch), `comment`

**DELETE** — `request_id` (✓), `timetracking_id` (✓), `user_id` (✓)

### `timetracking.attributes.category`
**GET** — `id` (opcional) → `[{ id, parent_id, name, cost_per_hour }]`

---

## Usuarios

### `user` — Un usuario

**GET** — `id` (✓), `include_disabled`

**POST** — Crea usuario. Requeridos: `name`, `lastname`, `email`. Opcionales: `username`, `pass`, `other-email`, `doc`, `mobile`, `office`, `other`, `fax`, `address`, `phone`, `city`, `country`, `birthday`, `position`, `department`, `manager_id`, `employee_number`, `localization`, `is_external`, `send_set_password_email`, `force_password_change`

**PUT** — `id` (✓) + campos opcionales a modificar (incluye `language`: es, en, en_GB, pt, fr, de, ca, it, nl, el)

**DELETE** — `id` (✓)

**Campos de respuesta:** `id`, `name`, `lastname`, `username`, `email`, `other_email`, `type` (1=System,2=LDAP,3=Webservice,4=Customer,5=Mailbox), `user_type` (1=agente, 2=usuario final), `role_name`, `is_external`, `is_disabled`, `is_deleted`, `employee_number`, `manager_id`, `position`, `department`, `location`, + datos de contacto

---

### `users` — Múltiples usuarios

**GET** — `ids[]` (opcional), `include_disabled`

---

### `user.by` — Buscar usuario
**GET** — `username` o `email` (al menos uno)

### `users.by` — Búsqueda extendida
**GET** — `username`, `email`, `phones`, `phone`, `office_phone`, `mobile_phone`, `fax_phone`, `other_phone`, `employee_number`, `exact_match`, `include_disabled`, `page_key`

### `users.by.extended` — Búsqueda con paginación offset
**GET** — Mismos params + `page`, `page_size` (max 100), `search` (por nombre/apellido)

### `users.groups` — Grupos, empresas, help desks y locaciones de usuarios
**GET** — `ids[]` (✓) → `[{ id, username, email, companies[], groups[], helpdesks[], locations[], locations_observed[], helpdesks_observed[], companies_observed[], groups_observed[] }]`

---

### Acciones de usuario

| Endpoint | Método | Acción |
|---|---|---|
| `user.token` | POST | Crea token de sesión. Parámetro: `id` |
| `user.convert` | POST | Convierte externo → interno. Parámetro: `id` |
| `user.disable` | PUT | Desactiva usuario. Parámetro: `id` |
| `user.enable` | PUT | Activa usuario. Parámetro: `id` |
| `user.password` | PUT | Cambia contraseña: `id`, `password`, `force_password_change` |
| `user.password.reset` | POST | Envía email: `id`, `type` (`NEW_USER` o `RESET_PASSWORD`) |

---

## Grupos

### `groups`

**GET** — `id`, `name` (opcionales) → `[{ id, name, total }]`

**POST** — `name` (✓) → `{ id }`

**DELETE** — `id` (✓)

### `groups.users`

**GET** — `id` (✓), `user_id` → `[{ id, name, username, email }]`

**POST** — `id` (✓), `users[]` (✓)

**DELETE** — `id` (✓), `users[]` (✓)

### `groups.observers`

**GET** — `ids[]` → `[{ id, name, observer_users[], observer_groups[] }]`

**POST** / **DELETE** — `id` (✓), `observer_users[]` (✓)

---

## Help Desks y Niveles

### `helpdesks`

**GET** — `id`, `name`, `include_deleted` → `[{ id, name, engine_id, status_id, parent_id, total_members }]`

### `levels`

**GET** — `id`, `include_deleted` → `[{ id, level_order, engine_id, status_id (1=Enabled,2=Suspend,3=OutTimeWork,4=Disabled), total_members, members_ids[] }]`

### `helpdesksandlevels`

**GET** — `id`, `include_deleted` → Combina helpdesks y niveles con estructura unificada.

### `helpdesks.observers` / `levels.observers`

**GET** — `ids[]` → `[{ id, name, observer_users[], observer_groups[] }]`

**POST** / **DELETE** — `id` (✓), `observer_users[]` (✓)

---

## Empresas (Companies)

### `companies`

**GET** — `id`, `name`, `external_id` → `[{ id, name, external_id, total }]`

**POST** — `name` (✓), `external_id`

**PUT** — `id` (✓), `name`, `external_id`

**DELETE** — `id` (✓)

### `companies.users`

**GET** — `id` (✓), `user_id`

**POST** / **DELETE** — `id` (✓), `users[]` (✓)

### `companies.groups`

**GET** — `id` (✓)

**POST** / **DELETE** — `id` (✓), `groups[]` (✓). DELETE también acepta `unlink_users_too` (BOOLEAN)

### `companies.observers`

**GET** — `ids[]`

**POST** / **DELETE** — `id` (✓), `observer_users[]` (✓)

---

## Locaciones

### `locations`

**GET** — `id` → `[{ id, name, total, parent_id }]`

**POST** — `name` (✓), `parent_id`

**DELETE** — `id` (✓) — Elimina la locación y sus sub-locaciones

### `locations.users`

**GET** — `id` (✓), `user_id`

**POST** / **DELETE** — `id` (✓), `users[]` (✓)

### `locations.observers`

**GET** — `ids[]`

**POST** / **DELETE** — `id` (✓), `observer_users[]` (✓)

---

## Base de Conocimiento (KB)

### `kb.articles`

**GET** — `sort_by` (asc/desc), `order_by` (last_update_date/id), `limit`, `offset`

→ `[{ id, title, author_id, content, creation_date, last_update_date, solved_requests, rating, views, category_id, is_private, attachments[], responsible_id }]`

**POST** — `title` (✓), `content` (✓), `author_id` (✓), `category_id` (✓), `description`, `is_private`, `attachments[]`, `responsible_id`

**PUT** — `id` (✓), `author_id` (✓), + campos opcionales

**DELETE** — `id` (✓)

### `kb.articles.by.ids`
**GET** — `ids[]` (✓)

### `kb.articles.by.keywords`
**GET** — `keywords` (✓), `min_search_scoring` (0-1), `limit` (default 25). Respuesta incluye `search_scoring`.

### `kb.articles.by.category`
**GET** — `category_id` (✓), `limit`, `offset`, `visibility` (1=Private, 2=Registered, 3=Public)

### `kb.articles.attachments`

**GET** — `article_id` (✓)

**POST** — `article_id` (✓), `author_id` (✓), `attachments[]`

**DELETE** — `article_id` (✓), `attachment_id` (✓)

### `kb.categories`

**GET** → `[{ id, parent_id, name }]`

**POST** — `name` (✓), `parent_id`

**PUT** — `id` (✓), `name`, `parent_id`

**DELETE** — `id` (✓)

### `kb.categories.by.ids`
**GET** — `ids[]` (✓)

---

## Workflows

### `wf.initialfields.by.category`
**GET** — `category_id` (✓) → `{ category_id, associated_workflow_id, associated_workflow_name, workflow_initial_fields[] }`

### `wf.deploy`
**PUT** — `workflow_id` (✓) → `{ status, workflow_id, description }`

### `workflow.process`
**GET** — `id` (opcional), `date_format`, `page_key`, `limit` (max 100)

→ `{ wf_id, name, description, author, created_at, status, status_name, versions[{ id, version, last_updated, last_editor, deployed_at, deployed_by, status, status_name, modified }] }`

### `workflow.field.list.values` — Campos lista en instancia de workflow

**GET** — `request_id` (✓), `field_id` (✓) → `{ request_id, field_id, status, values[] }`

**POST** — `request_id` (✓), `field_id` (✓), `value` o `values[]`, `author_id`, `operation` (`add` (default) o `set`)

---

## Breaking News

### `breakingnews`

**GET** — `id` (✓), `date_format` → `{ id, type_id, resolution_time, title, body, affected_helpdesk_ids[], affected_group_ids[], created_at, created_by_id, status_id }`

**POST** — `type_id` (✓, 1=High/2=Medium/3=Low), `title` (✓), `body` (✓), `affected_helpdesk_ids[]` o `affected_group_ids[]` (al menos uno), `resolution_time`, `creator_id`, `major_incident_id`

**PUT** — `id` (✓) + campos opcionales. `status_id`: 1=Open, 2=Closed.

### `breakingnews.all`
**GET** — `date_format` → lista completa

### `breakingnews.status` — Actualizaciones de una Breaking News

**GET** — `id` (✓), `date_format` → `[{ body, created_at, creator_id }]`

**POST** — `id` (✓), `body` (✓), `creator_id`, `is_solution` (1=cerrar)

### `breakingnews.attributes.type` / `breakingnews.attributes.status`
**GET** — `id` (opcional) → catálogo de IDs y nombres

---

## Notas Internas

### `internalnotes`

**GET** — `id` (✓), `object_id`, `object_type` (1=User,2=Helpdesk,3=Company,4=Group,5=ExternalEntity), `date_format`

**POST** — `object_id` (✓), `author_id` (✓), `object_type` (✓), `description` (✓), `title` (✓)

**PUT** — `id` (✓), `author_id` (✓), `description`, `title`

**DELETE** — `id` (✓), `author_id` (✓)

---

## Assets y CIs

### `assets`
**GET** — `assets_source_id` (✓), `assets_ids[]` (✓) → `[{ asset_id, status_id, incident_id }]`

### `cis.by.id`
**GET** — `ci_internal_ids[]` (✓), `ci_source_id` → `[{ ci_internal_id, ci_external_id, ci_name }]`

### `incidents.by.cis`
**GET** — `cis_source_id` (✓), `group` (✓, `Asset` o `BusinessApplication`), `ci_ids[]` (✓)

→ `[{ group, ci_id, requests }]`

---

## Triggers

### `triggers`
**GET** — `trigger_id` (opcional) → `[{ id, trigger_name }]`

### `triggers.executions`
**GET** — `trigger_id` (opcional) → `[{ id, trigger_id, request_id, executed_at }]` — últimas ejecuciones de la última hora

---

## Exportación y Versión

### `data.export`
**GET** — `id` (✓, UUID del reporte) → `{ status, url }`

### `sd.version`
**GET** → `{ version }` — Versión actual de la instancia InvGate

---

## Valores de referencia rápida

### `priority_id`
| ID | Nombre |
|---|---|
| 1 | Low |
| 2 | Medium |
| 3 | High |
| 4 | Urgent |
| 5 | Critical |

### `type_id`
| ID | Nombre |
|---|---|
| 1 | Incident |
| 2 | Service Request |
| 3 | Question |
| 4 | Problem |
| 5 | Change |
| 6 | Major Incident |

### `user.type`
| ID | Descripción |
|---|---|
| 1 | System |
| 2 | LDAP |
| 3 | Webservice |
| 4 | Customer |
| 5 | Mailbox |

### `user_type`
| ID | Descripción |
|---|---|
| 1 | Agente |
| 2 | Usuario final |

### `closed_reason`
| ID | Descripción |
|---|---|
| 1 | Solution accepted |
| 2 | Solution expired |
| 3 | Customer timeout |
| 4 | Finalized workflow |
