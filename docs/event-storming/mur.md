# Mur d'Event Storming QueenBee · groupe Data

Binôme : **Calixte Nguemo** et **Jule Rosier** · M1 Data · séance du 23/10/2026

Ce fichier est la transcription du mur (étapes B1 à B4 et C). Le tableau Miro
`QueenBee · ES · Data-<n° de groupe>` en est la version visuelle ; son export PDF
remplace `mur-complet.jpg`. Les instructions pour construire le tableau sont dans
[`../../miro/guide-tableau-miro.md`](../../miro/guide-tableau-miro.md).

Légende : 🟧 événement · 🟦 commande · 🟨 acteur · 🟨 agrégat (jaune vif) · 🟪 politique ·
🟪 système externe (rose) · 🟥 point chaud · ★ événement pivot.

## B2 · Chronologie (65 événements, 6 phases)

Chaque ligne donne l'événement et sa cause (B3), puis l'agrégat qui garantit les règles (B4)
et le contexte (C) auquel le post-it appartient.

### Phase 1 · Avant la saison : comptes, sites, ruches

| # | Événement 🟧 | Cause : acteur 🟨 + commande 🟦, système 🌸 ou politique 🟪 | Agrégat | Contexte | UC |
|---|---|---|---|---|---|
| 1 | Compte créé | Administrateur · « Créer un compte » | Compte | Identité et accès | UC-AD01 |
| 2 | Rôle assigné sur un site | Administrateur · « Assigner un rôle » | Compte | Identité et accès | UC-AD01 |
| 3 | Compte désactivé | Administrateur · « Désactiver un compte » | Compte | Identité et accès | UC-AD01 |
| 4 | Utilisateur authentifié | Tous les rôles · « S'authentifier » (MFA si coordinateur, gestionnaire, admin) | Compte | Identité et accès | UC-T01 |
| 5 | Authentification échouée | Tous les rôles · « S'authentifier » | Compte | Identité et accès | UC-T01 |
| 6 | Compte bloqué | 🟪 Quand 5 authentifications échouées consécutives, alors bloquer le compte | Compte | Identité et accès | UC-T01 |
| 7 | Site rucher créé | Administrateur · « Créer un site » | Site | Référentiel des ruchers | UC-AD01 |
| 8 | Ruche enregistrée | Coordinateur · « Enregistrer une ruche » | Ruche | Référentiel des ruchers | UC-C01 |
| 9 | Capteur associé à la ruche | Coordinateur · « Associer un capteur » | Ruche | Référentiel des ruchers | UC-T02 |
| 10 | Passerelle mise en service | Administrateur · « Mettre en service une passerelle » | Passerelle | Collecte IoT | UC-CY01 |
| 11 | Transhumance déclarée | Coordinateur · « Déclarer une transhumance » | Ruche | Référentiel des ruchers | UC-CY04 |

### Phase 2 · En continu : mesures et passerelles

| # | Événement 🟧 | Cause | Agrégat | Contexte | UC |
|---|---|---|---|---|---|
| 12 | Mesure relevée | 🌸 Capteur de ruche (toutes les 15 min, ou immédiatement si seuil local dépassé) | – | Collecte IoT | UC-T02 |
| 13 | Trame transmise à la passerelle | 🌸 Capteur → passerelle LoRaWAN | – | Collecte IoT | UC-T02 |
| 14 | Mesure reçue | 🌸 Passerelle LoRaWAN (souvent en différé) | Série de mesures | Collecte IoT | UC-T02 |
| 15 | **★ Mesure validée** | 🟪 Quand mesure reçue, alors valider (ruche connue, plages physiques, doublon, rejeu) | Série de mesures | Collecte IoT | UC-T02, UC-CY02 |
| 16 | Trame rejetée | 🟪 même politique : hors plage physique ou déjà reçue | Série de mesures | Collecte IoT | UC-CY02 |
| 17 | Incident d'intégrité ouvert | 🟪 Si plus de 5 trames rejetées en 1 h pour une même ruche, alors ouvrir un incident d'intégrité | Incident | Collecte IoT | UC-CY02 |
| 18 | Capteur devenu silencieux | 🟪 Si aucune mesure d'un capteur depuis 4 cycles alors que sa passerelle est en ligne, alors le déclarer silencieux | Série de mesures | Collecte IoT | UC-T02 |
| 19 | **★ Passerelle hors ligne détectée** | 🟪 Si aucun battement de cœur depuis 15 min, alors déclarer la passerelle hors ligne | Passerelle | Collecte IoT | UC-CY01 |
| 20 | Mode dégradé activé | 🌸 Passerelle LoRaWAN (cache local) | Passerelle | Collecte IoT | UC-CY01 |
| 21 | Incident de site ouvert | 🟪 Quand passerelle hors ligne détectée, alors ouvrir un incident de site et notifier le coordinateur | Incident | Collecte IoT | UC-CY01 |
| 22 | Passerelle resynchronisée | 🌸 Passerelle LoRaWAN au rétablissement du lien | Passerelle | Collecte IoT | UC-CY01 |
| 23 | Lot de mesures différées reçu | 🟪 Quand passerelle resynchronisée, alors vider le cache local vers le central | Série de mesures | Collecte IoT | UC-CY01 |
| 24 | Incident de site clos | 🟪 Quand passerelle resynchronisée et lot reçu, alors clore l'incident | Incident | Collecte IoT | UC-CY01 |
| 25 | Brouillage détecté | 🟪 Si le taux de trames perdues sur un canal dépasse le seuil, alors déclarer un brouillage | Passerelle | Collecte IoT | UC-CY03 |
| 26 | Changement de canal proposé | 🟪 Quand brouillage détecté, alors proposer un canal de repli | Passerelle | Collecte IoT | UC-CY03 |
| 27 | Canal changé | Administrateur · « Changer de canal » | Passerelle | Collecte IoT | UC-CY03 |
| 28 | Incident journalisé | 🟪 Quand un incident (site, intégrité, brouillage) est ouvert ou clos, alors le journaliser | Incident | Collecte IoT | UC-CY01 à 03 |
| 29 | Position hors tolérance détectée | 🟪 Quand mesure validée et position éloignée de la position de référence, alors signaler | Série de mesures | Collecte IoT | UC-CY04 |
| 30 | **★ Déplacement non déclaré détecté** | 🟪 Quand position hors tolérance et aucune transhumance déclarée, alors déclarer un déplacement non déclaré (alerte prioritaire coordinateur + admin) | Ruche | Référentiel des ruchers | UC-CY04 |
| 31 | Ruche déplacée | 🟪 Quand position hors tolérance et transhumance déclarée, alors mettre à jour la position de référence | Ruche | Référentiel des ruchers | UC-CY04 |

### Phase 3 · Quand un risque apparaît : détection et alerte

| # | Événement 🟧 | Cause | Agrégat | Contexte | UC |
|---|---|---|---|---|---|
| 32 | Données météo reçues | 🌸 OpenWeatherMap (relevé horaire par site) | – | Prévision | UC-A02 |
| 33 | Seuil dépassé | 🟪 Quand mesure validée, alors comparer aux seuils de la ruche | Colonie suivie | Santé des colonies | UC-A02 |
| 34 | Surveillance renforcée activée | 🟪 Quand seuil dépassé, alors passer la ruche à 1 mesure / 5 min pendant 24 h | Colonie suivie | Santé des colonies | UC-A02 |
| 35 | Score de santé recalculé | 🟪 Quand mesure validée, alors recalculer le score de la colonie | Colonie suivie | Santé des colonies | UC-A02 |
| 36 | Risque d'essaimage détecté | 🌸 Modèle de prédiction | Prédiction de risque | Prévision | UC-A02 |
| 37 | Risque d'affaiblissement détecté | 🌸 Modèle de prédiction | Prédiction de risque | Prévision | UC-A02 |
| 38 | **★ Alerte levée** | 🟪 Quand risque détecté avec une confiance ≥ seuil, alors lever une alerte | Alerte | Santé des colonies | UC-A02 |
| 39 | Apicultrice notifiée | 🟪 Quand alerte levée, alors notifier l'apicultrice (push, ou au prochain accès) | – | Santé des colonies | UC-A02 |
| 40 | Alerte prise en charge | Apicultrice · « Prendre en charge l'alerte » | Alerte | Santé des colonies | UC-A02 |
| 41 | Visite anticipée demandée | Apicultrice · « Demander une visite anticipée » | Visite | Interventions terrain | UC-A02 |
| 42 | **★ Alerte escaladée** | 🟪 Si aucune prise en charge sous 48 h, alors escalader | Alerte | Santé des colonies | UC-A02 |
| 43 | Coordinateur notifié de l'escalade | 🟪 Quand alerte escaladée, alors notifier le coordinateur du site | – | Santé des colonies | UC-A02 |
| 44 | Alerte résolue | Apicultrice · « Résoudre l'alerte » (avec l'inspection qui la justifie) | Alerte | Santé des colonies | UC-A01, UC-A02 |

### Phase 4 · Sur le terrain : visites et inspections

| # | Événement 🟧 | Cause | Agrégat | Contexte | UC |
|---|---|---|---|---|---|
| 45 | Ruches classées par risque | 🟪 Quand score de santé recalculé, alors reclasser les ruches du site | Plan de visites | Interventions terrain | UC-C01 |
| 46 | Visite planifiée | Coordinateur · « Planifier une visite » (criticité + géolocalisation) | Visite | Interventions terrain | UC-C01 |
| 47 | Visite assignée | Coordinateur · « Assigner une visite » | Visite | Interventions terrain | UC-C01 |
| 48 | **★ Inspection enregistrée hors ligne** | Apicultrice · « Enregistrer une inspection » (état, reine, comportement, photo) | Inspection | Interventions terrain | UC-A01 |
| 49 | Ruche introuvable signalée | Apicultrice · « Signaler une ruche introuvable » | Inspection | Interventions terrain | UC-A01 |
| 50 | Inspection synchronisée | 🌸 Application terrain au retour du réseau | Inspection | Interventions terrain | UC-A01 |
| 51 | Conflit de synchronisation détecté | 🟪 Quand deux versions d'une inspection divergent, alors signaler un conflit | Inspection | Interventions terrain | UC-A01 |
| 52 | Conflit de synchronisation résolu | Coordinateur · « Arbitrer le conflit » | Inspection | Interventions terrain | UC-A01 |
| 53 | Anomalie constatée | 🟪 Quand inspection synchronisée avec un état anormal, alors notifier le coordinateur | Inspection | Interventions terrain | UC-A01 |
| 54 | Visite réalisée | 🟪 Quand inspection synchronisée pour une visite assignée, alors clore la visite | Visite | Interventions terrain | UC-C01 |

### Phase 5 · À la récolte

| # | Événement 🟧 | Cause | Agrégat | Contexte | UC |
|---|---|---|---|---|---|
| 55 | **★ Récolte déclarée** | Apicultrice · « Déclarer une récolte » (ruche, volume, date) | Récolte | Production | UC-A03 |
| 56 | Prédiction de rendement calculée | 🌸 Modèle de prédiction (par ruche et par saison) | Prédiction de rendement | Prévision | UC-A03 |
| 57 | Écart de rendement calculé | 🟪 Quand récolte déclarée, alors comparer à la prédiction | Récolte | Production | UC-A03 |
| 58 | Coopérative et coordinateur notifiés | 🟪 Quand écart de rendement calculé, alors notifier coopérative et coordinateur | – | Production | UC-A03 |
| 59 | Récolte validée | Coordinateur · « Valider la récolte » (après comparaison aux poids mesurés) | Récolte | Production | UC-C02 |
| 60 | Écart mis en investigation | Coordinateur · « Signaler un écart » | Récolte | Production | UC-C02 |

### Phase 6 · En fin de période : indicateurs et rapports

| # | Événement 🟧 | Cause | Agrégat | Contexte | UC |
|---|---|---|---|---|---|
| 61 | Indicateurs consolidés | 🟪 Chaque nuit, consolider rendement, colonies à risque, couverture réseau par site et par pays | Tableau d'indicateurs | Impact | UC-G01 |
| 62 | Période de rapport choisie | Gestionnaire · « Choisir période, sites et indicateurs » | Rapport d'impact | Impact | UC-G02 |
| 63 | **★ Rapport d'impact généré** | 🟪 Quand période choisie, alors générer le PDF | Rapport d'impact | Impact | UC-G02 |
| 64 | Rapport téléchargé | Gestionnaire · « Télécharger le rapport » | Rapport d'impact | Impact | UC-G02 |
| 65 | Rapport envoyé au bailleur | Gestionnaire · « Envoyer au bailleur » | Rapport d'impact | Impact | UC-G02 |

Total : 65 événements (au-delà des 40 demandés), dont 8 pivots ★, 31 politiques, 5 systèmes externes.

## Zone « Écartés » (étape B2)

| Post-it retiré | Raison |
|---|---|
| Ligne insérée dans `measures` | Fait technique → « Mesure validée » |
| JSON de la trame parsé | Fait technique |
| Erreur 500 sur l'API d'ingestion | Technique, invisible pour le métier → « Trame rejetée » |
| Bouton « Valider » cliqué | Détail d'interface → « Récolte validée » |
| Envoyer un SMS | Commande, pas un événement → « Apicultrice notifiée » |
| Données reçues | Trop vague → « Mesure reçue » ou « Trame rejetée » |
| Miel récolté | Doublon de « Récolte déclarée » (mot du cahier des charges conservé) |
| Alerte | Un nom, pas un fait → « Alerte levée », « Alerte résolue » |
| Cache vidé | Technique → « Lot de mesures différées reçu » |

## B4 · Agrégats et invariants (12)

| Agrégat | Contexte | Commandes reçues | Invariants |
|---|---|---|---|
| Compte | Identité et accès | Créer, Assigner un rôle, Désactiver, S'authentifier | un rôle est toujours rattaché à un site ; un compte bloqué ou désactivé refuse toute authentification ; MFA obligatoire pour coordinateur, gestionnaire, admin |
| Site | Référentiel des ruchers | Créer un site | un site appartient à un seul pays ; un site actif a au moins une passerelle déclarée |
| Ruche | Référentiel des ruchers | Enregistrer, Associer un capteur, Déclarer une transhumance | identifiant unique au format `PP-SITE-NNN` ; au plus un capteur actif par ruche ; une transhumance a une date de début et une destination |
| Passerelle | Collecte IoT | Mettre en service, Changer de canal ; politiques de battement de cœur | rattachée à un seul site ; un seul canal actif à la fois ; hors ligne si aucun battement depuis 15 min |
| Série de mesures (une par ruche) | Collecte IoT | Valider une mesure | une seule mesure par couple (ruche, horodatage) ; une mesure hors plage physique n'est jamais publiée ; horodatage ≤ réception + 5 min |
| Incident | Collecte IoT | Ouvrir, Clore | un incident clos ne se rouvre pas (on en ouvre un nouveau) ; tout incident est journalisé |
| Colonie suivie | Santé des colonies | Évaluer une mesure, Passer en surveillance renforcée | un seul score courant par colonie ; la surveillance renforcée a une date de fin |
| Alerte | Santé des colonies | Lever, Prendre en charge, Escalader, Résoudre | type de risque connu, confiance entre 0 et 1 ; résolution seulement après prise en charge et avec une inspection ; une alerte résolue n'accepte plus de transition ; escalade seulement après 48 h (cf. kit `queenbee-alerte`) |
| Prédiction de risque / de rendement | Prévision | Calculer | une prédiction cite la version du modèle et la fenêtre de données utilisée ; une prédiction publiée n'est jamais modifiée (on en publie une nouvelle) |
| Visite / Plan de visites | Interventions terrain | Planifier, Assigner, Demander une visite anticipée | une visite est assignée à une personne ayant un rôle sur le site ; une visite réalisée ne se replanifie pas |
| Inspection | Interventions terrain | Enregistrer, Arbitrer un conflit | une inspection porte sur une ruche existante ; une inspection synchronisée en conflit ne peut pas être utilisée avant arbitrage |
| Récolte | Production | Déclarer, Valider, Signaler un écart | volume positif ; une récolte validée ne change plus ; un écart signalé exige un commentaire |
| Rapport d'impact | Impact | Choisir la période, Générer, Envoyer | une période a un début avant sa fin ; un rapport envoyé est figé (archivé tel quel) |
