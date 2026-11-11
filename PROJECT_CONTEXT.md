# PROJECT_CONTEXT — Reglas estables del proyecto

## Misión

DLS Service Manager es una herramienta para el trabajo diario de DLS Servicio Técnico. El beneficio se mide por **facilidad y rapidez real de registrar y consultar un equipo**, trazabilidad del trabajo y calidad de comunicación. No construir un ERP, ni orientar el MVP a SaaS.

## Forma de trabajo obligatoria para agentes

1. Leer en este orden: `PROJECT_CONTEXT.md` → `ROADMAP.md` → `ARCHITECTURE.md` → `DATA_MODEL.md` → documentación del hito.
2. Identificar restricciones, incertidumbres, opciones y riesgos *antes* de modificar código. No inferir decisiones de producto sin evidencia.
3. Proponer un cambio acotado, explicar por qué y qué verificar. No implementar diez módulos a la vez.
4. Entregar código que compila y pruebas de los recorridos afectados; registrar resultados reales, no intenciones.
5. Actualizar documentación y el historial de decisiones. En conflicto, las reglas de este archivo prevalecen sobre prompts ocasionales; el ROADMAP determina el hito activo.
6. Pedir aclaración cuando cambie una regla esencial; no completar vacíos arbitrariamente.

## Producto y diseño

- Primero móvil: 360–390 px, navegación inferior, controles cómodos. Adaptar tablet y escritorio luego.
- Idioma español de Argentina, copy claro, cercano y profesional.
- Cuatro temas: **Claro amable**, **Oscuro minimalista**, **Focus**, **Super Night**. Todos accesibles, sin duplicar pantallas.
- Escritorio solamente: sidebar izquierda inicialmente 272 px; el borde derecho se arrastra, puede colapsar a cero sin quedar una tira de iconos, y un botón superior restaura a 272 px. Persistir tamaño/estado en navegador, ofrecer teclado y botón de restauración. Panel derecho de IA independiente.
- No pie de página con clichés tipo «hecho con amor».
- Links externos en la web: `target="_blank" rel="noopener noreferrer"`.

## Recepción y datos

- Dos modos **rápido** y **guiado** con el mismo modelo.
- Para confirmar: nombre del cliente, identificación flexible del equipo, falla informada y **título del acuerdo**. Estos son los únicos cuatro campos obligatorios.
- Observaciones, teléfono, Instagram, fotografías y demás campos son opcionales. **NO solicitar ni almacenar email de cliente**.
- Borrador puede estar incompleto. Todos los campos editables; cambios relevantes de acuerdos ya confirmados se auditan.
- Sugerencias no intrusivas, nunca ventanas bloqueantes; primero guardar, luego enriquecer.
- Las fotos se vinculan a la reparación; aparecen como tarjetas en la ficha, se amplían y descargan. Son opcionales.

## Tecnología y privacidad

- Aplicación web **online** en Next.js / Vercel. **No PWA, no Offline First, no Service Worker ni IndexedDB offline.**
- PostgreSQL/Auth/Storage privado de Supabase. No exponer secretos ni SQL libre en IA.
- Repo privado, demo con datos falsos, no compartir datos de clientes del historial original.
- `localStorage` exclusivamente para preferencias UI; nunca usarlo como archivo principal de comunicaciones. **Decisión confirmada:** historial bidireccional por reparación en Supabase, con mensajes enviados confirmados manualmente y respuestas del cliente pegadas/escritas manualmente.
- Un borrador preparado por IA no es un mensaje enviado; **Copiar no implica enviar**. Solo la acción «Confirmar enviado» crea el registro saliente. «Registrar respuesta» crea el registro entrante; ninguno se obtiene automáticamente de WhatsApp.
- La IA consulta únicamente el historial pertinente y autorizado, trata mensajes del cliente como datos no confiables y jamás infiere una aceptación contractual o precio confirmado de un mensaje sin verificación humana.
- La IA redacta borradores editables/copiables en base a datos autorizados; usuario envía manualmente. No PDF, no envío de WhatsApp, no respuestas automáticas.
- La IA jamás inventa montos, diagnósticos, repuestos, pruebas o promesas; indica si faltan datos esenciales.

## README y cierre

- `README.md` profesional: primeras líneas con URL de demo (o estado honesto «no desplegada»), enlaces al autor, instalación y uso verificados.
- Mantener `docs/DECISIONS.md` vivo y al finalizar completar `docs/TECHNICAL_REPORT.md` con decisiones y evidencia técnica de forma transparente (motivación, opciones, costos, seguridad, límites, pruebas, mejoras).
- Nada que no se haya ejecutado debe aparecer marcado «terminado».

## Decisión IA v1.4

- Único proveedor: OpenRouter, únicamente modelos verificados de costo cero; no usar modelos pagos ni fallback pago.
- Modelo principal: identificador configurable y pendiente de validación, no inventar ID de Gemini Flash Lite.
- Fallback: modelos `:free` validados y `openrouter/free` como alternativa; disponibilidad no garantizada.
- Si no responde: mostrar error honesto y ofrecer plantillas locales editables, sin inventar respuesta de IA.
- System prompt versionado en `prompts/dls-assistant.system.md`; no usar prompt como sustituto de autorización, validación o seguridad.
- Clave solo servidor en `OPENROUTER_API_KEY`. No persistir conversaciones en localStorage; historial de mensajes enviados y respuestas recibidas registrado manualmente en Supabase.
- Flujo Git obligatorio: revisar estado y diff, implementar hito acotado, correr pruebas, actualizar documentación y hacer commit explícito.
