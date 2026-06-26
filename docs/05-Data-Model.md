# 05 - Data Model

## Objectif

Définir un modèle métier simple mais suffisamment propre pour éviter les refontes rapides.

La V1 ne gère qu'une seule maison, mais le modèle conserve l'entité `Property`.

---

## Vue d'ensemble

```mermaid
erDiagram
    PROPERTY ||--o{ PHOTO : has
    PROPERTY ||--o{ AMENITY : has
    PROPERTY ||--o{ LOCALIZED_CONTENT : has
    PROPERTY ||--o{ FAQ_ITEM : has
    PROPERTY ||--o{ PRICING_PERIOD : has
    PROPERTY ||--o{ CALENDAR_BLOCK : has
    PROPERTY ||--o{ STAY_REQUEST : receives
    PROPERTY ||--o{ RESERVATION : has

    STAY_REQUEST ||--|| QUOTE_SNAPSHOT : contains
    RESERVATION ||--|| QUOTE_SNAPSHOT : freezes

    PROPERTY {
        uuid id
        string slug
        string name
        string baseline
        int max_guests
        string address
        datetime created_at
        datetime updated_at
    }

    PHOTO {
        uuid id
        uuid property_id
        string url
        string alt
        int sort_order
        bool is_main
    }

    PRICING_PERIOD {
        uuid id
        uuid property_id
        date start_date
        date end_date
        int nightly_price_cents
        int priority
    }

    STAY_REQUEST {
        uuid id
        uuid property_id
        date arrival_date
        date departure_date
        string status
        string first_name
        string last_name
        string email
        string phone
        int adults
        int children
    }

    RESERVATION {
        uuid id
        uuid property_id
        date arrival_date
        date departure_date
        string status
        int total_cents
    }
```

---

## Entités principales

### Property

Représente la maison.

Même si une seule maison existe en V1, cette entité évite de disperser les paramètres globaux.

### LocalizedContent

Contenus éditoriaux traduits.

Champs recommandés :
- `entity_type`
- `entity_id`
- `locale`
- `field`
- `value`

Cette approche évite de créer `title_fr`, `title_en`, `title_es`.

### Photo

Photo affichée sur le site.

Responsabilités :
- URL ;
- ordre ;
- photo principale ;
- texte alternatif.

### Amenity

Équipement affiché sur le site.

Exemples :
- Piscine
- Wifi
- Parking
- Garage vélo
- Climatisation

### FAQItem

Question / réponse affichée sur la landing.

Chaque question est traduisible.

### PricingPeriod

Définit un prix par nuit sur une période.

V1 :
- prix fixe par nuit ;
- priorité prévue pour gérer plus tard exceptions / promotions.

### AdditionalFee

Frais additionnel.

V1 :
- ménage : 400 €.

Modèle générique afin de pouvoir ajouter plus tard linge, animal, chauffage piscine, etc.

### StayRequest

Demande envoyée par un voyageur.

Ne bloque pas les dates.

### Reservation

Séjour validé par le propriétaire.

Bloque les dates.

### QuoteSnapshot

Capture du devis au moment de la demande ou de la validation.

Important : si les prix changent plus tard, l'ancien devis reste inchangé.

---

## Diagramme métier

```mermaid
classDiagram
    class AvailabilityService {
        +check(arrival, departure)
    }

    class PricingService {
        +calculateNights(arrival, departure)
        +findNightlyPrice(date)
    }

    class QuoteService {
        +buildQuote(arrival, departure, guests)
    }

    class StayRequestService {
        +create(request)
        +approve(id)
        +reject(id)
    }

    AvailabilityService --> PricingService
    PricingService --> QuoteService
    QuoteService --> StayRequestService
```

---

## Règle importante

Le devis (`Quote`) est un objet métier calculé.

Le snapshot (`QuoteSnapshot`) est une version persistée du devis.
