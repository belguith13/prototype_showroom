# HEARTBEAT.md — Alex

Exécute cette séquence à chaque déclenchement (planifié : mar–sam, toutes les heures 10h–19h CET).

## Règle de silence

Si la boîte est vide et qu'il n'y a aucune relance en attente : terminer silencieusement.
Ne pas créer de ticket inutile.

## 1. Lire les nouveaux e-mails via `email-handler`

- Appeler `fetch_unread()` sur contact@bellacucina.fr
- Si aucun nouvel e-mail : passer directement à l'étape 3

## 2. Traiter chaque e-mail non lu

Pour chaque e-mail :

**a. Classifier** (prospect_nouveau / client_existant / fournisseur / spam_irrelevant / autre)

**b. Si `prospect_nouveau` :**
1. Extraire les données structurées (nom, email, tél, projet, surface, budget, délai, urgence)
2. Vérifier doublon CRM via `crm-lookup` → `lookup_contact()`
3. Si nouveau contact : créer fiche via `crm-lookup` → `create_prospect()`
4. Rédiger draft de réponse (template §4 du skill email-handler)
5. Créer ticket Paperclip :
   - Titre : "Nouveau lead — [NOM_CONTACT]"
   - Assigné à : Sophie
   - Priorité : haute si urgence=haute, normale sinon
   - Description : données extraites + draft en commentaire

**c. Si `client_existant` :**
- Identifier le dossier CRM concerné
- Créer ticket d'alerte assigné à la commerciale responsable

**d. Si `spam_irrelevant` :** ignorer, ne pas créer de ticket

**e. Si `autre` et sujet potentiellement important :** escalader à Sophie via ticket

## 3. Vérifier les relances en attente

Consulter le CRM : prospects avec statut "prospect contacté" depuis > 3 jours

| Délai | Action |
|-------|--------|
| J+3   | Préparer draft relance douce + créer ticket assigné à Sophie |
| J+5   | Créer ticket d'alerte "Relance J+5 — [NOM]" assigné à Sophie |
| J+10  | Escalader à Max si ticket J+5 non traité |
