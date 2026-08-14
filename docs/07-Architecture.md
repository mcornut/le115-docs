# 07 - Architecture

## Objectif

Proposer une architecture simple, testable et adaptée à un projet personnel.

---

## Stack recommandée

| Couche | Choix |
|---|---|
| Site public visiteur | **Next.js / React — confirmé et livré** (dépôt `le115-frontend`), rendu au serveur **à chaque requête** |
| Back-office | SPA React (Vite), dépôt `le115-dashboard` — cf. DEC-014 |
| Backend | Go |
| Base | PostgreSQL |
| Auth admin | Cookie HttpOnly |
| Déploiement | Docker Compose |
| Stockage photos | Système de fichiers local (racine `LE115_MEDIA_DIR`) ; variantes responsive générées à l'upload |

---

## Découpage backend Go

```text
backend/
├── cmd/
│   └── server/
├── internal/
│   ├── auth/
│   ├── property/
│   ├── cms/
│   ├── media/
│   ├── availability/
│   ├── pricing/
│   ├── quote/
│   ├── stay/
│   ├── reservation/
│   ├── calendar/
│   └── notification/
└── migrations/
```

---

## Responsabilités

### availability

Détermine si une période peut être sélectionnée.

Ne calcule jamais les prix.

### pricing

Trouve les prix par nuit selon les périodes tarifaires.

Ne connaît pas les formulaires.

### quote

Construit un devis détaillé à partir :
- des dates ;
- des prix ;
- des frais ;
- des règles métier.

### stay

Gère les demandes de séjour.

### reservation

Gère les réservations confirmées.

### cms

Gère les contenus éditoriaux multilingues.

### media

Gère les photos et leurs variantes responsive.

Caractéristiques V1 :
- Catégories fixes (enum) : `exterieur`, `interieur`, `chambres`, `salles-de-bain`, `autre`.
- Formats acceptés : JPEG, PNG.
- Variantes générées automatiquement : 400px, 800px, 1600px (pas d'upscale).
- Textes alternatifs bilingues (FR/EN) via `localized_content` (EAV).

---

## Site public visiteur — `le115-frontend`

Le site public est un **quatrième dépôt**, frère de `le115-backend`,
`le115-dashboard` et `le115-docs`. « Next.js / React », prévu ici de longue date, est
**confirmé** : le dashboard vit derrière une authentification et n’a aucun enjeu de
référencement (d’où Vite, cf. DEC-014), mais l’argument s’inverse pour le site
visiteur. Un rendu côté client laisserait les aperçus de partage vides et confierait
le référencement au bon vouloir des robots, sur un site dont la photo est l’argument
de vente.

**Rendu à chaque requête, sans cache.** Chaque visite lit l’API, pour qu’une
modification faite au dashboard soit en ligne immédiatement. Corollaire assumé : le
backend est sur le chemin critique de chaque visite, d’où un **délai d’attente de
5 secondes** sur l’appel d’API et une page de panne dans la langue du visiteur — un
backend qui ne répond plus rendrait sinon le site non pas lent, mais suspendu.

**Déploiement.** Un **processus Node** à côté du binaire Go, derrière le **même
reverse-proxy** que l’API et le dashboard. Le site parle au backend par le réseau
interne, jamais par l’internet public. Trois choses derrière un seul proxy : le
binaire Go (API), le bundle statique du dashboard, le processus Node du site public.

Structure du dépôt :

```text
le115-frontend/
├── src/
│   ├── app/            # routes (App Router), segment de langue [locale]
│   ├── components/     # sections de la landing et chrome
│   ├── content/        # pages d’audience, contenu statique FR/EN
│   ├── i18n/           # libellés d’interface, un fichier par langue
│   └── lib/            # api (seule frontière réseau), photos, métadonnées
└── public/             # carte statique du secteur, favicon
```

Le détail des choix techniques (segment de langue, routage, neutralisation de
l’optimisation d’images, îlots interactifs) vit dans le dépôt lui-même — il ne relève
pas de cette documentation produit.

---

## Composants landing

```text
LandingPage
├── Header             ← livré
├── Hero               ← livré
├── Highlights         ← livré (capacité + équipements, cf. 02-UX.md)
├── About              ← livré
├── Amenities          ← livré
├── Gallery            ← livré (filtre + plein écran : îlot interactif)
├── Location           ← livré (carte statique, cf. DEC-021/DEC-022)
├── FaqPreview         ← livré (quatre questions, puis /informations-pratiques)
├── StayPlanner        ← À VENIR, parcours de demande
│   ├── DateSelector
│   ├── GuestFields
│   ├── QuoteCard
│   └── StayRequestForm
└── Footer             ← livré
```

Le bouton « Estimer mon séjour » pointe aujourd’hui vers un module qui n’existe pas
encore : il est traité comme une **ancre inerte**, jamais comme un lien mort.

---

## Principe clé

La logique métier ne doit jamais être portée par les composants React.

React affiche :
- les données ;
- les erreurs ;
- les états.

Le backend décide :
- disponibilités ;
- règles ;
- prix ;
- validité d'une demande.
