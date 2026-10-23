# Points chauds · groupe Data

Binôme : Calixte Nguemo et Jule Rosier · M1 Data

| # | Question | Impact si on se trompe | Qui peut répondre | Devient un ADR ? |
|---|---|---|---|---|
| 1 | « Alerte » : colonie ou infrastructure ? Même mot, deux sens ? | Un seul agrégat aux règles contradictoires (48 h contre 15 min) | Équipe (tranché : Alerte = colonie, Incident = infrastructure) | non (glossaire) |
| 2 | Cadence nominale : 1 mesure / 30 min (UC-T02, annexe A1) ou 1 / 15 min (partie E, 38 400 mesures / jour = 400 × 96) ? | Volumétrie, autonomie des batteries et détection « capteur silencieux » faussées d'un facteur 2 | BeeAfrica + fournisseur des capteurs | non (paramètre du contrat ; 15 min retenu en hypothèse) |
| 3 | Plages physiques valides (poids max d'une ruche, températures possibles au Sahel) | Rejet de vraies mesures ou publication de mesures absurdes | Apicultrices référentes BeeAfrica + fiche technique des capteurs | non (paramètre) |
| 4 | Retard maximal accepté pour une mesure différée (capacité du cache des passerelles) | Alertes levées sur des données périmées, ou mesures perdues | Fournisseur des passerelles + équipe Data | **oui** (ADR Data : ingestion lots ou flux) |
| 5 | Lever une alerte sur une mesure arrivée 6 h après son relevé ? | Alerte inutile (la colonie a déjà essaimé) ou manquée | Coordinateurs + équipe Data | **oui** (lié au n° 4) |
| 6 | Seuil de confiance pour lever une alerte (0,7 ?) ; par type de risque ? | Trop d'alertes (lassitude) ou pas assez | Équipe Data + coordinateurs | non (paramètre, à calibrer) |
| 7 | Délai d'escalade de 48 h : fixé par qui ? Identique pour la mortalité et l'essaimage ? | Escalade trop tardive sur un risque de mortalité | BeeAfrica (direction technique) | non (paramètre par type) |
| 8 | Seuil d'écart de rendement : 20 % ? par coopérative ? | Investigations inutiles ou fraudes non vues | Coordinateurs BeeAfrica | non (paramètre) |
| 9 | Tolérance GPS avant « déplacement non déclaré » (50 m ? 200 m ?) vu la précision des capteurs | Fausses alertes de vol | Administrateur + fournisseur des capteurs | non (paramètre) |
| 10 | Conflit de synchronisation d'inspection : quelle version gagne ? | Perte d'une observation terrain | Équipe Dev + coordinateurs | **oui** (ADR Dev) |
| 11 | Changement de canal en cas de brouillage : automatique ou validé par l'administrateur ? | Coupure d'un site ou attaque non contrée | Équipe Cyber + administrateur | **oui** (ADR Cyber) |
| 12 | Authentification des passerelles auprès du central (clé partagée, mTLS, VPN) | Trames forgées acceptées comme mesures validées | Équipe Cyber | **oui** (ADR Cyber) |
| 13 | Quelles données personnelles (noms des apicultrices, positions) peuvent figurer dans un rapport bailleur ? | Non-conformité à la protection des données | Gestionnaire + DPO BeeAfrica | oui (ADR transverse) |
| 14 | Format `hive_id` `PP-SITE-NNN` : le troisième pays et ses codes de site sont-ils fixés ? | Ruches refusées à l'ingestion | BeeAfrica | non |
| 15 | Une ruche sans capteur (inspection seule) existe-t-elle dans le référentiel ? | Référentiel incomplet, indicateurs faussés | Coordinateurs | non |
| 16 | Les mesures rejetées doivent-elles alimenter l'indicateur de couverture réseau ? | Couverture surestimée | Gestionnaire + équipe Data | non |
