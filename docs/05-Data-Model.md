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
    PROPERTY ||--o{ EXTERNAL_CALENDAR_SOURCE : imports
    PROPERTY ||--o{ ADDITIONAL_FEE : has
    PROPERTY ||--o{ STAY_RULE : has
    PROPERTY ||--o{ STAY_REQUEST : receives
    PROPERTY ||--o{ RESERVATION : has
    STAY_REQUEST ||--o| RESERVATION : becomes
    EXTERNAL_CALENDAR_SOURCE ||--o{ EXTERNAL_CALENDAR_EVENT : contains

    STAY_REQUEST ||--|| QUOTE_SNAPSHOT : contains
    RESERVATION ||--|| QUOTE_SNAPSHOT : freezes

    PROPERTY {
        uuid id
        string slug
        string name
        string baseline
        int max_guests
        string address
        numeric rating
        int review_count
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
        uuid stay_request_id
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

Contenus éditoriaux traduits (modèle EAV).

Champs :
- `entity_type` : type d'entité (`property`, `amenity`, `faq_item`)
- `entity_id` : UUID de l'entité
- `locale` : `fr` ou `en`
- `field` : clé du champ (voir ci-dessous)
- `value` : contenu texte

Champs par entité :

| entity_type | field | Exemple |
|---|---|---|
| `property` | `title` | « La Provençale » |
| `property` | `subtitle` | « Maison d'exception » |
| `property` | `description` | « Au cœur de la Provence... » |
| `property` | `location` | « Luberon » |
| `amenity` | `label` | « Wifi haute vitesse » |
| `faq_item` | `question` | « Puis-je amener un animal ? » |
| `faq_item` | `answer` | « Oui, chiens et chats bienvenus. » |

Cette approche évite de créer des colonnes comme `title_fr` et `title_en` sur chaque table métier.

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

### StayRule

Règle de séjour applicable à une période.

Champs recommandés :
- `start_date` / `end_date` (nulles = règle par défaut) ;
- `min_nights` ;
- `allowed_checkin_dows` / `allowed_checkout_dows` (jours d'arrivée / de départ autorisés) ;
- `derogation_note` ;
- `priority`.

V1 :
- basse saison : 3 nuits, arrivée/départ tous les jours ;
- 14 juin → 28 août : 7 nuits, arrivée/départ le samedi, avec message de dérogation.

La règle applicable est celle de plus haute priorité dont la période contient la date d'arrivée.

### StayRequest

Demande envoyée par un voyageur.

Ne bloque pas les dates.

### Reservation

Séjour validé par le propriétaire.

Bloque les dates.


### ExternalCalendarSource

Source de calendrier externe importée dans Le 115.

Exemple V1 probable : URL iCal Abritel.

Champs recommandés :
- `provider` (`abritel`, `ical`) ;
- `name` ;
- `ical_url` chiffrée ou stockée de manière sécurisée ;
- `enabled` ;
- `last_sync_at` ;
- `last_sync_status` ;
- `last_error`.

### ExternalCalendarEvent

Événement importé depuis une source externe.

Responsabilités :
- bloquer les dates correspondantes ;
- conserver l'identifiant externe si disponible ;
- permettre la mise à jour ou la suppression lors des imports suivants.

Un événement externe n'est pas une réservation Le 115 : c'est un blocage de disponibilité issu d'une autre plateforme.

### QuoteSnapshot

Capture du devis au moment de la demande ou de la validation.

Le snapshot inclut le détail des nuits, les frais et d'éventuels ajustements admin (montant signé, négatif = remise). Un ajustement régénère un nouveau snapshot en conservant le détail d'origine.

Important : si les prix changent plus tard, l'ancien devis reste inchangé.

### Reservation ↔ StayRequest

Une réservation issue d'une demande référence celle-ci via `stay_request_id`. Ce lien est nul pour une réservation créée manuellement depuis le dashboard.

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
