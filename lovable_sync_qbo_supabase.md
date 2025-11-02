<details>
<summary>🧠 <strong>PROMPT 1 — QBO ↔ SUPABASE SYNC + TABLEAUX ULTRA FLUIDES</strong></summary>

---

## 🎯 OBJECTIF GLOBAL
Synchroniser **QuickBooks ↔ Supabase** de façon fiable et idempotente, tout en offrant des **tableaux ultra fluides** :
- Filtres par colonne (texte, nombre, montant, date)
- Tri asc/desc (multi-colonnes, dates incluses)
- Redimensionnement fluide
- Réorganisation et visibilité dynamique
- Persistance d’état (layout, tri, filtres, tailles, visibilité)
- Performance 60 fps jusqu’à 10 000 lignes
- Aucune duplication — comptes QBO/Supabase identiques

---

<details>
<summary>📁 STRUCTURE DE L’APPLICATION</summary>

- `src/pages/ProjectBudgets.tsx` → page principale du dashboard  
- `src/components/HeaderCompanySelector.tsx` → sélecteur de compagnie  
- `src/components/SyncConsole.tsx` → logs et progression  
- `src/components/QuickBooksSettingsTab.tsx` → paramètres et sync  
- `src/components/Background.tsx` → fond animé (canvas, étoiles)  
- `*Tab.tsx` → tables par entité (Invoices, Bills, Payments, Items, Accounts…)

</details>

---

<details>
<summary>⚙️ SYNCHRONISATION FIABLE ET COMPLÈTE</summary>

### 🔁 Pipeline
1. UI → Edge Function `sync_qbo_<entity>`  
2. Edge Function → pagination complète QBO → upsert Supabase `(company_id,qbo_id)`  
3. UI → rafraîchit table + compteurs + logs  

### 🔒 Intégrité
- `on conflict (company_id,qbo_id) do update`  
- Idempotence UI → aucune duplication visuelle  
- FK clients/fournisseurs/paiements validées  

### 🧮 Compteurs
- Afficher `Supabase:<count>` | `QBO:<count>` | `Table:<count>` | `Δ:<diff>`  
- Dernière sync ISO visible dans chaque onglet  

</details>

---

<details>
<summary>📊 RÉCONCILIATION AUTOMATIQUE (ANTI-ÉCARTS)</summary>

- **GL balances** : Trial Balance QBO = Supabase → `[RECON_GL_MISMATCH]`  
- **AR/AP** : Clients / Fournisseurs → `[RECON_AR/AP_MISMATCH]`  
- **Paiements orphelins** → `[PAYMENT_ORPHAN]`  
- **FK manquantes** → `[FK_MISSING]`  
- Logs dans `SyncConsole` → **SUCCESS / FAILED WITH ISSUES**

</details>

---

<details>
<summary>🧰 TABLEAUX ULTRA FLUIDES (UX COMPLÈTE)</summary>

### 🔍 Filtres par colonne
- Inline dans l’en-tête  
- Types auto : texte, numérique, montant, date  
- Debounce 300 ms + bouton Reset  

### 🔽 Tri multi-colonnes
- Clic = asc/desc, Shift+clic = secondaire  
- Dates → timestamp (`America/Toronto`)  
- Tri numérique réel  

### ↔️ Redimensionnement
- Poignée par colonne, min 80 px / max 600 px  
- Scroll horizontal fluide, virtualisation > 1 000 lignes  

### 🧩 Colonnes visibles / ordre
- Menu visibilité + drag-drop d’ordre  
- “Reset layout” = état par défaut  

### 💾 Persistance d’état
Stockage local (ou Supabase si multi-device) :

```json
ui:<company_id>:<entity>{
  "columns": { "order": [], "widths": {}, "visibility": {} },
  "sorting": [{ "id": "txn_date", "desc": true }],
  "filters": { "customer_name": { "type": "text", "value": "smith" } },
  "page": { "index": 0, "size": 50 }
}
🧱 Exemple d’upsert
sql
Copy code
insert into invoices (company_id,qbo_id,doc_number,txn_date,customer_id,amount)
values (...)
on conflict (company_id,qbo_id) do update set amount = excluded.amount;
📦 Exemple de réponse sync
json
Copy code
{
  "entity": "invoices",
  "total_qbo": 12458,
  "total_supabase_after": 12458,
  "delta": 0,
  "errors": []
}
</details>
<details> <summary>✅ ACCEPTANCE & QA RUNBOOK</summary>
1️⃣ Full sync → Δ = 0
2️⃣ No change → 0 upsert
3️⃣ Ajout facture/paiement → Δ = 0 après resync
4️⃣ Tri, filtres, resize OK
5️⃣ 10k lignes → fluide

Librairies : @tanstack/react-table, @tanstack/react-virtual, dayjs/luxon

</details> </details>
<!-- 🚧 Fin du PROMPT 1 🚧 -->
<details> <summary>🧠 <strong>PROMPT 2 — INTÉGRATION COMPLÈTE DES TABLES & RÉCONCILIATION DYNAMIQUE</strong></summary>
🎯 OBJECTIF
Brancher EntityCounters et EnhancedDataTable dans tous les onglets

Implémenter la réconciliation dynamique via les Edge Functions Supabase

Finaliser la pagination QBO

Vérifier FK et orphelins

Cohérence temps réel QBO ↔ Supabase ↔ UI

<details> <summary>🧩 INTÉGRATION DES COMPOSANTS PAR ONGLET</summary>
À intégrer dans :
InvoicesTab.tsx, BillsTab.tsx, PaymentsTab.tsx, ItemsTab.tsx, AccountsTab.tsx, CustomersTab.tsx, VendorsTab.tsx, TransactionsTab.tsx

Exemple :

tsx
Copy code
<EntityCounters entity="invoices" />
<EnhancedDataTable entity="invoices" data={invoicesData} />
</details>
<details> <summary>🔁 PAGINATION QBO & EDGE FUNCTIONS</summary>
Pagination complète avec backoff 429 + retour JSON enrichi :

json
Copy code
{
  "entity": "invoices",
  "total_qbo": 12458,
  "total_supabase_after": 12458,
  "delta": 0,
  "duration_ms": 9823,
  "errors": [],
  "reconciliation": {
    "gl_mismatch": [],
    "ar_mismatch": [],
    "ap_mismatch": [],
    "payment_orphan": [],
    "fk_missing": []
  }
}
Stocker dans sync_status : entity, total_qbo, total_supabase_after, delta, reconciliation (jsonb), errors[], started_at, ended_at.
SyncConsole → afficher dynamiquement ✅/⚠️/🔥 selon résultats.

</details>
<details> <summary>🧠 RÉCONCILIATION AUTOMATIQUE</summary>
GL : débits/crédits Supabase = Trial Balance QBO

AR/AP : factures − paiements = solde QBO

Payments : chaque ligne référence Invoice/Bill

FK : loguer manquantes, pas de crash

Δ : cohérence QBO vs Supabase

Exemples :

scss
Copy code
⚠️ 2 paiements orphelins (PAYMENT_ORPHAN)
⚠️ 3 factures sans client (FK_MISSING)
🔥 Solde AR incohérent (RECON_AR_MISMATCH)
</details>
<details> <summary>🧩 VALIDATION DES FK & ORPHELINS</summary>
FK principales :

invoice.customer_id → customers.id

bill.vendor_id → vendors.id

payment.invoice_id → invoices.id

payment.vendor_id → vendors.id

transaction.account_id → accounts.id

Détection automatique → reconciliation.fk_missing / reconciliation.payment_orphan
Remédiation optionnelle : fonction “fix orphelins”.

</details>
<details> <summary>📊 VALIDATION & ACCEPTANCE</summary>
Domaine	Attente	Résultat
Integration	Tous les onglets branchés	✅
Pagination	Aucune perte de pages	✅
Réconciliation	GL/AR/AP cohérents	✅
FK	0 manquante	✅
Orphelins	0 paiement orphelin	✅
UI	60 fps, persistance OK	✅

</details>
<details> <summary>🧠 BONUS – TESTS RAPIDES À AUTOMATISER</summary>
1️⃣ Sync Invoices → Δ = 0
2️⃣ Sync Bills → 0 mismatch
3️⃣ Ajouter paiement → delta ajusté
4️⃣ Supprimer client → FK_MISSING détecté
5️⃣ Tester tri/filtres/resize → persistants

</details> </details> ```
