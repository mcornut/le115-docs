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
