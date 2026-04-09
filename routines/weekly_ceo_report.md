# Routine : Rapport hebdomadaire CEO — chaque lundi 08h00

schedule: "0 8 * * 1"   # cron — lundi 08h00
agent: max_ceo_assistant
trigger: schedule

## Instructions pour Max

À chaque déclenchement de cette routine, tu dois :

1. **Consulter le CRM** via le skill `crm-lookup`
   - Compter les nouveaux prospects créés cette semaine
   - Compter les RDV planifiés et tenus
   - Compter les devis envoyés
   - Compter les commandes signées + somme du CA
   - Identifier les leads sans activité depuis > 5 jours

2. **Lire la boîte e-mail team@bellacucina.fr** via `email-handler`
   - Vérifier s'il y a des e-mails non traités importants

3. **Rédiger le rapport hebdomadaire** et le poster en commentaire
   sur le ticket de rapport de Marc (ticket récurrent "Rapport CEO")

### Format du rapport

```
Bonjour Marc,

Rapport semaine [N] — [DATE]

CHIFFRES CLÉ
• Nouveaux leads : [N]
• RDV tenus : [N]
• Devis envoyés : [N]
• Commandes signées : [N] — [CA €] TTC
• CA cumulé mois : [X €] / objectif [Y €] ([Z]%)

PIPELINE ACTIF
[Liste top 5 deals chauds avec statut et montant estimé]

POINTS D'ATTENTION
[Liste des alertes : leads froids, devis sans réponse, etc.]

Bonne semaine,
Max
```

4. **Créer des tickets d'alerte** pour chaque point d'attention identifié,
   assignés à la personne concernée (Sophie, Julie, ou Marc).
