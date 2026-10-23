# QueenBee : Event Storming et architecture métier (TP 5)

- **Spécialité :** Data
- **Membres :** Jovani et Patrick 
- **Module :** Architecture logicielle · M1 INGETIS · TP 5 du 23/10/2026

## 1. Présentation

QueenBee est la plateforme de suivi de ruchers connectés de l'association BeeAfrica : 3 pays, 8 sites, 400 ruches, 25 coopératives. Des capteurs mesurent poids, températures, humidité et activité acoustique ; une passerelle LoRaWAN par site transmet les mesures, souvent en différé ; un modèle prédit les risques ; les apicultrices et les coordinateurs agissent sur le terrain ; le gestionnaire rend compte aux bailleurs.

Ce dépôt contient la première étape de la conception de l'architecture : de l'Event Storming aux bounded contexts, puis la context map et le livrable de spécialité Data. Il prépare la séance 6 (modèle C4 et ADR).

## 2. Structure du dépôt

```text
queenbee-archi-Jovani-Patrick/
├── README.md
├── docs/
│   ├── event-storming/
│   │   └── mur-complet.png
│   ├── glossaire.md
│   ├── bounded-contexts.md
│   ├── context-map.md
│   ├── points-chauds.md
│   └── specialite/
│       └── data-contrat-mesure-validee.md
└── adr/
    ├── ADR-001-ingestion-mesures.md
    ├── ADR-002-gestion-mesures-en-retard.md
    └── ADR-003-versionnement-mesure-validee.md
```

## 3. Livrables

| Livrable | Contenu |
|---|---|
| [Mur Event Storming](docs/event-storming/mur-complet.png) | 60 événements métier répartis par bounded context, avec commandes, acteurs, systèmes externes, politiques, agrégats et points chauds |
| [Glossaire](docs/glossaire.md) | 70 termes avec définition, contexte, nom dans le code et UC source ; mots à plusieurs sens tranchés (dont « Alerte » et « Incident ») |
| [Bounded contexts](docs/bounded-contexts.md) | 8 contextes : responsabilité, type, agrégats et invariants, événements publiés et consommés, UC sources, flux entre contextes |
| [Context map](docs/context-map.md) | Carte Mermaid des 8 contextes et de 5 systèmes externes, chaque relation typée (OHS, PL, Partnership, ACL) |
| [Points chauds](docs/points-chauds.md) | 14 questions ouvertes avec impact, interlocuteur et décision d'ADR éventuelle |
| [Contrat Data](docs/specialite/data-contrat-mesure-validee.md) | Contrat de l'événement `MesureValidée.v1` : champs, enveloppe, déduplication, rejet, retard, versionnement |
| [ADR](adr/) | 3 décisions d'architecture (voir section 5) |

## 4. Les 8 bounded contexts

| Contexte | Type |
|---|---|
| Référentiel ruchers | support |
| Identité et accès | générique |
| Collecte et supervision | support |
| Prévision | cœur |
| Santé des colonies | cœur |
| Intervention terrain | cœur |
| Production | cœur |
| Impact et reporting | support |

**Décision de langage :** le mot « alerte » est réservé à la santé d'une colonie. Une panne d'infrastructure (passerelle muette, brouillage, déplacement non déclaré) est un *incident*, dans Collecte et supervision.

## 5. Spécialité Data

Le contrat `MesureValidée.v1` est publié par **Collecte et supervision** vers Santé des colonies et Prévision (Published Language), et sous forme d'indicateurs de qualité vers Impact et reporting. Il distingue `occurredAt` (date de la mesure) et `receivedAt` (date de réception), et la clé de déduplication est `hive_id` + `timestamp`.

| ADR | Décision | Statut |
|---|---|---|
| ADR-001 | Ingestion des mesures en flux, avec cache local en mode dégradé ; batch pour l'analytique | Acceptée |
| ADR-002 | Gestion des mesures reçues en retard : `occurredAt` comme référence métier, retard maximal de 24 h proposé | Acceptée |
| ADR-003 | Versionnement du contrat `MesureValidée` par un champ `version` | Proposée |

## 6. État d'avancement

- [x] Mur Event Storming (60 événements, 8 contextes)
- [x] Glossaire, bounded contexts, context map, points chauds
- [x] Contrat Data `MesureValidée.v1`
- [x] ADR-001, ADR-002, ADR-003
- [ ] Aligner le mur, le glossaire et le contrat Data sur les 8 contextes (supervision fusionnée dans la collecte)
- [ ] Renommer les événements « Alerte d'intégrité levée » et « Alerte prioritaire envoyée » en incident et notification
- [ ] Valider avec BeeAfrica les valeurs proposées (plages physiques, retard maximal de 24 h, seuil d'écart de 20 %)
- [ ] Séance 6 : modèle C4 et ADR complémentaires

## 7. Points restant à clarifier

- **Cadence des mesures :** le CDC indique 30 minutes dans UC-T02 et 15 minutes (5 en renforcé) en section 6.4. Le contrat retient 15 minutes, ce qui correspond aux 38 400 mesures par jour du CDC.
- **Seuils d'alerte :** le CDC les fait configurer par l'administrateur, par type de ruche et par site ; leurs valeurs restent à définir.
- **Retard maximal d'une mesure :** 24 h proposé, à confirmer.
- **Propriété des associations** ruche–capteur–passerelle : Référentiel seul, ou aussi Collecte.
- **Notifications :** quel contexte les orchestre, ou un service dédié.

Les valeurs non imposées par le CDC sont des propositions, pas des exigences confirmées.

## 8. Sources

Les livrables s'appuient sur l'énoncé du TP 5 et sur le cahier des charges commun QueenBee (chapitres 3, 4, 6 et 7).