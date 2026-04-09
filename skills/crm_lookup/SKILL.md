---
name: crm-lookup
description: >
  Use this skill to consult or update the CRM (customer records).
  Covers: looking up a contact by name or email, creating a new prospect record,
  updating lead status, and listing active pipeline.
  Use when: processing a new lead, checking if a contact already exists,
  or updating a deal status after an action.
  Don't use when: the task involves email reading/writing — use email-handler instead.
---

# CRM Lookup Skill

## Backend CRM cible
Ce prototype utilise **Notion** comme CRM léger (base de données Notion).
La même logique s'applique à HubSpot ou Airtable via leur API REST respective.

## 1. Connexion Notion API

```python
import requests
import os

NOTION_TOKEN   = os.environ.get("NOTION_TOKEN")
NOTION_DB_ID   = os.environ.get("NOTION_CRM_DB_ID")
HEADERS = {
    "Authorization": f"Bearer {NOTION_TOKEN}",
    "Content-Type": "application/json",
    "Notion-Version": "2022-06-28"
}

BASE_URL = "https://api.notion.com/v1"
```

## 2. Rechercher un contact

```python
def lookup_contact(email: str = None, name: str = None) -> dict | None:
    """Retourne la fiche CRM d'un contact ou None si introuvable."""
    filters = []
    if email:
        filters.append({
            "property": "Email",
            "email": {"equals": email}
        })
    if name:
        filters.append({
            "property": "Nom",
            "title": {"contains": name}
        })
    payload = {"filter": {"or": filters}} if filters else {}
    r = requests.post(
        f"{BASE_URL}/databases/{NOTION_DB_ID}/query",
        headers=HEADERS, json=payload
    )
    results = r.json().get("results", [])
    return results[0] if results else None
```

## 3. Créer une fiche prospect

```python
def create_prospect(data: dict) -> str:
    """
    Crée une fiche prospect. Retourne l'ID Notion de la page créée.
    data = {
        "nom": str,
        "email": str,
        "tel": str,
        "projet": str,
        "surface": str,
        "budget": str,
        "delai": str,
        "source": str,       # "e-mail entrant", "téléphone", "walk-in"…
        "statut": str,       # "prospect contacté", "RDV planifié", "devis envoyé"…
        "assigne_a": str     # "Sophie" ou "Julie"
    }
    """
    payload = {
        "parent": {"database_id": NOTION_DB_ID},
        "properties": {
            "Nom":      {"title": [{"text": {"content": data["nom"]}}]},
            "Email":    {"email": data["email"]},
            "Tél":      {"phone_number": data.get("tel", "")},
            "Projet":   {"rich_text": [{"text": {"content": data.get("projet", "")}}]},
            "Surface":  {"rich_text": [{"text": {"content": data.get("surface", "")}}]},
            "Budget":   {"rich_text": [{"text": {"content": data.get("budget", "")}}]},
            "Délai":    {"rich_text": [{"text": {"content": data.get("delai", "")}}]},
            "Source":   {"select": {"name": data.get("source", "e-mail entrant")}},
            "Statut":   {"select": {"name": data.get("statut", "prospect contacté")}},
            "Assigné à":{"select": {"name": data.get("assigne_a", "Sophie")}},
        }
    }
    r = requests.post(f"{BASE_URL}/pages", headers=HEADERS, json=payload)
    return r.json().get("id", "")
```

## 4. Mettre à jour le statut d'un lead

Statuts disponibles (dans l'ordre du pipeline) :
`prospect contacté` → `RDV planifié` → `RDV tenu` → `devis envoyé` → `devis relancé` → `commande signée` → `perdu`

```python
def update_status(page_id: str, new_status: str) -> bool:
    payload = {
        "properties": {
            "Statut": {"select": {"name": new_status}}
        }
    }
    r = requests.patch(
        f"{BASE_URL}/pages/{page_id}",
        headers=HEADERS, json=payload
    )
    return r.status_code == 200
```

## 5. Variables d'environnement requises

```
NOTION_TOKEN        secret_xxxx (token d'intégration Notion)
NOTION_CRM_DB_ID    ID de la base de données CRM Notion
```
