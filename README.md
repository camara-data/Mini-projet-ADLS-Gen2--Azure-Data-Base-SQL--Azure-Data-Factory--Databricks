# Projet ADF Metadata-Driven - Architecture d’ingestion automatisée

## 1 - Contexte
L’ingestion de données provenant de plusieurs sources, notamment Azure SQL Database et Azure Data Lake Storage Gen2 (ADLS Gen2), nécessite généralement la création et la maintenance de pipelines spécifiques pour chaque table ou fichier.
L’objectif de ce projet est de mettre en place une architecture d’ingestion **Metadata-Driven** avec Azure Data Factory, afin de centraliser la configuration des chargements et de rendre les pipelines réutilisables, évolutifs et plus faciles à maintenir.

Les principaux enjeux sont de :

- Automatiser l’ingestion des données depuis plusieurs sources.
- Réduire la duplication des pipelines grâce à une architecture générique.
- Centraliser les paramètres et les règles de chargement.
- Gérer les chargements complets et incrémentaux.
- Assurer la traçabilité des exécutions et la gestion des erreurs.
- Alimenter automatiquement la couche Bronze dans Databricks.

## 2 - Approche technique choisie

Le projet repose sur une architecture **Metadata-Driven**, orchestrée avec Azure Data Factory (ADF), et s’appuie sur Databricks pour la préparation et la publication des données dans la couche Bronze.

Les principaux composants sont :

- **Sources de données :**
  - Azure SQL Database : ingestion des données provenant de tables relationnelles.
  - ADLS Gen2 : ingestion des fichiers sources (CSV, Excel, Parquet, etc., selon les formats configurés).

- **Azure Data Factory – Orchestration :**
  - `ORCHESTRATEUR` : point d’entrée pour déclencher les traitements.
  - `INGEST_TOP` : lecture de la table de métadonnées `ingestion_config` et organisation des traitements par `batch_group`.
  - `INGEST_MIDDLE` : parcours des configurations et routage vers le pipeline Bottom correspondant à la source.
  - `INGEST_BOTTOM_SQL` : exécution des chargements depuis Azure SQL Database.
  - `INGEST_BOTTOM_ADLS` : exécution des chargements depuis ADLS Gen2.

- **Gestion des métadonnées :**
  - `ingestion_config` : centralisation des paramètres d’ingestion, des sources, des destinations, des modes de chargement et des règles de traitement.
  - Organisation des traitements par `batch_group`, permettant de respecter les dépendances entre les groupes.

- **Modes de chargement :**
  - FULL : chargement complet des données.
  - INCREMENTAL : chargement des nouvelles données ou des données modifiées à partir d’un mécanisme de watermark.

- **Databricks :**
  - Exécution de notebooks pour préparer les données, gérer le staging et publier les tables dans la couche Bronze.
  - Stockage des données ingérées au format Delta Lake dans le catalogue Databricks.

- **Journalisation et suivi :**
  - `ingestion_run_log` : conservation de l’historique des exécutions, notamment les dates de début et de fin, le statut, la durée et les éventuels messages d’erreur.
  - Gestion des statuts SUCCESS et FAILED à la fin des traitements.

Le fonctionnement du processus est organisé en plusieurs niveaux : les groupes de traitement sont parcourus séquentiellement dans `INGEST_TOP`, tandis que les éléments d’un même groupe peuvent être traités en parallèle dans `INGEST_MIDDLE`, avec une concurrence configurée à `batchCount = 5`.

Pour les chargements incrémentaux, le watermark n’est mis à jour qu’après la réussite complète du chargement, afin de garantir la continuité des données en cas d’échec.

## 3 - Résultats et bénéfices attendus

Cette architecture permet de :

- Centraliser les configurations d’ingestion dans une table de métadonnées.
- Réutiliser les mêmes pipelines pour plusieurs tables et fichiers.
- Simplifier l’ajout de nouvelles sources sans multiplier les pipelines génériques.
- Automatiser les chargements FULL et INCREMENTAL.
- Contrôler les exécutions grâce à une journalisation centralisée.
- Limiter les risques de perte de données grâce à la gestion du watermark.
- Alimenter une couche Bronze centralisée dans Databricks.

L’approche Metadata-Driven apporte ainsi une meilleure maintenabilité, une meilleure évolutivité et une gestion plus structurée des flux d’ingestion.

## Auteur

Mamadou Mouctar CAMARA
