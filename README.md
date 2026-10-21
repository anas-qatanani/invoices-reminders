# Invoices and payment reminders

Invoices made from your own data. Reminders follow. Customers, orders and time reports (an export like Fortnox or Visma) become invoices with number, OCR, VAT, rotavdrag or reverse charge, and due date. Then the bank file is matched, and every overdue invoice gets a reminder draft in Swedish or English. A person approves each invoice run and each reminder. Nothing is sent by the machine.

**Try it:** https://anas-qatanani.github.io/invoices-reminders/

The firm (Kustby Ventilation AB, 12 people), its customers, orders and payments are all made up. No client data. Personnummer are Skatteverket's published test numbers. Org and VAT numbers have a wrong check digit on purpose, so none can belong to a real firm. Emails use the reserved `.example` domain; the phone number is from PTS's series for fiction.

## Result on 77 test orders

| Check | Result |
|---|---|
| Wrong amounts (invoice net, VAT, rotavdrag, to pay; left to pay, fee, amount in the reminder) | **0** |
| Orders handled right (invoiced, or stopped for a person, or left alone) | 77 / 77 |
| Invoices made | 66 (6 with rotavdrag, 2 with reverse charge, 16 in English) |
| Orders stopped for a person (no invoice, no number) | 9 |
| Invoices missing a field from the checked list (ML 17 kap. 24 §, Skatteverket rot list) | 0 |
| Bank payments placed right (or sent to a person when unclear) | 41 / 41 |
| Invoice status right (paid, not due, overdue, held, ...) | 66 / 66 |
| Reminders drafted | 22 |
| Tampered drafts caught by the number check | 22 / 22 drafts, 468 / 468 bad copies |

Every expected amount was written by hand per case, not by the program. Six planted bugs (rot rounded up, VAT per row, discount dropped, no rot cap, fixed 30-day terms, travel counted as rot work) each made the score card fail.

## How it works
1. **Make invoices.** Every finished order: its rows and reported hours, discount, VAT worked out once on the net sum, rotavdrag or reverse charge, invoice number in one series, OCR with check digit, due date from the customer's terms. An order with a missing or doubtful fact goes to a person and gets no number, so the series has no gaps.
2. **Match.** The bank file: a valid OCR first, then invoice numbers in the message, also one payment for two invoices. A Skatteverket rot payout is never booked on a customer's invoice. Anything unclear goes to a person.
3. **Find late.** What is left on each invoice and how many days late. 1-14 days: friendly. 15-30: firmer. 31 or more: last notice. Disputed invoices, overpayments, tiny rests and customers with an unplaced payment go to a person.
4. **Draft and check.** One reminder per invoice, in the customer's language. A check proves that every number, date and reference in the draft equals the books, and that no other number is in it.

All numbers come from plain code, never from an AI model.

## Orders it stops for a person
Rot without a valid personnummer, rot without fastighetsbeteckning, rot for a company, reverse charge without the buyer's VAT number, reverse charge on goods only, an unknown article, a row without a price, a total below zero (credit invoice), a customer without an address. A job not marked done waits. An order already invoiced is never invoiced twice.

## Swedish rules it follows
- **Invoice contents:** date, unique number in a series, both names and addresses, VAT numbers, what was done, net per rate, VAT rate and amount. Mervärdesskattelag (2023:200) 17 kap. 24-25 §§.
- **VAT 25 %**, on the net sum. ML (2023:200) 9 kap. 2 §.
- **Rotavdrag:** up to 30 % of the work cost incl. moms, at most 50 000 kr per person and year; not material or travel. Skatteverket, "Så fungerar rotavdraget". The demo reproduces Skatteverket's own example to the krona.
- **Reverse charge** for building work sold to a builder: no VAT, "Omvänd betalningsskyldighet" and the buyer's VAT number. Goods only: normal VAT. Skatteverket, "Omvänd betalningsskyldighet inom byggsektorn".
- **Reminder fee 60 kr** only when agreed at the latest when the debt arose. Lag (1981:739) 2 § and 4 §.
- **Late payment interest** = referensränta + 8 procentenheter from the due date. Räntelagen (1975:635) 3 §, 6 §, 9 §. Referensränta 2,00 % from 1 July 2026.

Sources: [riksdagen.se, ML 2023:200](https://www.riksdagen.se/sv/dokument-och-lagar/dokument/svensk-forfattningssamling/mervardesskattelag-2023200_sfs-2023-200/) · [skatteverket.se, rotavdrag](https://www.skatteverket.se/foretag/skatterochavdrag/rotochrut/safungerarrotavdraget.4.2ef18e6a125660db8b080002709.html) · [skatteverket.se, omvänd betalningsskyldighet bygg](https://www.skatteverket.se/foretag/moms/sarskildamomsregler/byggverksamhet/omvandbetalningsskyldighetinombyggsektorn.4.47eb30f51122b1aaad28000545.html) · [riksdagen.se, lag 1981:739](https://www.riksdagen.se/sv/dokument-och-lagar/dokument/svensk-forfattningssamling/lag-1981739-om-ersattning-for-inkassokostnader_sfs-1981-739/) · [riksdagen.se, räntelag 1975:635](https://www.riksdagen.se/sv/dokument-och-lagar/dokument/svensk-forfattningssamling/rantelag-1975635_sfs-1975-635/) · [riksbank.se, referensränta](https://www.riksbank.se/sv/statistik/rantor-och-valutakurser/referensranta/). This is not legal or tax advice.

## What it does not do
- It does not decide if a job is building work or rot work. The order says so; the code checks the facts each needs and stops if one is missing.
- The 50 000 kr rot limit is counted on this firm's invoices only. The customer may have used rot elsewhere this year.
- If Skatteverket refuses the rotavdrag, the customer owes that part. A person sends that invoice.
- Credit invoices, sales of goods abroad and VAT rates other than 25 % go to a person.
- Rot is rounded down to whole kronor. That is a choice of the demo, not a rule found in Skatteverket's text.
- It sends nothing. Invoices and reminders wait for a person.

## How it maps to a real firm
| Demo | Real setup |
|---|---|
| `customers.csv`, `articles.csv`, `orders.csv`, `order_rows.csv`, `time.csv` | Export or API from the accounting or job program (Fortnox, Visma, a time app) |
| Invoice on the page | Invoice draft in the accounting program; a person approves the run |
| `bank.csv` | The bank's payment file (for example Bankgirot) |
| Reminder on the page | Draft saved in the mail program; a person presses Send |

## How it was built
Python standard library only, no cloud, no API costs. The page runs offline in the browser and plays one real run. The source code is private. Ask on a call if you want to see it.

---
Built by Anas Qatanani · [AI-Driven](https://www.linkedin.com/company/ai-driven-tech) · [Book a 15-min call](https://calendly.com/anas-aidriven/15min)
