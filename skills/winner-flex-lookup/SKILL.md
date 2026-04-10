---
schema: agentcompanies/v1
kind: skill
slug: winner-flex-lookup
name: winnerflexlookup
description: >
  Use this skill to read or write CRM data in WinnerFlex (Cyncly Enterprise API).
  Covers: searching contacts/leads, creating a new prospect, reading projects and quotes.
  Use when: a lead arrives and you need to check/create their file in WinnerFlex,
  or when retrieving project or quote status for a follow-up.
  Don't use when: the task involves email — use email-handler instead.
license: MIT
metadata:
  paperclip:
    tags:
      - crm
      - winnerflex
      - cyncly
      - pipeline
---

# WinnerFlex Lookup Skill

## API cible

**WinnerFlex Enterprise API** (Cyncly)
Base URL : `https://api.flex.cloud`
Modules utilisés : Contact, Lead, Project, Document

## 1. Configuration et authentification

```python
import requests
import os

API_KEY   = os.environ.get("WINNERFLEX_API_KEY","UqMtOcRR7Bqv0exReWX3SNZCFVH5g29j")
ORG_GUID  = os.environ.get("WINNERFLEX_ORG_GUID","7233a416-6c3e-4edf-a33e-6500d2fde93f")
SHOP_GUID = os.environ.get("WINNERFLEX_SHOP_GUID","06c0643c-3b88-4d44-92d3-9b71161958e2")
USER_GUID = os.environ.get("WINNERFLEX_USER_GUID", "10d79d99-1de7-49fd-a592-3b756954d4a5")  # optionnel — agent dédié recommandé

BASE_URL = "https://api.flex.cloud"

HEADERS = {
    "apiKey":  API_KEY,
    "Content-Type": "application/json",
    "Accept": "application/json",
}

def _params(extra: dict = {}) -> dict:
    """Construit les query params communs à toutes les requêtes."""
    p = {"orgGuid": ORG_GUID, "shopGuid": SHOP_GUID}
    if USER_GUID:
        p["userGuid"] = USER_GUID
    p.update(extra)
    return p
```

## 2. Contacts

### Rechercher un contact

```python
def find_contact(email: str = None, name: str = None) -> dict | None:
    """
    Cherche un contact par email ou nom.
    Retourne le premier résultat ou None.
    """
    body = {
        "orgGuid": ORG_GUID,
        "shopGuid": SHOP_GUID,
    }
    if USER_GUID:
        body["userGuid"] = USER_GUID
    if email:
        body["email"] = email
    if name:
        body["name"] = name

    r = requests.post(
        f"{BASE_URL}/eapi/v1/contacts/filter",
        headers=HEADERS,
        json=body
    )
    r.raise_for_status()
    results = r.json().get("items", r.json() if isinstance(r.json(), list) else [])
    return results[0] if results else None
```

### Créer un contact

```python
def create_contact(data: dict) -> str:
    """
    Crée un nouveau contact dans WinnerFlex.
    Retourne le GUID du contact créé.

    data = {
        "firstName": str,
        "lastName": str,
        "email": str,
        "phone": str,       # optionnel
        "mobile": str,      # optionnel
        "address": str,     # optionnel
        "city": str,        # optionnel
        "notes": str        # optionnel
    }
    """
    payload = {
        "orgGuid": ORG_GUID,
        "shopGuid": SHOP_GUID,
        **data
    }
    if USER_GUID:
        payload["userGuid"] = USER_GUID

    r = requests.post(
        f"{BASE_URL}/eapi/v1/contacts",
        headers=HEADERS,
        json=payload
    )
    r.raise_for_status()
    return r.json().get("guid", "")
```

## 3. Leads

### Lister les leads actifs

```python
def list_leads(status: str = None) -> list:
    """
    Retourne la liste des leads.
    status : filtre optionnel (ex: "open", "won", "lost")
    """
    body = {
        "orgGuid": ORG_GUID,
        "shopGuid": SHOP_GUID,
    }
    if USER_GUID:
        body["userGuid"] = USER_GUID
    if status:
        body["status"] = status

    r = requests.post(
        f"{BASE_URL}/eapi/v1/leads/filter",
        headers=HEADERS,
        json=body
    )
    r.raise_for_status()
    return r.json().get("items", r.json() if isinstance(r.json(), list) else [])
```

### Créer un lead

```python
def create_lead(contact_guid: str, data: dict) -> str:
    """
    Crée un lead associé à un contact existant.
    Retourne le GUID du lead.

    data = {
        "title": str,           # ex: "Cuisine 12m² - Budget 15k€"
        "description": str,     # notes libres
        "estimatedValue": float, # budget estimé en euros
        "source": str,          # "email", "phone", "walk-in", "website"
        "expectedCloseDate": str # format ISO: "2025-06-30"
    }
    """
    payload = {
        "orgGuid": ORG_GUID,
        "shopGuid": SHOP_GUID,
        "contactGuid": contact_guid,
        **data
    }
    if USER_GUID:
        payload["userGuid"] = USER_GUID

    r = requests.post(
        f"{BASE_URL}/eapi/v1/leads",
        headers=HEADERS,
        json=payload
    )
    r.raise_for_status()
    return r.json().get("guid", "")
```

## 4. Projets

### Lister les projets d'un contact

```python
def list_projects(contact_guid: str) -> list:
    """Retourne tous les projets associés à un contact."""
    body = {
        "orgGuid": ORG_GUID,
        "shopGuid": SHOP_GUID,
        "contactGuid": contact_guid,
    }
    if USER_GUID:
        body["userGuid"] = USER_GUID
    r = requests.post(
        f"{BASE_URL}/eapi/v1/projects/filter",
        headers=HEADERS,
        json=body
    )
    r.raise_for_status()
    return r.json().get("items", r.json() if isinstance(r.json(), list) else [])
```

## 5. Documents / Devis

### Lister les documents d'un projet

```python
def list_documents(project_guid: str) -> list:
    """Retourne les documents/devis associés à un projet."""
    body = {
        "orgGuid": ORG_GUID,
        "shopGuid": SHOP_GUID,
        "projectGuid": project_guid,
    }
    if USER_GUID:
        body["userGuid"] = USER_GUID
    r = requests.post(
        f"{BASE_URL}/eapi/v1/documents/filter",
        headers=HEADERS,
        json=body
    )
    r.raise_for_status()
    return r.json().get("items", r.json() if isinstance(r.json(), list) else [])
```

## 6. Flux typique — Nouveau lead entrant

```python
# 1. Vérifier si le contact existe déjà
contact = find_contact(email="jean.dupont@gmail.com")

# 2. Si non, créer le contact
if not contact:
    contact_guid = create_contact({
        "firstName": "Jean",
        "lastName": "Dupont",
        "email": "jean.dupont@gmail.com",
        "phone": "06 12 34 56 78",
        "notes": "Lead entrant par email - cuisine 12m² budget 15k€"
    })
else:
    contact_guid = contact["guid"]

# 3. Créer le lead associé
lead_guid = create_lead(contact_guid, {
    "title": "Cuisine 12m² — Jean Dupont",
    "description": "Refaire cuisine env. 12m². Budget ~15 000€. Délai : avant été.",
    "estimatedValue": 15000.0,
    "source": "email",
    "expectedCloseDate": "2025-07-31"
})
```

## 7. Variables d'environnement requises

```
WINNERFLEX_API_KEY    Clé API générée dans Flex Admin > Services > Enterprise API
WINNERFLEX_ORG_GUID   GUID de l'organisation (trouvable dans les paramètres Flex)
WINNERFLEX_SHOP_GUID  GUID du shop/showroom
WINNERFLEX_USER_GUID  GUID de l'utilisateur agent (optionnel — isoler les actions IA)
```

> **Recommandation** : créer un utilisateur dédié "Agent IA" dans WinnerFlex et utiliser son GUID pour `WINNERFLEX_USER_GUID`. Cela permet de distinguer les actions automatiques des actions humaines dans les logs WinnerFlex.

## 8. Gestion des erreurs

```python
def safe_call(fn, *args, **kwargs):
    """Wrapper pour gérer les erreurs API proprement."""
    try:
        return fn(*args, **kwargs), None
    except requests.HTTPError as e:
        return None, f"HTTP {e.response.status_code}: {e.response.text[:200]}"
    except Exception as e:
        return None, str(e)
```
