# Routine : Vérification e-mails entrants — toutes les heures (heures ouvrées)

schedule: "0 10-19 * * 2-6"   # cron — mar-sam, toutes les heures de 10h à 19h
agent: alex_sales_assistant
trigger: schedule

## Instructions pour Alex

À chaque déclenchement, tu dois :

1. **Lire les nouveaux e-mails** de contact@bellacucina.fr
   - Utiliser `email-handler` → fonction `fetch_unread()`
   - Si aucun nouvel e-mail : terminer la routine silencieusement (pas de ticket)

2. **Pour chaque e-mail non lu** :
   a. Classifier (prospect_nouveau / client_existant / fournisseur / spam / autre)
   b. Si `prospect_nouveau` :
      - Extraire les données structurées (§3 du skill email-handler)
      - Vérifier si le contact existe déjà dans le CRM via `crm-lookup`
      - Si nouveau : créer la fiche CRM
      - Rédiger le draft de réponse
      - Créer un ticket Paperclip assigné à Sophie
   c. Si `client_existant` :
      - Identifier le dossier concerné dans le CRM
      - Alerter la commerciale responsable via ticket

3. **Vérifier les relances en attente** :
   - Consulter le CRM : prospects avec statut "prospect contacté" depuis > 3 jours
   - Pour chaque cas : préparer un draft de relance et créer un ticket

## Règle de silence
Si la routine tourne en dehors des horaires showroom (lundi, dimanche)
ou si la boîte est vide : ne rien poster, ne pas créer de ticket inutile.
