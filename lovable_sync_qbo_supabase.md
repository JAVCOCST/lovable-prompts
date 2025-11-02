<details> <summary>🧠 PROMPT 1 — QBO ↔ SUPABASE SYNC + TABLEAUX ULTRA FLUIDES</summary>
OBJECTIF GLOBAL

Synchroniser QuickBooks ↔ Supabase de façon fiable et idempotente, tout en offrant des tableaux ultra fluides :

Filtres par colonne (texte, nombre, montant, date)

Tri ascendant/descendant (multi-colonnes, dates incluses)

Redimensionnement fluide

Réorganisation et visibilité dynamique

Persistance d’état (layout, tri, filtres, tailles, visibilité)

Performance 60 fps jusqu’à 10 000 lignes

Aucune duplication — comptes QBO/Supabase identiques

<details> <summary>STRUCTURE DE L’APPLICATION</summary>

src/pages/ProjectBudgets.tsx → page principale du dashboard

src/components/HeaderCompanySelector.tsx → sélecteur de compagnie

src/components/SyncConsole.tsx → logs et progression

src/components/QuickBooksSettingsTab.tsx → paramètres et sync

src/components/Background.tsx → fond animé (canvas, étoiles)

*Tab.tsx → tables par entité (Invoices, Bills, Payments, Items, Accounts…)

</details> <details> <summary>SYNCHRONISATION FIABLE ET COMPLÈTE</summary>

Pipeline :

UI → Edge Function sync_qbo_<entity>

Edge Function → pagination complète QBO → upsert Supabase (company_id, qbo_id)

UI → rafraîchit table + compteurs + logs

Intégrité :

on conflict (company_id, qbo_id) do update

Idempotence UI → aucune duplication visuelle

FK clients/fournisseurs/paiements validées

Compteurs :

Afficher Supabase:<count> | QBO:<count> | Table:<count> | Δ:<diff>

Dernière sync ISO visible dans chaque onglet

</details> <details> <summary>RÉCONCILIATION AUTOMATIQUE</summary>

GL balances : Trial Balance QBO = Supabase → RECON_GL_MISMATCH

AR/AP : Clients / Fournisseurs → RECON_AR/AP_MISMATCH

Paiements orphelins → PAYMENT_ORPHAN

FK manquantes → FK_MISSING

Logs dans SyncConsole → SUCCESS / FAILED WITH ISSUES

</details> <details> <summary>TABLEAUX ULTRA FLUIDES</summary>

Filtres par colonne : inline dans l’en-tête, debounce 300 ms
Tri multi-colonnes : clic pour asc/desc, Shift+clic = secondaire
Redimensionnement : poignée par colonne, min 80 px / max 600 px
Colonnes visibles / ordre : menu visibilité + drag-drop
Persistance : localStorage ou Supabase si multi-device

</details> <details> <summary>ACCEPTANCE & QA RUNBOOK</summary>

Full sync → Δ = 0

No change → 0 upsert

Ajout facture/paiement → Δ = 0 après resync

Tri, filtres, resize OK

10 k lignes → fluide

Librairies : @tanstack/react-table, @tanstack/react-virtual, dayjs/luxon

</details> </details> <details> <summary>🧠 PROMPT 2 — INTÉGRATION COMPLÈTE DES TABLES ET RÉCONCILIATION DYNAMIQUE</summary>

OBJECTIF

Brancher EntityCounters et EnhancedDataTable dans tous les onglets

Implémenter la réconciliation dynamique via les Edge Functions Supabase

Finaliser la pagination QBO

Vérifier FK et orphelins

Cohérence temps réel entre QBO, Supabase et UI

<details> <summary>INTÉGRATION DES COMPOSANTS PAR ONGLET</summary>

À intégrer dans :
InvoicesTab.tsx, BillsTab.tsx, PaymentsTab.tsx, ItemsTab.tsx, AccountsTab.tsx, CustomersTab.tsx, VendorsTab.tsx, TransactionsTab.tsx

Exemple :
<EntityCounters entity="invoices" />
<EnhancedDataTable entity="invoices" data={invoicesData} />

</details> <details> <summary>PAGINATION QBO ET EDGE FUNCTIONS</summary>

Pagination complète avec backoff 429 et retour JSON enrichi.
Stocker dans sync_status : entity, total_qbo, total_supabase_after, delta, reconciliation (jsonb), errors[], started_at, ended_at.
SyncConsole → afficher dynamiquement ✅ ⚠️ 🔥 selon résultats.

</details> <details> <summary>RÉCONCILIATION AUTOMATIQUE</summary>

GL : débits/crédits Supabase = Trial Balance QBO
AR/AP : factures − paiements = solde QBO
Payments : chaque ligne référence Invoice/Bill
FK : loguer manquantes sans crash
Δ : cohérence QBO vs Supabase

Exemples :
⚠️ 2 paiements orphelins (PAYMENT_ORPHAN)
⚠️ 3 factures sans client (FK_MISSING)
🔥 Solde AR incohérent (RECON_AR_MISMATCH)

</details> <details> <summary>VALIDATION DES FK ET ORPHELINS</summary>

FK principales :
invoice.customer_id → customers.id
bill.vendor_id → vendors.id
payment.invoice_id → invoices.id
payment.vendor_id → vendors.id
transaction.account_id → accounts.id

Détection automatique : reconciliation.fk_missing / reconciliation.payment_orphan
Remédiation optionnelle : fonction “fix orphelins”

</details> <details> <summary>VALIDATION ET ACCEPTANCE</summary>

Tous les onglets branchés

Pagination complète sans perte

Réconciliation GL/AR/AP cohérente

0 FK manquante

0 paiement orphelin

UI fluide (60 fps)

</details> <details> <summary>TESTS RAPIDES À AUTOMATISER</summary>

Sync Invoices → Δ = 0

Sync Bills → 0 mismatch

Ajouter paiement → delta ajusté

Supprimer client → FK_MISSING détecté

Tester tri, filtres et resize → persistants

</details> </details>
<details> <summary>🧠 PROMPT 3 — CHARGEMENT GLOBAL + COMPTEURS MULTI-ONGLETS (RÉSOLUTION DÉSYNCHRONISATION)</summary>
OBJECTIF

Uniformiser et fiabiliser le chargement des données et des compteurs dans tous les onglets QuickBooks ↔ Supabase.
Corriger le problème où certains onglets (ex. Dépenses) affichent 0 résultat malgré des enregistrements existants.
Garantir la cohérence des compteurs Supabase / QBO / Table / Δ / Dernière Sync pour toutes les entités.

<details> <summary>STRUCTURE OU MODULES À IMPLÉMENTER</summary>

Nouveaux hooks et composants

useEntityData(entity: string) → centralise fetch, filtres, tri, pagination, counters.

EntityCounters.tsx → affiche les totaux (Supabase, QBO, Table, Δ, Dernière sync).

DebugPanel.tsx → panneau pliable de diagnostic (entity, company_id, total_supabase, total_qbo, delta, erreurs).

Normalisation du mapping entité → table

invoices → invoices

bills → bills

payments → payments

items → items

accounts → accounts

customers → customers

vendors → vendors

transactions → transactions

expenses_lines → vw_expense_lines (vue matérialisée combinant toutes les lignes de dépenses).

Fichiers à modifier

Tous les QuickBooks*Tab.tsx (Invoices, Bills, Payments, Items, Accounts, Customers, Vendors, Transactions, Expenses).

SyncConsole.tsx → relier fin de sync à refetch automatique des compteurs et tableaux.

Flux global

Détection du company_id actif (HeaderCompanySelector).

Chargement via useEntityData(entity) (fetch Supabase + fetch counters).

Affichage combiné : EntityCounters + EnhancedDataTable.

DebugPanel actif (pliable) pour vérification locale rapide.

Rechargement automatique après chaque sync ou changement de compagnie.

</details>
<details> <summary>LOGIQUE TECHNIQUE</summary>

Hook useEntityData(entity)

Inputs : entity, filters, sorting, page, size.

Étapes :

Récupère le company_id courant.

Résout la table réelle via VIEW_TO_QUERY[entity].

Fait un select('*', { count: 'exact' }) avec eq('company_id', company_id).

Applique les filtres colonne (texte, nombre, date).

Retourne { data, count, error }.

Fetch parallèle de sync_status pour total_qbo, total_supabase_after, delta, ended_at.

Return : { data, total, counters, error, isLoading, refetch }.

Gestion du mapping de vue

export const VIEW_TO_QUERY = {
  invoices: { table: "invoices" },
  bills: { table: "bills" },
  payments: { table: "payments" },
  items: { table: "items" },
  accounts: { table: "accounts" },
  customers: { table: "customers" },
  vendors: { table: "vendors" },
  transactions: { table: "transactions" },
  expenses_lines: { table: "vw_expense_lines" },
} as const;


Refetch automatique

Sur SyncConsole → une fois la sync terminée → refetch() de tous les onglets montés.

Sur changement de company_id → rechargement global des onglets visibles.

Gestion des cas “0 résultat”

Si data.length === 0 mais total_supabase_after > 0 → afficher un badge “Filtres actifs” + bouton Reset filtres.

Bouton “Tester requête brute” → refait un fetch sans filtre, log console.

Uniformisation des colonnes (exemple Dépenses)

Colonnes par défaut :

Date (txn_date)

Fournisseur (vendor_name)

Compte (account_name)

Item (item_name)

Quantité (qty)

Montant unitaire (unit_price)

Total (amount)

Tri par défaut : txn_date DESC.

Compteurs

Supabase = count(*) sur la table.

QBO = champ total_qbo de sync_status.

Δ = total_qbo - total_supabase_after.

Dernière sync = ended_at.

</details>
<details> <summary>VALIDATION ET TESTS</summary>

Tests unitaires par onglet

Les données s’affichent pour chaque entité (data.length > 0).

Les compteurs Supabase/QBO/Table sont cohérents.

Δ = 0 après une sync complète.

Changement de compagnie → data + counters mis à jour.

Sync QuickBooks → refetch automatique des compteurs.

Aucun onglet n’affiche “vide” si des données existent.

Dépenses → 828 lignes visibles (pas de perte).

Critères d’acceptation

Tous les onglets utilisent le même hook (useEntityData).

Les compteurs affichent les valeurs exactes Supabase + QBO.

Aucun “faux 0 résultat”.

Le rafraîchissement est instantané après sync ou changement de compagnie.

Performance maintenue à 60fps (virtualisation active).

</details> </details>
<details>
<summary>🧠 PROMPT 4 — VALIDATION DÉPENSES / BUDGET / TRANSACTIONS + RÉCONCILIATION FILTRÉE PAR PROJET</summary>

## OBJECTIF
Corriger les incohérences d’affichage et de synchronisation dans les modules **Dépenses**, **Budget** et **Transactions**.  
Assurer que :
- les lignes d’items de dépenses apparaissent correctement dans le tableau,  
- les sections de diagnostic affichent le `QBO count`,  
- le **budget** affiche tous les comptes (revenus, dépenses, heures, profit net),  
- le **filtre projet** agit sur toutes les entités (factures, dépenses, heures, devis, etc.),  
- et que la **synchronisation des transactions** fonctionne sans doublon ni erreur silencieuse.

---

<details>
<summary>STRUCTURE OU MODULES À REVALIDER</summary>

### 🔹 Dépenses
**Fichier :** `ExpensesTab.tsx` (ou `QuickBooksExpensesTab.tsx`)

- Vérifier que le hook `useEntityData('expenses_lines')` pointe bien vers la vue matérialisée `vw_expense_lines`.  
- Corriger le mapping dans le `entityMap` global (`expenses_lines → vw_expense_lines`).  
- Relier le diagnostic (`EntityCounters`) pour afficher :
Supabase:<count> | QBO:<count> | Table:<count> | Δ:<diff>

markdown
Copy code
- Rafraîchir automatiquement après chaque synchronisation.  
- Vérifier la cohérence du `company_id` et du `project_id` dans les requêtes Supabase.

---

### 🔹 Budget
**Fichier :** `src/pages/ProjectBudgets.tsx` → onglet “Budget”

- Le tableau doit afficher **tous les comptes du plan comptable QBO** :
- Revenus (`account_type = INCOME`)
- Dépenses (`account_type = EXPENSE`)
- Heures travaillées (via `time_activities`)
- Ajouter une ligne **profit net = revenus − dépenses − main-d’œuvre**.
- Brancher le **dropdown projet** (`SelectTrigger`) pour filtrer :
- factures, dépenses, heures, devis, paiements.
- S’assurer que le calcul agrégé est dynamique selon le projet sélectionné.

---

### 🔹 Transactions
**Fichier :** `TransactionsTab.tsx`

- Vérifier le comportement du bouton **Sync** :
- Utiliser `on conflict (company_id, qbo_id) do update` pour éviter les doublons.
- Journaliser les erreurs dans `sync_status` (`errors[]`).
- Mettre à jour automatiquement les compteurs après la sync :
  ```
  Supabase:<count> | QBO:<count> | Table:<count> | Δ:<diff>
  ```
- Ajouter un message visuel si une erreur est détectée ou si la sync échoue partiellement.

---

### 🔹 Diagnostic global
**Composant :** `DebugPanel.tsx` ou `DiagnosticPanel.tsx`

- Centraliser les compteurs `QBO`, `Supabase`, `Table`, `Δ`.
- Requêter ces compteurs pour **chaque entité** (invoices, bills, payments, items, accounts, transactions, expenses_lines).
- Rafraîchir les données après chaque `sync_qbo_<entity>`.

---

</details>

<details>
<summary>LOGIQUE TECHNIQUE</summary>

#### 1. **Lignes de dépenses**
- Requêter `vw_expense_lines` (join `bills`, `vendors`, `accounts`, `projects`).
- Mapper correctement les FK :
- `expense_line.account_id → accounts.id`
- `expense_line.project_id → projects.id`
- Vérifier que la pagination fonctionne (pas de limite de 100 par défaut).

#### 2. **Compteurs et Diagnostic**
- Chaque onglet doit charger les compteurs avec :
```ts
const { totalQBO, totalSupabase, totalUI, delta } = useEntityCounters(entity);
Les compteurs doivent refléter le dernier sync_status enregistré.

3. Budget
Requêter :

accounts (revenus/dépenses)

transactions groupées par account_id

time_activities groupées par projet

Générer une table consolidée :

Compte	Type	Montant	Heures	Total

Ajouter un résumé total en bas du tableau.

4. Filtre projet
Le SelectTrigger de projet doit :

passer project_id à tous les hooks useEntityData(entity) concernés,

rafraîchir automatiquement les tables et compteurs concernés.

5. Transactions Sync
Utiliser un traitement idempotent :

ts
Copy code
supabase.from('transactions')
  .upsert(data, { onConflict: 'company_id,qbo_id' })
Logger les erreurs et l’état final (SUCCESS, FAILED_WITH_ISSUES).

Ajouter un toast ou badge visuel pour avertir en cas de désynchronisation.

</details>
<details> <summary>VALIDATION ET TESTS</summary>
 Les lignes de dépenses s’affichent (aucun tableau vide si data Supabase).

 Le diagnostic affiche bien le compteur QBO count.

 Le budget montre tous les comptes et calcule le profit net.

 Le filtre projet actualise toutes les données reliées (factures, dépenses, heures, devis).

 La sync des transactions ne crée aucun doublon (Δ = 0 après resync).

 Les compteurs QBO/Supabase/Table/Δ sont identiques dans chaque onglet.

 Les erreurs sont correctement journalisées et visibles dans le DebugPanel.

 Performance fluide (aucun freeze à 10k lignes).

</details> </details>

<details>
<summary>🧠 PROMPT 4 — À DÉFINIR (NOUVELLE SECTION)</summary>

## OBJECTIF
[Décris ici le but général du prochain module, exemple : monitoring, logs, auto-heal, etc.]

<details>
<summary>STRUCTURE OU MODULES À IMPLÉMENTER</summary>

- [Liste des fichiers ou composants]
- [Flux ou logique à suivre]

</details>

<details>
<summary>LOGIQUE TECHNIQUE</summary>

- [Description des étapes principales à exécuter]
- [Fonctions clés à coder]
- [Validation à effectuer]

</details>

<details>
<summary>VALIDATION ET TESTS</summary>

- [Tests à exécuter]
- [Critères d’acceptation]

</details>

</details>
