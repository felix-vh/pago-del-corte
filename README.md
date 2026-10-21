# Pago del Corte

Credit card tracker built around how cards actually work in Mexico: billing cycles (*fecha de corte*), payment due dates, *meses sin intereses* (MSI) and purchases you pay for but someone else pays back.

Most budgeting apps think in calendar months. Mexican credit cards don't. What you spend between the 17th of one month and the 16th of the next is what you pay in early December, and an MSI purchase from June is still on your statement in November. Pago del Corte answers one question per card and per month: **how much do I pay, and how much of that is actually mine?**

> Estado: prototipo funcional. La app está en español porque está pensada para tarjetas mexicanas.

## What it does

- **Billing cycles per card.** Each card has its own cut-off day and days-to-pay. Purchases fall into the right statement automatically.
- **MSI plans.** Enter the total, the number of months and (optionally) the exact monthly amount the bank charges. Banks round up and adjust the last payment; the app does the same.
- **Recurring charges.** Subscriptions show up as *expected* in future cycles and disappear once the real charge is recorded.
- **Shared expenses.** Mark who pays part or all of a purchase. Split MSI plans are divided proportionally on every installment, with rounding that adds up to the cent.
- **Who owes you.** A checklist of pending reimbursements per month.
- **All cards view.** Groups every card by the month the payment is due, not by cut-off date.
- **Income, fixed payments and savings.** Salary, side income (counted only once received), cash expenses and set-asides, ending in what's left for savings.

## Try the prototype

The prototype is a single HTML file with no build step and no dependencies.

```bash
git clone https://github.com/<your-user>/pago-del-corte.git
cd pago-del-corte/prototype
python3 -m http.server 8000   # or just open index.html in a browser
```

Data is stored in your browser's `localStorage`. Nothing leaves your device.

## Roadmap

The plan for turning the prototype into a full app is in [`CLAUDE.md`](CLAUDE.md). In short:

1. **Next.js + TypeScript** port, with the domain logic (cycles, MSI, shares) extracted into a tested pure module.
2. **Supabase** for auth and storage, with row-level security so each user only sees their own data.
3. **Statement import**: upload a bank statement PDF and have an LLM extract the movements, then reconcile the totals against the statement before saving.
4. Bank presets (BBVA, HSBC, Banorte, Nu, etc.) for cut-off and due-date rules.

## Design notes

- Amounts are stored as integer cents to avoid floating point errors.
- Cycle logic is in [`docs/domain.md`](docs/domain.md), which is the spec the rewrite has to pass.
- Privacy first: financial data never goes in the repo, logs or test fixtures. Fixtures use made-up numbers.

## License

MIT. See [LICENSE](LICENSE).
