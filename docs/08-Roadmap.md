# 08 - Roadmap

## V1

- Landing page premium
- Internationalisation FR / EN
- Galerie
- Disponibilités
- Estimation détaillée
- Demande de séjour
- Site public visiteur — **en chantier** (dépôt `le115-frontend`, cf. DEC-020 à
  DEC-022), découpé en **trois sous-projets** :

  | Sous-projet | Contenu | Livrable | État |
  |---|---|---|---|
  | 1. Socle et contenus | Coquille, navigation, FR/EN, accueil, FAQ, contact, localisation, une page d’audience | Un site qu’on peut montrer | **livré** |
  | 2. Parcours de demande | Calendrier de disponibilité, devis, formulaire, confirmation, erreurs métier | Un site qui convertit | à venir |
  | 3. Mise en ligne | `sitemap.xml`, `robots.txt`, `CSP`/`HSTS`, CDN des médias, domaine | Un site accessible au public | à venir |

  Deux fonctionnalités écartées de ce découpage, chacune demandant du backend neuf :
  le **formulaire de contact** (les coordonnées sont affichées, sans formulaire) et
  l’**exposition publique des règles de séjour**.
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
- Quatre autres pages d’audience (groupe d’amis, tourisme et culture, cyclistes,
  télétravail) et leur **édition depuis le dashboard** — en V1 elles sont rédigées
  dans le dépôt du site
- Promotions
- Export CSV

## V3

- Synchronisation Airbnb / Booking
- Export iCal depuis Le 115
- Signature électronique
- Multi-propriétés
- Espace voyageur
