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

- `src/pages/ProjectBudgets.tsx` → Page principale du dashboard  
- `src/components/HeaderCompanySelector.tsx` → Sélecteur de compagnie  
- `src/components/SyncConsole.tsx` → Logs et progression  
- `src/components/QuickBooksSettingsTab.tsx` → Paramètres et sync  
- `src/components/Background.tsx` → Fond animé (canvas, étoiles)  
- `*Tab.tsx` → Tables par entité (Invoices, Bills, Payments, Items, Accounts…)  

</details>

---

<details>
<summary>⚙️ SYNCHRONISATION FIABLE ET COMPLÈTE</summary>

### 🔁 Pipeline  
1. UI → Edge Function `sync_qbo_<entity>`  
2. Edge Function → pagination complète QBO → upsert Supabase `(company_id, qbo_id)`  
3. UI → rafraîchit table + compteurs + logs  

### 🔒 Intégrité  
- `on conflict (company_id, qbo_id) do update`  
- Idempotence UI → aucune duplication visuelle  
- FK clients/fournisseurs/paiements validées  

### 🧮 Compteurs  
- Afficher `Supabase: <count>` | `QBO: <count>` | `Table: <count>` | `Δ: <diff>`  
- Dernière sync ISO, visible dans chaque onglet  

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
insert into invoices (company_id, qbo_id, doc_number, txn_date, customer_id, amount)
values (...)
on conflict (company_id, qbo_id) do update set amount = excluded.amount;
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
<details> <summary>✅ ACCEPTANCE & QA RUNBOOK</summary>
🔬 Tests de synchronisation
1️⃣ Full sync → Δ = 0, soldes cohérents
2️⃣ No change → 0 upsert
3️⃣ Ajout facture/paiement → Δ = 0 après resync
4️⃣ Tri dates/nombres OK
5️⃣ Filtres texte/montant/date OK
6️⃣ Resize persisté OK
7️⃣ Masquage/ordre OK
8️⃣ 10k lignes → fluide

📦 Librairies
@tanstack/react-table, @tanstack/react-virtual, dayjs/luxon, Intl.NumberFormat('fr-CA')

</details> </details>
<details> <summary>🧠 <strong>PROMPT 2 — INTÉGRATION COMPLÈTE DES TABLES & RÉCONCILIATION DYNAMIQUE</strong></summary>
🎯 OBJECTIF
Terminer la mise en place du système complet :

Brancher les composants EntityCounters et EnhancedDataTable dans tous les onglets principaux

Implémenter la réconciliation dynamique via les Edge Functions Supabase

Finaliser la pagination QBO complète

Vérifier les FK et orphelins

Assurer la cohérence temps réel entre QBO ↔ Supabase ↔ UI

<details> <summary>🧩 INTÉGRATION DES COMPOSANTS PAR ONGLET</summary>
🔗 À intégrer :
InvoicesTab.tsx

BillsTab.tsx

PaymentsTab.tsx

ItemsTab.tsx

AccountsTab.tsx

CustomersTab.tsx

VendorsTab.tsx

TransactionsTab.tsx

Chaque onglet doit :

Importer EntityCounters et EnhancedDataTable

Charger les données Supabase de l’entité correspondante (select * from invoices, etc.)

Récupérer les compteurs (Supabase / QBO / Δ / Dernière sync) depuis sync_status

Injecter ces compteurs dans le bandeau (EntityCounters)

Rendre le tableau avec filtres, tri et persistance

Exemple :

tsx
Copy code
<EntityCounters entity="invoices" />
<EnhancedDataTable entity="invoices" data={invoicesData} />
Les colonnes sont générées dynamiquement selon le schéma Supabase ou configurées via un mapping local.

</details>
<details> <summary>🔁 PAGINATION QBO & EDGE FUNCTIONS</summary>
📘 Implémentation complète
Pagination via startPosition / maxResults jusqu’à QueryResponse.totalCount atteint

Backoff exponentiel en cas de 429

Retour JSON enrichi :

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
🧮 Table sync_status
entity

total_qbo

total_supabase_after

delta

reconciliation (jsonb)

started_at

ended_at

errors[]

🧩 SyncConsole
Lire reconciliation et afficher dynamiquement :

✅ Aucun écart

⚠️ 2 FK manquantes / 1 paiement orphelin

🔥 4 GL mismatches

</details>
<details> <summary>🧠 RÉCONCILIATION AUTOMATIQUE</summary>
💡 Vérifications automatiques (Edge Function)
GL (Comptes) : somme des débits/crédits Supabase = Trial Balance QBO

AR/AP : factures / bills ouverts - paiements = solde QBO

Payments : chaque PaymentLine doit référencer un Invoice ou Bill

FK : loguer toute FK manquante sans crash

Δ : comparer nombre de lignes QBO vs Supabase

Les anomalies sont ajoutées à reconciliation et affichées dans SyncConsole.

Exemples :

scss
Copy code
⚠️ 2 paiements orphelins détectés (PAYMENT_ORPHAN)
⚠️ 3 factures sans client associé (FK_MISSING)
🔥 Solde AR incohérent (RECON_AR_MISMATCH)
</details>
<details> <summary>🧩 VALIDATION DES FK & ORPHELINS</summary>
🔗 FK à appliquer dans Supabase
invoice.customer_id → customers.id

bill.vendor_id → vendors.id

payment.invoice_id → invoices.id

payment.vendor_id → vendors.id

transaction.account_id → accounts.id

🔎 Détection automatique
FK inexistante → reconciliation.fk_missing

Paiement sans facture → reconciliation.payment_orphan

🛠️ Remédiation optionnelle
Fonction “fix orphelins” : désactiver ou associer à un placeholder.

</details>
<details> <summary>📊 VALIDATION & ACCEPTANCE</summary>
Domaine	Attente	Résultat
Integration	Tous les onglets utilisent EntityCounters & EnhancedDataTable	✅
Pagination	Toutes les pages QBO traitées sans perte	✅
Réconciliation	GL/AR/AP cohérents avec QBO	✅
FK	Zéro FK manquante après sync complète	✅
Orphelins	Zéro paiement orphelin	✅
UI	Tables fluides, persistantes et rapides (60 fps)	✅

</details>
<details> <summary>🧠 BONUS – TESTS RAPIDES À AUTOMATISER</summary>
1️⃣ Sync Invoices → vérifier EntityCounters affiche Δ = 0
2️⃣ Sync Bills → vérifier SyncConsole affiche 0 mismatch
3️⃣ Ajouter un paiement → vérifier delta ajusté
4️⃣ Supprimer un client → FK_MISSING détecté
5️⃣ Tester tri, filtres, resize → persistance confirmée

</details> </details> ```
