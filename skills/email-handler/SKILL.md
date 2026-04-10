---
schema: agentcompanies/v1
kind: skill
slug: email-handler
name: Email Handler
description: >
  Use this skill whenever you need to read, classify, or draft emails.
  Covers: reading the inbox (IMAP), classifying incoming messages by type
  (prospect / existing client / supplier / other), extracting structured data
  from prospect emails, and drafting reply emails for human validation.
  Use when: a heartbeat fires and you need to check new emails, or when asked
  to reply to or follow up on an email thread.
  Don't use when: the task is purely CRM-based with no email interaction needed.
license: MIT
metadata:
  paperclip:
    tags:
      - email
      - imap
      - drafts
---

# Email Handler Skill

## 1. Connexion IMAP

```python
import imaplib
import email
from email.header import decode_header
import os

IMAP_HOST = os.environ.get("EMAIL_IMAP_HOST", "imap.gmail.com")
IMAP_PORT = int(os.environ.get("EMAIL_IMAP_PORT", "993"))
EMAIL_USER = os.environ.get("EMAIL_USER")
EMAIL_PASS = os.environ.get("EMAIL_PASS")

def connect_imap():
    mail = imaplib.IMAP4_SSL(IMAP_HOST, IMAP_PORT)
    mail.login(EMAIL_USER, EMAIL_PASS)
    return mail

def fetch_unread(mailbox="INBOX", limit=20):
    """Retourne les N derniers e-mails non lus."""
    mail = connect_imap()
    mail.select(mailbox)
    _, msg_ids = mail.search(None, "UNSEEN")
    ids = msg_ids[0].split()[-limit:]
    messages = []
    for mid in ids:
        _, data = mail.fetch(mid, "(RFC822)")
        raw = data[0][1]
        msg = email.message_from_bytes(raw)
        messages.append(parse_message(msg, mid))
    mail.logout()
    return messages

def parse_message(msg, mid):
    """Extrait les champs utiles d'un message brut."""
    subject, enc = decode_header(msg["Subject"])[0]
    if isinstance(subject, bytes):
        subject = subject.decode(enc or "utf-8", errors="replace")
    sender = msg.get("From", "")
    date   = msg.get("Date", "")
    body   = ""
    if msg.is_multipart():
        for part in msg.walk():
            if part.get_content_type() == "text/plain":
                body = part.get_payload(decode=True).decode("utf-8", errors="replace")
                break
    else:
        body = msg.get_payload(decode=True).decode("utf-8", errors="replace")
    return {
        "id":      mid.decode(),
        "from":    sender,
        "subject": subject,
        "date":    date,
        "body":    body[:2000]   # tronqué pour économiser les tokens
    }
```

---

## 2. Classification d'un e-mail entrant

Quand tu reçois un e-mail, classe-le dans l'une de ces catégories avant tout traitement :

| Catégorie | Critères |
|---|---|
| `prospect_nouveau` | Première prise de contact, mention d'un projet cuisine |
| `client_existant` | Mention d'une commande, d'un devis, ou nom connu dans le CRM |
| `fournisseur` | Expéditeur identifié comme fournisseur (Schmidt, Häcker, Elica…) |
| `spam_irrelevant` | Publicité, démarchage, hors-sujet |
| `autre` | Ne rentre dans aucune des catégories ci-dessus |

**Traiter uniquement** `prospect_nouveau` et `client_existant`.
**Ignorer** (marquer comme lu, ne pas créer de ticket) : `spam_irrelevant`.
**Escalader à Sophie** : `autre` si le sujet semble important.

---

## 3. Extraction structurée — prospect nouveau

Pour tout e-mail classé `prospect_nouveau`, extraire ces champs :

```
NOM_CONTACT    : Prénom Nom (depuis signature ou adresse e-mail)
EMAIL          : adresse e-mail de l'expéditeur
TEL            : numéro si mentionné, sinon "non renseigné"
PROJET         : description courte du projet (ex: "cuisine ouverte sur salon")
SURFACE        : en m² si mentionné, sinon "non renseignée"
BUDGET         : montant si mentionné, sinon "non renseigné"
DELAI          : délai ou date travaux si mentionné, sinon "non renseigné"
URGENCE        : haute / normale / faible (estimer depuis le ton et les délais)
```

Si un champ est absent, noter `"non renseigné"` — ne jamais inventer.

---

## 4. Rédaction d'un e-mail de réponse (draft)

### Structure obligatoire

```
OBJET  : [8 mots max, clair, sans ponctuation finale]
        Exemples :
        "Votre projet cuisine — nous vous rappelons rapidement"
        "Demande de renseignements reçue — Bella Cucina"

CORPS  :
  §1 — Accusé de réception chaleureux (1-2 phrases)
  §2 — Proposition concrète : RDV au showroom avec 2 créneaux proposés
  §3 — Phrase de clôture + invitation à rappeler

SIGNATURE :
  L'équipe Bella Cucina
  Showroom ouvert mardi–samedi, 10h–19h
  📍 [adresse] | 📞 01 42 XX XX XX | bellacucina.fr
```

### Règles impératives
- **Toujours vouvoyer** le client, même s'il tutoie dans son e-mail
- Ne jamais promettre un délai ou un prix sans validation humaine
- Ne jamais mentionner les noms des commerciales (signer "L'équipe Bella Cucina")
- Ton : professionnel mais chaleureux, jamais froid ni formel à l'excès
- Longueur corps : 80–120 mots maximum

### Template de base — prospect nouveau

```
Objet : Votre projet cuisine — nous vous recontactons rapidement

Bonjour [Prénom],

Merci pour votre message ! Nous sommes ravis de votre intérêt pour Bella Cucina
et serions heureux de discuter de votre projet de cuisine.

Afin de mieux vous conseiller, nous vous proposons de vous accueillir au showroom
pour une première rencontre sans engagement. Seriez-vous disponible
[CRÉNEAU 1, ex: jeudi 15 mai à 10h30] ou [CRÉNEAU 2, ex: vendredi 16 mai à 14h] ?

N'hésitez pas à nous appeler si vous préférez convenir d'un autre horaire.
Nous nous réjouissons de découvrir votre projet !

L'équipe Bella Cucina
Showroom ouvert mardi–samedi, 10h–19h
📞 01 42 XX XX XX | bellacucina.fr
```

---

## 5. Ticket Paperclip à créer après traitement

Après chaque e-mail `prospect_nouveau` traité, créer un ticket Paperclip avec :

```
TITRE      : "Nouveau lead — [NOM_CONTACT]"
ASSIGNÉ À  : Sophie (commerciale senior par défaut)
PRIORITÉ   : haute si URGENCE=haute, normale sinon
DESCRIPTION:
  Source : e-mail entrant (contact@bellacucina.fr)
  Reçu le : [DATE]
  ---
  [DONNÉES EXTRAITES — tous les champs du §3]
  ---
  Draft réponse rédigé : oui / non
  Action requise : valider et envoyer le draft e-mail
```

---

## 6. Gestion des relances

Quand tu identifies un prospect sans réponse depuis plusieurs jours :

| Délai | Action |
|---|---|
| J+3 | Préparer draft de relance douce (voir template ci-dessous) |
| J+5 | Créer ticket d'alerte assigné à Sophie : "Relance J+5 — [NOM]" |
| J+10 | Escalader à Max si ticket J+5 non traité |

### Template relance J+3

```
Objet : Suite à votre demande — Bella Cucina

Bonjour [Prénom],

Nous revenons vers vous suite à votre message du [DATE].
Nous espérons que votre projet avance bien !

Nous restons à votre disposition pour organiser une visite au showroom
et répondre à toutes vos questions. N'hésitez pas à nous contacter.

L'équipe Bella Cucina
📞 01 42 XX XX XX | bellacucina.fr
```

---

## 7. Variables d'environnement requises

Ces secrets doivent être configurés dans le module **Secrets** de Paperclip :

```
EMAIL_IMAP_HOST   imap.gmail.com (ou serveur du client)
EMAIL_IMAP_PORT   993
EMAIL_SMTP_HOST   smtp.gmail.com (ou serveur du client)
EMAIL_SMTP_PORT   587
EMAIL_USER        contact@bellacucina.fr
EMAIL_PASS        [mot de passe app ou token OAuth]
```

> Toujours utiliser `os.environ.get("NOM_VARIABLE")` — ne jamais hardcoder.

---

## 8. Limites et sécurité

- **Lecture seule par défaut** — Alex ne peut pas envoyer d'e-mail sans validation humaine
- Tout draft est posté comme commentaire dans le ticket Paperclip, pas envoyé directement
- En cas d'erreur IMAP : créer un ticket d'alerte technique assigné à Marc
- Ne jamais logger le contenu des e-mails dans les fichiers de log (données personnelles)
- Tronquer les corps d'e-mails à 2000 caractères avant envoi au LLM
