# queenbee-archi-nguemo-rosier

Architecture logicielle · M1 INGETIS · TP 5 · Event Storming QueenBee (23/10/2026)

- **Spécialité** : Data
- **Binôme** : Calixte Nguemo et Jule Rosier (M1 Data)
- **Groupe** : Data-&lt;n°&gt; (à compléter avec le numéro de groupe donné en séance)
- **Tableau Miro** : `QueenBee · ES · Data-<n°>` (lien à ajouter)

## Contenu

| Fichier | Contenu | État |
|---|---|---|
| `docs/event-storming/mur.md` | transcription du mur : 65 événements ordonnés en 6 phases, causes, 31 politiques, agrégats et invariants, écartés | fait |
| `docs/event-storming/mur-complet.pdf` | export PDF du tableau Miro | à exporter depuis Miro |
| `docs/glossaire.md` | 32 termes, 4 mots à plusieurs sens tranchés (« Alerte », « Ruche », « Site », « Validation ») | fait |
| `docs/bounded-contexts.md` | 8 contextes : 3 cœur, 4 support, 1 générique | fait |
| `docs/context-map.md` (+ `context-map.png`) | context map Mermaid, 16 relations typées, 4 ACL | fait |
| `docs/points-chauds.md` | 16 points chauds, dont 6 menant à un ADR | fait |
| `docs/specialite/data-contrat-mesure-validee.md` | contrat de l'événement `MesureValidée.v1` : propriétaire, consommateurs, champs, enveloppe, exemple, déduplication, rejet, retard, versionnement | amorcé, complet sauf vérification des plages (H) |
| `docs/specialite/mesure-validee.v1.schema.json` | schéma JSON du contrat ; l'exemple du contrat y est conforme | fait |
| `adr/0001-ingestion-lots-ou-flux.md` | amorce de l'ADR de spécialité | proposé |
| `miro/guide-tableau-miro.md` | construction du tableau Miro, post-it prêts à coller | fait |

## Suite pour le dossier du 30/10
1. Vérifier les plages marquées (H) du contrat avec le § 6.5 du cahier des charges.
2. Faire trancher la cadence nominale (15 ou 30 min, point chaud n° 2) et le retard maximal (n° 4).
3. Finaliser l'ADR 0001 en séance 6, puis 2 à 4 autres ADR à partir des points chauds.
4. Exporter le tableau Miro en PDF dans `docs/event-storming/`.
