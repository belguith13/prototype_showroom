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

Auth : header `apiKey` sur toutes les requêtes.
`shopGuid` est passé en **query param** pour les créations, en **body PascalCase** pour les filtres.
`orgGuid` / `userGuid` ne sont **pas** utilisés dans les corps de requête (auth uniquement via apiKey).

## 1. Configuration et authentification

```python
import requests
import os

API_KEY   = os.environ.get("WINNERFLEX_API_KEY")
SHOP_GUID = os.environ.get("WINNERFLEX_SHOP_GUID")
USER_GUID = os.environ.get("WINNERFLEX_USER_GUID")  # optionnel — agent dédié recommandé

BASE_URL = "https://api.flex.cloud"

HEADERS = {
    "apiKey": API_KEY,
    "Content-Type": "application/json",
    "Accept": "application/json",
}
```

## 2. Contacts

### Rechercher un contact

```python
def find_contact(search: str) -> dict | None:
    """
    Cherche un contact par email, nom ou téléphone (recherche full-text).
    Retourne le premier résultat ou None.

    search : email, nom complet, ou numéro de téléphone
    """
    body = {
        "ShopGuid": SHOP_GUID,
        "Search": search,
    }

    r = requests.post(
        f"{BASE_URL}/eapi/v1/contacts/filter",
        headers=HEADERS,
        json=body,
    )
    r.raise_for_status()
    data = r.json()
    results = data.get("items", data if isinstance(data, list) else [])
    return results[0] if results else None
```

### Créer un contact

```python
def create_contact(data: dict) -> str:
    """
    Crée un nouveau contact dans WinnerFlex.
    Retourne le GUID du contact créé.

    data = {
        "ContactName": str,      # "Jean Dupont"
        "ContactEmail": str,
        "ContactPhone": str,     # optionnel
        "FreeText": str,         # notes libres, optionnel
        "Enabled": bool,         # défaut True
    }
    """
    payload = {"Enabled": True, **data}

    r = requests.post(
        f"{BASE_URL}/eapi/v1/contacts",
        params={"shopGuid": SHOP_GUID},
        headers=HEADERS,
        json=payload,
    )
    r.raise_for_status()
    return r.json().get("ContactGuid", r.json().get("guid", ""))
```

## 3. Leads

### Rechercher des leads

```python
def search_leads(keyword: str = None, page: int = 1, page_size: int = 50) -> list:
    """
    Retourne une liste paginée de leads.
    keyword : recherche libre (nom contact, email…)
    """
    body = {
        "PageNumber": page,
        "PageSize": page_size,
    }
    if keyword:
        body["Keyword"] = keyword

    r = requests.post(
        f"{BASE_URL}/eapi/v1/leads/search",
        headers=HEADERS,
        json=body,
    )
    r.raise_for_status()
    data = r.json()
    return data.get("Items", data if isinstance(data, list) else [])
```

### Créer un lead

```python
def create_lead(contact_info: dict, lead_info: dict) -> str:
    """
    Crée un lead (avec contact embarqué ou existant).
    Retourne le GUID du lead créé.

    contact_info = {
        "FirstName": str,
        "LastName": str,
        "Email": str,
        "PhoneMobile": str,   # optionnel
        "City": str,          # optionnel
    }

    lead_info = {
        "ShopId": SHOP_GUID,  # obligatoire
        "FreeText": str,      # notes libres, description du projet
        "AssignedTo": str,    # USER_GUID du vendeur, optionnel
    }
    """
    payload = {
        "Lead": {
            "ShopId": SHOP_GUID,
            **lead_info,
        },
        "Contact": contact_info,
        "CustomFields": [],
    }
    if USER_GUID:
        payload["Lead"].setdefault("AssignedTo", USER_GUID)

    r = requests.post(
        f"{BASE_URL}/eapi/v1/leads",
        headers=HEADERS,
        json=payload,
    )
    r.raise_for_status()
    data = r.json()
    return data.get("LeadId", data.get("guid", ""))
```

## 4. Projets

### Lister les projets d'un contact

```python
def list_projects(contact_guid: str) -> list:
    """Retourne tous les projets associés à un contact."""
    body = {
        "ShopGuid": SHOP_GUID,
        "ContactGuid": contact_guid,
    }

    r = requests.post(
        f"{BASE_URL}/eapi/v1/projects/filter",
        headers=HEADERS,
        json=body,
    )
    r.raise_for_status()
    data = r.json()
    return data.get("items", data if isinstance(data, list) else [])
```

## 5. Documents / Devis

### Lister les documents d'un projet

```python
def list_documents(project_guid: str) -> list:
    """Retourne les documents/devis associés à un projet."""
    body = {
        "ShopGuid": SHOP_GUID,
        "ProjectGuid": project_guid,
    }

    r = requests.post(
        f"{BASE_URL}/eapi/v1/documents/filter",
        headers=HEADERS,
        json=body,
    )
    r.raise_for_status()
    data = r.json()
    return data.get("items", data if isinstance(data, list) else [])
```

## 6. Flux typique — Nouveau lead entrant

```python
# 1. Vérifier si le contact existe déjà
contact = find_contact("jean.dupont@gmail.com")

# 2. Créer le lead (contact embarqué dans la payload)
lead_guid = create_lead(
    contact_info={
        "FirstName": "Jean",
        "LastName": "Dupont",
        "Email": "jean.dupont@gmail.com",
        "PhoneMobile": "06 12 34 56 78",
    },
    lead_info={
        "FreeText": "Refaire cuisine env. 12m². Budget ~15 000€. Délai : avant été.",
    },
)

# 3. Si le contact existait déjà, récupérer ses projets
if contact:
    projects = list_projects(contact["ContactGuid"])
```

## 7. Variables d'environnement requises

```
WINNERFLEX_API_KEY    Clé API générée dans Flex Admin > Services > Enterprise API
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
