# BioCleanProject App — Claude Code Instructions

## Project
Single-file PWA για καταχώρηση εσόδων/εξόδων μικρής επιχείρησης.
Client: Bio Clean Project (βιολογικοί καθαρισμοί).

## Stack
- **Ένα αρχείο:** `index.html` — όλο το app (HTML + CSS + JS) σε ένα αρχείο
- **Storage:** localStorage μόνο — χωρίς server, χωρίς database, χωρίς API calls
- **Hosting:** Render Static Site (auto-deploy από GitHub push)
- **URL:** https://bioclean-app.onrender.com

## Κανόνες που ΔΕΝ αλλάζουν
- Κανένα framework (no React, no Vue, no Node) — vanilla HTML/CSS/JS μόνο
- Κανένα npm package, κανένο build step
- Κανένα toISOString() — χρήση isoOf(y,m,d) για ημερομηνίες (timezone bug fix)
- Τα δεδομένα μένουν 100% local στο κινητό του χρήστη

## Χρώματα brand (Bio Clean Project logo)
- `--accent: #3D7E92` (βαθύ πετρόλ)
- `--accent-bright: #4997AD` (τιρκουάζ)
- `--accent-sky: #7CD3EA` (ουρανί)
- `--bg: #F5F9FA`

## Αρχιτεκτονική
- `render()` → router που καλεί renderEntry/renderHistory/renderInvoices/renderAppointments/renderStats
- `persist()` → αποθηκεύει όλα στο localStorage
- `materializeFixed()` → δημιουργεί πραγματικές κινήσεις από πάγια έξοδα (τρέχει στο init)
- `repairLegacyFixed()` → one-time migration παλιών δεδομένων (τρέχει στο init)
- Ημερομηνίες: πάντα με `isoOf(y, mZero, day)` — ποτέ `new Date().toISOString()`

## Εκκρεμείς διορθώσεις (TODO)
1. **Φίλτρα ιστορικού:** ΦΠΑ να είναι ξεχωριστό toggle (συνδυάζεται με όλα τα άλλα φίλτρα)
   - Άξονας 1: Όλα / Έσοδα / Έξοδα
   - Άξονας 2: Όλοι / Μετρητά / Κάρτα / Τράπεζα
   - Άξονας 3: Όλα / Απόδειξη / Τιμολόγιο / Κανένα
   - Άξονας 4: toggle ΦΠΑ (cross-cutting με όλα τα παραπάνω)
2. **ΦΠΑ εικόνα στατιστικών:** όταν υπάρχει ποσό στο κουμπί "Πληρωμή →" χαλάει το layout

## Workflow
```
edit index.html → git add . && git commit -m "msg" && git push → Render auto-deploy (~2 min)
```

## Test μετά από κάθε αλλαγή
1. Βάλε έσοδο → φαίνεται στο Ιστορικό;
2. Φαίνεται στα Στατιστικά (κάρτα μήνα);
3. Το πορτοφόλι ενημερώνεται;
