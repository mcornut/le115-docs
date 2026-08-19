# 00 - Product Decisions

Ce document liste les décisions produit importantes afin d'éviter de les rediscuter plus tard.

## DEC-001 — Nom commercial

Le site utilisera le nom :

> **Le 115, Maison de Provence**

Le nom conserve l'identité existante tout en explicitant immédiatement la nature et la localisation du bien.

## DEC-002 — Landing page proche de la maquette fournie

Le site doit rester relativement fidèle à la direction artistique transmise par le conseiller en communication.

Conséquence :
- pas de refonte radicale de la navigation ;
- conserver une image premium, claire et élégante ;
- les propositions UX doivent respecter cette base visuelle.

## DEC-003 — CTA principal : “Estimer mon séjour”

Le CTA principal n'est pas “Réserver”.

Raison :
- moins engageant pour le visiteur ;
- plus adapté à une réservation non instantanée ;
- encourage l'utilisateur à tester les dates.

## DEC-004 — Réservation manuelle

Le site permet une demande de séjour, pas une réservation instantanée.

Le propriétaire s'engage à répondre sous 48 h.

## DEC-005 — Nombre de voyageurs facultatif

Le formulaire conserve les champs :
- adultes ;
- enfants.

Mais ils restent facultatifs en V1 afin de réduire la friction.

## DEC-006 — Dashboard orienté calendrier

Le premier écran du dashboard est une vue calendrier.

Raison :
- c'est l'information la plus utile au propriétaire ;
- elle permet de voir rapidement les réservations, blocages et disponibilités.

> **Remplacée par DEC-014** : l'écran d'accueil du dashboard V1 livré est le
> **Tableau de bord** (synthèse), pas directement le Calendrier — le
> Calendrier reste une entrée de navigation à part entière, accessible en un
> clic.

## DEC-007 — Détail du prix obligatoire

Le devis affiche le détail :
- prix de chaque nuit ;
- sous-total ;
- frais de ménage ;
- total.

Raison :
- transparence ;
- confiance ;
- meilleure compréhension des périodes tarifaires.

## DEC-008 — Documentation exploitable par IA

Le dossier `docs/` décrit le produit.

Le dossier `specs/` décrit les comportements attendus de manière plus directe, afin de pouvoir servir de contexte à Claude Code ou un autre assistant de développement.

## DEC-009 — Vidéo hero souhaitée

Une vidéo courte en hero est une amélioration souhaitée.

Elle n'est pas bloquante pour la V1 si les assets vidéo ne sont pas disponibles.


## DEC-010 — Langues V1 : français et anglais uniquement

La V1 ne propose que deux langues :

- français (`fr`) ;
- anglais (`en`).

L'espagnol n'est pas prévu en V1 afin de réduire le volume de contenu à produire et à maintenir.

## DEC-011 — Synchronisation Abritel à prévoir

Le calendrier devra probablement se synchroniser avec les réservations effectuées via Abritel.

Conséquence :
- prévoir une source de disponibilité externe ;
- privilégier un import iCal en V1 ;
- distinguer les réservations locales des blocages importés ;
- afficher dans le dashboard l'état de la dernière synchronisation.

## DEC-012 — Ajustement de prix par l'administrateur

Le propriétaire peut ajuster le prix final d'une demande ou d'une réservation, par exemple un geste commercial (−200 €).

Modélisation :
- l'ajustement est une ligne de devis à montant signé (négatif = remise), distincte des frais ;
- réservé à l'administrateur (jamais côté voyageur) ;
- il régénère un devis figé (`QuoteSnapshot`) incluant la ligne d'ajustement ;
- le détail d'origine est conservé (transparence, cf. DEC-007) ;
- l'action est journalisée dans l'activité.

## DEC-013 — Synchronisation externe bloquante à la validation

Prolonge DEC-011 (synchronisation Abritel via import iCal).

- Les événements d'un calendrier externe (Abritel/Vrbo, import iCal) **bloquent** les dates au même titre qu'une réservation ou un blocage manuel, sans être des réservations Le 115.
- **L'approbation d'une demande déclenche d'abord une synchronisation fraîche** des sources externes activées, puis vérifie la disponibilité. Objectif : **jamais de sur-réservation** avec Abritel.
- **Si la synchronisation échoue** (source injoignable), l'approbation est **refusée** avec un message explicite ; l'administrateur réessaie une fois la source de nouveau joignable. On préfère bloquer plutôt que valider sur un calendrier potentiellement périmé.
- Côté voyageur, la disponibilité affichée reflète le **dernier** sync (le temps réel n'est pas requis).
- V1 : déclenchement **manuel** de la synchronisation (pas de planification automatique) ; le dashboard affiche l'état/la date du dernier sync.

## DEC-014 — Dashboard = SPA séparée, déploiement même origine que l'API

Le dashboard admin est développé comme une **application front séparée**
(Vite / React / TypeScript, dépôt `le115-dashboard`), distincte du site
public. Il consomme l'API backend existante (`/api/admin/...`) sans jamais
réimplémenter de logique métier côté front.

Topologie retenue : **même origine en production**. Le bundle statique du
dashboard (`dist/`) est servi **derrière le même reverse-proxy / même
domaine** que l'API — pas de sous-domaine ni d'hébergement séparé pour la
V1.

Conséquences :
- le cookie de session admin reste `SameSite=Lax` (déjà en place côté
  backend), sans configuration `SameSite=None` ni CORS à ouvrir ;
- en développement, un proxy du serveur de dev (Vite) reproduit la même
  origine vers le backend local, pour un comportement identique à la
  production ;
- l'écran d'accueil du dashboard V1 est le **Tableau de bord** (synthèse),
  pas directement le Calendrier — cf. remarque sous DEC-006 ;
- la navigation V1 du dashboard compte douze entrées : Tableau de bord,
  Calendrier, Demandes, Réservations, Tarifs, Règles de séjour, Maison,
  Équipements, FAQ, Photos, Synchronisations, Activité (détail :
  `04-Dashboard.md`) ;
- le chrome du dashboard (navigation, libellés d'interface) est **en
  français uniquement** — aucune internationalisation de l'interface admin
  elle-même ; seuls les contenus édités (site public) restent bilingues
  FR/EN (cf. DEC-010).

## DEC-015 — Périmètre du dashboard V1 : mono-bien

Le dashboard V1 est conçu et livré pour **un seul bien** (Le 115), sur le
backend existant, lui-même mono-bien en V1.

Conséquence :
- aucune notion de sélection de bien/logement dans l'interface V1 ;
- les modules imaginés au-delà de ce périmètre (multi-bien, gestion de
  logements, utilisateurs multiples, etc.) sont **reportés post-V1** —
  cf. `04-Dashboard.md` (« Modules reportés post-V1 ») et `08-Roadmap.md`.

## DEC-016 — Équipements et FAQ : écrans dédiés dans le dashboard (2026-08-01)

Les équipements et la FAQ quittent le module `Maison` du dashboard admin pour
devenir chacun un **module à part entière**, avec son propre écran et sa
propre entrée de menu (`/equipements`, `/faq`). L’écran `Maison` (`/maison`)
se limite désormais aux informations principales du bien ; Photos y reste la
seule extension prévue, dans un second temps.

Portée et limites :
- décision **limitée au dashboard admin** — c’est une réorganisation
  d’écrans, pas un changement de modèle de données ni de contrat API ;
- le **site public n’est pas concerné** : `GET /api/public/property` continue
  de renvoyer équipements et FAQ dans le même payload, sans changement ;
- l’opportunité d’une **page FAQ dédiée côté site public** reste une question
  ouverte, à trancher quand le site visiteur sera cadré ;
- la navigation V1 du dashboard passe de huit à dix entrées (cf. DEC-014,
  mis à jour en conséquence).

## DEC-017 — Photos : écran dédié dans le dashboard (2026-08-02)

Prolongement de DEC-016. Les photos du bien ont leur propre écran `/photos` et
leur entrée de menu, plutôt qu’une seconde section de « Maison ». Motif :
galerie, ajout en lot, catégories et réordonnancement forment le plus lourd des
écrans de contenu ; les empiler sous les informations du bien recréerait la page
longue dont Équipements et FAQ viennent de sortir.

Conséquence : **« Maison » se limite définitivement aux informations du bien** —
photos était sa dernière extension prévue. La navigation passe à onze entrées.

**Portée : écran d’administration seul.** Le site visiteur n’est pas concerné :
`GET /api/public/property` continue de servir les photos avec le reste du
contenu.

## DEC-018 — Tableau de bord : poste de pilotage, sans chiffres d’agrégation (2026-08-05)

L’écran d’accueil du dashboard répond à « qu’est-ce qui m’attend » et non à
« comment se porte l’affaire ». Il affiche quatre cartes — demandes en attente,
arrivées et départs, dernières actions, état des synchronisations — et **aucun
indicateur d’agrégation**.

Sont donc **reportés post-V1**, sans être annulés : taux d’occupation, chiffre
d’affaires, nombre de nouvelles réservations, donut d’occupation, courbe de
revenus, sélecteur de période et comparaisons « vs période précédente » — tous
présents dans la maquette `assets/dashboard_v1.png` (écran 01).

Motif : sur un bien unique dont le volume annuel se compte en dizaines de
séjours, une comparaison période à période est du bruit statistique présenté
comme une mesure. Le propriétaire ouvre ce dashboard pour **traiter** quelque
chose.

Le calendrier détaillé n’est pas repris sur l’accueil : il reste à un clic dans
la navigation.

## DEC-019 — Les règles de séjour ont leur écran, et leur aperçu ne résout rien (2026-08-06)

**Décision.** Les règles de séjour obtiennent une entrée de navigation dédiée
(`/regles-de-sejour`, douzième), et non une section des Tarifs. L’écran porte un aperçu annuel
en **bandes superposées** : il montre les périodes et leurs priorités, sans jamais calculer
quelle règle l’emporte à une date donnée.

**Pourquoi.** Résoudre la priorité côté interface dupliquerait une règle métier qui vit dans
le domaine serveur. Deux implémentations d’une même règle divergent tôt ou tard, et l’aperçu
mentirait alors sur ce que le visiteur subit réellement. Les bandes superposées, triées par
priorité et accompagnées d’une légende, donnent la même lecture sans créer cette seconde
source de vérité.

**Conséquence.** Si les chevauchements se révèlent illisibles à l’usage, la réponse sera un
endpoint de résolution côté serveur — pas un calcul côté interface.

## DEC-020 — Direction graphique du site public : l’hybride des deux maquettes (2026-08-15)

**Décision.** Le site public visiteur retient la **mise en page de
`landing-reference.png`** — deux niveaux d’en-tête, logo en arche, accent terracotta,
dégradé latéral, note et avis en en-tête, bandeau d’atouts à icônes chevauchant le
visuel — avec trois corrections :

- le bouton principal dit **« Estimer mon séjour »** (DEC-003), et non « Réserver » :
  rien ne se réserve en ligne (DEC-004) ;
- la navigation garde les **ancres réelles** du produit, et non les libellés de la
  maquette (« Commodités », « Les environs ») qui pointent vers des contenus
  inexistants ;
- les **onglets d’audience mènent à des pages** de contenu, pas à un filtre de la
  galerie.

Rendu de référence : [`assets/landing-hybride.html`](assets/landing-hybride.html).

**Pourquoi.** Les deux maquettes ne disent pas la même chose. `front_v1.png` est
fidèle aux décisions déjà prises mais compose le titre en blanc sur photo plein
cadre — or la photo principale réelle est une **cour close en plein jour**, ce qui
imposerait un voile sombre marqué et assombrirait une belle lumière pour pouvoir
écrire dessus. Le dégradé latéral de `landing-reference.png` n’occupe que le tiers
gauche et laisse la photo respirer. À l’inverse, la chaleur de cette seconde maquette
tient largement à ses **convives attablés**, alors que les photos réelles sont
architecturales et vides.

**Écartés.** La direction sobre de `front_v1.png` seule (voile sombre imposé par la
photo de jour) ; la direction chaleureuse appliquée telle quelle (promesses fausses,
contenus inexistants).

## DEC-021 — Aucun tiers sur le site public, donc aucun bandeau de consentement (2026-08-15)

**Décision.** Le site public **ne charge aucun service tiers**. La carte de la section
Localisation est une **image statique** servie par le site lui-même, doublée d’un lien
« Ouvrir dans Google Maps » qui, lui, part chez eux — au clic du visiteur, jamais
avant. Aucune mesure d’audience pour l’instant.

**Pourquoi.** En France, tout tiers déposant des cookies impose un bandeau de
consentement conforme : stockage du choix, blocage des scripts avant acceptation, page
de politique de confidentialité. C’est une fonctionnalité transverse entière, et une
dette de conformité permanente, pour un gain nul en V1.

**Écartés.** OpenStreetMap via Leaflet (sans cookie, mais un îlot JavaScript et des
tuiles peu soignées en zone rurale) ; Google Maps embarqué et une mesure d’audience
(le bandeau et tout ce qu’il traîne).

**Conséquence.** Le jour où une mesure d’audience devient nécessaire, elle rouvre cette
décision : ce n’est pas un réglage, c’est un chantier.

## DEC-022 — Adresse approximative avant confirmation du séjour (2026-08-15)

**Décision.** Le site public situe la maison par son **secteur**, jamais par son
portail : la carte montre la zone, le texte situe sans localiser, et une mention le dit
au visiteur. **L’adresse exacte part par email une fois le séjour confirmé.**

`02-UX.md` laissait le choix au propriétaire (« position approximative ou exacte selon
décision propriétaire ») ; c’est ce choix qui est tranché ici.

**Pourquoi.** Le site publierait sinon, au même endroit, **où** se trouve la maison
**et quand** elle est inoccupée — le calendrier de disponibilité étant public par
nature. Le coût pour le visiteur est nul : à ce stade il veut savoir dans quelle
région il atterrit, pas quelle rue.

**À confirmer par le propriétaire.** Les coordonnées affichées aujourd’hui (téléphone,
email) et l’image de carte du secteur sont des **valeurs de remplacement** posées côté
site pour livrer le socle ; elles doivent être remplacées par les vraies avant toute
mise en ligne (cf. `../le115-backend/docs/DEBTS.md`).

**Le secteur, lui, n’est pas au choix du site.** Il doit dire ce que dit la donnée
saisie par le propriétaire (`property.location`), aujourd’hui « À 15 minutes
d’Avignon ». Le site a un temps annoncé Aix-en-Provence, à environ soixante-dix
kilomètres de là : erreur corrigée le 2026-08-15, cf. `02-UX.md` et `DEBTS.md`.

## DEC-023 — La demande de séjour a sa page, pas une ancre de l’accueil (2026-08-18)

**Décision.** Le parcours de demande vit sur une **URL dédiée**,
`/[langue]/demande`, au même titre que `/contact`, `/informations-pratiques` et les
pages d’audience. L’appel à l’action du site — « Estimer mon séjour » (DEC-003) —
pointe vers elle, et vers elle seule.

**Pourquoi.** Une demande sans URL propre ne se partage pas, ne se référence pas, ne
se reprend pas. Un visiteur qui hésite doit pouvoir s’envoyer le lien, le rouvrir le
lendemain, ou le recevoir de quelqu’un d’autre. Une ancre de l’accueil ne permet
aucun des trois.

**Écartée.** La section sous une ancre `#estimation` de l’accueil, que prévoyait la
maquette d’origine. Cette ancre n’a d’ailleurs jamais existé dans le site : le bouton
principal a pointé un temps vers une cible morte, puis vers la page de contact faute
de mieux.

**Conséquence.** Rien ne change au contrat de DEC-003 ni de DEC-004 : la page ne
réserve pas, elle **demande**. Le parcours s’arrête à « demande envoyée », la
propriétaire arbitre depuis son dashboard et s’engage à répondre sous 48 h.

## DEC-024 — Le formulaire n’apparaît qu’une fois le devis soumissible (2026-08-18)

**Décision.** Le visiteur choisit d’abord ses **dates** et son nombre de
**voyageurs** ; le devis se met à jour à chaque changement ; **les coordonnées ne
sont demandées que lorsque le serveur déclare le devis soumissible.** Tant qu’une
règle de séjour est violée, le devis reste affiché — avec son détail et son obstacle
— mais le formulaire n’est pas là.

**Pourquoi.** Réclamer nom, email et téléphone pour un séjour que les règles
refuseront est une friction gratuite, et une déception à retardement. Le backend
porte déjà le verdict (`submittable` sur le devis) : le site l’**affiche**, il ne le
décide pas.

**Écartés.** Tout afficher d’emblée (le visiteur remplit ses coordonnées, puis se
voit refuser) ; un assistant en étapes numérotées (plus lourd, et il cache le devis
pendant qu’on choisit ses dates).

**Conséquence.** DEC-005 tient toujours : adultes et enfants restent facultatifs au
sens où le visiteur n’a rien à saisir pour obtenir une estimation — ils sont
présélectionnés. Mais un nombre de voyageurs supérieur à la capacité du bien rend le
devis non soumissible, donc **`max_guests` doit être juste**. Il n’est éditable nulle
part aujourd’hui : dette consignée dans `../le115-backend/docs/DEBTS.md`, remontée en
priorité par cette décision.

## DEC-025 — Horizon de réservation : douze mois glissants (2026-08-18)

**Décision.** Le calendrier de disponibilité montre et laisse choisir **de demain
jusqu’à un an**. Le passé n’est pas sélectionnable ; au-delà de douze mois, rien
n’est proposé.

**Pourquoi.** Les périodes tarifaires sont saisies par la propriétaire saison par
saison, et elles ne le sont pas au-delà d’un an. Un devis calculé si loin
retomberait sur le seul prix de base et **annoncerait un montant qui ne sera pas le
bon** — un chiffre faux vaut moins qu’une absence de chiffre.

**Écarté.** Un horizon ouvert, ou un horizon fixe à l’année civile (qui rétrécirait
jusqu’à quelques semaines en décembre).

**Conséquence.** L’horizon suit la date du jour, il ne se règle nulle part. Le jour
où la propriétaire saisira ses tarifs plus loin, c’est cette décision qu’il faudra
rouvrir — pas un paramètre.

## DEC-026 — L’appel à l’action devient « Réserver », la page détrompe (2026-08-19)

**Décision.** Le CTA principal du site public — bouton d’en-tête et bouton du
hero — dit désormais **« Réserver »** (« Book » en anglais), et non plus le
libellé retenu depuis DEC-003 (« Estimer mon séjour » sur ce document,
« Demander une estimation » dans le code livré — les deux jamais recalés
l’un sur l’autre). Il mène toujours à `/[langue]/demande` (DEC-023), dont le
titre devient **« Réserver votre séjour »** et dont la première phrase dit
d’emblée : la demande est étudiée sous 48 heures, rien n’est engagé tant
qu’elle n’est pas acceptée.

**Pourquoi.** Arbitrage du propriétaire : « Réserver » attire davantage
qu’une formule prudente, et un bouton qui n’invite pas franchement à l’action
convertit moins.

**Ça ne contredit ni DEC-003 ni DEC-004.** Les deux tiennent : rien ne se
réserve en ligne, la propriétaire arbitre chaque demande depuis son
dashboard. Aucune étape du parcours ne change de sens — le bouton d’envoi dit
toujours « Envoyer ma demande », la confirmation dit toujours que la demande
est envoyée, pas acceptée. Le nuancier retenu : le bouton **attire**, la page
qu’il ouvre **détrompe aussitôt**, avant même le calendrier.

**Risque assumé.** Un visiteur qui clique « Réserver » puis lit dans la même
respiration « rien n’est engagé » peut se sentir découragé, voire dupé — un
« Demander une estimation » ne créait pas cette attente à détromper. C’est le
prix accepté pour un bouton plus engageant ; à surveiller si le taux
d’abandon sur `/demande` augmente après ce changement.

**Écarté.** Garder le libellé prudent (fidèle à DEC-003, mais jugé moins
engageant) ; un intermédiaire du type « Faire une demande », écarté par le
propriétaire au profit de « Réserver ».

**Conséquence.** `nav.cta` et `hero.cta` valent désormais le même texte dans
les deux langues côté site — la distinction libellé court / libellé long,
qui existait pour tenir la rangée d’en-tête à 320 px, devient sans objet tant
que « Réserver » reste le texte retenu (cf. commentaire dans
`le115-frontend/src/i18n/fr.ts`). `02-UX.md` est mis à jour en conséquence.
