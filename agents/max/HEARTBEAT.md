# HEARTBEAT.md — Max

Exécute cette séquence à chaque déclenchement (planifié : lundi 08h00 CET).

## 1. Consulter le CRM via `crm-lookup`

- Compter les nouveaux prospects créés cette semaine
- Compter les RDV planifiés et tenus
- Compter les devis envoyés
- Compter les commandes signées + somme du CA
- Identifier les leads sans activité depuis > 5 jours

## 2. Lire la boîte e-mail team@bellacucina.fr via `email-handler`

- Vérifier s'il y a des e-mails non traités importants
- Ne lire qu'en lecture seule — ne jamais rédiger de draft depuis cette boîte

## 3. Rédiger le rapport hebdomadaire

Poster en commentaire sur le ticket récurrent "Rapport CEO" de Marc.

### Format obligatoire

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
[Top 5 deals chauds — nom, statut, montant estimé]

POINTS D'ATTENTION
[Alertes : leads froids, devis sans réponse, RDV non confirmés]

Bonne semaine,
Max
```

## 4. Créer des tickets d'alerte

Pour chaque point d'attention identifié : créer un ticket Paperclip assigné
à la personne concernée (Sophie, Julie, ou Marc).

- Titre : "Alerte — [NOM_LEAD] sans activité depuis [N] jours"
- Assigné à : commerciale responsable du dossier, ou Marc si deal > 20k€
