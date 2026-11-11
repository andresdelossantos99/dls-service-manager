# ROADMAP — DLS Service Manager v1.4

**Estado:** documentación lista para copiar al repositorio; Next.js, Supabase e IA todavía no implementados.

## Hito 0 — Inicializar proyecto y repositorio [AHORA]

- [ ] Crear Next.js App Router, TypeScript, Tailwind, ESLint y carpeta `src/`.
- [ ] Copiar documentación v1.4 y crear `.env.local` sin secretos públicos.
- [ ] Iniciar GitHub privado y primer commit de documentación.
- [ ] Confirmar `npm run lint`, `npx tsc --noEmit`, `npm run build`.
- [ ] Verificar URL de GitHub del autor y futuro deploy para README.

## Hito 1 — Diseño mobile-first

- [ ] Cuatro temas y tokens.
- [ ] Navegación móvil y sidebar escritorio arrastrable, colapsable a cero y restaurable a 272px.
- [ ] Componentes accesibles, pantallas vacías y errores claros.

## Hito 2 — Supabase y recepción

- [ ] Auth privada, RLS, clientes (nombre obligatorio; teléfono/Instagram opcionales, sin email de cliente).
- [ ] Equipos, órdenes, borradores, recepción rápida y guiada.
- [ ] Solo cuatro requisitos al confirmar: nombre, equipo, falla y título del acuerdo; resto editable.

## Hito 3 — Taller y fotografías

- [ ] Eventos técnicos, estados, fotos opcionales privadas, miniaturas, zoom y descarga.
- [ ] Entrega y búsqueda.

## Hito 4 — Comunicaciones e IA

- [ ] Historial de mensajes enviados y respuestas recibidas, ambos registrados manualmente.
- [ ] Editor, copiar y confirmar envío (copiar no equivale a enviado).
- [ ] OpenRouter gratuito: verificar IDs/precios, modelos de respaldo, errores y plantillas locales.
- [ ] System prompt versionado y evaluaciones; no automatizar WhatsApp ni generar PDFs.

## Hito 5 — Calidad y publicación

- [ ] Pruebas, accesibilidad, seguridad, documentación técnica transparente y Vercel.

**No incluido:** PWA, offline, emails de clientes, envío automático, PDFs generados, monetización SaaS.
