# Contrat de données · événement « Mesure validée » (MesureValidée.v1)

Binôme : **Calixte Nguemo** et **Jule Rosier** · M1 Data · parcours Data du TP 5
Statut : v1 proposée le 23/10/2026, à finaliser pour le dossier du 30/10/2026.

> Sources : UC-T02, UC-CY01, UC-CY02, UC-CY04 et la partie E de l'énoncé. Le format 6.5 du
> cahier des charges n'était pas joint au kit : les plages ci-dessous sont nos hypothèses,
> marquées **(H)**, à confronter au § 6.5 avant le dossier (point chaud n° 3).

## 1. Propriétaire et consommateurs

- **Propriétaire (producteur)** : contexte **Collecte IoT** (support). Il est le seul à pouvoir
  émettre `MesureValidée` ; c'est la sortie de l'agrégat *Série de mesures* après les contrôles
  d'ingestion.
- **Relation sur la context map** : `U/D · OHS · PL : MesureValidée.v1`, de Collecte IoT vers
  ses quatre consommateurs. Le contrat est le *published language* : les consommateurs ne
  voient jamais la trame LoRaWAN, qui reste derrière l'ACL capteurs → Collecte.

| Consommateur | Ce qu'il en fait | Champs utilisés | Exigence de fraîcheur |
|---|---|---|---|
| Santé des colonies (cœur) | seuils, surveillance renforcée, score de santé | tous les champs capteur | forte : alerte en moins de 30 min après réception |
| Prévision (cœur) | variables explicatives des modèles de risque et de rendement ; réentraînement | tous ; historique complet | modérée (risque) à faible (rendement) |
| Production (cœur) | comparer une récolte déclarée aux poids mesurés (UC-C02) | `hive_id`, `timestamp`, `weight_kg` | faible (jour) |
| Impact (support) | indicateurs multi-sites, couverture réseau | `site_id`, `timestamp`, `receivedAt`, `weight_kg`, `quality` | faible (consolidation nocturne) |

Le Référentiel des ruchers ne consomme pas `MesureValidée` : il reçoit l'événement dérivé
`PositionHorsToléranceDétectée` (relation Customer / Supplier).

## 2. Enveloppe de l'événement

| Champ | Type | Oblig. | Règle |
|---|---|---|---|
| `eventId` | string UUID v4 | oui | unique ; sert à l'idempotence des consommateurs |
| `type` | string | oui | toujours `"MesureValidée"` |
| `version` | integer | oui | version majeure du schéma : `1` |
| `occurredAt` | string ISO 8601 UTC | oui | date du relevé par le capteur = `payload.timestamp` |
| `receivedAt` | string ISO 8601 UTC | oui | date de réception au central ; `receivedAt ≥ occurredAt − 5 min` |
| `validatedAt` | string ISO 8601 UTC | oui | date de publication par Collecte IoT |
| `producer` | string | oui | `"collecte-iot"` |
| `gatewayId` | string | oui | passerelle qui a relayé la trame (format `GW-SITE-NN`) **(H)** |
| `deliveryMode` | enum | oui | `TEMPS_REEL` si `receivedAt − occurredAt ≤ 30 min`, sinon `DIFFERE` |
| `samplingMode` | enum | oui | `NORMAL` (15 min), `RENFORCE` (5 min), `SEUIL` (envoi immédiat sur dépassement local) |
| `dedupKey` | string | oui | `hive_id + "|" + timestamp` (voir § 5) |
| `schemaUri` | string | non | lien vers le schéma JSON publié dans le registre |

## 3. Tableau des champs du payload

| Champ | Type | Unité | Plage valide | Oblig. | Règle / remarque |
|---|---|---|---|---|---|
| `hive_id` | string | – | `^[A-Z]{2}-[A-Z]{3,4}-\d{3}$` (`PP-SITE-NNN`) | oui | ruche connue et active dans le Référentiel, sinon rejet `RUCHE_INCONNUE` |
| `site_id` | string | – | `^[A-Z]{2}-[A-Z]{3,4}$` | oui | doit être le préfixe de `hive_id` et le site de la passerelle, sinon rejet `SITE_INCOHERENT` |
| `timestamp` | string ISO 8601 UTC | – | ≤ `receivedAt` + 5 min ; ≥ `receivedAt` − 72 h (voir § 6) | oui | avec `hive_id` : clé de déduplication ; arrondi à la minute |
| `weight_kg` | number (1 déc.) | kg | 0 – 150 **(H)** | oui | variation > 3 kg en 15 min hors visite ou récolte : publiée avec `quality.flags = ["VARIATION_BRUTALE"]` ; Santé tranche (essaimage ? vol ? erreur ?) |
| `temp_internal_c` | number (1 déc.) | °C | −10 – 60 **(H)** | oui | couvain sain ≈ 33 – 36 °C ; hors de 25 – 40 °C : signal pour Santé, pas un rejet |
| `temp_external_c` | number (1 déc.) | °C | −20 – 60 **(H)** | oui | écart > 10 °C avec OpenWeatherMap du site : flag `SONDE_SUSPECTE` |
| `humidity_pct` | number (entier) | % | 0 – 100 | oui | humidité intérieure relative |
| `acoustic_activity_score` | number (2 déc.) | sans unité | 0 – 1 | oui | score calculé par le capteur ; son algorithme est versionné dans `sensor_firmware` |
| `battery_pct` | integer | % | 0 – 100 | non | < 15 % : flag `BATTERIE_FAIBLE` ; absent : `null` |
| `position_lat` | number (5 déc.) | degré décimal WGS 84 | −90 – 90, pratique 4 – 17 (pays couverts) **(H)** | non | lien avec UC-CY04 : distance à la position de référence > tolérance → `PositionHorsToléranceDétectée` |
| `position_lon` | number (5 déc.) | degré décimal WGS 84 | −180 – 180, pratique −18 – 3 **(H)** | non | obligatoire si `position_lat` est présent |
| `sensor_id` | string | – | identifiant matériel | oui | doit être le capteur actif associé à `hive_id` dans le Référentiel |
| `sensor_firmware` | string | – | semver | non | trace la version de calcul du score acoustique |
| `frame_counter` | integer | – | 0 – 2³² − 1 | oui | compteur LoRaWAN (FCnt) : sert à détecter le rejeu (voir § 5) |
| `quality.flags` | array<string> | – | `VARIATION_BRUTALE`, `SONDE_SUSPECTE`, `BATTERIE_FAIBLE`, `DIFFERE`, `GPS_ABSENT` | oui (peut être vide) | qualifie une mesure valide ; ne remplace jamais un rejet |

**Règle générale** : un champ obligatoire absent ou hors de sa plage physique entraîne le
rejet de toute la mesure (`TrameRejetée`, motif `HORS_PLAGE:<champ>`) ; un champ optionnel hors
plage est remplacé par `null` avec un flag. Une mesure n'est jamais publiée « à moitié fausse ».

## 4. Exemple JSON valide

```json
{
  "eventId": "6f1c2a9e-5b7d-4c1e-9a3f-2d8e0b4c7a11",
  "type": "MesureValidée",
  "version": 1,
  "occurredAt": "2027-03-14T08:15:00Z",
  "receivedAt": "2027-03-14T08:41:12Z",
  "validatedAt": "2027-03-14T08:41:13Z",
  "producer": "collecte-iot",
  "gatewayId": "GW-CASA-01",
  "deliveryMode": "TEMPS_REEL",
  "samplingMode": "NORMAL",
  "dedupKey": "SN-CASA-014|2027-03-14T08:15:00Z",
  "schemaUri": "https://schemas.queenbee.example/mesure-validee/v1.json",
  "payload": {
    "hive_id": "SN-CASA-014",
    "site_id": "SN-CASA",
    "sensor_id": "QB-S-00314",
    "sensor_firmware": "2.3.1",
    "timestamp": "2027-03-14T08:15:00Z",
    "frame_counter": 48211,
    "weight_kg": 34.2,
    "temp_internal_c": 34.8,
    "temp_external_c": 27.1,
    "humidity_pct": 58,
    "acoustic_activity_score": 0.72,
    "battery_pct": 91,
    "position_lat": 12.58330,
    "position_lon": -16.28330,
    "quality": { "flags": [] }
  }
}
```

Exemple de mesure différée (même ruche, passerelle revenue après 6 h de coupure) :
`"occurredAt": "2027-03-14T02:00:00Z"`, `"receivedAt": "2027-03-14T08:05:40Z"`,
`"deliveryMode": "DIFFERE"`, `"quality": { "flags": ["DIFFERE"] }`.

Le schéma JSON (draft 2020-12) correspondant est fourni à côté :
[`mesure-validee.v1.schema.json`](mesure-validee.v1.schema.json).

## 5. Déduplication et rejeu (UC-CY02)

- **Clé de déduplication métier** : `(hive_id, timestamp)`. L'agrégat *Série de mesures* garantit
  qu'une seule mesure validée existe par clé.
- **Même clé, même contenu** (renvoi d'un lot après resynchronisation) : doublon bénin, ignoré
  sans rejet, compté dans une métrique `doublons_ignores`. C'est le cas normal du mode dégradé.
- **Même clé, contenu différent**, ou **`frame_counter` inférieur ou égal au dernier reçu pour
  ce capteur** hors redémarrage déclaré : **rejeu suspect** → `TrameRejetée` motif `REJEU`,
  journalisé avec la trame brute (hash), la passerelle et `receivedAt`.
- **Répétition** : plus de 5 rejets (tous motifs) en 1 h pour une même ruche, ou plus de 20 pour
  une passerelle → `IncidentD'IntégritéOuvert` (Collecte IoT), visible de l'administrateur.
- **Côté consommateurs** : idempotence par `eventId` (livraison au moins une fois) ; un
  consommateur ne doit jamais supposer l'ordre d'arrivée, il trie par `occurredAt`.

## 6. Fréquence, retard et mode dégradé (UC-T02, UC-CY01)

- **Fréquence nominale** : 1 mesure / 15 min / ruche → 96 par jour, soit **38 400 mesures par jour**
  pour 400 ruches. **Surveillance renforcée** : 1 / 5 min (288 par jour) pour les ruches désignées par
  Santé des colonies. **Mode SEUIL** : envoi immédiat en plus de la cadence.
  *Point chaud n° 2 : l'annexe A1 indique 30 min pour UC-T02 ; nous retenons 15 min, cohérent
  avec la volumétrie de l'énoncé.*
- **Ruche silencieuse** : aucune mesure pendant 4 cycles (1 h en nominal, 20 min en renforcé)
  alors que la passerelle est en ligne → `CapteurDevenuSilencieux`.
- **Pourquoi deux dates** : `occurredAt` dit quand l'abeille a vécu la situation, `receivedAt`
  quand nous l'avons apprise. En mode dégradé l'écart peut atteindre plusieurs heures.
  Conséquences sur les alertes :
  1. les seuils et modèles travaillent sur `occurredAt` (la série temporelle réelle) ;
  2. une mesure `DIFFERE` est rejouée dans la série et peut lever une alerte **rétroactive** ;
     l'alerte porte alors `detectedLate = true` et les deux dates, pour que le coordinateur
     juge si une visite a encore du sens ;
  3. le délai d'escalade de 48 h court à partir de la levée de l'alerte (date de détection), pas
     de `occurredAt` : sinon une alerte arriverait déjà escaladée ;
  4. la couverture réseau (Impact) se calcule sur `receivedAt − occurredAt`.
- **Retard maximal accepté** : 72 h **(H, point chaud n° 4)**, borne liée à la capacité du cache
  des passerelles. Au-delà : la mesure n'est pas publiée en `MesureValidée` (aucune alerte
  possible), elle est archivée avec le motif `TROP_TARDIVE` pour Prévision et Impact.
- **Horloge** : `timestamp` > `receivedAt` + 5 min → rejet `HORODATAGE_FUTUR` (horloge capteur
  dérivée).

## 7. Versionnement

- Le nom porte la version majeure : `MesureValidée.v1`, champ `version: 1`, schéma publié dans un
  registre (`schemaUri`).
- **Évolution compatible** (sans changement de version) : ajout d'un champ **optionnel**, ajout
  d'une valeur de flag. Les consommateurs appliquent la règle du *tolerant reader* : ils ignorent
  les champs inconnus.
- **Évolution cassante** (→ `v2`) : renommer, supprimer, changer le type ou l'unité d'un champ,
  rendre obligatoire un champ optionnel, changer la clé de déduplication.
- **Migration** : Collecte IoT publie `v1` et `v2` en parallèle pendant au moins 3 mois ; chaque
  consommateur annonce sa bascule ; `v1` est retirée quand le dernier a basculé.
- **Propriété** : toute modification du contrat est validée par Collecte IoT et revue par les
  quatre consommateurs (relation OHS : un contrat pour tous, pas d'arrangement bilatéral).

## 8. Où se trouve l'événement sur le mur et la context map

- Mur : événement pivot ★ n° 15 « Mesure validée », phase « En continu », causé par la politique
  « Quand mesure reçue, alors valider », agrégat *Série de mesures*, contexte Collecte IoT ; son
  alternative est « Trame rejetée » (n° 16), posée au-dessus.
- Context map : quatre flèches doubles `U/D · OHS · PL : MesureValidée.v1` de Collecte IoT vers
  Santé des colonies, Prévision, Production et Impact ([`../context-map.md`](../context-map.md)).

## 9. Suite pour le dossier du 30/10

- Confronter les plages **(H)** au § 6.5 du cahier des charges et aux fiches des capteurs.
- Faire valider la cadence (15 ou 30 min) et le retard maximal par BeeAfrica.
- Finaliser l'ADR [`../../adr/0001-ingestion-lots-ou-flux.md`](../../adr/0001-ingestion-lots-ou-flux.md).
