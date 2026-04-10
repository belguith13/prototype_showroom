---
schema: agentcompanies/v1
kind: company
slug: bella-cucina
name: Bella Cucina
description: Showroom de cuisines sur mesure haut de gamme à Paris — conseil, conception et pose pour particuliers et promoteurs
version: 1.0.0
license: MIT
authors:
  - name: Belguith C.
goals:
  - Atteindre 1 200 000€ de CA signé sur l'année 2025
  - Répondre à tout lead entrant sous 2h (heures ouvrées)
  - Taux de prise de RDV sur lead entrant ≥ 50%
  - Maintenir un pipeline actif ≥ 15 projets à tout moment
  - Envoyer tout devis dans les 48h après visite showroom
includes:
  - agents/max/AGENTS.md
  - agents/alex/AGENTS.md
  - skills/email-handler/SKILL.md
  - skills/crm-lookup/SKILL.md
  - skills/kitchen-knowledge/SKILL.md
  - skills/winner-flex-lookup/SKILL.md
  - projects/operations/PROJECT.md
  - tasks/hourly-email-check/TASK.md
  - tasks/weekly-ceo-report/TASK.md
requirements:
  secrets:
    - ANTHROPIC_API_KEY
    - NOTION_TOKEN
    - NOTION_CRM_DB_ID
    - EMAIL_IMAP_HOST
    - EMAIL_IMAP_PORT
    - EMAIL_SMTP_HOST
    - EMAIL_SMTP_PORT
    - EMAIL_USER
    - EMAIL_PASS
metadata:
  tags:
    - showroom
    - cuisine
    - paris
    - b2c
    - france
    - haut-de-gamme
  brandColor: "#8B1A1A"
---

# Bella Cucina — Showroom Cuisine sur Mesure

Bella Cucina est un showroom de cuisines sur mesure haut de gamme situé à Paris.
L'équipe conseille, conçoit et pose des cuisines pour des particuliers et des promoteurs immobiliers.

## Valeurs

- Qualité et finition irréprochable
- Conseil personnalisé, pas de vente forcée
- Délais respectés, transparence sur les prix

## Chiffres clés

- Panier moyen : 12 000€ – 25 000€
- Délai moyen de signature : 3 à 6 semaines après 1er RDV
- Taux de transformation lead → RDV : ~40%
- Fournisseurs principaux : Schmidt, Häcker, Elica, Blanco

## Équipe humaine

- **Marc** — PDG / Directeur
- **Sophie** — Commerciale senior (référente vente)
- **Julie** — Commerciale junior
- **Léa** — Designer cuisine

## Horaires showroom

Mardi–Samedi : 10h–19h — Fermé dimanche et lundi

## Contact

- Email prospects : contact@bellacucina.fr
- Email interne équipe : team@bellacucina.fr
- Téléphone : 01 42 XX XX XX

## Objectifs opérationnels 2025

### Commercial
- Répondre à tout lead entrant sous 2h (heures ouvrées)
- Taux de prise de RDV sur lead entrant : ≥ 50%
- Envoyer tout devis dans les 48h après visite showroom
- Relancer tout devis non signé à J+5, J+10, J+20

### Pipeline
- Maintenir un pipeline actif ≥ 15 projets à tout moment
- Seuil d'alerte PDG : tout deal > 20 000€ ou tout lead sans suivi > 48h

### Satisfaction client
- Répondre à tout e-mail client sous 4h (heures ouvrées)
- Aucun RDV annulé sans reprogrammation dans les 24h

## KPIs hebdomadaires (rapport CEO lundi 08h00)

- Nouveaux leads reçus
- RDV planifiés et tenus
- Devis envoyés
- Commandes signées + CA associé
- Leads sans activité depuis > 5 jours (alerte)
