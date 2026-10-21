# Pago del Corte: guía para Claude Code

## Qué es
Rastreador de tarjetas de crédito para México: periodos de corte, fechas límite, meses sin intereses y gastos compartidos. Hoy existe un prototipo funcional en un solo archivo (`prototype/index.html`). El objetivo es convertirlo en una app completa y publicable, que también sirva como proyecto de portafolio.

## Fuente de verdad
- `docs/domain.md` define las reglas de negocio. Cualquier código nuevo debe reproducir esos resultados.
- `prototype/index.html` es la referencia de comportamiento y de UI. No lo borres; es útil para comparar.

## Stack objetivo
- Next.js (App Router) + TypeScript estricto
- Supabase: Auth, Postgres y RLS (cada fila con `user_id`, políticas `auth.uid() = user_id`)
- Tailwind para estilos, respetando la paleta del prototipo (verde petróleo como acento, ámbar para MSI)
- Vitest para pruebas de la lógica de dominio
- Despliegue en Vercel

## Plan por fases
1. **Dominio** (`src/domain/`): funciones puras en TypeScript para periodos, MSI, recurrentes, partes compartidas, vista por mes de pago y colchón. Pruebas con los ejemplos de `docs/domain.md`. Sin UI todavía.
2. **UI local**: portar el prototipo a componentes con estado en `localStorage`, misma funcionalidad.
3. **Supabase**: tablas `cards`, `items`, `plans`, `recurring`, `incomes`, `extras`, `cash`, `accounts`, `collected`; RLS en todas; migraciones versionadas.
4. **Importar estado de cuenta**: subir PDF, extraer movimientos con un LLM, mostrar una vista previa editable y validar los totales de la sección 8 de `docs/domain.md` antes de guardar.
5. **Presets por banco** y pulido (onboarding, exportar CSV).

## Reglas del proyecto
- Montos siempre en centavos (`number` entero). Formatear solo al mostrar, con `Intl.NumberFormat('es-MX', { style: 'currency', currency: 'MXN' })`.
- Fechas como `AAAA-MM-DD` en hora local; no usar `new Date('AAAA-MM-DD')` sin hora, porque se interpreta en UTC.
- La UI está en español de México.
- **Privacidad**: nunca subir datos financieros reales al repo, a fixtures, a logs ni a issues. Los ejemplos usan cifras inventadas y lo dicen. No guardar números de tarjeta completos, CLABE ni RFC; solo los últimos 4 dígitos.
- Las API keys (LLM, Supabase service role) van en variables de entorno y nunca en el cliente. Para importar PDFs, preferir que cada usuario use su propia key o que el procesamiento ocurra en el servidor sin persistir el PDF.
- Commits pequeños y descriptivos. Cada regla de dominio nueva llega con su prueba.

## Comandos (cuando exista la app)
- `npm run dev`: servidor local
- `npm test`: pruebas de dominio
- `npm run lint`
