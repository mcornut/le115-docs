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
  | 3. Mise en ligne | `sitemap.xml`, `robots.txt`, `CSP`/`HSTS`, CDN des médias, domaine ; mentions légales et confidentialité publiées dans les deux langues, conditions de location relues puis publiées (DEC-035) ; vrais téléphone et email du site (aujourd’hui fictifs dans `le115-frontend/src/lib/contact.ts`) | Un site accessible au public | à venir |

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

  Une fonctionnalité écartée de ce découpage, demandant du backend neuf :
  l’**exposition publique des règles de séjour**. (Le **formulaire de contact**, écarté
  alors, est livré depuis par B1 — voir plus bas, DEC-035.) L’**édition des pages d’audience
  depuis le dashboard**, décidée le 2026-08-16, est en revanche livrée (DEC-032,
  chantier du 2026-08-31) : c’était le préalable aux quatre pages restantes (groupe
  d’amis, tourisme et culture, cyclistes, télétravail), qui restent elles-mêmes à
  écrire — la capacité de les créer existe, leur contenu non.

  Cette absence se paie désormais comptant : le calendrier ne peut annoncer ni la durée
  minimale ni les jours d’arrivée autorisés, faute de route publique qui les expose. Le
  visiteur choisit ses dates, puis **découvre la contrainte par le message d’erreur du
  devis**. Assumé au cadrage du sous-projet 2, à traiter dans le plan qui figera le
  contrat d’exposition publique des règles.
- **Architecture de la maquette Figma** (DEC-034, DEC-035), en trois sous-projets :

  | Sous-projet | Contenu | État |
  |---|---|---|
  | B1 | Formulaire de contact, pages légales, pied de page | **livré** |
  | B2 | Pages « Le 115 » et « Commodités » (DEC-036) | en trois temps, ci-dessous |
  | — B2a | Le texte riche des corps de section | **livré** |
  | — B2b | Le bien : liste d’atouts, icônes de règles, adultes au plus | à venir |
  | — B2c | Les pages éditoriales « Le 115 » et « Commodités » | à venir |
  | B3 | « Les environs » et le détail d’un lieu | à cadrer |

  B1 est livré le **2026-10-08** : un formulaire « Nous écrire » qui envoie un simple email
  à la propriétaire (rien n’est stocké), trois pages légales éditables depuis le dashboard
  mais livrées en **brouillons non publiés** — leurs trous marqués « À COMPLÉTER », que
  la propriétaire doit combler —, l’adresse affichée, Facebook et Instagram saisis dans
  Maison, et le pied de page à trois colonnes de la maquette. Reste, avant la mise en
  ligne, à compléter et publier ces pages (voir la ligne « 3. Mise en ligne » ci-dessus).

  B2a est livré le **2026-10-09** (DEC-036) : les corps de section des pages légales et des
  pages d’audience acceptent le gras, l’italique, les liens et les listes, édités
  visuellement au dashboard et rendus par le site sans HTML interprété. B2b et B2c restent à
  faire.
- Dashboard admin (SPA séparée, déploiement même origine que l’API, mono-bien — cf. DEC-014, DEC-015), treize écrans :
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
  - Pages (pages d’audience du site public — cf. DEC-032)
- Préparation synchronisation Abritel / iCal

### Dashboard — modules reportés post-V1

Envisagés en amont, non retenus dans la nav V1 livrée (détail dans
`04-Dashboard.md`) :

- Clients (fiche transverse)
- Avis unitaires — et depuis DEC-031, plus seulement « pas encore » : les avis font foi
  sur la fiche Google Business, le produit n'en tient aucune copie
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
- **Images dans le texte et téléversement d’images** — le **texte riche est livré** (B2a,
  DEC-036) : gras, italique, liens et listes dans les corps de section, édités visuellement
  et stockés en Markdown restreint, sans jamais interpréter de HTML. L’édition des pages
  d’audience elle-même est livrée aussi (DEC-032). Ce qui reste : les **images dans le
  texte** et leur téléversement (images servies par le stockage média, sous la CSP de
  production). À cadrer dans son propre spec.
- Promotions
- Export CSV

## V3

- Synchronisation Airbnb / Booking
- Export iCal depuis Le 115
- Signature électronique
- Multi-propriétés
- Espace voyageur
