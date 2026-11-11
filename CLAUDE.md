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
5. **Presets por banco** y pulido: onboarding para alguien que nunca la ha usado (agregar tarjetas con su banco, corte y días para pagar), exportar CSV.
6. **Beta con usuarios reales**: desplegar la web en Vercel y compartirla con un grupo pequeño de amigos. Medir qué registran, dónde se atoran y qué piden antes de invertir en la app móvil.
7. **App móvil** (iOS y Android) sobre la misma base de código, con Capacitor o Expo:
   - Notificaciones locales antes de cada fecha de corte y fecha límite de pago.
   - Desbloqueo con Face ID o huella.
   - Widget con el pago del mes y lo que falta por cobrar.
   - Registro rápido de un gasto desde la pantalla de inicio.
   - Beta por TestFlight y Google Play (prueba interna) antes de publicar.
   - Esto también cumple la regla de Apple de que la app aporte funciones propias y no sea solo una página web envuelta.
8. **Captura automática de gastos**: que los cargos se registren solos, con revisión contra el estado de cuenta en cada corte.
   - **iOS**: automatización de Atajos con el disparador "Transacción" de Apple Wallet. Recibe monto, comercio y tarjeta de cada pago con Apple Pay y los manda a la app (App Intent o endpoint autenticado). No cubre tarjeta física ni compras en línea con el número de la tarjeta.
   - **Android**: NotificationListenerService con permiso explícito del usuario para leer las notificaciones de las apps de bancos (BBVA, HSBC, etc.) y extraer monto, comercio y tarjeta.
   - **Cualquier plataforma**: leer los correos de alerta de compra del banco (Gmail API con permiso de solo lectura, o reenvío a un buzón de la app).
   - Todo cargo capturado automáticamente entra como "por confirmar" hasta que el usuario lo revisa o se concilia con el estado de cuenta (fase 4), para evitar duplicados.
   - Nunca guardar el texto completo de la notificación ni del correo; solo los campos extraídos.
   - A futuro: agregadores de open banking en México (por ejemplo Belvo) para conectar directo con el banco.
   - FinanceKit de Apple no aplica por ahora: solo funciona con Apple Card y bancos fuera de México.

## Antes de abrirla a otras personas
- **Aviso de privacidad** conforme a la LFPDPPP (ley mexicana de datos personales) y política de privacidad pública: las tiendas la exigen.
- Opción de **borrar la cuenta y todos sus datos** desde la app (requisito de Apple).
- Nada de datos financieros en analítica ni en reportes de errores.
- Costos: Apple Developer USD 99 al año, Google Play USD 25 una vez. El plan gratuito de Supabase y Vercel alcanza para la beta.


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
