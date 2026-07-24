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
- la navigation V1 du dashboard compte huit entrées : Tableau de bord,
  Calendrier, Demandes, Réservations, Tarifs, Maison, Synchronisations,
  Activité (détail : `04-Dashboard.md`) ;
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
