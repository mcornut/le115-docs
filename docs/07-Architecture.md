# 07 - Architecture

## Objectif

Proposer une architecture simple, testable et adaptée à un projet personnel.

---

## Stack recommandée

| Couche | Choix |
|---|---|
| Frontend | Next.js / React |
| Backend | Go |
| Base | PostgreSQL |
| Auth admin | Cookie HttpOnly |
| Déploiement | Docker Compose |
| Stockage photos | Local en dev, S3-compatible possible ensuite |

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

---

## Frontend

```text
frontend/
├── app/
├── components/
│   ├── landing/
│   ├── booking/
│   ├── dashboard/
│   └── ui/
├── features/
│   ├── quote/
│   ├── availability/
│   ├── stay-request/
│   └── admin/
└── lib/
```

---

## Composants landing

```text
LandingPage
├── Header
├── HeroSection
├── KeyFactsSection
├── DescriptionSection
├── AmenitiesSection
├── GallerySection
├── FAQSection
├── LocationSection
├── StayPlannerSection
│   ├── DateSelector
│   ├── GuestFields
│   ├── QuoteCard
│   └── StayRequestForm
└── Footer
```

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
