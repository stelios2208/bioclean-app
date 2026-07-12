# Αρχιτεκτονική — BioCleanProject App

Σύντομος οδηγός για όποιον/όποια (άνθρωπο ή AI βοηθό) πιάσει το project πρώτη φορά.

## Framework / Γλώσσα

- **Καθόλου framework.** Vanilla HTML + CSS + JavaScript, όλα μέσα σε **ένα αρχείο**: `index.html`.
- Δεν υπάρχει build step, δεν υπάρχει `package.json`, δεν υπάρχει npm/node_modules.
- Δεν υπάρχει backend/server/API. Το app τρέχει εξ ολοκλήρου στον browser του χρήστη.
- UI rendering γίνεται με χειροκίνητο `innerHTML` templating (template literals σε JS συναρτήσεις `renderX()`), όχι Virtual DOM.

## Δομή φακέλων

```
/
├── index.html      # ΟΛΟ το app: <style> (CSS) + <body> (markup) + <script> (JS)
├── CLAUDE.md        # Οδηγίες project για AI βοηθούς (κανόνες, brand χρώματα, TODOs)
└── ARCHITECTURE.md  # Αυτό το αρχείο
```

Δεν υπάρχουν subfolders, assets, εικόνες ή static files εκτός του `index.html` (το λογότυπο είναι inline SVG μέσα στο HTML).

## Πού & πώς αποθηκεύονται τα δεδομένα

**Αποκλειστικά `localStorage`** του browser — καμία βάση δεδομένων, κανένα API call, καμία σύγχρονη αποστολή δεδομένων εκτός συσκευής. Όλα τα δεδομένα είναι 100% local στο κινητό/browser του χρήστη.

Κεντρικό object με τα localStorage keys (`K`, γύρω στη γραμμή 237 του `index.html`):

| Key                | JS μεταβλητή     | Περιεχόμενο                          |
|--------------------|------------------|----------------------------------------|
| `bpc_transactions`  | `transactions`   | Έσοδα & έξοδα (κύριο ledger)          |
| `bpc_fixed`         | `fixedExpenses`  | Πάγια έξοδα (recurring)               |
| `bpc_appointments`  | `appointments`   | Ραντεβού                              |
| `bpc_settings`      | `settings`       | Ρυθμίσεις (ΦΠΑ, κατηγορίες, κ.λπ.)    |
| `bpc_transfers`     | `transfers`      | Μεταφορές μεταξύ μεθόδων πληρωμής     |
| `bpc_adjustments`   | `adjustments`    | Χειροκίνητες διορθώσεις υπολοίπου     |

- `load(key, fallback)` διαβάζει και κάνει `JSON.parse`.
- `save(key, val)` κάνει `JSON.stringify` και γράφει.
- `persist()` αποθηκεύει **όλα** τα παραπάνω μαζί — καλείται μετά από κάθε αλλαγή δεδομένων.
- Δεν υπάρχει καμία εξωτερική sync/cloud λειτουργία. Export/Import backup γίνεται χειροκίνητα ως αρχείο JSON (κουμπιά στα Στατιστικά).

### Σημαντικό: ημερομηνίες

Ποτέ `new Date().toISOString()` (bug λόγω timezone — γλιστράει μέρα). Πάντα μέσω `isoOf(y, mZero, day)` που φτιάχνει `'YYYY-MM-DD'` string χειροκίνητα.

## Ποια σημεία του κώδικα χειρίζονται τι

Το `index.html` είναι router-based SPA με state σε global JS μεταβλητές. `render()` (~γραμμή 429) διαλέγει ποια `renderX()` να καλέσει ανάλογα με `activeTab`.

| Λειτουργία            | Render function      | State μεταβλητές              | Helper functions                                  |
|------------------------|-----------------------|--------------------------------|----------------------------------------------------|
| **Νέα καταχώρηση** (έσοδο/έξοδο) | `renderEntry()` (~446)  | `entryType`, `entryDraft`, `editingId` | `saveEntry()`, `editTx()`, `deleteTx()`             |
| **Ιστορικό** (λίστα + φίλτρα) | `renderHistory()` (~575) | `histFilterType`, `histFilterPay`, `histFilterDoc`, `histFilterVat`, `histSelectMode` | `histMatches()`, `setHistType/Pay/Doc()`, `toggleHistVat()` |
| **Τιμολόγια**          | `renderInvoices()` (~670) | — (φιλτράρει `transactions` με `doc==='invoice'`) | `toggleInvoiceSent()`                              |
| **Ραντεβού**           | `renderAppointments()` (~809), `renderCalendar()` (~777) | `apptDraft`, `apptView`, `apptFilter`, `calMonth`, `payingApptId` | `saveAppt()`, `markDone()`, `openPay()`/`confirmPay()` (μετατρέπει ραντεβού σε έσοδο) |
| **Στατιστικά / Πορτοφόλι** | `renderStats()` (~958)   | `currentMonth`, `expandedStat`, `vatPayOpen` | `walletBalance()`, `vatPortion()`, `materializeFixed()`, `editWallet()` |

Άλλα βοηθητικά κομμάτια:
- **Πάγια έξοδα**: `materializeFixed()` (~360) δημιουργεί πραγματικές εγγραφές `transactions` από τα `fixedExpenses` κάθε μήνα, τρέχει στο init.
- **Αριθμομηχανή (calculator overlay)**: `calcXxx()` συναρτήσεις (~1397+), ανεξάρτητο UI component, δεν αγγίζει δεδομένα.
- **Backup**: `exportBackup()` / `importBackup()` (~1183) — export/import ολόκληρου του state ως JSON αρχείο.

## Schema δεδομένων (όπως αποθηκεύονται σήμερα)

Δεν υπάρχει επίσημο schema/TypeScript types — τα object shapes προκύπτουν από το πώς γράφονται στον κώδικα. Ενδεικτικά:

### `transaction` (μέσα στο `transactions[]`)
```js
{
  id: 'uid string',
  type: 'income' | 'expense',
  amount: Number,
  category: 'string',           // από serviceCategories ή expenseCategories
  date: 'YYYY-MM-DD',
  payment: 'Μετρητά' | 'Κάρτα' | 'Τράπεζα',
  vat: Boolean,
  doc: 'none' | 'receipt' | 'invoice',
  sent: Boolean,                 // μόνο για doc==='invoice': εστάλη στον λογιστή;
  note: 'string',
  // προαιρετικά πεδία που εμφανίζονται σε ειδικές εγγραφές:
  fixedId: 'string',             // αν προήλθε από πάγιο έξοδο
  auto: Boolean,                 // αν δημιουργήθηκε αυτόματα (π.χ. από materializeFixed)
  vatPayment: Boolean            // αν είναι εγγραφή πληρωμής ΦΠΑ (εξαιρείται από "πραγματικά έξοδα")
}
```

### `fixedExpense` (μέσα στο `fixedExpenses[]`)
```js
{
  id: 'uid string',
  name: 'string',
  amount: Number,
  dayOfMonth: Number,            // 1-31
  payment: 'Μετρητά' | 'Κάρτα' | 'Τράπεζα',
  vat: Boolean,
  active: Boolean,
  startMonth: 'YYYY-MM-DD',      // πρώτος μήνας ισχύος
  createdAt: 'YYYY-MM-DD'
}
```

### `appointment` (μέσα στο `appointments[]`)
```js
{
  id: 'uid string',
  client: 'string',
  service: 'string',             // από serviceCategories
  date: 'YYYY-MM-DD',
  price: Number | null,
  note: 'string',
  status: 'pending' | 'done',
  paid: Boolean,
  txId: 'string' | null          // id της transaction που δημιουργήθηκε όταν πληρώθηκε
}
```

### `transfer` (μέσα στο `transfers[]`)
```js
{ id, date: 'YYYY-MM-DD', from: 'Μετρητά'|'Κάρτα'|'Τράπεζα', to: same, amount: Number }
```

### `adjustment` (μέσα στο `adjustments[]`)
```js
{ id, date: 'YYYY-MM-DD', method: 'Μετρητά'|'Κάρτα'|'Τράπεζα', delta: Number }
```

### `settings`
```js
{
  vatRate: Number,               // π.χ. 24
  serviceCategories: ['Καναπέδες', ...],
  expenseCategories: ['Ενοίκιο', ...],
  genFixed: ['fixedId:YYYY-MM', ...],  // markers ήδη υλοποιημένων πάγιων ανά μήνα
  fixedRepaired: Boolean,        // one-time migration flag
  dateRepairDone: Boolean        // one-time migration flag
}
```

## Deploy

- **Hosting:** Render Static Site, auto-deploy από push στο GitHub `main` branch.
- **URL:** https://bioclean-app.onrender.com
- Ροή: `edit index.html` → `git commit` → `git push origin main` → Render auto-deploy (~2 λεπτά).
