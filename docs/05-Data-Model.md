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
    PROPERTY ||--o{ AUDIENCE_PAGE : has
    AUDIENCE_PAGE ||--o{ AUDIENCE_PAGE_SECTION : has
    AUDIENCE_PAGE ||--o{ AUDIENCE_PAGE_LOCALE : publishes
    PROPERTY ||--o{ LEGAL_PAGE : has
    LEGAL_PAGE ||--o{ LEGAL_PAGE_SECTION : has
    LEGAL_PAGE ||--o{ LEGAL_PAGE_LOCALE : publishes
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
        int max_adults
        string address
        string reviews_url
        string public_address
        string facebook_url
        string instagram_url
        datetime created_at
        datetime updated_at
    }

    PHOTO {
        uuid id
        uuid property_id
        string category
        string content_type
        int width
        int height
        int byte_size
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
        string locale
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

    AUDIENCE_PAGE {
        uuid id
        uuid property_id
        string icon
        int sort_order
    }

    AUDIENCE_PAGE_SECTION {
        uuid id
        uuid page_id
        int sort_order
    }

    AUDIENCE_PAGE_LOCALE {
        uuid page_id
        string locale
        uuid property_id
        string slug
        datetime published_at
    }
```

---

## Entités principales

### Property

Représente la maison.

Même si une seule maison existe en V1, cette entité évite de disperser les paramètres globaux.

Colonnes non traduites ajoutées par B1 (DEC-035), toutes `text NOT NULL DEFAULT ''` —
la chaîne vide dit « non renseigné », comme pour `reviews_url` :
- `public_address` : l'**adresse affichée** sur le site, sans numéro de rue (DEC-022
  amendée), 200 caractères au plus. Jamais déduite de `address`, l'adresse exacte, qui
  n'est exposée par aucune route publique. Vide : le site affiche le secteur ;
- `facebook_url`, `instagram_url` : vides, ou une URL `https` sur `facebook.com` /
  `instagram.com` (règle tenue par l'application, pas par une contrainte de base).

Colonne ajoutée par B2b (DEC-037) :
- `max_adults` : `int NULL`, **adultes au plus**. `NULL` : aucune limite (le comportement
  d'avant B2b) ; la migration ne la remplit pas. Contrainte de base
  `property_max_adults_check` : `NULL`, ou entre 1 et `max_guests` — elle double la
  validation de l'application (qui rend un `422` nommé) pour qu'aucun chemin (psql, seed,
  écriture concurrente) ne laisse une limite qu'aucun devis ne pourrait satisfaire. Aucun
  âge n'y est attaché : elle borne les adultes que le visiteur déclare.

### LocalizedContent

Contenus éditoriaux traduits (modèle EAV).

Champs :
- `entity_type` : type d'entité (`property`, `amenity`, `faq_item`, `photo`, `additional_fee`,
  `stay_rule`, `audience_page`, `audience_page_block`, `audience_page_block_card`, `legal_page`, `legal_page_section`,
  `highlight`, `site_page`, `site_page_block`, `place`, `surroundings_page`)
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
| `highlight` | `label` | « Cour intérieure » / « Inner courtyard » |
| `faq_item` | `question` | « Puis-je amener un animal ? » |
| `faq_item` | `answer` | « Oui, chiens et chats bienvenus. » |
| `photo` | `alt` | « Vue de la piscine » |
| `additional_fee` | `label` | « Ménage » / « Cleaning » |
| `stay_rule` | `derogation_note` | « Hors samedi : nous contacter. » / « Outside Saturdays: please contact us. » |
| `audience_page` | `nav_label` | « Familles » / « Families » |
| `audience_page` | `title` | « La maison en famille » |
| `audience_page` | `intro` | « Une cour close, une piscine... » |
| `audience_page_block` | `title` | « Les enfants y sont chez eux » |
| `audience_page_block` | `body` | « La cour est close et **sans vis-à-vis**. » — Markdown restreint (DEC-036) |
| `audience_page_block_card` | `subtitle` | « Un village à découvrir à pied » |
| `legal_page` | `title` | « Mentions légales » / « Legal notice » |
| `legal_page_section` | `heading` | « Éditeur du site » |
| `legal_page_section` | `body` | Le texte de la section, en Markdown restreint (DEC-036) ; une ligne vide sépare deux paragraphes |
| `site_page` | `hero_title` | « Le 115, une maison où l’on prend le temps de vivre » |
| `site_page` | `hero_lede` | Le chapeau du grand visuel |
| `site_page_block` | `eyebrow` | « Au fil des pièces » — bloc `media` ou bloc vitrine, facultatif |
| `site_page_block` | `title` | Le titre du bloc `media` ou d’un bloc vitrine |
| `site_page_block` | `body` | Le corps du bloc `media`, en Markdown restreint (DEC-036) |
| `site_page_block` | `text` | Le texte du bloc `quote` |
| `site_page_block` | `button_label` | « Découvrir le 115 » / « Discover Le 115 » |
| `place` | `name` | « Avignon, cité des Papes » |
| `place` | `tagline` | « Un patrimoine mondial à une demi-heure » |
| `place` | `summary` | Le résumé de la carte |
| `place` | `body` | Le corps du détail, en Markdown restreint (DEC-036) |
| `surroundings_page` | `eyebrow` | « Entre nature et patrimoine » — facultatif |
| `surroundings_page` | `title` | « Découvrez les environs du 115 » |
| `surroundings_page` | `lede` | Le chapeau de la liste, facultatif |

Cette approche évite de créer des colonnes comme `title_fr` et `title_en` sur chaque table métier.

### Photo

Photo affichée sur le site.

Responsabilités :
- catégorie (enum fixe V1 : `exterieur`, `interieur`, `chambres`, `salles-de-bain`, `autre`) ;
- dimensions (`width`, `height`, `byte_size`) et format (`content_type` : JPEG ou PNG) ;
- ordre d'affichage ;
- photo principale (une seule par bien via `is_main` unique) ;
- texte alternatif bilingue via `localized_content` (`entity_type='photo'`, `field='alt'`, `locale` en `fr`/`en`).
- **collection** (DEC-039) : `maison` (par défaut) ou `environs`, fixée au téléversement et jamais
  changée. Toutes les lectures de la galerie — admin par défaut, publique, photo principale, photos
  des pages du site — se limitent à `maison` ; une photo `environs` n'est jamais principale et sa
  catégorie vaut `autre` ;
- **credit** (DEC-039) : texte court (200 caractères au plus), le même dans les deux langues,
  **obligatoire** pour une photo `environs` (contrainte `CHECK`), toujours `NULL` pour une photo
  `maison` (contrainte `CHECK`) ;
- l'**ordre d'affichage** (`sort_order`) est propre à chaque collection.

**Pas de colonne `url`** : les URLs sont calculées à partir de l'id photo (`/api/public/media/{id}`) et
permettent le service avec variantes responsive (widths 400/800/1600/original).

### Amenity

Équipement affiché sur le site.

Exemples :
- Piscine
- Wifi
- Parking
- Garage vélo
- Climatisation

### Highlight

Atout du **bandeau** de l'accueil (DEC-037) : une arche et un libellé, composés par la
propriétaire.

Colonnes : `id`, `property_id` (`ON DELETE CASCADE`), `icon` (`text NOT NULL`, jamais
vide — un code du catalogue d'icônes, qui comprend `guests`, réservé aux atouts),
`sort_order` (ordre d'affichage). Index `highlight_property_idx (property_id, sort_order)`.
Le libellé vit dans `localized_content` (`entity_type='highlight'`, `field='label'`), comme
celui d'un équipement ; l'anglais vide retombe sur le français à la lecture.

- **Six au plus par bien**, règle tenue par l'application (et éprouvée par un test), pas
  par une contrainte de base.
- **Naissance par déclencheur.** Un bien reçoit ses six atouts de la maquette (Cour
  intérieure, Piscine, 12 couchages, Chambres climatisées, Commerces de qualité, 4 salles de
  bain) à son insertion — `AFTER INSERT` sur `property`, même patron que les pages légales
  — et, une fois, au passage de la migration pour les biens déjà là. La pose est
  idempotente : un bien qui a déjà au moins un atout n'en reçoit pas, et une liste que la
  propriétaire a vidée n'est jamais re-remplie.
- **Écriture entière** : la liste reçue remplace l'existante, dans l'ordre reçu ; un atout
  absent du corps est supprimé avec ses libellés.

### FAQItem

Question / réponse affichée sur la landing.

Chaque question est traduisible.

### AudiencePage / AudiencePageBlock / AudiencePageLocale

Pages « pour qui » du site public (« Familles », « Cyclistes »…), éditables et
créables depuis le dashboard (DEC-032) — ce ne sont **pas** des colonnes de
`Property` : un agrégat à elles, publié langue par langue.

`AudiencePage` :
- `icon` (catalogue fermé propre aux pages, vingt et un codes, `guests` et les quatre icônes d'audience compris — distinct de celui
  des équipements, qui n'a pas `guests`) ;
- `sort_order` ;
- `card_photo_id` : photo de la carte de l'accueil, **facultative**, de la collection `maison` ou
  `environs` du même bien, `ON DELETE SET NULL` ; hors complétude (DEC-040) ;
- `hero_photo_id` : photo du grand visuel, **facultative**, de la collection `maison` ou
  `environs` du même bien, `ON DELETE SET NULL` ; hors complétude (DEC-041).

`AudiencePage` ne porte plus de `slug` : l'adresse d'une page est désormais un
contenu **par langue** (DEC-033), portée par `AudiencePageLocale`.

`AudiencePageBlock` (DEC-041) : les blocs de la page, calqués sur `SitePageBlock` — `sort_order`,
`type` (`media`, `quote`, `highlights`, `amenities`, `cards`), `photo_id` et `image_side` (bloc
`media`, photo de la Maison), `button_target`. `highlights` et `amenities` au plus une fois.
`AudiencePageBlockCard` : les cartes d'un bloc `cards`, une à trois — `sort_order`, `photo_id`
(Maison ou Environs). Les sections d'avant DEC-041 sont devenues des blocs `media` (migration
00030) ; la table `audience_page_section` n'existe plus.

`AudiencePageLocale` : une ligne `(page_id, locale, slug, published_at)` par
langue — **la ligne ne signifie plus « publiée » mais « la page a une identité
dans cette langue »**. Le `slug` (unique par bien et par langue, minuscules/
chiffres/tirets, refusé s'il appartient à la liste des routes fixes du site
public — dans les deux langues) vit ici, et `published_at`, **nullable**, est
seul à porter la publication : une langue qui a un slug mais aucun
`published_at` reste un brouillon. **Aucun repli** : une page publiée en
français seulement rend un 404 sur son adresse anglaise, jamais le texte
français (DEC-032). Une langue ne se publie pas tant qu'un champ requis — le
slug (DEC-033), le libellé d'onglet, le titre, le chapeau, ou un champ
requis d'un bloc (titre et corps d'un image et texte, texte d'une citation, titre et texte de
chaque carte) — est vide, ni tant que la page n'a aucun bloc.

Les textes (`nav_label`, `title`, `intro` de la page ; `eyebrow`, `title`,
`body`, `text`, `lede`, `button_label` d'un bloc ; `title`, `subtitle`, `text` d'une carte) vivent dans `LocalizedContent`, comme le reste de l'éditorial
— voir la table ci-dessus. La suppression d'une page cascade ses blocs, leurs cartes et
ses lignes de publication ; les lignes `LocalizedContent`, polymorphes et sans
clé étrangère, sont supprimées par l'application dans la même transaction.

Le `body` d'un bloc `media` est du **Markdown restreint** (DEC-036) : gras, italique, liens
(`https://`, `/fr/`, `/en/`, `mailto:`), listes à puces ou numérotées sur un niveau. Il est
validé à l'écriture ; tout le reste est refusé (`422 RICH_TEXT_INVALID`). Les autres champs
(`nav_label`, `title`, `intro`, en-têtes et textes des cartes) restent du texte simple.

### LegalPage / LegalPageSection / LegalPageLocale

Les trois pages légales du site (DEC-035) : mentions légales, confidentialité et
cookies, conditions de location. Un agrégat **propre**, sur le patron des pages
d'audience sans en être une variante : une page légale n'a ni slug, ni icône, ni
libellé d'onglet, ni place dans le menu, et ne se crée ni ne se supprime.

`LegalPage` :
- `kind` : `legal-notice`, `privacy` ou `rental-terms` — liste **fermée** (contrainte
  `CHECK`), unique par bien. Le `kind` désigne la page dans l'API ; il n'y a pas d'`id`
  côté contrat ;
- `updated_at` : avance à chaque écriture du **contenu**, et seulement à celle-là (publier
  ne change pas le texte) ; le site l'affiche (« Mis à jour le … »).

**Trois lignes par bien, créées par migration** — ni création ni suppression par l'API.
Elles naissent d'un déclencheur `AFTER INSERT` sur `property` : un bien créé plus tard
(base neuve, environnement de test) reçoit lui aussi ses trois pages et leurs brouillons,
non publiés. Aucune migration ne crée la ligne `property`, d'où le déclencheur plutôt
qu'une insertion ponctuelle.

Le texte des brouillons vit dans une fonction SQL (`legal_page_drafts`, migration 00024).
La migration 00025 la remplace pour ajouter, au marqueur « Adresse » des mentions légales,
une mise en garde (l'adresse de l'éditeur peut être un domicile) ; elle ne corrige les pages
existantes que si leur texte est encore exactement le brouillon d'origine.

`LegalPageSection` : les paragraphes de la page, une table fille ordonnée par
`sort_order`, supprimée en cascade avec sa page.

`LegalPageLocale` : une ligne `(page_id, locale, published_at)` par langue, présente dès
la création de la page. `published_at`, **nullable**, est **seul** porteur de la
publication, avec la même sémantique que `AudiencePageLocale`. **Aucun repli** de langue.

Les textes (`title` de la page ; `heading`, `body` de chaque section) vivent dans
`LocalizedContent`. Le `body` est du **Markdown restreint**, validé à l'écriture
(DEC-036, voir les pages d'audience) ; `title` et `heading` restent du texte simple.
Règles de complétude et d'écriture (DEC-035) :
- une langue est **complète** quand elle a un titre, au moins une section, et l'intitulé
  comme le corps de chaque section — le corps se juge sur son **texte extrait**, pas sur le
  Markdown brut (DEC-036) ;
- elle ne se publie pas tant qu'elle est incomplète (`PAGE_INCOMPLETE`) ni tant qu'un
  texte contient le marqueur des brouillons **« À COMPLÉTER »** (`PLACEHOLDER_REMAINING`) —
  reconnu sans égard à la casse ni aux accents, avec des frontières de mot ;
- une écriture qui rendrait ainsi **impubliable une langue publiée** est **refusée** et
  n'écrit rien, au lieu de la dépublier comme le fait une page d'audience ;
- les six adresses publiques sont réservées aux pages d'audience (`SLUG_RESERVED`), et la
  migration refuse de s'appliquer si une page d'audience en occupe déjà une.

### SitePage / SitePageBlock / SitePageLocale

Les pages éditoriales du site : « Le 115 » et « Commodités » (DEC-038), et depuis DEC-040
l’**accueil**. Un agrégat **propre**,
sur le patron des pages légales : trois pages fixes par bien, qu'aucune route ne crée ni ne
supprime. Une page est un grand visuel suivi d'une liste ordonnée de blocs typés.

`SitePage` :
- `kind` : `home`, `house` ou `amenities` — liste **fermée** (contrainte `CHECK`), unique par bien.
  Il désigne la page dans l'API ; pas d'`id` côté contrat. `home` n'a **jamais** de ligne
  `SitePageLocale` : il est toujours en ligne (DEC-040) ;
- `hero_photo_id` : la photo du grand visuel, **facultative**, `ON DELETE SET NULL` : une photo
  supprimée de la galerie laisse la page sans photo, sans bloquer la suppression ;
- `updated_at` : avance à chaque écriture du **contenu**, jamais à la publication.

`SitePageBlock` (table fille, supprimée en cascade avec sa page) :
- `sort_order` : le rang dans la page ;
- `type` : `media`, `quote`, `highlights`, `amenities` (toutes pages) ; `audiences`,
  `surroundings`, `gallery`, `location`, `faq` (blocs vitrine, accueil seulement — règle du
  service, le genre vivant sur la page). Tous sauf `media` et `quote` sont uniques par page
  (index partiel) ; les blocs vitrine portent `eyebrow` et `title` dans `localized_content`,
  facultatifs ;
- `button_target` : `NULL` (pas de bouton) ou `house`, `amenities`, `surroundings`, `request` ;
  seulement sur `media` (contrainte `CHECK`). Son libellé `button_label` vit dans
  `localized_content`, 60 caractères au plus (DEC-040) ;
- `photo_id` : photo du bloc `media`, facultative, `ON DELETE SET NULL` ;
- `image_side` : `left` ou `right`, **propre à `media`** (obligatoire pour lui, absent pour les
  autres) ; une contrainte `CHECK` interdit aussi `photo_id` hors d'un bloc `media` ;
- un **index unique partiel** `(page_id, type)` sur `highlights` et `amenities` : au plus une fois
  chacun par page. Le service refuse avant (`BLOCK_DUPLICATED`) ; l'index tient pour deux
  écritures simultanées, qui rendent le même code.

`SitePageLocale` : une ligne `(page_id, locale, published_at)` par langue, présente dès la
création de la page. `published_at`, **nullable**, est seul porteur de la publication, comme
`LegalPageLocale`. **Aucun repli** de langue.

**Trois pages par bien, créées par migration** (00027, puis 00029 pour l'accueil) puis par un
déclencheur `AFTER INSERT` sur `property` : un bien créé plus tard reçoit lui aussi ses pages, non
publiées sauf l'accueil, toujours en ligne, avec leur contenu de naissance (le texte de la
maquette, sans photo). La migration **refuse de
s'appliquer** si une page d'audience occupe déjà l'une des adresses désormais réservées.

Les textes vivent dans `LocalizedContent` : `hero_title` et `hero_lede` (entité `site_page`) ;
`eyebrow`, `title`, `body` d'un bloc `media` et `text` d'un bloc `quote` (entité
`site_page_block`). Le `body` d'un bloc `media` est du **Markdown restreint**, validé à
l'écriture (DEC-036, voir les pages d'audience) ; les autres champs sont du texte simple. Les
blocs `highlights` et `amenities` n'ont aucun champ : le site les remplit depuis
`GET /api/public/property`. Règles de complétude et d'écriture :
- une langue est **complète** quand elle a le titre du grand visuel, au moins un bloc, le titre et
  le corps de chaque `media` (le corps jugé sur son **texte extrait**) et le libellé de son bouton
  s'il en a un, le texte de chaque `quote`. Chapeau, surtitres, photos et en-têtes vitrine sont
  facultatifs. Pour `home`, la complétude est vérifiée dans les deux langues à chaque écriture ;
- une langue incomplète ne se publie pas (`PAGE_INCOMPLETE`) ; une écriture qui rendrait
  **incomplète une langue publiée** est **refusée** et n'écrit rien, jamais convertie en
  dépublication ;
- les quatre adresses publiques sont réservées aux pages d'audience (`SLUG_RESERVED`).

### Place / PlaceLocale / SurroundingsPage / SurroundingsPageLocale

La page « Les environs » et ses lieux (DEC-039). Deux agrégats : les **lieux**, ouverts (créés,
ordonnés, supprimés par la propriétaire, cinquante au plus par bien), et l'**en-tête** de la
liste, fermé (une ligne par bien).

`Place` :
- `position` : le rang dans la liste, unique par bien ; un lieu créé va en fin de liste ;
- `photo_id` : une photo de la collection `environs` du même bien, **facultative**,
  `ON DELETE SET NULL` : une photo supprimée laisse le lieu sans photo, toujours en ligne ;
- `created_at`, `updated_at`.

`PlaceLocale` : une ligne `(place_id, locale, slug, published_at)` par langue, créée avec le lieu.
Le `slug` suit le format des slugs d'audience (`^[a-z0-9]+(-[a-z0-9]+)*$`, 80 caractères au plus),
**unique par bien et par langue parmi les slugs non vides**, sans liste réservée (rien d'autre n'est servi sous
`/les-environs/`). `published_at`, nullable, porte seul la publication. Un slug **vide** est permis : la langue n'a pas encore d'adresse, elle est donc incomplète et ne peut pas être publiée ; deux lieux peuvent tous deux avoir un slug vide dans une langue (index d'unicité partiel). **Aucun repli** de langue.

Textes dans `LocalizedContent` (entité `place`) : `name`, `tagline`, `summary` (texte simple) et
`body` (Markdown restreint, DEC-036).

`SurroundingsPage` : une ligne par bien (`property_id` unique), créée par migration puis par un
déclencheur `AFTER INSERT` sur `property`, avec son contenu de naissance et les sept lieux non
publiés. `SurroundingsPageLocale` : `(page_id, locale, published_at)`. Textes (entité
`surroundings_page`) : `eyebrow` (facultatif), `title`, `lede` (facultatif).

Règles :
- un lieu est **complet** dans une langue avec son nom, son accroche, son résumé, un corps dont le
  **texte extrait** est non vide et un slug valide ; la photo est facultative ;
- la page est **complète** dans une langue avec son titre et **au moins un lieu publié** dans
  cette langue ;
- publier une langue incomplète est refusé (`PLACE_INCOMPLETE`, `PAGE_INCOMPLETE`) ; une écriture
  qui rendrait incomplète une langue publiée est **refusée**, jamais convertie en dépublication ;
- dépublier ou supprimer le **dernier lieu publié** d'une langue où la page est publiée est refusé
  (`LAST_PUBLISHED_PLACE`) ;
- créer ou réordonner les lieux verrouille aussi l'en-tête de la page (même ordre de verrous que DEC-038) : une création pendant un réordonnancement rend `ORDER_MISMATCH` ;
- `les-environs` et `surroundings` sont réservés aux pages d'audience (`SLUG_RESERVED`) ; la
  migration 00028 refuse de s'appliquer si une page d'audience en occupe déjà un.

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

Libellé localisé FR/EN via `localized_content` (`entity_type='additional_fee'`, `field='label'`, `locale` en `fr`/`en`).

### StayRule

Règle de séjour applicable à une période.

Champs :
- `start_date` / `end_date` (nulles = règle par défaut) ;
- `min_nights` ;
- `allowed_checkin_dows` / `allowed_checkout_dows` (jours d'arrivée / de départ autorisés,
  entiers `0`–`6`, dimanche = `0`) ;
- `priority`.

La note de dérogation n'est **pas** une colonne de `stay_rule` : elle vit dans
`localized_content` (`entity_type='stay_rule'`, `field='derogation_note'`, `locale` en
`fr`/`en`), comme les autres textes bilingues du modèle.

V1 :
- basse saison (règle par défaut) : 3 nuits, arrivée/départ tous les jours ;
- 14 juin → 28 août : 7 nuits, arrivée/départ le samedi, avec message de dérogation bilingue.

La règle applicable est celle de plus haute priorité dont la période contient la date d'arrivée.

**Invariants garantis en base :**
- **Anti-chevauchement à priorité identique** entre règles saisonnières : contrainte
  d'exclusion PostgreSQL `stay_rule_no_overlap` (`gist`, sur `property_id`, `priority`,
  `daterange(start_date, end_date, '[]')`), n'affectant que les règles saisonnières
  (`start_date IS NOT NULL`). Deux règles saisonnières peuvent en revanche se **superposer**
  si leurs priorités diffèrent (la plus haute gagne).
- **Une seule règle par défaut** par bien : index unique partiel `stay_rule_one_default`
  sur `(property_id) WHERE start_date IS NULL`. Cette règle est obligatoire (non
  supprimable) : sans elle, tout séjour hors saison serait soumissible sans contrainte.
- **Nature d'une règle immuable** : une règle saisonnière ne devient jamais une règle par
  défaut (ni l'inverse) après création — conséquence de l'invariant précédent.

### StayRequest

Demande envoyée par un voyageur.

Ne bloque pas les dates.

Mémorise la `locale` choisie par le voyageur (`fr` par défaut, ou `en`) au moment de la
demande. Cette langue sert à localiser l'email de refus envoyé au voyageur si le propriétaire
rejette la demande (le devis lui-même reste indépendant de la locale, figé dans
`QuoteSnapshot`).

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
