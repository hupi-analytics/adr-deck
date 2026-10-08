---
name: adr-extract
description: Crée une ou plusieurs ADR (Architecture Decision Records) au format MADR strict, quelle que soit l'entrée — fichier texte, Markdown, Word (.docx), PDF, compte rendu de réunion, notes, export Notion, fil de discussion, texte collé, simple description d'une décision ou discussion en cours — et produit des fichiers `NNNN-titre.md` (une ADR par fichier) valides pour adr-deck. À utiliser dès que l'utilisateur veut « créer une ADR », « rédiger les ADR », « transformer », « convertir » ou « importer » des décisions d'architecture, ou documenter un choix technique sous forme d'ADR.
argument-hint: <fichier, texte ou sujet de la décision> [dossier de sortie]
---

# Créer des ADR au format MADR

Objectif : quelle que soit l'entrée, produire **une ou plusieurs** ADR au format [MADR](https://adr.github.io/madr/) **strict**, fidèles à la source, validées par `adr-deck validate` (et `--strict` dès que la source le permet) et prêtes à être revues avec `adr-deck`. **La sortie est toujours du MADR conforme**, même si l'entrée est désordonnée, partielle ou dans une autre structure (ADR Nygard, tableau, liste de décisions, Y-statements…) : on reprend le fond, jamais la forme d'origine.

Entrée : `$ARGUMENTS` — au choix :

- un ou plusieurs chemins de fichiers, ou une URL déjà accessible via un outil ;
- du texte collé (notes, compte rendu, fil de discussion) ;
- une simple demande (« crée une ADR pour le choix de la base de données ») ou la discussion en cours dans la conversation : partir de ce que l'utilisateur a dit ; s'il manque l'essentiel (le problème, les options envisagées), poser **une seule** série de questions courtes, puis écrire. Ce qui reste inconnu reste absent.

Si rien n'est fourni, demander la source ou le sujet de la décision.

## 0. Prérequis

Dans ce dépôt, valider avec `npm run validate` (après `npm install`). Ailleurs, utiliser la CLI installée : `command -v adr-deck` ; si elle manque, le signaler (installation depuis ce dépôt : `npm run install:global`, Node ≥ 22.22 ; avec nvm, `adr-deck` n'existe que pour la version de Node active à l'installation). Sans outil de validation, écrire quand même les fichiers en suivant strictement le format ci-dessous, et dire qu'ils n'ont pas pu être validés.

## 1. Lire la source

| Source | Comment la lire |
| --- | --- |
| Demande directe, discussion en cours | Les messages de l'utilisateur et le code du projet si la décision le concerne (lire ce qui est utile pour le contexte, sans en déduire de décision). |
| Fichiers MADR existants | Déjà au format : seulement valider (étape 4) et corriger si besoin. |
| ADR dans un autre format (Nygard, Y-statement, tableau…) | Lecture directe, puis réécriture en MADR (Status → `status`, Context → « Context and Problem Statement », Decision → « Decision Outcome », Consequences → `### Consequences`). |
| `.docx` exporté par adr-deck | `npm run import -- <fichier.docx> [dossier]` dans ce dépôt, sinon `adr-deck import` (sans `--force`, n'écrase rien), puis valider. |
| Autre `.docx` | `textutil -convert txt -stdout "<fichier>"` (macOS), sinon le skill docx. |
| `.pdf` | Outil Read (paramètre `pages` au-delà de 10 pages). |
| `.md`, `.txt`, `.csv`, `.json`, transcription, notes, texte collé | Lecture directe. |
| Image / capture | Outil Read (vision). |

Toujours lire **toute** la source avant d'écrire.

## 2. Repérer les décisions

Une ADR = une question d'architecture à trancher ou tranchée. Pour chacune, relever ce que la source dit vraiment :

- **Numéro** : conserver le numéro d'origine s'il existe (`ADR-21` → fichier `0021-…md`). Sinon continuer la numérotation du dossier de sortie (dernier numéro + 1, dossiers de catégorie compris), dans l'ordre de la source. Les numéros sont uniques. Les renvois à d'autres ADR dans les textes s'écrivent `ADR-0021`.
- **Titre** : court, tel que dans la source si possible.
- **Statut** (front matter `status`) :

  | Dans la source | `status` |
  | --- | --- |
  | à prendre, ouvert, en discussion, à instruire, proposé, à valider | `proposed` |
  | validé, acté, accepté, adopté, décidé | `accepted` |
  | refusé, rejeté, abandonné | `rejected` |
  | reporté, en attente, hors périmètre, plus tard | `deferred` (+ raison dans « Decision Outcome ») |
  | remplacé par une autre ADR | `superseded by ADR-0012` |
  | obsolète, caduc | `deprecated` |

- **Tags** (`tags: [a, b]`, facultatif) : la partie, le thème ou le domaine.
- **Participants** (facultatif) : `decision-makers`, `consulted`, `informed` (noms séparés par des virgules), seulement s'ils sont cités.
- **Contexte** : le problème et les contraintes, repris fidèlement. Les informations sans place dédiée (échéance, blocage, tickets) vont en liste à puces sous le texte, ou dans « More Information ».
- **Options** : une par option envisagée, titre court commençant par une majuscule. Compléter une option elliptique pour qu'elle se lise seule.
- **Décision rédigée** : pour une ADR `accepted`, ou `proposed` avec une recommandation, écrire `Chosen option: "<titre exact de l'option>", because <justification>.` La recommandation d'une ADR proposée sera présélectionnée pendant la revue.

**Ne rien inventer** : ni date, ni décideur, ni option, ni argument. Une information absente reste absente (pas de `date` si la source n'en donne pas). Sans option formulée, pas de « Considered Options » : l'ADR pourra être refusée ou reportée, pas validée.

## 3. Écrire les fichiers

Un fichier par ADR, nommé `NNNN-titre-en-kebab-case.md` (4 chiffres). Modèles de référence : `templates/madr.md` et `templates/madr-minimal.md` ; exemples pour chaque statut dans `examples/decisions/`.

### Modèle complet

```markdown
---
status: proposed
date: 2026-10-05
decision-makers: Prénom Nom, Prénom Nom
tags: [backend]
---

# <Titre>

## Context and Problem Statement

<texte>

## Considered Options

* <Option 1>
* <Option 2>

## Decision Outcome

Chosen option: "<Option 1>", because <justification>.

### Consequences

* Good, because …
* Bad, because …

### Confirmation

<comment la mise en œuvre sera vérifiée, si la source le dit>

## Pros and Cons of the Options

### <Option 1>

<texte facultatif>

* Good, because …
* Neutral, because …
* Bad, because …

### <Option 2>

## More Information

<échéances, tickets, conditions de révision>
```

### Modèle minimal

Pour une source qui ne donne que contexte, options et décision :

```markdown
# <Titre>

## Context and Problem Statement

<texte>

## Considered Options

* <Option 1>
* <Option 2>

## Decision Outcome

Chosen option: "<Option 1>", because <justification>.
```

### Règles

- `date` et participants seulement si la source les donne.
- « Decision Outcome » : omis pour une ADR proposée sans recommandation ; sinon une phrase de tête (`Chosen option: …`, `Rejected, because …`, `Deferred, because …`), puis éventuellement `### Consequences`.
- Les titres de « Pros and Cons of the Options » reprennent **exactement** ceux de « Considered Options ».
- `### Confirmation` (MADR 4) seulement si la source dit comment la décision sera vérifiée (revue, test, règle de lint) ; `Neutral, because …` pour un argument ni pour ni contre.
- Gros projets : une ADR peut aller dans un **dossier de catégorie** (`docs/decisions/backend/0012-…md`, deux niveaux au plus) si l'utilisateur le demande ou si le dossier en a déjà ; la numérotation reste unique sur tout le dossier.
- Avant d'écrire, chercher dans le dossier une ADR existante sur le même sujet : la signaler plutôt que la dupliquer (au besoin, la nouvelle la remplace : `superseded by`).
- Titres de section MADR en anglais par défaut ; si l'utilisateur veut du français : `## Contexte et problématique`, `## Options envisagées`, `## Décision`, `## Avantages et inconvénients des options` (arguments `* Bon, car …` / `* Mauvais, car …`).
- Listes à puces avec `*`, une ligne vide entre chaque bloc, le fichier se termine par un saut de ligne.

**Emplacement** : le dossier demandé par l'utilisateur ; sinon un dossier d'ADR existant du projet courant (`docs/decisions/`, `docs/adr/`…) ; sinon `workspace/` dans ce dépôt, ou `docs/decisions/` (le créer) dans un autre projet. **Ne jamais écraser** un fichier existant sans le demander.

## 4. Valider et corriger

```sh
npm run validate -- <dossier>          # dans ce dépôt
npm run validate:strict -- <dossier>   # + modèles MADR et markdownlint
adr-deck validate <dossier>            # avec le paquet installé
adr-deck validate --strict <dossier>
```

Corriger toutes les **erreurs** et recommencer. `--strict` ajoute les règles des modèles MADR (dont `### Confirmation` pour une ADR acceptée) et de markdownlint : à viser pour des fichiers neufs, sans inventer de Confirmation que la source ne donne pas. Les avertissements (ex. « aucune option envisagée ») sont acceptables s'ils reflètent la source.

## 5. Rendre compte

Répondre en français, brièvement :

- dossier, fichiers créés et nombre d'ADR par statut ;
- les interprétations faites (statuts ramenés à un autre, options complétées, informations placées dans le contexte) ;
- ce qui manquait dans la source (ADR sans option, sans contexte…) ;
- comment ouvrir la revue : `adr-deck` depuis le dossier (ou son projet), `adr-deck timeline` pour la frise.
