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

<details>
<summary>🧠 PROMPT 3 — À DÉFINIR (NOUVELLE SECTION)</summary>

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
