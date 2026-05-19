# RunFood API v1.0.0

API REST para integrar tu aplicación con un punto de venta RunFood.
Te permite crear y consultar pedidos, leer el catálogo de productos y métodos de pago, y recibir eventos en tiempo real vía webhooks.

---

# Empieza en 5 minutos

Antes de pedir credenciales de producción, prueba todo contra nuestro **ambiente público de demo**. No hace falta pedir permiso ni pagar nada — toma la API key de abajo y empieza.

## Ambiente de demo

**URL base:**
```
https://ec-s2.runfoodapp.com/apps/demo/api/v1
```

**API Key (compartida, solo para pruebas):**
```
184bc6459fe440e1318852fb074f1d7f3c5036a049d5245beea810e13227a503
```

Cuando autentiques con esta key y llames a `GET /apps/me`, vas a ver esta identidad:

| Campo | Valor |
|---|---|
| `id` | `fantastic_app` |
| `name` | `Fantastic App` |
| `icon_url` | `https://runfoodapp.com/fantastic_app.png` |
| `scopes` | *(todos)* |

> Es una cuenta compartida entre todos los que están probando. Los datos del demo los puede modificar cualquiera — no guardes nada crítico ahí.

## Prueba rápida con curl

Verifica que estás conectado y que tu key funciona:

```bash
# 1. Health check (no necesita auth)
curl https://ec-s2.runfoodapp.com/apps/demo/api/v1/health

# 2. Tu identidad como app (requiere X-Api-Key)
curl -H "X-Api-Key: 184bc6459fe440e1318852fb074f1d7f3c5036a049d5245beea810e13227a503" \
     https://ec-s2.runfoodapp.com/apps/demo/api/v1/apps/me
```

Si los dos responden 200, ya estás listo para explorar los demás endpoints.

---

# Autenticación

Todos los endpoints (excepto los marcados como **Public**) necesitan un solo header:

```
X-Api-Key: <tu_api_key>
```

Cada endpoint está protegido por un **scope** (permiso):
`orders:read`, `orders:write`, `products:read`, `payment-methods:read`, `apps:read`.

Tu API key tiene asignados los scopes que hayas solicitado al momento de generarla. Si llamas a un endpoint sin el scope correspondiente, recibes un `403 insufficient_scope`.

---

# Pasar a producción

El demo sirve para desarrollar, pero cuando quieras conectarte con un cliente real necesitas credenciales nuevas, específicas para ese local.

## Antes de solicitar acceso

El cliente (el restaurante o local que usa RunFood) tiene que ser alcanzable desde internet. Hay dos caminos:

1. **El cliente pone su propia infraestructura.** IP pública fija más un certificado HTTPS válido instalado en su servidor RunFood. Es lo ideal si ya tiene equipo técnico.
2. **Nosotros le damos un túnel gestionado.** Le asignamos un subdominio tipo `mi-restaurante.runfoodapp.com` y nos encargamos del HTTPS y de exponer el servicio. Útil cuando el cliente no quiere lidiar con redes.

Además, necesitas tener claros los datos de tu app antes de enviar la solicitud:

| Campo | Descripción |
|---|---|
| `id` | Identificador único (ej. `fantastic_app`). Debe ser único en todo RunFood: no se puede repetir en ningún otro cliente. Piénsalo como un handle global. |
| `name` | Nombre que se va a mostrar al usuario dentro del POS (ej. *Fantastic App*). |
| `icon_url` | URL pública de tu ícono. PNG cuadrado, mínimo 128x128. |
| `scopes` | Los permisos que tu app necesita (lista separada por comas). Pide solo los que vas a usar. |

## Cómo solicitarla

Todo se tramita por correo a **soporte@runfoodapp.com**. Lo más limpio es que el **dueño del local** sea quien envíe la solicitud desde su correo, porque así queda autorizada al instante.

Si prefieres encargarte tú como integrador, puedes hacerlo — pero entonces el dueño tiene que mandarnos por separado una autorización corta desde su correo (abajo hay una plantilla para cada caso).

El correo de solicitud debe incluir:

- **Del cliente:** RUC, razón social, ciudad y sector
- **Del servidor del cliente:** IP pública o URL del túnel, más el puerto
- **De tu app:** los 4 campos de la tabla anterior (`id`, `name`, `icon_url`, `scopes`)

### Plantilla — solicitud enviada por el dueño del local

Copia este texto, reemplaza los `[corchetes]` y envíalo desde el correo del dueño:

```text
Asunto: Solicitud de credenciales API — [RUC] — [Nombre de la app]

Hola equipo de RunFood,

Autorizo la creación de credenciales de API de terceros para mi
establecimiento con los siguientes datos:

DATOS DEL ESTABLECIMIENTO
- Razón social: [Razón social]
- RUC: [RUC]
- Ciudad / sector: [Ciudad, sector]
- IP pública o URL del túnel: [ej. 181.xxx.xxx.xxx:3200]

APLICACIÓN A CONECTAR
- Nombre visible: [Nombre de la app]
- ID (identificador único): [identificador_app]
- URL del ícono: https://...
- Permisos solicitados: [ej. orders:read, orders:write, products:read]

Confirmo haber leído los costos de activación vigentes y estoy de acuerdo.

Saludos,
[Nombre del dueño]
[Correo]
[Teléfono]
```

### Plantilla — autorización corta del dueño (cuando el integrador hace la solicitud)

Si la solicitud técnica la envía el integrador, necesitamos que el **dueño** nos mande por separado este correo desde su cuenta. Sin él no podemos avanzar.

```text
Asunto: Autorización de integración API — [RUC]

Hola equipo de RunFood,

Autorizo a [Nombre del integrador], correo [correo@integrador.com],
a solicitar y gestionar credenciales de API de terceros en mi cuenta
de RunFood.

RUC: [RUC]
Razón social: [Razón social]
Ciudad / sector: [Ciudad, sector]

Saludos,
[Nombre del dueño]
```

## Costos

| Concepto | Costo | Modalidad |
|---|---|---|
| **Generación de API Key** | **USD 50** | Pago único, de por vida, por cada credencial que se genere en un cliente |
| **Túnel gestionado — activación** | **USD 30** | Pago único al activar el subdominio |
| **Túnel gestionado — mantenimiento** | **USD 10 / mes** | Recurrente, a partir del segundo mes |

> El túnel es **opcional**. Si el cliente ya tiene su propia IP pública con HTTPS, no lo necesitas y no pagas nada por ese concepto.

---

# Consideraciones importantes antes de integrar

**RunFood es on-premise.** Esto es clave y tiene que entenderlo bien antes de escribir código: el servidor no vive en la nube, vive físicamente dentro del local del cliente. Está así a propósito — así la caja, el monitor de cocina y las impresoras funcionan a velocidad de red local aunque se vaya el internet.

La consecuencia para ti como integrador es que **el servidor del cliente no siempre va a estar disponible**, y eso no es un error: es parte del día a día. Se va la luz, reinician el router, cambia la IP, el ISP del local tiene problemas. Tu integración tiene que asumir esto como un estado normal, no como una excepción.

## Cómo preparar tu integración

Estas son las estrategias que recomendamos a los equipos que ya llevan tiempo integrados con RunFood. No son opcionales si quieres que tu producto sea estable en producción.

### 1. Cola de reintentos con backoff exponencial

Cuando una request falla por timeout o devuelve 5xx, no la pierdas: guárdala en una cola local en tu lado y reinténtala con delays crecientes (por ejemplo 1s, 5s, 30s, 5min, 30min). Después de 24 horas sin éxito, escala el incidente y avisa al equipo que corresponde.

### 2. Idempotencia con `external_id`

`POST /orders` acepta un campo `external_id` que debe ser único en tu sistema. Si reenvías un pedido con el mismo `external_id`, la API te devuelve `409` con el `id` del pedido que ya existía, en lugar de crear uno duplicado.

**Úsalo siempre.** Es tu seguro contra envíos duplicados cuando la red te juegue sucio — sin él, un reintento puede cobrarle dos veces al cliente final.

### 3. Webhooks en vez de polling

En lugar de preguntarle cada 10 segundos al servidor "¿pasó algo nuevo?", suscríbete a los webhooks que te interesan (`ORDER.CREATED`, `ORDER.CLOSED`, `INVOICE.CREATED`, etc.). RunFood te empuja el cambio cuando ocurre.

Esto reduce la carga en el servidor del cliente y tu latencia de reacción. El polling constante, en un ambiente on-premise con ancho de banda limitado, es una mala idea.

### 4. Valida la firma HMAC en cada webhook que recibas

Cada webhook viene con un header `X-Signature-256`. Antes de procesar el payload, calcula `HMAC_SHA256(secret, raw_body)` y compáralo con lo que viene en ese header. Si no coinciden, descarta el mensaje.

Esto protege tu endpoint aunque sea público: sin el secret, nadie puede falsificar un payload que tu servicio acepte.

### 5. Health check periódico

Llama a `GET /health` cada 1 a 5 minutos desde tu backend y guarda el resultado. Así armas un histórico de disponibilidad del cliente y detectas caídas antes que tus propios usuarios se quejen.

### 6. Reconcilia cuando el cliente vuelva a estar en línea

Después de una caída, no asumas que todo quedó sincronizado. Corre una pasada de reconciliación:

- Los pedidos que tú tienes con `external_id` y que no aparecen en RunFood: reenvíalos.
- Los pedidos que aparecen en RunFood y que tú no tienes indexados (porque perdiste webhooks durante la caída): bájalos y guárdalos.

Esta rutina es la que hace la diferencia entre una integración que "más o menos funciona" y una que es confiable.

---

# Webhooks salientes

Además de las peticiones que tú haces a RunFood, RunFood también puede hacer peticiones a **tu** servidor. Cuando pasa algo importante (se crea un pedido, se cierra una factura, se anula una venta), RunFood envía un `POST` HTTP al endpoint que hayas registrado.

- **Webhooks** — cómo registrar, listar y eliminar tus suscripciones
- **Webhook Events** — qué formato tiene el payload que vas a recibir

## Base URL
- `https://ec-s2.runfoodapp.com/apps/public.demo/api/v1` — Demo público (Ecuador)

## Authentication
- **apiKeyAuth**: apiKey in header (`X-Api-Key`)
  Autenticación por API key en el header `X-Api-Key`. Una sola clave — no hay header adicional.

## Endpoints

### GET /health
**Health check**
Retorna el estado del servidor, versión, uptime y metadata del runtime.
*No authentication required.*

**Responses:**
- `200` — Server is healthy
  - `status`: string
  - `version`: string (e.g. `17.7.2`)
  - `commitHash`: string (e.g. `abc1234`)
  - `buildDate`: string (e.g. `2026-03-01T00:00:00.000Z`)
  - `startedAt`: string
  - `uptime`: integer (e.g. `86400`) — Tiempo activo del servidor, en segundos
  - `uptimeHuman`: string (e.g. `1d 0h 0m`)
  - `runtime`: object
  - `platform`: string (e.g. `win32-x64`)
  - `hostname`: string
  - `checkedAt`: string
  - `timeZone`: string (e.g. `America/Guayaquil`)
  - `UTC`: string (e.g. `UTC-5`)

---

### GET /identity
**Identidad del servidor**
Retorna la información del establecimiento para el flujo de conexión del RunFood Shell.
Incluye RUC, razón social, capabilities y un fingerprint estable.
*No authentication required.*

**Responses:**
- `200` — Identidad del servidor
  - `ruc`: string (e.g. `0992877878001`)
  - `razonSocial`: string (e.g. `RESTAURANTE DEMO S.A.`)
  - `nombreComercial`: string (e.g. `La Esquina del Sabor`)
  - `establecimiento`: string (e.g. `001`)
  - `bodega`: integer (e.g. `1`)
  - `ciudad`: string (e.g. `Guayaquil`)
  - `version`: string (e.g. `17.7.2`)
  - `channel`: string (e.g. `stable`)
  - `fingerprint`: string (e.g. `fp-a1b2c3d4`) — Hash estable derivado de RUC + razón social + bodega. Sirve para identificar al cliente de forma consistente entre reinicios.
  - `capabilities`: array (e.g. `electronic-invoicing,tables,kitchen-monitor`)
  - `requiresApproval`: boolean (e.g. `false`)

---

### POST /orders
**Crear pedido**
Crea un pedido con una o más cuentas (tabs), cada una con sus items.
Es idempotente por `external_id`: si ya existe un pedido con ese mismo `external_id`
para tu app, te devuelve `409` con el `id` del pedido existente en lugar de crear uno nuevo.

**Request body:** `application/json`
- `external_id`: string (e.g. `order-12345`) — ID único del pedido dentro de tu sistema. Se usa para idempotencia — si reenvías el mismo pedido, te devolvemos 409 en vez de duplicarlo.
- `reference`: string (e.g. `REF-001`) — Referencia visible para el usuario (ej. el número de pedido que vio el cliente)
- `tabs`: array
- `service_type`: string (e.g. `delivery`) — Tipo de servicio (delivery, pickup, dine-in, etc.)
- `table_number`: string (e.g. `12`)
- `customer`: object
- `delivery_address`: object

**Responses:**
- `201` — Pedido creado
  - `id`: integer (e.g. `1042`)
  - `external_id`: string
  - `external_app`: string
  - `reference`: string
  - `status`: string (e.g. `open`)
  - `order_number`: integer (e.g. `157`)
  - `service_type`: string
  - `table_number`: string
  - `total`: number (e.g. `13.77`)
  - `currency`: string (e.g. `USD`)
  - `tabs`: array
  - `created_at`: string
  - `notes`: string
  - `guest_count`: integer (e.g. `0`)
- `400` — Error de validación (campos faltantes o producto no encontrado)
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)
- `401` — Autenticación faltante o inválida
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)
- `409` — Pedido duplicado (mismo external_id)


---

### GET /orders/{id}
**Detalle de un pedido**
Retorna un pedido completo con sus tabs e items.
Solo devuelve pedidos que pertenezcan a tu app — no puedes leer los de otras apps, aunque conozcas el ID.

**Parameters:**
- `id` (required) in path — integer

**Responses:**
- `200` — Pedido encontrado
  - `id`: integer (e.g. `1042`)
  - `external_id`: string
  - `external_app`: string
  - `reference`: string
  - `status`: string (e.g. `open`)
  - `order_number`: integer (e.g. `157`)
  - `service_type`: string
  - `table_number`: string
  - `total`: number (e.g. `13.77`)
  - `currency`: string (e.g. `USD`)
  - `tabs`: array
  - `created_at`: string
  - `notes`: string
  - `guest_count`: integer (e.g. `0`)
- `401` — Autenticación faltante o inválida
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)
- `404` — Pedido no encontrado o no pertenece a esta app
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### DELETE /orders/{id}
**Eliminar pedido**
Elimina un pedido que todavía esté abierto (estado `open`).
Los pedidos cerrados o cancelados ya no se pueden eliminar — para esos casos tendrías que hacer una anulación desde el POS.

**Parameters:**
- `id` (required) in path — integer

**Request body:** `application/json`
- `reason`: string (e.g. `El cliente canceló el pedido`) — Motivo de eliminación (opcional, queda registrado en el historial)

**Responses:**
- `200` — Pedido eliminado
  - `id`: integer
  - `deleted`: boolean (e.g. `true`)
- `400` — El pedido no está en estado open (ya fue cerrado o anulado)
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)
- `404` — Pedido no encontrado
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### PATCH /orders/{id}/items
**Modificar items de un pedido**
Permite hacer múltiples operaciones (`add`, `remove`, `update`) sobre los items de un pedido abierto, en una sola llamada.

Cada operación se procesa de forma independiente: si una falla (producto no encontrado, item inexistente, etc.), las demás continúan y te devolvemos el resultado de cada una por separado.

**Parameters:**
- `id` (required) in path — integer

**Request body:** `application/json`
- `operations`: array

**Responses:**
- `200` — Resultado detallado de cada operación
  - `applied`: array
  - `order_total`: number (e.g. `25.5`)
- `400` — El pedido no está en estado open, o hay un error de validación
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)
- `404` — Pedido no encontrado
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### GET /products
**Listar productos**
Retorna todos los productos activos del catálogo. Puedes filtrar por nombre o código usando el parámetro `search`.

**Parameters:**
- `search` (optional) in query — Filtro por nombre o código (no distingue mayúsculas/minúsculas, busca coincidencias parciales)

**Responses:**
- `200` — Lista de productos
  - `data`: array
  - `count`: integer (e.g. `42`)
- `401` — Autenticación faltante o inválida
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### GET /products/{sku}
**Detalle de un producto**
Retorna un producto a partir de su SKU (código).

**Parameters:**
- `sku` (required) in path — Código único del producto

**Responses:**
- `200` — Producto encontrado
  - `sku`: string (e.g. `PROD-001`)
  - `name`: string (e.g. `Hamburguesa Clásica`)
  - `price`: number (e.g. `5.99`)
  - `tax_applicable`: boolean (e.g. `true`)
  - `active`: boolean (e.g. `true`)
- `401` — Autenticación faltante o inválida
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)
- `404` — Producto no encontrado
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### GET /payment-methods
**Listar métodos de pago**
Retorna los métodos de pago configurados en el establecimiento (efectivo, tarjetas, transferencias, etc.).

**Responses:**
- `200` — Lista de métodos de pago
  - `data`: array
- `401` — Autenticación faltante o inválida
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### GET /apps/me
**Perfil de tu app**
Retorna el perfil de la app que está autenticando en este momento (id, nombre, ícono, scopes, fecha de creación).

**Responses:**
- `200` — Perfil de la app
  - `id`: string
  - `name`: string
  - `icon_url`: string
  - `scopes`: string (e.g. `orders:read,orders:write,products:read`) — Lista de scopes separados por coma. Si viene `null`, la app tiene acceso a todos los scopes (compatibilidad con apps antiguas).
  - `created_at`: string
- `401` — Autenticación faltante o inválida
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### POST /webhooks
**Registrar webhook**
Crea una suscripción para que RunFood te envíe un HTTP POST cuando ocurra un evento.

**Eventos disponibles:**
| Evento | Cuándo se dispara |
|--------|-------------------|
| `ORDER.CREATED` | Se creó un pedido nuevo |
| `ORDER.UPDATED` | Se modificó un pedido existente (items, estado) |
| `ORDER.SAVED` | Atajo: se disparó un CREATED o un UPDATED |
| `ORDER.CLOSED` | Se cerró/facturó un pedido |
| `ORDER.REVERSED` | Se anuló un pedido completo |
| `ORDER.PRODUCT.REVERSED` | Se anuló un item individual dentro de un pedido |
| `INVOICE.CREATED` | Se generó una factura o venta |

**Headers que vas a recibir en cada POST:**
- `Content-Type: application/json`
- `User-Agent: runfoodcore/{version}`
- `X-Event-Type: {nombre_del_evento}`
- `X-Request-ID: {id_único_del_request}`
- `X-Signature-256: sha256={hmac}` — solo si configuraste un secret al registrar el webhook

**Request body:** `application/json`
- `event`: string (e.g. `ORDER.CREATED`) — Nombre del evento al que quieres suscribirte
- `endpoint`: string (e.g. `https://miapp.com/webhooks/runfood`) — URL (idealmente HTTPS) donde RunFood va a enviar los webhooks

**Responses:**
- `201` — Webhook creado
  - `id`: integer (e.g. `15`)
  - `event`: string (e.g. `ORDER.CREATED`)
  - `endpoint`: string
- `400` — Faltan campos requeridos o el endpoint no es una URL válida
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### GET /webhooks
**Listar webhooks**
Devuelve todas las suscripciones que tu app tiene registradas en este cliente.

**Responses:**
- `200` — Lista de webhooks
  - `data`: array

---

### DELETE /webhooks/{id}
**Eliminar webhook**
Elimina una suscripción de webhook. Después de esto, RunFood deja de enviarte POSTs para ese evento.

**Parameters:**
- `id` (required) in path — integer

**Responses:**
- `200` — Webhook eliminado
  - `deleted`: boolean (e.g. `true`)
  - `deleted_at`: string
- `404` — Webhook no encontrado
  - `error`: string (e.g. `not_found`)
  - `message`: string (e.g. `Product 'XXXX' not found`)

---

### POST /webhook-events/order
**Webhook: ORDER.CREATED / ORDER.UPDATED / ORDER.CLOSED**
**Esto NO es un endpoint que tú llames.** Es la documentación del payload que RunFood te envía cuando ocurre un evento relacionado con pedidos.

RunFood hace un `POST` HTTP a la URL que registraste, con el body que ves aquí abajo.

### Verificar autenticidad (recomendado)

Si configuraste un `secret` al registrar el webhook, cada request va a incluir un header `X-Signature-256`. Antes de procesar el payload, valídalo:

```
firma_esperada = "sha256=" + HMAC_SHA256(secret, raw_body)
if (firma_esperada != request.headers["X-Signature-256"]) descartar()
```

Si las firmas no coinciden, el request no viene de RunFood — descártalo sin procesarlo.
*No authentication required.*

**Request body:** `application/json`
- `event`: string (e.g. `ORDER.CREATED`) — Nombre del evento
- `timestamp`: string
- `schema_version`: string (e.g. `2025-02-12`)
- `data`: object — Contenido del evento. La estructura varía según el tipo de evento (`ORDER.CREATED`, `INVOICE.CREATED`, etc.).
- `metadata`: object

**Responses:**
- `200` — Tu servidor debe responder con un código 2xx para confirmar que recibiste el evento. Si respondes con error o no respondes, RunFood lo registra como fallo.

---

