# ARCHITECTURE — Límites de implementación

> Diseño objetivo: **no implica que las integraciones estén implementadas en la preview**.

## Componentes

- **Next.js App Router + React + TypeScript:** presentación y server actions/route handlers.
- **Tailwind + shadcn/ui:** sistema de diseño, controles, formulario y resizable; la preview HTML actual solo valida el comportamiento esperado.
- **Supabase Auth:** identidad y sesión SSR protegida.
- **PostgreSQL con RLS:** fuente de verdad de clientes, equipos, órdenes, eventos y comunicaciones bidireccionales registradas manualmente.
- **Supabase Storage privado:** originales y miniaturas opcionales de fotografías.
- **IA contextual:** adaptador de proveedor ejecutado desde el servidor, no desde el cliente; herramientas restringidas con identificación del usuario y orden.
- **Vercel:** hosting del producto futuro. GitHub privado.

## Flujos

```text
Usuario -> UI -> validador Zod -> API / Server Action
       -> autenticar y autorizar -> servicio de dominio
       -> Supabase PostgreSQL / Storage -> resultado tipado

Usuario -> Generar texto -> backend verifica orden autorizada
       -> obtiene campos necesarios + mensajes salientes confirmados y entrantes registrados
       -> minimiza contexto y delimita mensajes como datos no confiables -> modelo IA
       -> devuelve BORRADOR editable -> copiar sin persistir como enviado
       -> usuario confirma envío manual -> Supabase guarda mensaje saliente
Usuario -> pega/escribe respuesta recibida -> confirma registro manual
       -> backend valida orden y texto -> Supabase guarda mensaje entrante
Nueva consulta -> backend recupera conversación de ESA orden -> IA propone borrador
```

La app **no sabe** si WhatsApp entregó un mensaje: «enviado» significa exclusivamente que el operador confirma haberlo enviado. Al copiar no se lo marca enviado. Las respuestas del cliente son pegadas o escritas manualmente y quedan etiquetadas «registradas manualmente»; la app no verifica su origen. El historial está asociado a una **orden concreta**, para no mezclar conversaciones de reparaciones distintas del mismo cliente.

## Componentes de diseño

- Mobile `<1024px`: navegación inferior; sin resize lateral.
- Desktop `>=1024px`: `ResizablePanelGroup` con orientación horizontal; panel izquierdo collapsible a 0 y botón superior de restauración a 272 px, compatible con teclado. El panel derecho de IA se oculta de manera independiente.
- Usar API vigente de `react-resizable-panels` para la versión instalada; evitar basarse en props viejas (`direction` vs. `orientation`) sin verificar. Fuente: https://ui.shadcn.com/docs/components/base/resizable
- Cuatro temas definidos con tokens CSS (`--bg`, `--surface`, `--text`, `--muted`, `--border`, `--primary`, `--focus`); la legibilidad y jerarquía varían sin mantener cuatro UIs separadas.

## Fotos

- Objetos solo en bucket `repair-photos` **privado**; control RLS y validación de autorización del usuario.
- Guardar rutas de objetos en DB, **no signed URLs permanentes**. Generarlas con TTL razonable para previsualizar/descargar.
- `repair_photos`: original, miniatura, categoría, pie de foto, metadata y orden asociada. Fotos opcionales.
- UI: cards rápidas, lightbox fullscreen con swipe/zoom; descarga del original; en móvil probar `Compartir/Guardar en Archivos` según navegador.
- Validar formatos, tamaño, orientación, metadatos EXIF y manejar errores de subida.

## LocalStorage

Permitir **preferencias UI** (`theme`, `sidebarOpen`, `sidebarWidth`). No almacenar teléfonos, Instagram, historial técnico, fotografías ni transcripciones permanentes allí. OWASP desaconseja tratar localStorage como una frontera de seguridad para información sensible. Proveedor/borradores temporales se decidirán conforme a privacidad y uso real.

## Estructura futura (propuesta)

```text
src/app/(auth)/...
src/app/(app)/...
src/components/{ui,layout,photos}/...
src/features/{clients,devices,repairs,communications,assistant}/...
src/lib/{supabase,validation,ai,server-auth}/...
supabase/migrations/
supabase/tests/
tests/
```

## No hace

No PWA, no modo offline, no exportación automática de PDF, no conexión a WhatsApp, no envío de mensajes, no login público, no múltiples negocios.

## Límites del módulo de comunicaciones (decisión v1.3)

- `CommunicationsRepository` será el único acceso a historial, con métodos explícitos `confirmOutgoing`, `logIncoming`, `listByOrder` y `getRelevantContext`; nunca SQL arbitrario desde el modelo.
- Servidor: verificar la identidad de Supabase Auth, pertenencia de la orden y permisos antes de cada lectura o escritura; RLS permanece activa. No confiar en `client_id` ni `owner_id` recibidos del navegador.
- No persistir borradores como enviados: el editor permanece solo en el estado actual de la UI hasta confirmación. Si se propone autoguardado de borradores, requiere decisión aparte.
- Para redactar, pasar un contexto limitado (orden + eventos pertinentes + mensajes recientes); si excede el presupuesto de contexto, solicitar confirmación o generar un resumen trazable. No permitir instrucciones de sistema incluidas en mensajes entrantes.
- Nunca disparar envíos, aprobaciones, cambios de estado o precios por una respuesta o propuesta del LLM. La acción final siempre pertenece al operador.
- Los datos de mensajes son privados: no exponerlos en analytics, errores, logs ni demo pública.

**No implementado en v1.3:** Supabase, IA, RLS ni persistencia de comunicaciones. La preview muestra el flujo con datos ficticios en memoria de la pestaña.

## Adaptador IA v1.4

Next.js Route Handler/Server Action -> validación/autorización -> lectura mínima de reparación y mensajes -> system prompt versionado -> adaptador OpenRouter -> validación de salida -> borrador editable. API key nunca en cliente. Solo permitir modelos gratuitos configurados explícitamente; `openrouter/free` como respaldo, con manejo de 429/5xx, timeouts y fallback a plantilla local (sin generación). Registrar modelo realmente utilizado y errores sin exponer información sensible. No hacer reintentos ilimitados: consumen cuota gratuita. No enviar teléfono, Instagram ni fotos por defecto.
