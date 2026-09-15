# BENNZ · Guía completa de logística

Actualizado 15 Sep 2026 · Para Tomaturri

Esta guía explica **paso por paso** qué tenés que hacer para integrar envíos reales con Correo Argentino, Andreani y OCA. Incluye qué datos vas a pedir al dueño de BENNZ, cómo funcionan las APIs, y cómo probar todo antes de ir a producción.

---

## Índice

1. Los 3 modos de entrega
2. Antes de contratar — decisiones y checklist
3. Onboarding paso a paso — Correo Argentino
4. Onboarding paso a paso — Andreani
5. Onboarding paso a paso — OCA (opcional)
6. Cómo funcionan las APIs — ejemplos reales
7. Flujo end-to-end en el storefront
8. Sucursales — cómo listarlas y seleccionarlas
9. Emisión de etiquetas
10. Seguimiento (tracking) — cron + UI
11. Modo prueba / sandbox
12. Costos operativos reales
13. n8n workflows de logística
14. Checklist final antes de ir a producción

---

## 1 · Los 3 modos de entrega que ofrece el storefront

### Modo A · Retiro en local · Te Al 436, Tolhuin
- Costo cliente: **$0**
- Sin cotizador
- No pasa por ningún carrier
- Estado de la orden pasa por: `unfulfilled → preparing → ready → delivered`
- Cuando está **ready**, workflow n8n dispara WhatsApp: *"Tu pedido BNZ-2026-000042 está listo. Podés pasarlo a retirar de Lun-Sáb 10-20."*
- Pago puede ser online (todos los métodos) o **efectivo al retirar**
- Cliente lleva su DNI + número de orden

### Modo B · Envío a domicilio
- Cliente ingresa su CP
- Sistema cotiza contra la tarifa cargada (o contra la API del carrier)
- Muestra 1-3 opciones (Correo, Andreani) con costo + ETA
- Cliente elige, se persiste en `checkout_session`
- Al pagar → workflow genera etiqueta → carrier retira o BENNZ despacha en centro de imposición
- Cliente recibe mail + WhatsApp con **número de tracking + link**

### Modo C · Envío a sucursal del carrier
- Cliente elige "Retiro en sucursal de Andreani/Correo cercana"
- Sistema **consulta la API del carrier** con el CP y trae sucursales cercanas
- Cliente elige una sucursal
- Se despacha etiqueta con dirección de esa sucursal como destino
- Cliente recibe mail: *"Tu pedido llegó a la sucursal Andreani Palermo. Tenés 10 días para retirarlo con tu DNI."*
- **Ventaja** para el cliente: 30-40 % más barato que envío a domicilio, retiro cuando le queda cómodo
- **Ventaja** para BENNZ: menos entregas fallidas, menos redespachos

---

## 2 · Antes de contratar — decisiones y checklist

### 2.1 Datos que necesitás recolectar del dueño de BENNZ

Todo esto va a nombre de la persona/empresa titular:

- **CUIT** (obligatorio para todos los carriers)
- **Razón social exacta** como figura en AFIP
- **Condición ante IVA** (Monotributo, Responsable Inscripto, Exento)
- **Categoría de Monotributo** si aplica
- **Domicilio fiscal**
- **Actividad económica principal** (código AFIP — para venta de ropa online suele ser 477320)
- **CBU/CVU** para acreditación de reembolsos si los hay
- **Email operativo** (idealmente `envios@bennz.ar` o `admin@bennz.ar`, no personal)
- **Teléfono comercial**
- **Datos del centro de imposición** — desde qué CP van a despacharse los paquetes (Tolhuin CP 9412)
- **Foto DNI del titular** frente y dorso
- **Volumen mensual estimado** (crítico para negociar tarifas — sub-declararlo genera problemas después)

### 2.2 Decisión estratégica — ¿uno solo, dos, o los tres carriers?

**Recomendación por volumen esperado:**

| Volumen mensual | Estrategia |
|---|---|
| < 30 envíos | Sólo **Correo Argentino** — más simple, cobertura nacional |
| 30-150 envíos | **Correo Argentino + Andreani** — CA para tarifa base, Andreani para grandes ciudades (más rápido, tracking mejor) |
| > 150 envíos | Los 3 con reglas de ruteo automático por zona |

**Argumento comercial:** ofrecer a la cliente elegir entre 2-3 opciones aumenta la conversión ~8 % (dato de estudios de UX ecommerce AR).

### 2.3 Timeline realista de onboarding

- Correo Argentino → 7-10 días hábiles desde que mandás formulario
- Andreani → 10-15 días (comercial + contrato marco)
- OCA → 10-14 días
- Sandbox de todos → inmediato una vez firmado el contrato

**Empezá esto lo antes posible** — mientras el resto del ecommerce se desarrolla.

---

## 3 · Onboarding paso a paso — Correo Argentino

### 3.1 Servicio a contratar
**"Servicio Empresa" con acceso API** (no el servicio individual, ese no da API).

### 3.2 Pasos

1. **Entrar a** `https://www.correoargentino.com.ar/formularios/oca`
2. Elegir **"Soy Empresa" → "Envíos ecommerce"**
3. Rellenar formulario con:
   - Datos fiscales
   - URL del sitio (`bennz.ar`)
   - Descripción del negocio ("Venta de ropa por internet")
   - Volumen mensual estimado
   - Peso promedio del paquete (400-600 g)
4. Subir DNI + constancia CUIT
5. Firmar contrato electrónico
6. **Espera** ~7-10 días hábiles

### 3.3 Qué te va a mandar Correo Argentino

- **Cliente Nº** (`CUIT_XXXXXXXXX`)
- **Centro de imposición** asignado (para Tolhuin es CI 9412)
- **Credenciales API** (usuario + password) — vía mail al operativo
- **URL de API productiva y sandbox**
- **Manual de tarifas actualizado** (Excel con tarifas por zona × peso)

### 3.4 Servicios disponibles vía API

- **Clásico** — 3-8 días hábiles, más barato
- **Expreso** — 24-48 hs, ~30 % más caro, sólo cabeceras
- **Sucursales** — retiro en sucursal, ~30 % más barato que domicilio

---

## 4 · Onboarding paso a paso — Andreani

### 4.1 Servicio a contratar
**"Andreani ecommerce"** con contrato marco y acceso a API v2.

### 4.2 Pasos

1. **Entrar a** `https://www.andreani.com/ecommerce/`
2. Click **"Quiero contratar"** → completar formulario
3. **Los llama un comercial** en 24-48 hs
4. **Reunión de onboarding** (Zoom, ~30 min) para explicar tu operación
5. Ellos preparan **propuesta con tarifas** basadas en tu volumen
6. Firmás **contrato marco** (electrónico)
7. Recibís credenciales **sandbox** para probar
8. Cuando aprobás las pruebas, pasás a **producción**

### 4.3 Qué te va a mandar Andreani

- **Número de contrato** (`CTR_XXXXXX`)
- **Cliente** (`CL_XXXXXX`)
- **Sucursal origen** (código Andreani de tu punto de despacho — para Tolhuin puede ser una sucursal en Río Grande o USH según donde tengas más volumen)
- **API Key + Secret** para OAuth2
- **URL de sandbox y producción**
- **Manual API v2** (PDF con endpoints y esquemas)

### 4.4 Servicios Andreani disponibles

- **Estándar** — 3-6 días hábiles, mejor cobertura
- **Urgente** — 2-3 días hábiles, sólo grandes ciudades
- **Sucursal** — retiro en las ~800 sucursales del país
- **Andreani Solutions** — logística integrada (fulfillment) — para volúmenes altos

---

## 5 · Onboarding OCA (opcional)

Similar proceso pero API menos documentada. Recomendable **saltear al inicio** y sumarla más adelante si Andreani no cubre alguna zona.

Contacto comercial: `https://www.oca.com.ar/contacto/ecommerce`

---

## 6 · Cómo funcionan las APIs — ejemplos reales

### 6.1 Correo Argentino — cotización

```bash
curl -X POST https://apis.correoargentino.com.ar/rate-quote \
  -H "Authorization: Bearer $CORREO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "postalCodeOrigin": "9412",
    "postalCodeDestination": "1414",
    "weight": 600,
    "declaredValue": 89000,
    "service": "CLASICO"
  }'
```

Respuesta:
```json
{
  "cost": 4850,
  "deliveryTime": { "min": 3, "max": 5 },
  "service": "CLASICO",
  "code": "SC0000123"
}
```

### 6.2 Correo Argentino — emisión de etiqueta

```bash
curl -X POST https://apis.correoargentino.com.ar/generate-label \
  -H "Authorization: Bearer $CORREO_TOKEN" \
  -d '{
    "sender": {
      "name": "BENNZ",
      "postalCode": "9412",
      "address": "Te Al 436",
      "phone": "..."
    },
    "recipient": {
      "name": "María Pérez",
      "dni": "30111222",
      "postalCode": "1414",
      "address": "Av. Corrientes 1234, CABA",
      "phone": "1155669988"
    },
    "package": {
      "weight": 600,
      "declaredValue": 89000
    },
    "service": "CLASICO",
    "externalReference": "BNZ-2026-000042"
  }'
```

Respuesta:
```json
{
  "trackingNumber": "CB123456789AR",
  "labelUrl": "https://apis.correoargentino.com.ar/labels/xxx.pdf",
  "estimatedDelivery": "2026-09-20"
}
```

### 6.3 Andreani — cotización

Andreani usa OAuth2. Primero token:

```bash
curl -X POST https://apis.andreani.com/login \
  -H "Content-Type: application/json" \
  -d '{"username":"tu_usuario","password":"tu_password"}'
```

Devuelve `{"token": "eyJ..."}`. Ese token dura 8 horas.

Cotización:
```bash
curl -X POST https://apis.andreani.com/v2/tarifas/nacional \
  -H "x-authorization-token: eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "codigoContrato": "CTR_100200",
    "cliente": "CL_100200",
    "sucursalOrigen": "TOLH",
    "cpDestino": "1414",
    "peso": 0.6,
    "valorDeclarado": 89000
  }'
```

Respuesta:
```json
{
  "servicios": [
    {
      "servicio": "ESTANDAR",
      "tarifa": 5850,
      "diasHabiles": { "min": 3, "max": 5 }
    },
    {
      "servicio": "URGENTE",
      "tarifa": 9200,
      "diasHabiles": { "min": 2, "max": 3 }
    }
  ]
}
```

### 6.4 Andreani — sucursales cercanas al CP

```bash
curl -X GET "https://apis.andreani.com/v2/sucursales?codigoPostal=1414" \
  -H "x-authorization-token: eyJ..."
```

Respuesta:
```json
{
  "sucursales": [
    {
      "id": "TERM_PALERMO",
      "nombre": "Andreani Palermo",
      "direccion": "Av. Santa Fe 4300, CABA",
      "horarios": "Lun-Vie 9-18, Sáb 9-13",
      "distanciaKm": 1.2
    },
    { ... }
  ]
}
```

### 6.5 Andreani — emisión de orden de envío

```bash
curl -X POST https://apis.andreani.com/v2/ordenes-de-envio \
  -H "x-authorization-token: eyJ..." \
  -d '{
    "contrato": "CTR_100200",
    "origen": { "postal": "9412", "calle": "Te Al", "numero": "436", "localidad": "Tolhuin" },
    "destinatario": {
      "nombreCompleto": "María Pérez",
      "dni": "30111222",
      "email": "maria@example.com",
      "celular": "1155669988",
      "postal": "1414",
      "calle": "Av. Corrientes",
      "numero": "1234",
      "localidad": "CABA"
    },
    "peso": 0.6,
    "valorDeclarado": 89000,
    "servicio": "ESTANDAR",
    "referencia": "BNZ-2026-000042"
  }'
```

Respuesta:
```json
{
  "numeroAndreani": "500123456789",
  "linking": "https://andreani.com/tracking/500123456789",
  "constancia": "https://apis.andreani.com/constancias/xxx.pdf"
}
```

### 6.6 Andreani — consulta de estado (tracking)

```bash
curl -X GET https://apis.andreani.com/v2/tracking/500123456789 \
  -H "x-authorization-token: eyJ..."
```

Respuesta:
```json
{
  "numeroAndreani": "500123456789",
  "estadoActual": "EN_TRANSITO",
  "eventos": [
    { "fecha": "2026-09-18T14:23:00Z", "descripcion": "Ingreso al centro de distribución" },
    { "fecha": "2026-09-19T09:45:00Z", "descripcion": "En camino a sucursal destino" }
  ],
  "fechaEstimadaEntrega": "2026-09-21"
}
```

---

## 7 · Flujo end-to-end en el storefront

### 7.1 Cliente elige envío

En el cart drawer (ya está funcionando en la maqueta):
- Radio 1: **Retiro en local** — gratis, sin cotizador
- Radio 2: **Envío a domicilio** — ingresa CP → cotiza → elige carrier
- Radio 3: **Retiro en sucursal** (NUEVO) — ingresa CP → lista sucursales → elige

### 7.2 Cliente paga

Al confirmar carrito → Edge Function `checkout-create` de Supabase:
1. Valida stock
2. Recalcula subtotal + envío server-side (nunca confía en el cliente)
3. Persiste `checkout_session` en DB
4. Crea preferencia en Mercado Pago con `external_reference = checkout_id`
5. Redirige al cliente al Checkout Pro

### 7.3 Mercado Pago confirma pago

Webhook a `POST /api/webhook/mp`:
1. Valida firma HMAC
2. Consulta el pago a la API MP (nunca confía en el body)
3. Si `status = approved` → RPC `create_order_from_checkout()`

### 7.4 Se dispara emisión de etiqueta

`pg_notify('order.paid', order_id)` → n8n workflow #05:
1. Lee la orden
2. Determina carrier según `orders.shipping_carrier`
3. Llama al endpoint apropiado (Correo Argentino o Andreani)
4. Persiste `tracking_number`, `label_url` en `shipments`
5. Notifica al cliente por mail (Resend) + WhatsApp (Twilio) con el número + link

### 7.5 Tracking sincronizado

Cron n8n #06 cada 4 h:
1. Selecciona shipments con estado activo (`in_transit`, `shipped`, `ready`)
2. Consulta API del carrier
3. Actualiza `shipment_events`
4. Si estado pasa a `delivered` → dispara mail *"¿Cómo te llegó?"* + solicita review

---

## 8 · Sucursales — cómo listarlas y seleccionarlas

### 8.1 Sincronización periódica

Recomendación: **no consultar la API de sucursales cada vez** que un cliente ingresa un CP. En su lugar:

1. Cron n8n #08 (semanal) baja todas las sucursales de Andreani y Correo
2. Persiste en tabla `carrier_sucursales`:

```sql
create table carrier_sucursales (
  id uuid primary key default uuid_generate_v4(),
  carrier text not null,
  external_id text not null,
  name text not null,
  address text not null,
  city text not null,
  province text not null,
  postal_code text not null,
  geo jsonb,
  hours jsonb,
  active boolean default true,
  synced_at timestamptz default now(),
  unique (carrier, external_id)
);
create index on carrier_sucursales (postal_code, carrier) where active;
```

3. Cuando el cliente ingresa CP → query local (~10 ms) en vez de API remota (~500 ms)

### 8.2 Filtrado por distancia

Si querés mostrar sucursales por proximidad, agregar función Postgres:

```sql
create or replace function nearest_sucursales(
  p_cp text,
  p_carriers text[] default array['andreani','correo-argentino']
) returns setof carrier_sucursales
language sql stable as $$
  select cs.* from carrier_sucursales cs
  where cs.active and cs.carrier = any(p_carriers)
    and (cs.postal_code = p_cp or left(cs.postal_code, 4) = left(p_cp, 4))
  order by
    case when cs.postal_code = p_cp then 0 else 1 end,
    cs.name
  limit 12;
$$;
```

### 8.3 UI en el checkout

- Cliente elige "Retiro en sucursal"
- Ingresa CP
- Aparece scroll horizontal con tarjetas de sucursales
- Cada tarjeta muestra: nombre, dirección, horarios, distancia, costo
- Al seleccionar → guarda `shipment_sucursal_id` en `checkout_session`

---

## 9 · Emisión de etiquetas

### 9.1 Cuándo se emite

**Al momento del pago aprobado**, no antes. Motivo: si el cliente cancela, no queremos etiquetas huérfanas que después haya que anular manualmente.

### 9.2 Dónde se guarda

- `shipments.tracking_number` — el número del carrier
- `shipments.label_url` — URL del PDF en bucket privado Supabase (bajarlo del carrier y persistir)
- `shipments.carrier` — 'andreani' | 'correo-argentino' | 'oca' | 'retiro-local'

### 9.3 Cómo se imprime

Panel admin → Órdenes → click en orden → "Ver etiqueta" → abre PDF. Se imprime en A4 y se pega en el paquete.

### 9.4 Casos borde

- **Cancelación después de emitida**: hay que llamar al endpoint `DELETE /ordenes-de-envio/{id}` del carrier antes del despacho físico. Si ya se despachó → gestión manual con soporte del carrier.
- **Reintento por dirección incorrecta**: emitir nueva etiqueta anulando la anterior.
- **Múltiples paquetes en una orden**: soportado — un order puede tener N shipments.

---

## 10 · Seguimiento (tracking) — cron + UI

### 10.1 Cron de sincronización

**n8n workflow #06**, corre cada 4 horas:

```sql
select id, carrier, tracking_number
from shipments
where status not in ('delivered', 'returned', 'failed')
  and tracking_number is not null;
```

Por cada uno → llama a API del carrier → hace UPSERT en `shipment_events`.

### 10.2 Página pública de seguimiento

URL: `https://bennz.ar/seguimiento/BNZ-2026-000042`

- No requiere login
- Muestra timeline con eventos de `shipment_events`
- Auto-refresca cada 2 minutos si la orden está en tránsito
- Datos personales enmascarados excepto para el dueño logueado
- Link se manda por mail al despachar

### 10.3 Página en la maqueta actual

Ya la agregué en esta iteración — click en el link "Seguimiento" del footer, se abre modal con input de número de orden y timeline de ejemplo.

---

## 11 · Modo prueba / sandbox

### 11.1 Antes de tener cuentas reales

**Todo el flujo se prueba con datos mock** — como ya funciona en la maqueta actual. El calculador usa las tarifas hardcodeadas por zona, muestra opciones, ETA, y el "checkout" es simulado.

### 11.2 Sandbox de Andreani

Cuando firmes contrato, Andreani te da credenciales sandbox con las mismas APIs pero **no cobra ni envía**. Podés emitir 100 etiquetas de prueba, ver tracking eventos, sin costo.

### 11.3 Sandbox Correo Argentino

Más limitado — sólo cotización funciona en sandbox. Emisión de etiqueta hay que hacerlo directo en producción con orden de prueba de valor bajo ($100).

### 11.4 Test del flujo completo antes de ir productivo

1. Setear en `.env` `SHIPPING_MODE=sandbox`
2. En admin, marcar 1 producto de prueba como "internal_test"
3. Comprar con tarjeta de test de MP ($100)
4. Verificar que se emita etiqueta sandbox
5. Verificar mail + WhatsApp mock (Resend en modo dev + Twilio sandbox)
6. Simular estados de tracking manualmente vía admin
7. Marcar entregado → verificar workflow de review

---

## 12 · Costos operativos reales

### 12.1 Correo Argentino — Clásico

| Peso hasta | Tarifa base | +/kg extra |
|---|---|---|
| 500 g | $2.900 | — |
| 1 kg | $3.800 | +$400 |
| 3 kg | $5.400 | +$500 |
| 5 kg | $7.100 | +$600 |

Multiplicadores por zona: TDF local ×1.0 · Patagonia sur ×1.4 · Centro ×1.3 · NEA ×1.5

### 12.2 Andreani — Estándar

| Peso hasta | Tarifa base | +/kg |
|---|---|---|
| 500 g | $3.500 | — |
| 1 kg | $4.400 | +$500 |
| 3 kg | $6.800 | +$700 |
| 5 kg | $9.200 | +$800 |

Multiplicadores similar a Correo pero ~15 % más barato en grandes ciudades.

### 12.3 Retiro en sucursal — descuento aproximado

- Correo Argentino sucursal: **30-35 % menos** que domicilio
- Andreani sucursal: **35-40 % menos** que domicilio

**Los tarifarios cambian trimestralmente.** Todos estos números son orientativos — con contrato firmado tenés acceso al tarifario real que actualizás en tabla `shipping_rates` de Supabase.

---

## 13 · n8n workflows de logística

Todos ya diseñados en el blueprint anterior. Los específicos de logística son:

| # | Nombre | Trigger | Acción |
|---|---|---|---|
| 05 | Orden pagada → emitir etiqueta | Webhook Supabase `order.paid` | API carrier · guarda tracking · notifica cliente |
| 06 | Sync tracking cada 4h | Cron */4 * * * * | Consulta carrier · actualiza `shipment_events` |
| 07 | Solicitud review +3 días | Evento `shipment.delivered` +3d | Mail al cliente pidiendo review |
| 08 | Sync sucursales | Cron semanal | Baja lista completa · UPSERT `carrier_sucursales` |
| 09 | Alerta despacho pendiente 48h | Cron diario | Alerta a operación si hay shipments `ready` sin despachar |
| 10 | Devolución solicitada | Trigger insert `return_requests` | Notifica soporte · genera etiqueta de retorno prepaga |

---

## 14 · Checklist final antes de ir a producción

Marcá cada uno cuando lo cumplas:

- [ ] Contrato firmado con Correo Argentino
- [ ] Contrato firmado con Andreani (si aplica)
- [ ] Credenciales productivas cargadas en Vercel env vars (`CORREO_TOKEN`, `ANDREANI_USER`, `ANDREANI_PASS`)
- [ ] Webhook URL del carrier configurada apuntando a tu backend
- [ ] Sucursales sincronizadas al menos una vez (tabla `carrier_sucursales` poblada)
- [ ] Tarifas cargadas en `shipping_rates` (fallback si API cae)
- [ ] Templates de mail transaccional aprobados en Resend (verificación DNS)
- [ ] Twilio WhatsApp Business aprobado por Meta (7-14 días)
- [ ] n8n workflows 05, 06, 07, 08 activos
- [ ] Test end-to-end completo pasado (compra + etiqueta + tracking + entregado)
- [ ] Política de envíos publicada en la web (`/envios`)
- [ ] Política de devoluciones publicada (`/cambios-devoluciones`)
- [ ] Backup del contrato + credenciales en 1Password del equipo

---

## Contacto de emergencia por carrier

- **Correo Argentino ecommerce**: 0800-777-3400 opción 3
- **Andreani ecommerce**: 0810-122-1111 opción 2
- **Ambos:** priorizar apertura de ticket vía panel web para tener número de reclamo

---

**Con esta guía tenés todo lo que necesitás para empezar el proceso hoy mismo.** Los 3 primeros pasos que te recomiendo:

1. **Esta semana:** juntar todos los datos fiscales del dueño de BENNZ (sección 2.1)
2. **La semana que viene:** iniciar formulario de Correo Argentino ecommerce
3. **En paralelo:** contactar comercial de Andreani

Mientras tanto la maqueta sigue funcionando con datos mock — así podés mostrar el flujo completo a cualquiera sin haber firmado ningún contrato todavía.
