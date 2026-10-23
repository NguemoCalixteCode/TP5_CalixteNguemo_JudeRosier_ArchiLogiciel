# Glossaire QueenBee · groupe Data

Binôme : Calixte Nguemo et Jule Rosier · M1 Data

| Terme | Définition validée | Contexte | Dans le code | Source |
|---|---|---|---|---|
| Ruche (actif) | Emplacement physique identifié `PP-SITE-NNN`, rattaché à un site et à une coopérative, avec une position de référence et au plus un capteur actif | Référentiel des ruchers | `Ruche` | UC-C01, UC-CY04 |
| Ruche (source de mesures) | Identifiant `hive_id` porté par une trame ; seule une ruche connue du référentiel produit des mesures validées | Collecte IoT | `hiveId` (champ `hive_id`) | UC-T02 |
| Colonie | Population d'abeilles vivant dans une ruche ; c'est elle qui a un score de santé, pas la boîte | Santé des colonies | `ColonieSuivie` | UC-A02 |
| Site (rucher) | Lieu géographique regroupant des ruches, rattaché à un pays et à des comptes par rôle | Référentiel des ruchers | `Site` | UC-AD01, UC-C01 |
| Site (couverture radio) | Zone couverte par une passerelle LoRaWAN ; un site rucher a une passerelle par site | Collecte IoT | `siteId` sur `Passerelle` | UC-CY01 |
| Mesure | Relevé d'un capteur à un instant donné : poids, températures, humidité, activité acoustique, batterie, position | Collecte IoT | `Mesure` | UC-T02 |
| Trame | Message radio LoRaWAN brut, encore non interprété, qui transporte une ou plusieurs mesures | Collecte IoT | `Trame` (couche anticorruption) | UC-T02, UC-CY02 |
| Mesure validée | Mesure qui a passé les contrôles d'ingestion (ruche connue, plages physiques, non doublon, non rejouée) et qui est publiée aux autres contextes | Collecte IoT | événement `MesureValidée.v1` | UC-T02, UC-CY02 |
| Trame rejetée | Trame refusée à l'ingestion (hors plage physique, rejeu, ruche inconnue), journalisée avec son motif | Collecte IoT | événement `TrameRejetée` | UC-CY02 |
| Passerelle | Équipement LoRaWAN d'un site qui relaie les trames vers le central, avec un cache local en mode dégradé | Collecte IoT | `Passerelle` | UC-CY01, UC-CY03 |
| Mode dégradé | État d'une passerelle sans lien vers le central : elle conserve les mesures en cache et les envoie à la resynchronisation | Collecte IoT | `Passerelle.modeDegrade` | UC-CY01 |
| Incident | Problème d'infrastructure ou d'intégrité (passerelle muette, rejets répétés, brouillage), ouvert puis clos, toujours journalisé | Collecte IoT | `Incident` | UC-CY01 à UC-CY03 |
| Alerte | Signal qu'une **colonie** est en risque (essaimage, affaiblissement, stress thermique, mortalité), levée par la prédiction, prise en charge puis résolue après inspection | Santé des colonies | `Alerte` | UC-A02, UC-A01 |
| Seuil | Valeur limite d'une grandeur pour une ruche (ex. chute de poids sur 24 h) au-delà de laquelle on déclenche une surveillance renforcée | Santé des colonies | `Seuil` | UC-A02, UC-T02 |
| Surveillance renforcée | Cadence de mesure portée à 1 mesure / 5 min pour une ruche, pour une durée limitée | Santé des colonies | `surveillanceRenforcee` | UC-T02 |
| Score de santé | Indicateur de 0 à 100 de l'état d'une colonie, recalculé à chaque mesure validée | Santé des colonies | `scoreSante` | UC-A02, UC-C01 |
| Niveau de confiance | Probabilité, entre 0 et 1, que le risque prédit soit réel | Prévision | `confiance` | UC-A02 |
| Prédiction de rendement | Volume de miel attendu calculé par le modèle pour une ruche et une saison | Prévision | `PredictionRendement` | UC-A03 |
| Escalade | Passage d'une alerte non prise en charge sous 48 h au coordinateur du site | Santé des colonies | `alerte.escalader()` | UC-A02 |
| Visite | Déplacement planifié d'une apicultrice vers une ou plusieurs ruches, assigné par le coordinateur | Interventions terrain | `Visite` | UC-C01 |
| Inspection | Constat fait à la ruche (état visuel, reine, comportement, photo), saisi même hors ligne | Interventions terrain | `Inspection` | UC-A01 |
| Conflit de synchronisation | Deux versions divergentes d'une même inspection arrivées au central | Interventions terrain | `ConflitSynchronisation` | UC-A01 |
| Récolte | Volume de miel déclaré pour une ruche, à une date | Production | `Recolte` | UC-A03 |
| Écart de rendement | Différence, en %, entre la récolte déclarée et la prédiction de rendement | Production | `ecartRendement` | UC-A03, UC-C02 |
| Validation (d'une mesure) | Contrôle automatique d'une mesure à l'ingestion ; aucun humain n'intervient | Collecte IoT | `validerMesure()` | UC-CY02 |
| Validation (d'une récolte) | Acceptation de la déclaration par le coordinateur, après comparaison aux poids mesurés | Production | `recolte.valider()` | UC-C02 |
| Transhumance | Déplacement de ruches **déclaré** par le coordinateur, avec destination et dates | Référentiel des ruchers | `Transhumance` | UC-CY04 |
| Déplacement non déclaré | Position GPS hors tolérance sans transhumance déclarée : suspicion de vol ou d'erreur, alerte prioritaire | Référentiel des ruchers | `DeplacementNonDeclare` | UC-CY04 |
| Indicateur | Valeur consolidée par site et par pays (rendement, colonies à risque, couverture réseau) | Impact | `Indicateur` | UC-G01 |
| Rapport d'impact | Document PDF sur une période, des sites et des indicateurs, destiné à un bailleur | Impact | `RapportImpact` | UC-G02 |
| Bailleur | Organisme qui finance BeeAfrica et reçoit les rapports d'impact | Impact | `Bailleur` | UC-G02 |
| Coopérative | Groupement d'apicultrices auquel appartiennent des ruches et qui est notifié des récoltes | Référentiel des ruchers | `Cooperative` | UC-A03 |

32 termes, dont 4 mots à plusieurs sens.

## Mots à plusieurs sens

- **« Alerte »** : sur le mur, le mot désignait à la fois le risque d'une colonie (UC-A02) et
  une passerelle muette ou des trames rejetées (UC-CY01 à CY03). Ce n'est ni le même cycle de vie
  (levée → prise en charge → escaladée → résolue contre ouvert → clos), ni les mêmes personnes
  (apicultrice contre administrateur), ni les mêmes règles (48 h contre 15 min). **Décision : un
  seul sens.** « Alerte » est réservé à la colonie, dans Santé des colonies. Côté infrastructure,
  on dit **« Incident »** (incident de site, d'intégrité, de brouillage), dans Collecte IoT.
  L'« alerte prioritaire » de UC-CY04 devient l'événement « Déplacement non déclaré détecté ».
- **« Ruche »** : actif physique dans Référentiel des ruchers ; simple source `hive_id` dans
  Collecte IoT ; dans Santé des colonies on parle de la **colonie**. Décision : trois termes,
  l'identifiant `hive_id` est la seule chose partagée.
- **« Site »** : rucher géographique (Référentiel) contre zone couverte par une passerelle
  (Collecte IoT). Aujourd'hui une passerelle par site, donc les identifiants coïncident ; on garde
  deux lignes pour le jour où un site aura deux passerelles.
- **« Validation »** : contrôle automatique d'une mesure (Collecte IoT) contre décision humaine
  du coordinateur sur une récolte (Production). Décision : « Mesure validée » et « Récolte
  validée » sont deux événements de deux contextes, jamais un même service de validation.
