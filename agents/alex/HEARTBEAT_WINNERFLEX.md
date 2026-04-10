# HEARTBEAT_WINNERFLEX.md — Alex

Exécute cette séquence à chaque déclenchement de la routine `winnerflex-sync`.
Test : toutes les heures. Production : tous les jours à 08h00 CET.

## Règle de silence

Si aucun changement détecté et aucune alerte à remonter : terminer silencieusement.
Ne pas créer de ticket inutile.

## 1. Vérifier la connexion WinnerFlex

- Appeler `test_connection()` via `winner-flex-lookup`
- Si erreur : créer ticket "Alerte — WinnerFlex inaccessible" assigné à Marc (PDG), stopper

## 2. Lister les projets actifs

- Appeler `list_projects()` pour récupérer tous les projets du shop
- Comparer avec la liste du ticket précédent (ou mémoriser via notes Paperclip)
- **Nouveaux projets détectés** → créer ticket "Nouveau projet WinnerFlex — [NOM]" assigné à Sophie

## 3. Pour chaque projet actif : vérifier les designs

- Appeler `get_project_designs(project_guid)` pour les projets modifiés récemment
- Détecter les designs avec `totalSalesPriceExVAT > 0` (devis chiffré disponible)
- **Nouveau design chiffré** → créer ticket "Devis disponible — [NOM_PROJET] — [MONTANT]€ HT"
  - Assigné à : commerciale responsable
  - Priorité : haute si montant > 20 000€

## 4. Vérifier les documents récents

- Appeler `get_project_documents(project_guid, type_guids=[DOC_TYPES["QUOTATION"], DOC_TYPES["SIGNED_CONTRACT"]])` pour les projets actifs
- **Nouveau devis (QUOTATION)** → ticket "Devis émis — [NOM_PROJET]" assigné à Sophie
- **Contrat signé (SIGNED_CONTRACT)** → ticket "Contrat signé — [NOM_PROJET]" assigné à Marc
  - Inclure le montant du design associé dans la description

## 5. Alertes pipeline

Vérifier dans la liste des projets :
- Projets sans design depuis > 7 jours → ticket "Alerte relance — [NOM_PROJET] sans devis" assigné à Sophie
- Projets sans activité depuis > 14 jours → escalader à Marc

## Résumé de fin d'exécution

Si des actions ont été prises, créer un ticket récapitulatif :
- Titre : "WinnerFlex Sync — [DATE] — [N] changements"
- Lister les projets mis à jour, nouveaux devis, contrats signés
