# Receipt Tracker: instructions for Claude

This folder is a simple, free, no-login expense tracker that lives entirely on the user's own computer. The user drops a receipt (photo, screenshot or PDF) into the chat and says something like "log this". Your job is to register it.

Files:
- `settings.js`: the user's home currency and symbol (default GHS / ₵).
- `data.js`: the ledger, one JS object per receipt inside `window.RECEIPTS = [ ... ];`
- `receipts/`: the saved originals.
- `dashboard.html`: opened by double-clicking. Never needs editing for a new receipt.

## First run

If `data.js` does not exist, create it by copying `data.sample.js`. If the `receipts/` folder does not exist, create it. Ask the user once for their **name** (used in `by`) and their **home currency**; if it isn't GHS, update `settings.js` (code and symbol). Don't ask again after that.

## Logging a receipt

1. **Read it** (look at the image; for PDFs extract the text). Work out store, date, total, currency, category, payment method, items.
2. **Show a short summary** and ask, never guessing:
   - Is it **Personal, Work or Reimbursable**?
   - "Is this right? Once I save it, it's locked."
3. Only after a clear yes:
   1. If the receipt currency differs from the home currency, get that day's rate (home units per 1 unit) from `https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@YYYY-MM-DD/v1/currencies/usd.json` (replace `usd` with the receipt currency in lowercase; read the field named after the home currency in lowercase, e.g. `usd.ghs`). If that fails use `@latest`, else ask the user. Round to 6 decimals. `rate` is 1 when the currencies match.
   2. Copy the original into `receipts/` named `R-0007_2026-10-24_Melcom_345.50GHS.ext` (id, date, store, amount+currency).
   3. Append **one line** to the end of the list in `data.js`, using the next free id (R-0001, R-0002, ...):

```js
  { id: "R-0001", date: "2026-10-23", store: "Example Store", amount: 24.00, currency: "USD", rate: 11.6, category: "Subscriptions", payment: "Card", purpose: "Personal", by: "Alex", status: "Sealed", corrects: "", items: "1 x Item @ 24.00", notes: "", file: "receipts/R-0001_2026-10-23_Example-Store_24.00USD.pdf" },
```
   Categories: Groceries, Food & Drinks, Transport, Fuel, Utilities, Airtime & Data, Household, Health, Clothing, Electronics, Education, Entertainment, Gifts & Family, Personal Care, Subscriptions, Other.
   4. Tell the user it's saved ("Sealed R-0007") and that they can refresh `dashboard.html`.

## The lock rule

- **Never edit or delete an existing line in `data.js`, or anything in `receipts/`.** Only append.
- A mistake becomes a **correction**: a new line with `status: "Correction"`, `corrects: "R-0007"`, the same store/date/currency/rate, `amount` = the difference (negative to reduce), and the reason in `notes`.
- If asked to "just fix" or delete a receipt, explain this and offer a correction instead.

## Reports

For "expense report for Sam, reimbursable, October": filter `data.js` and give a table with totals in the home currency and original currencies, or tell the user to use the dashboard filters and **Download CSV**.

## Privacy

Nothing is online. `data.js` and `receipts/` are the user's private data: never upload, commit or share them unless the user asks.
