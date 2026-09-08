# Automatisation des résumés de visioconférences

Automatisation de la transcription, du résumé personnalisé et de la distribution par e-mail de réunions enregistrées.

## Avis de confidentialité

Ce projet a été mené dans un cadre professionnel.

Le code source, les données de production, les informations relatives aux clients et les captures d'écran originales ne sont pas accessibles au public pour des raisons de confidentialité.

Ce référentiel présente la portée du projet et l’architecture technique.

Certains diagrammes et illustrations ont pu être recréés à partir d’informations anonymisées ou fictives.

## Table des matières

- [Présentation du projet](#aperçu-projet)
- [Contexte métier](#contexte-metier)
- [Objectifs](#objectifs)
- [Architecture technique](#architecture-technique)
- [Étapes du workflow](#etapes-du-workflow)
- [Stack technique](#stack-technique)
- [Compétences développées](#compétences-développées)

## Présentation du projet

Ce projet consiste à automatiser la captation, la transcription, la personnalisation et l'envoi de résumés de visioconférences.

L'objectif est de permettre aux utilisateurs d'obtenir rapidement un compte rendu adapté au format souhaité, tout en limitant les manipulations manuelles.

---

## Contexte métier

L’organisation avait besoin de résumé fiable et personnalisé, avec la possibilité de modifier eux même le contenu des prompts.

Avant la mise en œuvre de la solution, des informations se voyaient perdues après les réunions. Plusieurs solutions ont été testées (Fireflies.ai, Otter.ai, Avoma, Krisp, TL;DV), voici les points problématiques qui revenaient majoritairement :

- limite de temps ou nombre de captations de résumé
- mauvaise qualité de transcription
- non pertinence des résumés
- aucune ou peu de flexibilité sur l'ajustement de la forme et les points essentiels sur les résumés 
- prix élevé des licences premium

---

## Objectifs

Les principaux objectifs de la solution étaient les suivants :

- enregistrer automatiquement les visioconférences sur plusieurs outils de visioconférence
- récupérer les transcriptions à la fin des réunions
- générer des résumés personnalisés (dans l'idéal, créer un prompt template de base utilisable pour la génération de tout les résumés), avec la possibilité de donner la main au client pour la personnalisation
- envoyer automatiquement les résultats par e-mail
- mettre en place une politique de suppression des fichiers de transcription, pour la sécurité des données sensibles
- possibilité de regénérer et d'envoyer à nouveau un résumé par mail, après modification du prompt
- aucun coût d'abonnement, seulement les coûts d'utilisation de l'IA et du serveur n8n

---

## Architecture technique

```text
Logiciel de visioconférence : Google Meet, Teams, etc.
        │
        ▼
TL;DV : enregistrement et transcription
        │
        ▼
Zapier : récupération et transfert de la transcription
        │
        ▼
Google Drive / Google Sheets
        │
        ▼
n8n : génération du résumé personnalisé
        │
        ▼
E-mail : envoi du résumé et des informations associées
        │
        ▼
n8n : suppression automatique des fichiers TXT après 2 jours
        │
        ▼
n8n/Type Form : modification du prompt et nouvelle génération du compte rendu de la visioconférence sélectionnée
```

---

## Étapes du workflow

<img width="4494" height="2742" alt="Bot IA Visio TLDV - Re traitement (1)" src="https://github.com/user-attachments/assets/80077f2a-7495-4672-8870-a835ee7d9427" />


### 1. Enregistrement de la visioconférence

Le logiciel sélectionné pour la captation est TL;DV, il se connecte au logiciel de visioconférence utilisé par les collaborateurs. Le choix de TL;DV, provient du fait qu'il propose une version gratuite sans limite de captation, avec une transcription de qualité et une connexion native possible à Zapier (idéal pour l'extraction automatique des transcriptions).

Il permet :

- d'enregistrer la visioconférence
- de générer automatiquement sa transcription
- d'identifier les participants et les métadonnées de la visioconférence

### 2. Transfert de la transcription

À la fin de la visioconférence, une automatisation Zapier est déclenchée. Cette automatisation utilise le connecteur natif de TL;DV afin de :

- extraire les données de transcription
- extraire les données de la visioconférence
- ajouter les données de la visioconférence dans le fichier Gsheet ciblé (Métadonnées de la visioconférence (id, email participant, nom, durée, etc.))
- créer le fichier .txt de la transcription dans le dossier ciblé

### 3. Stockage dans Google Drive

Les fichiers sont organisés dans une arborescence Google Drive dédiée, voici celle utilisée :

```text
Dossier principal
├── Notes.gsheet
└── Transcription
    ├── titre_reunion01-lien_enregistrement.txt
    └── titre_reunion02-lien_enregistrement.txt
```

Le fichier Notes.gsheet centralise les informations relatives aux visioconférences :

- date et heure
- identifiant de la visioconférence
- titre de la réunion
- liste des participants
- lien vers l'enregistrement

Le dossier Transcription contient les fichiers texte générés à partir des réunions enregistrées. Le matching entre les données de la visioconférence (Notes.gsheet) et la transcription (fichier .txt dans dossier Transcription) se fait avec un identifiant qui combine le titre de la réunion et le lien d'enregistrement.

### 4. Génération et envoi du résumé avec n8n

Une automatisation n8n récupère les données nécessaires depuis Google Drive et Google Sheets.

Le workflow permet notamment de :

- identifier une nouvelle transcription (déclenchement quand nouvelle transcription dans le dossier Drive Transcription)
- lire le contenu du fichier .txt
- récupérer les informations de la visioconférence dans le fichier Notes.gsheet
- appliquer le format de résumé demandé
- générer un résumé personnalisé à l'aide de l'intelligence artificielle
- préparer le contenu de l'e-mail et envoi

Les résumés peuvent être adaptés selon les besoins des utilisateurs, à travers la modification d'un prompt.

Un workflow n8n est spécifiquement existant pour cette étape.

### 5. Suppression des fichiers temporaires

Afin de limiter la conservation des données sensibles, une automatisation n8n planifiée supprime les fichiers .txt présents dans le dossier Transcription après deux jours (modifiable).

Cette étape permet de réduire la durée de conservation des transcriptions pour limiter les risques liés à l'exposition de données confidentielles et également éviter l'accumulation de fichiers inutiles.

Un workflow n8n est spécifiquement existant pour cette étape.

### 6. Possibilité de regénérer le résumé d'une réunion

Si le résumé ne convient pas, il est possible de modifier le fond et la forme attendu en structurant différemment le prompt, puis en demandant une nouvelle génération. Pour cela, on passe par l'outil de formulaire Type form, qui va recenser les réunions existantes et ou il est possible de demander une nouvelle génération.

Un workflow n8n est spécifiquement existant pour cette étape.

---

## Stack technique

| **Domaines** | **Technologies** |
|---|---|
| **Logiciel de captation** | `TL;DV` |
| **Logiciel d'automatisation** | `n8n`, `zapier` |
| **Langage** | `JS` |
| **Organisation des fichiers** | `Google Drive` |

---

## Compétences développées

Ce projet m'a permis de développer et de mettre en œuvre mes compétences dans les domaines suivants :

- analyse métier et recueil des besoins/problématiques de l'existant
- mise en forme et documentation d'une solution
- autonomnie dans la gestion d'un projet et la communication avec un client
- utilisation de logiciel no code/low code (n8n et zapier)
- optimisation et règles de prompting selon les LLM

Pour ce qui est des outils, j'ai développé des compétences dans ceux qui sont présents dans la partie stack technique.

