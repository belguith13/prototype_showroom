---
schema: agentcompanies/v1
kind: skill
slug: winner-flex-lookup
name: winnerflexlookup
description: >
  Use this skill to read or write CRM data in WinnerFlex (Cyncly Enterprise API).
  Covers: listing/creating contacts and projects, reading designs and documents.
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
Auth : header `apiKey` sur toutes les requêtes (pas de bearer token).

Règles de nommage confirmées par intégration live :
- Endpoints **filter** (POST) : body en **camelCase** (`shopGuid`, `projectGuid`, `contactGuid`)
- Endpoints **documents/filter** : body en **PascalCase** (`ShopGuid`, `ProjectGuid`)
- Endpoints **création** : `shopGuid` en **query param** ; body avec noms métier (`ContactPersons`, `Addresses`, `ProjectName`…)

## 1. Configuration et authentification

```python
import requests
import os

API_KEY   = os.environ.get("WINNERFLEX_API_KEY")
SHOP_GUID = os.environ.get("WINNERFLEX_SHOP_GUID")
USER_GUID = os.environ.get("WINNERFLEX_USER_GUID")  # optionnel

BASE_URL = "https://api.flex.cloud"

# GUIDs fixes (récupérés via GET /eapi/v1/contacts/addresstypes et /contacts/countries)
MAIN_ADDRESS_TYPE_GUID = "524bb607-43ce-4de9-942e-936dc30e60d4"
FRANCE_COUNTRY_GUID    = "9d03832c-be57-45b3-bf5a-f90e81cabbe2"

HEADERS = {
    "apiKey": API_KEY,
    "Content-Type": "application/json",
    "Accept": "application/json",
}
```

## 2. Test de connexion

```python
def test_connection() -> dict:
    """Vérifie que l'API key est valide. GET /eapi/v1/company"""
    r = requests.get(f"{BASE_URL}/eapi/v1/company", headers=HEADERS)
    if r.ok:
        return {"connected": True}
    return {"connected": False, "error": f"HTTP {r.status_code}: {r.text[:200]}"}
```

## 3. Contacts

### Lister les contacts du shop

```python
def list_contacts() -> list:
    """Retourne tous les contacts du shop."""
    r = requests.post(
        f"{BASE_URL}/eapi/v1/contacts/filter",
        headers=HEADERS,
        json={"shopGuid": SHOP_GUID},
    )
    r.raise_for_status()
    data = r.json()
    return data if isinstance(data, list) else data.get("items", [])
```

### Rechercher un contact (par email ou nom)

```python
def find_contact(search: str) -> dict | None:
    """
    Cherche un contact par email, nom ou téléphone.
    Retourne le premier résultat ou None.
    """
    r = requests.post(
        f"{BASE_URL}/eapi/v1/contacts/filter",
        headers=HEADERS,
        json={"shopGuid": SHOP_GUID, "Search": search},
    )
    r.raise_for_status()
    data = r.json()
    results = data if isinstance(data, list) else data.get("items", [])
    return results[0] if results else None
```

### Créer un contact

```python
def create_contact(
    first_name: str,
    last_name: str,
    email: str = None,
    phone: str = None,
    address: str = None,
    city: str = None,
    postal_code: str = None,
    notes: str = None,
) -> dict:
    """
    Crée un contact dans WinnerFlex.
    Retourne {"contactGuid": str, "addressGuid": str}.

    La structure réelle : ContactPersons (tableau) + Addresses (tableau).
    shopGuid est passé en query param, pas dans le body.
    """
    body = {
        "ContactPersons": [
            {
                "FirstName": first_name,
                "LastName": last_name,
                "Email": email or None,
                "Phone": phone or None,
                "IsMainContact": True,
            }
        ],
        "Addresses": [
            {
                "AddressLine1": address or None,
                "City": city or None,
                "PostalCode": postal_code or None,
                "CountryTextGuid": FRANCE_COUNTRY_GUID,
                "AddressTypeGuid": MAIN_ADDRESS_TYPE_GUID,
                "IsEnabled": True,
            }
        ],
    }
    if notes:
        body["FreeText"] = notes

    r = requests.post(
        f"{BASE_URL}/eapi/v1/contacts",
        params={"shopGuid": SHOP_GUID},
        headers=HEADERS,
        json=body,
    )
    r.raise_for_status()
    result = r.json()

    contact_guid = result.get("contactGuid", "")
    addresses = result.get("addresses", [])
    main_addr = next(
        (a for a in addresses if a.get("addressTypeGuid") == MAIN_ADDRESS_TYPE_GUID),
        addresses[0] if addresses else {},
    )
    return {"contactGuid": contact_guid, "addressGuid": main_addr.get("addressGuid", "")}
```

### Récupérer l'adresse principale d'un contact existant

```python
def get_contact_address_guid(contact_guid: str) -> str:
    """Retourne l'addressGuid principal d'un contact (requis pour créer un projet)."""
    r = requests.get(
        f"{BASE_URL}/eapi/v1/contacts/{contact_guid}",
        headers=HEADERS,
    )
    r.raise_for_status()
    result = r.json()
    addresses = result.get("addresses", [])
    main_addr = next(
        (a for a in addresses if a.get("addressTypeGuid") == MAIN_ADDRESS_TYPE_GUID),
        addresses[0] if addresses else {},
    )
    return main_addr.get("addressGuid", "")
```

## 4. Projets

### Lister tous les projets du shop

```python
def list_projects(contact_guid: str = None) -> list:
    """Retourne les projets du shop, optionnellement filtrés par contact."""
    body = {"shopGuid": SHOP_GUID}
    if contact_guid:
        body["contactGuid"] = contact_guid

    r = requests.post(
        f"{BASE_URL}/eapi/v1/projects/filter",
        headers=HEADERS,
        json=body,
    )
    r.raise_for_status()
    data = r.json()
    return data if isinstance(data, list) else data.get("items", [])
```

### Créer un projet

```python
def create_project(
    contact_guid: str,
    address_guid: str,
    project_name: str,
    description: str = None,
    our_ref_guid: str = None,
) -> dict:
    """
    Crée un projet associé à un contact.
    Retourne {"projectGuid": str, "projectNumber": str}.

    contact_guid  : GUID du contact créé via create_contact
    address_guid  : GUID d'adresse (retourné par create_contact ou get_contact_address_guid)
    project_name  : nom du projet (ex: "Cuisine Jean Dupont")
    description   : informations complémentaires libres
    our_ref_guid  : GUID du vendeur responsable (optionnel)
    """
    project_address = {
        "ContactGuid": contact_guid,
        "AddressTypeGuid": MAIN_ADDRESS_TYPE_GUID,
    }
    if address_guid:
        project_address["AddressGuid"] = address_guid

    body = {
        "ProjectName": project_name,
        "ProjectAddresses": [project_address],
    }
    if description:
        body["ExtraInformation"] = description
    if our_ref_guid:
        body["OurRefGuid"] = our_ref_guid

    r = requests.post(
        f"{BASE_URL}/eapi/v1/projects",
        params={"shopGuid": SHOP_GUID},
        headers=HEADERS,
        json=body,
    )
    r.raise_for_status()
    result = r.json()
    return {"projectGuid": result.get("projectGuid", ""), "projectNumber": result.get("projectNumber", "")}
```

## 5. Designs (devis/plans)

### Lister les designs d'un projet

```python
def get_project_designs(project_guid: str) -> list:
    """
    Retourne les designs (plans + prix) d'un projet.
    Chaque design contient designGuid, designName, totalSalesPriceExVAT…
    """
    r = requests.post(
        f"{BASE_URL}/eapi/v1/projects/designs/filter",
        headers=HEADERS,
        json={"projectGuid": project_guid, "shopGuid": SHOP_GUID},
    )
    r.raise_for_status()
    data = r.json()
    return data if isinstance(data, list) else []
```

## 6. Documents

Types de documents utiles (GUIDs fixes WinnerFlex) :

```python
DOC_TYPES = {
    "SALES_DOCUMENT":  "7101bc1a-85d2-4545-b4fe-6d3d12497b55",
    "QUOTATION":       "dc345f15-cab5-4024-b343-047c555b1d6c",
    "SIGNED_CONTRACT": "71b3f715-c8cd-41a1-993f-16a4afd5391a",
    "SUPPLIER_ORDER":  "69fe2989-5b27-40ec-b8a5-966f366e0456",
    "INVOICE":         "c6ca57cb-0e2f-4954-b82d-8f1bf76c0670",
}
```

### Lister les documents d'un projet

```python
def get_project_documents(project_guid: str, type_guids: list = None) -> list:
    """
    Retourne les documents d'un projet (devis, contrats, factures…).
    type_guids : liste de GUIDs de types pour filtrer (optionnel).
    Note : documents/filter utilise PascalCase dans le body.
    """
    body = {
        "ShopGuid": SHOP_GUID,
        "ProjectGuid": project_guid,
    }
    if type_guids:
        body["Types"] = type_guids

    r = requests.post(
        f"{BASE_URL}/eapi/v1/documents/filter",
        headers=HEADERS,
        json=body,
    )
    r.raise_for_status()
    data = r.json()
    return data if isinstance(data, list) else []
```

## 7. Flux typique — Nouveau lead entrant

```python
# 1. Vérifier si le contact existe déjà
contact = find_contact("jean.dupont@gmail.com")

if contact:
    contact_guid = contact["contactGuid"]
    address_guid = get_contact_address_guid(contact_guid)
else:
    # 2. Créer le contact
    result = create_contact(
        first_name="Jean",
        last_name="Dupont",
        email="jean.dupont@gmail.com",
        phone="06 12 34 56 78",
        city="Paris",
        postal_code="75011",
        notes="Lead entrant par email - cuisine 12m² budget 15k€",
    )
    contact_guid = result["contactGuid"]
    address_guid = result["addressGuid"]

# 3. Créer le projet
project = create_project(
    contact_guid=contact_guid,
    address_guid=address_guid,
    project_name="Cuisine Jean Dupont",
    description="Refaire cuisine env. 12m². Budget ~15 000€. Délai : avant été.",
)

# 4. Lister les designs existants (si projet déjà travaillé)
designs = get_project_designs(project["projectGuid"])
```

## 8. Variables d'environnement requises

```
WINNERFLEX_API_KEY    Clé API générée dans Flex Admin > Services > Enterprise API
WINNERFLEX_SHOP_GUID  GUID du shop/showroom
WINNERFLEX_USER_GUID  GUID de l'utilisateur agent (optionnel — isoler les actions IA)
```

> **Recommandation** : créer un utilisateur dédié "Agent IA" dans WinnerFlex et utiliser son GUID pour `WINNERFLEX_USER_GUID`. Cela permet de distinguer les actions automatiques des actions humaines dans les logs WinnerFlex.

## 9. Gestion des erreurs

```python
def safe_call(fn, *args, **kwargs):
    try:
        return fn(*args, **kwargs), None
    except requests.HTTPError as e:
        return None, f"HTTP {e.response.status_code}: {e.response.text[:200]}"
    except Exception as e:
        return None, str(e)
```
