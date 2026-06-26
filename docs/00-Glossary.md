# 00 - Glossary

Ce document fixe le vocabulaire métier utilisé dans toute la documentation et dans le code.

## Property

La maison louée.

Même si la V1 ne gère qu'une seule maison, le domaine utilise le terme `Property` pour conserver une modélisation propre.

## Traveler

Visiteur intéressé par un séjour.

## Stay Request

Demande de séjour envoyée par un voyageur.

Une `Stay Request` n'est pas une réservation confirmée. Elle ne bloque pas automatiquement les dates.

## Reservation

Séjour confirmé manuellement par le propriétaire.

Une réservation bloque les dates dans le calendrier.

## Availability

Disponibilité d'une période donnée.

Elle dépend :
- des réservations confirmées ;
- des dates bloquées par le propriétaire ;
- des règles de séjour.

## Quote

Estimation tarifaire calculée pour une période.

Un devis contient :
- dates ;
- nombre de nuits ;
- prix par nuit ;
- frais additionnels ;
- total.

## Pricing Period

Période tarifaire définissant un prix par nuit.

## Calendar Block

Blocage manuel d'une période par le propriétaire.

Exemples :
- usage personnel ;
- travaux ;
- maintenance ;
- fermeture exceptionnelle.

## Owner

Administrateur du site et propriétaire de la maison.
