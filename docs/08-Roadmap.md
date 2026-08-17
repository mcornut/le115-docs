# 08 - Roadmap

## V1

- Landing page premium
- Internationalisation FR / EN
- Galerie
- Disponibilités
- Estimation détaillée
- Demande de séjour
- Site public visiteur — **en chantier** (dépôt `le115-frontend`, cf. DEC-020 à
  DEC-025), découpé en **trois sous-projets**, dont **deux sont livrés** :

  | Sous-projet | Contenu | Livrable | État |
  |---|---|---|---|
  | 1. Socle et contenus | Coquille, navigation, FR/EN, accueil, FAQ, contact, localisation, une page d’audience | Un site qu’on peut montrer | **livré** |
  | 2. Parcours de demande | **Calendrier de disponibilité**, devis, formulaire, confirmation, erreurs métier | Un site qui convertit | **livré** |
  | 3. Mise en ligne | `sitemap.xml`, `robots.txt`, `CSP`/`HSTS`, CDN des médias, domaine | Un site accessible au public | à venir |

  Le sous-projet 2 est livré le **2026-08-18** (cf. DEC-023 à DEC-025). Le visiteur a
  désormais une page dédiée, `/[langue]/demande`, où le seul appel à l’action du site le
  mène : un calendrier de disponibilité sur douze mois glissants, un devis recalculé au
  serveur à chaque changement, un formulaire qui n’apparaît qu’une fois le devis
  soumissible, et une confirmation qui fige ce qui a été envoyé. **Rien ne se réserve en
  ligne** (DEC-003, DEC-004) : le parcours s’arrête à « demande envoyée », la
  propriétaire arbitre depuis son dashboard.

  Une seule addition côté backend : la création d’une demande envoie désormais **deux**
  emails, celui de la propriétaire et un **accusé de réception au voyageur**
  (cf. `06-API.md`). Tout le reste tient sur les endpoints publics existants.

  Deux fonctionnalités écartées de ce découpage, chacune demandant du backend neuf :
  le **formulaire de contact** (les coordonnées sont affichées, sans formulaire) et
  l’**exposition publique des règles de séjour**. À quoi s’ajoute, décidé le
  2026-08-16, l’**édition des pages d’audience depuis le dashboard** (voir V2) —
  préalable aux quatre pages restantes.

  La seconde se paie désormais comptant : le calendrier ne peut annoncer ni la durée
  minimale ni les jours d’arrivée autorisés, faute de route publique qui les expose. Le
  visiteur choisit ses dates, puis **découvre la contrainte par le message d’erreur du
  devis**. Assumé au cadrage du sous-projet 2, à traiter dans le plan qui figera le
  contrat d’exposition publique des règles.
- Dashboard admin (SPA séparée, déploiement même origine que l’API, mono-bien — cf. DEC-014, DEC-015), douze écrans :
  - Tableau de bord (accueil — cf. DEC-018)
  - Calendrier
  - Demandes
  - Réservations
  - Tarifs
  - Règles de séjour
  - Maison (informations du bien)
  - Équipements (cf. DEC-016)
  - FAQ (cf. DEC-016)
  - Photos (cf. DEC-017)
  - Synchronisations
  - Activité
- Préparation synchronisation Abritel / iCal

### Dashboard — modules reportés post-V1

Envisagés en amont, non retenus dans la nav V1 livrée (détail dans
`04-Dashboard.md`) :

- Clients (fiche transverse)
- Avis unitaires
- Messages
- Rapports
- Multi-bien / Logements
- Utilisateurs (comptes admin multiples)
- Paiements
- KPI d'agrégation (occupation, chiffre d'affaires, revenus)
- Création manuelle de réservation
- Nuits max. (règle de séjour)

## V1.1

- Vidéo hero si asset disponible
- Amélioration SEO
- Emails plus travaillés
- Journal d'activité enrichi

## V2

- Paiement acompte
- Import Abritel / iCal opérationnel
- Page “Découvrir la région”
- **Pages d’audience éditables depuis le dashboard** — décidé le 2026-08-16, et c’est
  le préalable aux quatre pages restantes (groupe d’amis, tourisme et culture,
  cyclistes, télétravail), non l’inverse. En V1 elles sont rédigées dans le dépôt du
  site, ce qui pose un vrai problème : **leur intérêt est le référencement, donc leur
  contenu doit être long et concret** — marché, boucles à vélo, distances, ce qu’on
  fait avec des enfants — et rien de tout cela ne vit dans l’API. La page « Familles »
  a été écrite en comblant ces trous, et deux de ses affirmations sont depuis
  consignées comme non corroborées. Rédiger les quatre autres de la même façon
  multiplierait le problème par quatre ; les faire éditer par la propriétaire le
  supprime à la racine.

  Chantier sur les **trois dépôts** : modèle et routes côté backend, écran d’édition
  au dashboard, et le site qui lit l’API au lieu de ses fichiers statiques. **Une
  décision commande tout le reste et n’est pas tranchée : la liberté de mise en
  page** — des sections typées (titre, chapeau, blocs « titre + paragraphe », ce que
  porte déjà le gabarit) ou du texte riche. La première est simple à éditer,
  impossible à casser et toujours cohérente avec le site ; la seconde est plus libre,
  mais demande de choisir un format, de le nettoyer à l’entrée, et d’accepter qu’une
  page puisse devenir laide. À cadrer dans son propre spec.
- Promotions
- Export CSV

## V3

- Synchronisation Airbnb / Booking
- Export iCal depuis Le 115
- Signature électronique
- Multi-propriétés
- Espace voyageur
