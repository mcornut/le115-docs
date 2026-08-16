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
  | 2. Parcours de demande | **Calendrier de disponibilité**, devis, formulaire, confirmation, erreurs métier | Un site qui convertit | à venir, **rien n’en est commencé** |
  | 3. Mise en ligne | `sitemap.xml`, `robots.txt`, `CSP`/`HSTS`, CDN des médias, domaine | Un site accessible au public | à venir |

  Le sous-projet 1 ne livre **aucun affichage de disponibilité ni aucun moyen de
  demander un séjour** : le seul appel à l’action mène à la page de contact, qui
  affiche un téléphone et un email. Tout le parcours — voir les dates libres, obtenir
  une estimation, envoyer sa demande, la voir confirmée — est le sous-projet 2, et
  rien n’en est commencé.

  Deux fonctionnalités écartées de ce découpage, chacune demandant du backend neuf :
  le **formulaire de contact** (les coordonnées sont affichées, sans formulaire) et
  l’**exposition publique des règles de séjour**. À quoi s’ajoute, décidé le
  2026-08-16, l’**édition des pages d’audience depuis le dashboard** (voir V2) —
  préalable aux quatre pages restantes.
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
