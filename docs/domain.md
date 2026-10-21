# Reglas de dominio

Este documento es la especificación de la lógica. La versión en Next.js debe reproducir exactamente estos resultados; cada regla debe tener al menos una prueba.

Todos los montos son **enteros en centavos**. Las fechas son `AAAA-MM-DD` en hora local.

## 1. Periodos de corte

Cada tarjeta tiene `cut` (día de corte, 1 a 31) y `dueDays` (días entre el corte y la fecha límite de pago).

- Un periodo se identifica con una llave `k = año * 12 + mes0`, donde `mes0` es el mes (0 a 11) en el que **termina** el periodo.
- Día de corte efectivo: `min(cut, último día del mes)`. Ejemplo: corte 31 en febrero es el 28 o 29.
- Una compra con fecha `d` pertenece al periodo del mes de `d` si `día(d) <= corte efectivo`; si no, al del mes siguiente.
- Inicio del periodo `k`: el día después del corte del periodo `k - 1`.
- Fecha límite: corte del periodo + `dueDays` días.

Ejemplo, corte 16: el periodo de noviembre va del 17 de octubre al 16 de noviembre.

La fecha que manda es la **fecha de cargo** del estado de cuenta, no la fecha de operación.

## 2. Meses sin intereses (MSI)

Plan: `total`, `months`, `date` (fecha de compra), `monthly` opcional.

- Mensualidad base: `monthly` si existe; si no, `floor(total / months)`.
- Mensualidad número `n` en el periodo `k`: `n = k - periodo(date) + 1`. La primera mensualidad cae en el periodo de la compra.
- Para `1 <= n < months` se cobra la base; en `n = months` se cobra `total - base * (months - 1)`.
- Fuera de `1..months` no se cobra nada.
- Saldo pendiente después del periodo `k`: `total - base * min(n, months)`, o 0 si ya terminó.

Ejemplo (cifras inventadas): `total = 125050`, `months = 6`, `monthly = 20900`. Mensualidades 1 a 5: 20900; la 6: 20550.

## 3. Cargos fijos (recurrentes)

`{desc, amount, day, card}`.

- Solo se proyectan en el periodo actual y los futuros (`k >= periodo de hoy`).
- La fecha proyectada es `day` dentro del periodo: si `day > corte`, cae en el mes anterior al de cierre.
- Un recurrente **no** se proyecta en un periodo donde ya existe una compra real de la misma tarjeta con el mismo `desc` (comparado sin mayúsculas ni espacios extremos) o con `recId` igual.
- Los proyectados cuentan en los totales y se muestran como "Esperado".

## 4. Pagos compartidos

Compras y planes pueden tener `who` (quién paga) y `whoAmt` (cuánto pone del total).

- Sin `whoAmt` (o 0): esa persona paga todo.
- Compra: parte de la otra persona = `min(whoAmt, amount)`.
- Plan MSI: la parte se reparte en proporción `r = whoAmt / total` sobre la **suma acumulada**, para que el redondeo cuadre al centavo:
  `parte(n) = round(acum(n) * r) - round(acum(n - 1) * r)`, con `acum(n)` = suma de mensualidades 1..n.

Ejemplo (cifras inventadas): `total = 900000`, 3 meses de 300000, `whoAmt = 520000`. Partes: 173333, 173334, 173333 (suman 520000).

- "Sale de tu bolsa" = total del banco − partes de otros − apoyos mensuales.

## 5. Vista de todas las tarjetas

Se agrupa por **mes de pago** (mes de la fecha límite), no por mes de corte. Para el mes `M`, cada tarjeta aporta el periodo `k` cuya fecha límite cae en `M` (buscar en `M - 3 .. M + 1`).

## 6. Ingresos y colchón

- Los ingresos del mes `M - 1` pagan los cortes que vencen en el mes `M` (el salario llega a fin de mes).
- Extras (ingresos variables) y gastos sin tarjeta cuentan en el mes calendario en que ocurrieron.
- Colchón del mes `M` = ingresos fijos + extras de `M - 1` − tu parte de tarjetas en `M` − pagos y apartados fijos − tu parte de gastos sin tarjeta de `M - 1`.

## 7. Por cobrar

Llaves estables para marcar lo cobrado:

- Compra: `i:<id>`
- Mensualidad de plan: `p:<id>:<n>`
- Gasto sin tarjeta: `c:<id>`
- Apoyo mensual: `a:<id>:<M>`

## 8. Pago para no generar intereses

Al importar un estado de cuenta, antes de guardar hay que verificar:

- Suma de cargos regulares importados = "Cargos regulares (no a meses)" del estado.
- Suma de mensualidades MSI = "Cargos y compras a meses".
- Contado + MSI + saldo a favor (negativo) = "Pago para no generar intereses".

Si algo no cuadra, no se guarda y se muestra la diferencia.
