Tractor Hisab Pro - Updated Receipt + Late Interest

Included files:
- index.html
- app.js
- style.css

Important:
1. Keep your existing firebase-config.js and other project files unchanged.
2. Replace the existing index.html, app.js and style.css with these files.
3. The app.js continues to import ./firebase-config.js as before.

Receipt update:
- The old compact payment receipt has been replaced by the A4 Cash Payment Receipt.

Late payment interest:
- 15 calendar days of grace from each work-entry date.
- After the grace period, interest is calculated daily at 2% per 30 days.
- Formula: outstanding principal x 0.02 x overdue days / 30.
- Customer payments are allocated to the oldest outstanding principal first.
- Interest is simple interest (no interest on accrued interest).
- Interest is calculated dynamically; no new Firebase fields or migrations are required.
