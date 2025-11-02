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
- Aucune duplication, comptes QBO/Supabase exactement égaux

---

<details>
<summary>📁 STRUCTURE DE L’APPLICATION</summary>

- `src/pages/ProjectBudgets.tsx` : page principale du dashboard  
- `src/components/HeaderCompanySelector.tsx` : sélecteur de compagnie  
- `src/components/SyncConsole.tsx` : logs et progression  
- `src/components/QuickBooksSettingsTab.tsx` : paramètres et sync  
- `src/components/Background.tsx` : fond animé (canvas, étoiles)  
- `*Tab.tsx` : tables par entité (Invoices, Bills, Payments, Items, Accounts…)  

</details>

---

<details>
<summary>⚙️ SYNCHRONISATION FIABLE ET COMPLÈTE</summary>

**Pipeline**  
1. UI → Edge Function `sync_qbo_<entity>`  
2. Edge Function → pagination complète QBO → upsert Supabase `(company_id, qbo_id)`  
3. UI → rafraîchit table + compteurs + logs  

**Intégrité**  
- `on conflict (company_id, qbo_id) do update`  
- Idempotence UI → pas de duplication visuelle  
- FK clients/fournisseurs/paiements valides  

**Compteurs**  

</details>

---

<details>
<summary>📊 RÉCONCILIATION AUTOMATIQUE (ANTI-ÉCARTS)</summary>

- **GL balances** : Trial Balance QBO = Supabase → `[RECON_GL_MISMATCH]`  
- **AR/AP** : Clients/Fournisseurs → `[RECON_AR/AP_MISMATCH]`  
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
- Debounce 300 ms + Clear/Reset  

### 🔽 Tri multi-colonnes
- Clic = asc/desc, Shift+clic = secondaire  
- Dates → timestamp (`America/Toronto`)  
- Tri numérique réel  

### ↔️ Redimensionnement
- Poignée par colonne, min 80 px / max 600 px  
- Scroll horizontal fluide, virtualisation > 1 000 lignes  

### 🧩 Colonnes visibles / ordre
- Menu visibilité + drag-drop ordre  
- “Reset layout” = état par défaut  

### 💾 Persistance
Stockage local ou Supabase :
```json
ui:<company_id>:<entity>{
  "columns":{ "order":[], "widths":{}, "visibility":{} },
  "sorting":[{ "id":"txn_date","desc":true }],
  "filters":{ "customer_name":{ "type":"text","value":"smith" } },
  "page":{ "index":0,"size":50 }
}
insert into invoices (company_id, qbo_id, doc_number, txn_date, customer_id, amount)
values (...)
on conflict (company_id, qbo_id) do update set amount = excluded.amount;
{
 "entity":"invoices",
 "total_qbo":12458,
 "total_supabase_after":12458,
 "delta":0,
 "errors":[]
}
<details> <summary>✅ ACCEPTANCE & QA RUNBOOK</summary>

Sync Tests
1️⃣ Full sync → Δ 0, soldes OK
2️⃣ No change → 0 upsert
3️⃣ Ajout facture/paiement → Δ 0 après resync
4️⃣ Tri dates/nombres OK
5️⃣ Filtres texte/montant/date OK
6️⃣ Resize persisté OK
7️⃣ Masquage/ordre OK
8️⃣ 10 k lignes → fluide

Librairies

@tanstack/react-table, react-virtual, dayjs/luxon, Intl.NumberFormat('fr-CA')

</details>
</details>
<details> <summary>🧠 <strong>PROMPT 2 — (RÉSERVÉ POUR FONCTIONNALITÉS FUTURES)</strong></summary>

(Ajoute ici ton deuxième prompt Lovable : nouveaux modules, intégration ClockShark, tableaux analytiques, ou autres flows n8n / Supabase à venir.)

</details> ```
