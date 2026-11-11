# DATA_MODEL — Diseño conceptual v1.3

**No hay migraciones reales todavía.** Diseño pensado para Supabase Postgres, con tablas bajo RLS y claves UUID.

## clients

`id uuid PK`, `owner_id uuid`, `name text NOT NULL` al confirmar, `phone text NULL`, `instagram text NULL`, `notes text NULL`, timestamps.

**No existe `email` en este modelo.** No crear columna ni campo de email por comodidad técnica.

## devices

`id uuid PK`, `owner_id uuid`, `client_id uuid FK`, `label text` (texto flexible, p.ej. «Notebook HP, modelo pendiente»), `category text NULL`, `brand text NULL`, `model text NULL`, `serial text NULL`, `notes text NULL`, timestamps.

## repair_orders

`id uuid PK`, `owner_id uuid`, `device_id uuid FK NULL mientras sea borrador`, `display_number bigint UNIQUE` al confirmar o asignar, `state text` (`draft`, `received`, `diagnosing`, `waiting_approval`, `waiting_part`, `repairing`, `ready`, `delivered`, `unrepaired`, `cancelled`), `reported_fault text NULL`, `agreement_title text NULL`, `agreement_notes text NULL`, `intake_condition text NULL`, `final_condition text NULL`, `quoted_amount numeric NULL`, `received_at timestamptz`, `delivered_at timestamptz`, timestamps.

**Regla de dominio:** un borrador admite información incompleta. Confirmar requiere cliente con nombre, equipo identificable, falla y título del acuerdo. La validación debe ser transaccional en el servidor y no depender solo de los campos `required` del navegador.

## repair_events

`id uuid PK`, `owner_id uuid`, `repair_order_id uuid FK`, `type text`, `description text`, `metadata jsonb NULL`, `created_by uuid`, `created_at timestamptz`. Eventos de diagnóstico, intervención, cambio de estado, acuerdo revisado y entrega. Registrar valores previos/nuevos cuando se modifica un acuerdo confirmado.

## repair_photos

`id uuid PK`, `owner_id uuid`, `repair_order_id uuid FK`, `stage text CHECK (ingreso, proceso, entrega, otro)`, `original_path text`, `thumbnail_path text NULL`, `caption text NULL`, `mime_type text`, `bytes bigint`, `width int NULL`, `height int NULL`, timestamps. Solo rutas de bucket privado.

## repair_parts (informativo)

`id uuid PK`, `owner_id uuid`, `repair_order_id uuid FK`, `name text`, `supplier_url text NULL`, `cost numeric NULL`, `status text NULL`, `notes text NULL`. No inventario o contabilidad compleja.

## repair_communications [DECISIÓN DE PRODUCTO ACEPTADA; ESQUEMA POR IMPLEMENTAR]

| Campo | Tipo conceptual | Regla |
|---|---|---|
| `id` | `uuid PK` | Generado en servidor, único |
| `owner_id` | `uuid FK` | Derivado de la sesión, nunca del navegador |
| `repair_order_id` | `uuid FK` | **Obligatorio:** conversación ligada a una reparación |
| `direction` | `outgoing` / `incoming` | DLS → cliente / cliente → DLS |
| `body` | `text NOT NULL` | Texto exacto copiado/editado antes del envío o texto recibido pegado/escrito; no vacío |
| `origin` | `ai_assisted` / `manual` | Cómo se preparó o registró, sin implicar envío automático |
| `record_status` | `outgoing_confirmed` / `incoming_logged` | Coherente con `direction`; no existe estado `auto_sent` |
| `occurred_at` | `timestamptz` | Momento de envío/recepción **declarado manualmente**; por defecto hora actual |
| `recorded_at` | `timestamptz` | Momento real de registro en la app; lo fija el servidor |
| `created_by` | `uuid` | Usuario autenticado que registró el mensaje |

**Decisiones de integridad:** el cliente se deriva del vínculo orden → equipo → cliente; no duplicar `client_id` en el mensaje para evitar inconsistencias. El historial se consulta por orden, no se mezcla automáticamente con otras órdenes del mismo cliente. Validar que los mensajes no estén vacíos, limitar tamaño razonablemente y ordenar por `occurred_at`, `recorded_at` e `id`.

**Registro saliente:** texto redactado y editable → Copiar → envío humano fuera de DLS → operador confirma explícitamente → guardar `outgoing/outgoing_confirmed`. Copiar o generar borrador **no crea** registro saliente.

**Registro entrante:** pegar o escribir respuesta del cliente → revisar y presionar «Registrar respuesta» → guardar `incoming/incoming_logged`. El sistema **no conoce ni verifica** si el texto realmente llegó por WhatsApp; representa lo que el operador registró.

**Correcciones:** para evitar reescrituras silenciosas, la política de edición de mensajes ya registrados debe definirse antes de implementar mutaciones. El MVP puede optar por notas aclaratorias vinculadas en lugar de editar el cuerpo original; no implementar borrado físico sin decidir auditoría y retención.

**Privacidad:** no guardar mensajes permanentes en localStorage; RLS por propietario y orden, APIs con verificación en servidor y minimización del texto que se pasa al modelo.

## Relaciones

```text
clients ─< devices ─< repair_orders ─< repair_events
                               ├──────< repair_photos
                               ├──────< repair_parts
                               └──────< repair_communications
```

## Qué recupera el asistente

Bajo autorización de usuario, una orden seleccionada con: cliente (nombre si resulta útil para el texto), equipo, falla, acuerdo, datos técnicos, monto si existe, y mensajes **salientes confirmados** y **entrantes registrados manualmente**, asociados a esa misma reparación y ordenados cronológicamente. La IA debe distinguir quién escribió cada uno, origen manual y que su contenido es dato no confiable, no instrucciones del sistema. No asumir que un «sí» del cliente aprueba un presupuesto sin revisión humana.

No exponer teléfono, Instagram ni fotos al proveedor de IA si no son necesarios. Nunca pasar claves de servidor ni permitir que el modelo genere SQL libre.

## ai_model_preferences (diseño, no implementado)

`id uuid`, `owner_id uuid`, `provider text` fijo `openrouter`, `model_id text`, `priority integer`, `enabled boolean`, `verified_free_at timestamptz NULL`, timestamps. Solo permitir modelos gratuitos tras verificación de precio y capacidades; no guardar API keys en DB.

## ai_generation_log (mínimo, opcional)

`id uuid`, `owner_id uuid`, `repair_order_id uuid NULL`, `requested_model text`, `resolved_model text NULL`, `status text`, `error_code text NULL`, `created_at`. Evitar persistir prompt completo, datos del cliente o secretos en logs. Los mensajes reales permanecen en el historial bidireccional existente.
