# 🏥 Architecture Big Data Distribuée pour l'Analyse des Risques Médicamenteux

[![Big Data](https://img.shields.io/badge/Big%20Data-Distributed-blue)](https://github.com)
[![Apache Kafka](https://img.shields.io/badge/Apache-Kafka-black)](https://kafka.apache.org/)
[![Apache Spark](https://img.shields.io/badge/Apache-Spark-orange)](https://spark.apache.org/)
[![Apache Hadoop](https://img.shields.io/badge/Apache-Hadoop-yellow)](https://hadoop.apache.org/)
[![Apache Hive](https://img.shields.io/badge/Apache-Hive-green)](https://hive.apache.org/)

## 📋 Table des Matières

- [Présentation](#-présentation)
- [Objectifs](#-objectifs)
- [Architecture](#-architecture)
- [Technologies](#-technologies)
- [Prérequis](#-prérequis)
- [Installation](#-installation)
- [Configuration Détaillée](#-configuration-détaillée)
- [Scripts de Code](#-scripts-de-code)
- [Démarrage et Utilisation](#-démarrage-et-utilisation)
- [Structure du Projet](#-structure-du-projet)
- [Monitoring et Supervision](#-monitoring-et-supervision)
- [Résultats et Visualisations](#-résultats-et-visualisations)
- [Contributeurs](#-contributeurs)

---

## 📌 Présentation

Ce projet Big Data vise à analyser les **risques médicamenteux** à partir des données publiques de l'**API OpenFDA**. Il permet d'identifier les effets indésirables, les médicaments à haut risque et d'extraire des indicateurs d'aide à la décision pour les professionnels de santé.

 

### 📊 Source de Données

| Élément | Description |
|---------|-------------|
| **Source** | API OpenFDA (https://api.fda.gov/drug/event.json) |
| **Format** | JSON semi-structuré |
| **Volume** | Millions d'événements indésirables par an |
| **Période** | Données de janvier à mars 2024 (configurable) |
| **Contenu** | Rapports de sécurité pharmaceutique, effets secondaires, indications médicamenteuses |

### 📈 Besoins Analytiques

- **Suivi de la sécurité** : Suivre la fréquence et les types d'événements indésirables pour chaque médicament
- **Profilage des risques** : Évaluer les médicaments selon la gravité des effets indésirables signalés
- **Analyse descriptive** : Examiner les tendances générales des événements indésirables
- **Analyse diagnostique** : Identifier les facteurs sous-jacents des médicaments fréquemment signalés

---

## 🎯 Objectifs

✅ **Surveiller les tendances de sécurité** - Identifier les tendances des événements indésirables signalés pour des médicaments spécifiques

✅ **Identifier les médicaments à haut risque** - Mettre en évidence les médicaments avec un grand nombre de rapports d'effets indésirables graves

✅ **Traitement temps réel** - Exploiter des données massives en streaming temps réel et en batch

✅ **Soutenir la prise de décision** - Fournir des informations exploitables pour les professionnels de santé et les patients

✅ **Visualisation interactive** - Créer des tableaux de bord pour l'aide à la décision médicale

---

## 🏗️ Architecture

### 📐 Vue d'Ensemble du Pipeline

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐     ┌─────────┐     ┌─────────┐     ┌──────────┐
│  OpenFDA    │────▶│   Kafka     │────▶│  Spark Stream   │────▶│  HDFS   │────▶│  Hive   │────▶│ Superset │
│    API      │     │  (3 nodes)  │     │   (on YARN)     │     │ (3x)    │     │ (SQL)   │     │  (BI)    │
└─────────────┘     └─────────────┘     └─────────────────┘     └─────────┘     └─────────┘     └──────────┘
      │                   │                      │                    │               │               │
  [Source]          [Ingestion]           [Traitement]          [Stockage]       [Analyse]    [Visualisation]
```

### 🔄 Flux de Données Détaillé

#### **Couche 1 : Ingestion (Apache Kafka)**

- **Producer Python** récupère les données de l'API OpenFDA toutes les secondes
- Les données sont transformées et envoyées au topic Kafka `devoir`
- **3 brokers Kafka** (huemaster, worker1, worker2) assurent la distribution
- **ZooKeeper** (3 serveurs en quorum) coordonne le cluster
- **Réplication factor : 2** pour la haute disponibilité
- **3 partitions** pour le parallélisme

#### **Couche 2 : Traitement (Apache Spark sur YARN)**

- **Spark Streaming** consomme les messages Kafka en micro-batches
- Traitement distribué sur **ResourceManager YARN** (huemaster)
- **NodeManagers** sur worker1 et worker2 exécutent les tâches
- Configuration mémoire : **4GB par executor**
- Transformation des données : extraction des médicaments et réactions
- Stockage dans Hive au format **Parquet**

#### **Couche 3 : Stockage (Hadoop HDFS + Hive)**

- **NameNode HDFS** sur huemaster gère les métadonnées
- **DataNodes** sur worker1 et worker2 stockent les blocs de données
- **Réplication HDFS : 3** pour la tolérance aux pannes
- **HiveServer2** et **Metastore MySQL** sur huemaster
- Tables Hive avec schéma optimisé pour l'analytique
- Format **Parquet** pour la compression et les performances

#### **Couche 4 : Visualisation (Apache Superset)**

- Déployé sur huemaster
- Connexion directe à **HiveServer2** (port 10000)
- Dashboards interactifs pour l'analyse des risques médicamenteux
- Interface web accessible sur le port 8088

### 🖥️ Topologie du Cluster (3 Machines)

#### **Machine 1 : huemaster (Master Node)**
```
┌─────────────────────────────────────┐
│          HUEMASTER (Master)         │
├─────────────────────────────────────┤
│ • Kafka Broker 1 (9092)            │
│ • ZooKeeper Server 1 (2181)         │
│ • HDFS NameNode (9000)              │
│ • YARN ResourceManager (8088)       │
│ • Spark Master (7077)               │
│ • HiveServer2 (10000)               │
│ • Hive Metastore (9083)             │
│ • MySQL Metastore (3306)            │
│ • Apache Superset (8088)            │
└─────────────────────────────────────┘
```

#### **Machine 2 : worker1 (Worker Node)**
```
┌─────────────────────────────────────┐
│          WORKER1 (Worker)           │
├─────────────────────────────────────┤
│ • Kafka Broker 2 (9092)            │
│ • ZooKeeper Server 2 (2181)         │
│ • HDFS DataNode                     │
│ • YARN NodeManager                  │
│ • Spark Worker                      │
└─────────────────────────────────────┘
```

#### **Machine 3 : worker2 (Worker Node)**
```
┌─────────────────────────────────────┐
│          WORKER2 (Worker)           │
├─────────────────────────────────────┤
│ • Kafka Broker 3 (9092)            │
│ • ZooKeeper Server 3 (2181)         │
│ • HDFS DataNode                     │
│ • YARN NodeManager                  │
│ • Spark Worker                      │
└─────────────────────────────────────┘
```

### ✨ Caractéristiques Clés de l'Architecture

- **Haute Disponibilité** : Réplication des données sur 3 nœuds
- **Tolérance aux Pannes** : Aucun point de défaillance unique (SPOF)
- **Scalabilité Horizontale** : Ajout simple de nouveaux workers
- **Performance Optimisée** : Compression, parallélisation, mise en cache
- **Streaming Temps Réel** : Traitement continu des données
- **Stockage Distribué** : HDFS avec réplication factor 3

---

## 🧩 Technologies

| Technologie | Version | Rôle | Configuration |
|-------------|---------|------|---------------|
| **Apache Kafka** | 3.x | Ingestion temps réel et messagerie | 3 brokers, 3 partitions, RF=2 |
| **Apache ZooKeeper** | 3.x | Coordination cluster Kafka | 3 serveurs en quorum |
| **Apache Spark** | 3.x | Traitement distribué streaming | Mode YARN, 4GB executors |
| **Apache Hadoop** | 3.x | Stockage HDFS + orchestration YARN | NameNode + 2 DataNodes, RF=3 |
| **Apache Hive** | 3.x | Data Warehouse SQL | HiveServer2, Metastore MySQL |
| **Apache Superset** | 2.x | Visualisation et BI | Dashboards interactifs |
| **Python** | 3.8+ | Scripts Producer/Consumer | kafka-python, pyspark |
| **MySQL** | 8.x | Metastore Hive | Stockage des métadonnées |

 
 
### Configuration Réseau

Les machines peuvent communiquer entre elles :

```bash
# Éditer /etc/hosts sur chaque machine
10.15.15.100   huemaster
10.15.15.101   worker1
10.15.15.102   worker2
```

**Ports à ouvrir** :
- Kafka : 9092
- ZooKeeper : 2181, 2888, 3888
- HDFS : 9000, 9870
- YARN : 8088, 8042
- Spark : 7077, 8080
- Hive : 9083, 10000
- Superset : 8088
- MySQL : 3306

---




## 📁 Structure du Projet

```
medical-risk-bigdata/
│
├── README.md                          # Documentation principale
├── requirements.txt                   # Dépendances Python
│
├── producer/
│   └── producer.py                   # Producer Kafka OpenFDA
│
├── consumer/
│   └── consumer.py                   # Consumer Spark Streaming
│
├── kafka/
│   ├── server.properties             # Configuration broker Kafka
│   ├── zookeeper.properties          # Configuration ZooKeeper
│   ├── start-kafka.sh               # Script démarrage Kafka
│   └── stop-kafka.sh                # Script arrêt Kafka
│
├── hadoop/
│   ├── core-site.xml                # Configuration Hadoop core
│   ├── hdfs-site.xml                # Configuration HDFS
│   ├── yarn-site.xml                # Configuration YARN
│   ├── mapred-site.xml              # Configuration MapReduce
│   └── workers                      # Liste des workers
│
├── spark/
│   ├── spark-env.sh                 # Variables d'environnement Spark
│   ├── spark-defaults.conf          # Configuration Spark
│   ├── start-cluster.sh            # Script démarrage cluster Spark
│   └── stop-cluster.sh             # Script arrêt cluster Spark
│
├── hive/
│   ├── hive-site.xml               # Configuration Hive
│   ├── start-hive.sh               # Script démarrage Hive
│   └── stop-hive.sh                # Script arrêt Hive
│
└── scripts/
    ├── start-all.sh                # Démarrage complet du cluster
    ├── stop-all.sh                 # Arrêt complet du cluster
    └── verify-cluster.sh           # Vérification de l'état du cluster
```

---

## 📊 Monitoring et Supervision

### 🔍 Interfaces Web de Monitoring

| Service | URL | Description |
|---------|-----|-------------|
| **HDFS NameNode** | http://huemaster:9870 | État du système de fichiers HDFS |
| **YARN ResourceManager** | http://huemaster:8088 | Gestion des applications YARN |
| **Spark Master** | http://huemaster:8080 | État du cluster Spark |
| **Spark History** | http://huemaster:18080 | Historique des jobs Spark |
| **HiveServer2** | http://huemaster:10002 | Interface web Hive |
| **Superset** | http://huemaster:8088 | Dashboards et visualisations |



## 🔗 Ressources Utiles

- [Documentation OpenFDA](https://open.fda.gov/apis/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Apache Spark Documentation](https://spark.apache.org/docs/latest/)
- [Apache Hadoop Documentation](https://hadoop.apache.org/docs/stable/)
- [Apache Hive Documentation](https://hive.apache.org/)
- [Apache Superset Documentation](https://superset.apache.org/)

---

**🎉 Félicitations ! Votre architecture Big Data pour l'analyse des risques médicamenteux est opérationnelle !**
