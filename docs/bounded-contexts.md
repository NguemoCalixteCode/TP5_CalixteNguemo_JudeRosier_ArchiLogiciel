# Bounded contexts QueenBee · groupe Data

Binôme : Calixte Nguemo et Jule Rosier · M1 Data

Huit contextes. Les frontières suivent les mots qui changent de sens (voir le glossaire) :
« Alerte » contre « Incident » sépare Santé des colonies de Collecte IoT ; « Ruche » actif contre
source de mesures sépare Référentiel de Collecte ; « Validation » sépare Collecte de Production.

| Contexte | Type | Spécialité la plus concernée |
|---|---|---|
| Santé des colonies | cœur | Dev |
| Prévision | cœur | Data |
| Production | cœur | Dev |
| Collecte IoT | support | Data, Cyber |
| Interventions terrain | support | Dev |
| Référentiel des ruchers | support | Data |
| Impact | support | Data |
| Identité et accès | générique | Cyber |

## Santé des colonies
- Responsabilité : savoir à tout instant quelles colonies sont en danger et faire agir quelqu'un à temps.
- Type : cœur (c'est ce qui distingue QueenBee d'un simple tableau de bord de capteurs) · Dev
- Agrégats : Alerte (type de risque connu, confiance dans [0 ; 1], résolution après prise en charge et inspection, escalade après 48 h, aucune transition après résolution) ; Colonie suivie (un seul score courant, surveillance renforcée bornée dans le temps)
- Événements publiés : Seuil dépassé, Surveillance renforcée activée, Score de santé recalculé, AlerteLevée.v1, AlertePriseEnCharge, AlerteEscaladée.v1, AlerteRésolue
- Événements consommés : MesureValidée.v1 (Collecte IoT), Risque d'essaimage / d'affaiblissement détecté (Prévision), Inspection synchronisée (Interventions terrain)
- Cas d'utilisation sources : UC-A02, UC-A01 (résolution), UC-C01 (classement)
- Mots propres : alerte, colonie, seuil, score de santé, escalade, prise en charge

## Prévision
- Responsabilité : transformer les mesures et la météo en prédictions de risque et de rendement, avec un niveau de confiance.
- Type : cœur (le modèle est l'avantage différenciant de BeeAfrica) · Data
- Agrégats : Prédiction de risque, Prédiction de rendement (chaque prédiction cite la version du modèle et la fenêtre de données ; une prédiction publiée n'est jamais modifiée)
- Événements publiés : Risque d'essaimage détecté, Risque d'affaiblissement détecté (RisqueDétecté.v1), Prédiction de rendement calculée
- Événements consommés : MesureValidée.v1 (Collecte IoT), Données météo reçues (OpenWeatherMap, via ACL), Récolte validée (Production, pour réentraîner le modèle de rendement), Inspection synchronisée (étiquettes de vérité terrain)
- Cas d'utilisation sources : UC-A02, UC-A03
- Mots propres : prédiction, confiance, fenêtre de données, version du modèle, variable explicative

## Production
- Responsabilité : enregistrer les récoltes et garantir qu'un volume validé est fiable.
- Type : cœur · Dev
- Agrégats : Récolte (volume positif, récolte validée immuable, écart signalé avec commentaire)
- Événements publiés : Récolte déclarée, Écart de rendement calculé, Récolte validée, Écart mis en investigation
- Événements consommés : Prédiction de rendement calculée (Prévision), MesureValidée.v1 (poids mesurés, Collecte IoT)
- Cas d'utilisation sources : UC-A03, UC-C02
- Mots propres : récolte, écart de rendement, validation (d'une récolte), coopérative notifiée

## Collecte IoT
- Responsabilité : recevoir les trames des capteurs via les passerelles, les contrôler et publier des mesures fiables ; surveiller l'infrastructure radio.
- Type : support (indispensable mais sans règle métier apicole) · Data et Cyber
- Agrégats : Série de mesures par ruche (déduplication, plages physiques, horodatage plausible) ; Passerelle (un site, un canal actif, battement ≤ 15 min) ; Incident (clos = définitif, toujours journalisé)
- Événements publiés : Mesure reçue, **MesureValidée.v1** (published language), Trame rejetée, Capteur devenu silencieux, Passerelle hors ligne détectée, Mode dégradé activé, Passerelle resynchronisée, Lot de mesures différées reçu, Incident de site ouvert / clos, Incident d'intégrité ouvert, Brouillage détecté, Changement de canal proposé, Canal changé, Position hors tolérance détectée
- Événements consommés : Ruche enregistrée, Capteur associé, Transhumance déclarée (Référentiel des ruchers) ; Surveillance renforcée activée (Santé des colonies, pour changer la cadence)
- Cas d'utilisation sources : UC-T02, UC-CY01, UC-CY02, UC-CY03, UC-CY04
- Mots propres : trame, mesure validée, passerelle, mode dégradé, incident, canal, rejeu

## Interventions terrain
- Responsabilité : organiser les visites et recueillir les inspections, y compris hors ligne.
- Type : support · Dev
- Agrégats : Visite / Plan de visites (assignée à une personne ayant un rôle sur le site ; visite réalisée non replanifiable) ; Inspection (porte sur une ruche existante ; inspection en conflit inutilisable avant arbitrage)
- Événements publiés : Ruches classées par risque, Visite planifiée, Visite assignée, Visite anticipée demandée, Inspection enregistrée hors ligne, Inspection synchronisée, Conflit de synchronisation détecté / résolu, Anomalie constatée, Ruche introuvable signalée, Visite réalisée
- Événements consommés : AlerteLevée.v1, AlerteEscaladée.v1, Score de santé recalculé (Santé des colonies)
- Cas d'utilisation sources : UC-A01, UC-C01
- Mots propres : visite, inspection, synchronisation, conflit, anomalie

## Référentiel des ruchers
- Responsabilité : être la source de vérité sur les sites, ruches, capteurs, coopératives et leurs positions.
- Type : support · Data (données de référence)
- Agrégats : Site (un pays ; au moins une passerelle si actif) ; Ruche (identifiant unique `PP-SITE-NNN`, un capteur actif au plus, transhumance datée)
- Événements publiés : Site rucher créé, Ruche enregistrée, Capteur associé à la ruche, Transhumance déclarée, Ruche déplacée, Déplacement non déclaré détecté
- Événements consommés : Position hors tolérance détectée (Collecte IoT)
- Cas d'utilisation sources : UC-C01, UC-CY04, UC-AD01 (création des sites)
- Mots propres : ruche (actif), site (rucher), coopérative, transhumance, position de référence

## Impact
- Responsabilité : consolider les indicateurs multi-sites et produire les rapports pour les bailleurs.
- Type : support · Data
- Agrégats : Tableau d'indicateurs (un indicateur par site, par jour et par définition versionnée) ; Rapport d'impact (période cohérente ; rapport envoyé figé)
- Événements publiés : Indicateurs consolidés, Période de rapport choisie, Rapport d'impact généré, Rapport téléchargé, Rapport envoyé au bailleur
- Événements consommés : MesureValidée.v1 et Incidents (couverture réseau, Collecte IoT), Récolte validée (Production), AlerteLevée / AlerteRésolue (Santé des colonies)
- Cas d'utilisation sources : UC-G01, UC-G02
- Mots propres : indicateur, période, rapport d'impact, bailleur

## Identité et accès
- Responsabilité : authentifier les personnes et dire qui a quel rôle sur quel site.
- Type : générique (un fournisseur d'identité standard ferait l'affaire) · Cyber
- Agrégats : Compte (rôle rattaché à un site ; compte bloqué ou désactivé refuse l'accès ; MFA pour coordinateur, gestionnaire, admin)
- Événements publiés : Compte créé, Rôle assigné sur un site, Compte désactivé, Utilisateur authentifié, Authentification échouée, Compte bloqué
- Événements consommés : aucun
- Cas d'utilisation sources : UC-AD01, UC-T01
- Mots propres : compte, rôle, permission, jeton hors ligne, MFA

## Systèmes externes
- **Capteurs et passerelles LoRaWAN** : en amont de Collecte IoT, derrière une couche anticorruption (trame binaire → Mesure).
- **OpenWeatherMap** : en amont de Prévision, derrière une couche anticorruption (format OWM → Relevé météo de site).
- **Service de messagerie** (push, SMS, e-mail) : en aval de Santé des colonies, Interventions terrain et Production, via un port « Notifier ».
- **Bailleurs** (e-mail) : destinataires des rapports d'Impact.
