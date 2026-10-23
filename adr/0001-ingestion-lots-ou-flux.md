# ADR 0001 · Ingestion des mesures : traitement par lots ou en flux ?
- Date : 2026-10-23 · Auteurs : Calixte Nguemo, Jule Rosier (M1 Data)

## Contexte
Collecte IoT reçoit les mesures des 400 ruches via les passerelles LoRaWAN des 8 sites et publie
`MesureValidée.v1` vers Santé des colonies, Prévision, Production et Impact
(voir `docs/specialite/data-contrat-mesure-validee.md`).

Contraintes du cahier des charges et du mur :
- **Latence des alertes** : un risque d'essaimage doit atteindre l'apicultrice vite (UC-A02) ;
  la politique d'escalade est à 48 h, mais la détection elle-même doit se faire en minutes.
- **Connectivité intermittente** : passerelles souvent en différé, mode dégradé avec cache local
  et resynchronisation (UC-CY01) ; les mesures arrivent en désordre et par rafales.
- **Volumétrie** : 38 400 mesures / jour en nominal (400 × 96), soit 0,44 / s en moyenne ;
  jusqu'à environ 3 fois plus si des ruches passent en surveillance renforcée (1 / 5 min).
  Pic de resynchronisation : un site de 50 ruches coupé 24 h renvoie 4 800 mesures d'un coup.
- **Intégrité** : déduplication et détection du rejeu à l'ingestion (UC-CY02).
- Points chauds d'origine : n° 4 (retard maximal accepté) et n° 5 (alerte sur donnée différée).

## Options envisagées
1. **Lots planifiés** (par exemple toutes les heures, ou chaque nuit) : les passerelles déposent
   des fichiers, un traitement les valide et publie.
   - Avantages : simple à opérer, rejouable, naturel pour Impact et le réentraînement de Prévision.
   - Inconvénients : latence d'alerte égale à la période du lot (jusqu'à 1 h ou plus) ; un lot
     raté retarde toutes les ruches d'un site.
2. **Flux événementiel** : chaque trame est validée à l'arrivée et `MesureValidée` est publiée
   sur un bus d'événements (journal persistant, consommateurs abonnés).
   - Avantages : latence de quelques secondes ; les rafales de resynchronisation sont absorbées
     par le journal ; chaque consommateur avance à son rythme et peut rejouer l'historique.
   - Inconvénients : il faut gérer le désordre (tri par `occurredAt`), l'idempotence et
     l'exploitation d'un broker ; plus de compétences requises.
3. **Hybride** : flux pour la validation et la publication de `MesureValidée` (Santé des colonies),
   plus des agrégations par lots sur le journal pour Impact et Prévision (consolidation nocturne,
   jeux d'entraînement).

## Critères de choix
| Critère | Poids | Lots | Flux | Hybride |
|---|---|---|---|---|
| Latence des alertes | fort | – – | + + | + + |
| Tolérance à la connectivité intermittente | fort | + | + (journal, idempotence) | + + |
| Volumétrie (0,44 / s, pics de 4 800) | faible : les trois tiennent | + | + | + |
| Simplicité d'exploitation | moyen | + + | – | – |
| Besoins analytiques (Impact, Prévision) | moyen | + + | + | + + |

## Décision (pressentie)
Option 3, hybride : ingestion et publication en flux, idempotentes par `(hive_id, timestamp)`,
sur un journal d'événements conservé au moins 90 jours ; consolidations par lots pour Impact et
Prévision lues sur ce même journal. La volumétrie ne justifie pas une plateforme de streaming
lourde : un broker simple (file persistante) suffit. À confirmer en séance 6.

## Conséquences
- Plus facile : alertes en minutes ; rejeu d'historique pour entraîner les modèles ; un seul
  contrat `MesureValidée.v1` pour tous.
- Plus difficile : traitement des mesures hors ordre et des alertes rétroactives
  (`detectedLate`) ; supervision du broker.
- À surveiller : écart `receivedAt − occurredAt` par site, taux de doublons et de rejets,
  retard des consommateurs sur le journal.
