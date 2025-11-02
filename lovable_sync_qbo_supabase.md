# 🧠 PROMPT LOVABLE — QBO ↔ SUPABASE SYNC + TABLEAUX ULTRA FLUIDES

## 🎯 OBJECTIF GLOBAL
Assurer une synchronisation **QuickBooks ↔ Supabase** complète, fiable et idempotente, tout en offrant des **tableaux ultra fluides** avec :
- Filtres par colonne (texte, nombre, montant, date)
- Tri ascendant/descendant (multi-colonnes, dates incluses)
- Redimensionnement fluide des colonnes
- Réorganisation et visibilité dynamique
- Persistance d’état (layout, tri, filtres, tailles, visibilité)
- Performance stable même sur 10 000 lignes

L’ensemble du système doit fonctionner **sans duplication**, **sans perte de données**, et fournir des **comptes exacts** entre QBO et Supabase.

---

## 🗺️ STRUCTURE CONNUE DE L’APPLICATION

### 📁 Pages et composants principaux
- `src/pages/ProjectBudgets.tsx` → page principale du dashboard  
- `src/components/HeaderCompanySelector.tsx` → sélection de compagnie QBO  
- `src/components/SyncConsole.tsx` → logs et progression de synchronisation  
- `src/components/QuickBooksSettingsTab.tsx` → onglet de configuration et sync  
- `src/components/Background.tsx` → fond animé (canvas, étoiles, nuages)  
- `*Tab.tsx` (Invoices, Bills, Payments, Items, Accounts, etc.) → tableaux par entité  

---

## ⚙️ SYNCHRONISATION COMPLÈTE ET FIABLE

### 🔁 Pipeline de synchronisation
1. **UI** → bouton “Sync” déclenche une Edge Function spécifique :  
   `sync_qbo_<entity>` avec payload `{ company_id, entity }`
2. **Edge Function Supabase**
   - Récupère **toutes les pages QBO** (pagination jusqu’à `hasMore=false`)
   - Applique **upsert Supabase** sur clé `(company_id, qbo_id)`
   - Écrit un log `sync_status` avec les compteurs et erreurs
3. **UI**
   - Recharge le tableau
   - Met à jour les **compteurs bandeau**
   - Affiche progression, logs et résumé dans `SyncConsole`

### ✅ Intégrité et idempotence
- `on conflict (company_id, qbo_id) do update`
- Pas de duplicata UI (composants ajoutés une seule fois)
- FK clients/fournisseurs/paiements valides
- Aucune transaction en double ni orpheline

### 🧮 Compteurs automatiques (en haut de chaque onglet)
Supabase : <count_supabase> | QBO : <count_qbo> | Tableau : <rows_displayed> | Δ : <count_qbo - count_supabase>
Dernière sync : <ISO datetime> | Entité : <invoices|bills|...>

markdown
Copy code
- `count_supabase` → `select('*', { count:'exact', head:true })`
- `count_qbo` → totalCount/pagination QBO
- Rafraîchir après sync ou clic “Rafraîchir les compteurs”

---

## 📊 RÉCONCILIATION AUTOMATIQUE (ANTI-ÉCARTS)
1. **Comptes GL** → soldes Supabase = Trial Balance QBO  
   → sinon `[RECON_GL_MISMATCH] {account, expected, got}`
2. **Clients/Fournisseurs** → AR/AP Supabase = QBO  
   → sinon `[RECON_AR/AP_MISMATCH]`
3. **Paiements orphelins** → `[PAYMENT_ORPHAN]`
4. **FK manquantes** → `[FK_MISSING]`
5. **Écarts volumétriques** → `[DELTA_WARNING]`

> Tous ces logs apparaissent dans `SyncConsole` avec le statut final :
> **SUCCESS** ou **FAILED WITH ISSUES**

---

## 🧰 TABLEAUX ULTRA FLUIDES (TOUS LES ONGLETS)

### 🔍 1. Filtres par colonne
- Filtres **inline** dans l’en-tête (un champ par colonne)
- Types auto-détectés :
  - **Texte** (fuzzy, insensible à la casse)
  - **Nombre/Montant** (=, ≠, >, ≥, <, ≤)
  - **Dates** (=, ≠, avant, après, entre)
- **Debounce 300ms** sur saisie
- Boutons : “Clear column” + “Reset filters”

### 🔽 2. Tri multi-colonnes (asc/desc)
- Clic simple = tri asc/desc  
- Shift+clic = tri secondaire  
- Dates → converties en timestamps (`YYYY-MM-DD`, `DD/MM/YYYY`, `MM/DD/YYYY`)  
- Tri robuste timezone : `America/Toronto`  
- Tri numérique = comparaison réelle (pas string)

### ↔️ 3. Redimensionnement fluide
- Poignée de resize sur chaque colonne
- Largeur min/max (80px à 600px)
- Scroll horizontal préservé
- Aucune cassure de layout
- Virtualisation active pour 1000+ lignes

### 🧩 4. Colonnes visibles et ordre
- Menu “Colonnes” → cocher/masquer, drag/drop pour réordonner
- Bouton “Reset layout” = état par défaut

### 💾 5. Persistance de l’état (localStorage / Supabase)
- Clé :  
ui:<company_id>:<entity>:{
columns:{order:[], widths:{}, visibility:{}},
sorting:[{id,desc}],
filters:{<columnId>:{type,value}},
page:{index,size}
}

markdown
Copy code
- Rechargé à chaque ouverture d’onglet
- Tolérant si colonne supprimée

### ⚡ 6. Performance et fluidité
- Virtualisation (TanStack Table + react-virtual)
- Pagination UI (25/50/100 lignes)
- Mémoisation des calculs de tri/filtre
- Aucun re-render inutile, 60 fps garanti

### ♿ 7. Accessibilité et UX
- `aria-sort` et tooltips sur icônes
- Focus clavier sur en-têtes
- Raccourci `/` = focus filtre global

---

## 🧮 RÈGLES DE RECONCILIATION TECHNIQUES

### 📘 SQL Upsert exemple (Invoices)
```sql
insert into invoices (company_id, qbo_id, doc_number, txn_date, customer_id, amount, qbo_last_updated_at, payload)
values (...)
on conflict (company_id, qbo_id) do update
set doc_number = excluded.doc_number,
  txn_date = excluded.txn_date,
  customer_id = excluded.customer_id,
  amount = excluded.amount,
  qbo_last_updated_at = excluded.qbo_last_updated_at,
  payload = excluded.payload;
🧾 Edge Function Response
json
Copy code
{
  "entity": "invoices",
  "total_qbo": 12458,
  "total_supabase_after": 12458,
  "delta": 0,
  "duration_ms": 9823,
  "errors": []
}
🧠 ACCEPTANCE CRITERIA (GLOBAL)
Domaine	Critère	Résultat attendu
Sync	Delta = 0, soldes GL = QBO, zéro doublon/orphelin	✅
Idempotence	Relancer la sync = 0 upsert	✅
UI	Aucun bouton supprimé ni ajouté deux fois	✅
Tables	Filtres colonne, tri dates/nombres correct, resize fluide	✅
Persistance	Layout conservé après reload	✅
Performance	10 000 lignes → fluide (virtualisation active)	✅
Logs	Résumé clair + erreurs dans SyncConsole	✅

🧪 TESTS RAPIDES (RUNBOOK QA)
Run 1 – Full sync → Delta 0, tout OK

Run 2 – Aucun changement QBO → 0 upsert (idempotence)

Run 3 – Ajout d’une facture + paiement QBO → Delta 0 après resync

Tri → colonnes txn_date et due_date triables asc/desc

Filtres → texte (“Smith”), montant (> 1000), date (entre)

Redimensionnement → modifier 3 colonnes, recharger → largeur persistée

Colonnes visibles → masquer/afficher, reset layout OK

Performance → scroller 10 k lignes → aucune latence

🧩 NOTES TECHNIQUES D’IMPLÉMENTATION
Librairies :

@tanstack/react-table (filtres/tri/resize)

react-virtual (virtualisation)

dayjs ou luxon (dates, TZ)

Intl.NumberFormat('fr-CA') (montants)

API QBO :

Gestion maxResults et startPosition

Backoff 429 (1 s → 2 s → 4 s)

Edge Function :

Batch upsert 200–500 lignes

Stocke qbo_last_updated_at pour différentiel

Table sync_status = journal complet de chaque sync

Erreurs :

Loggées avec {entity, id, message, page} dans sync_status.errors

💡 CONCLUSION
L’application doit offrir :

Une synchronisation exacte entre QBO et Supabase

Une interface fluide et persistente

Une traçabilité totale via SyncConsole

Une performance irréprochable (60 fps, 10 k lignes)

Des tableaux professionnels comparables à QuickBooks Desktop ou Airtable
