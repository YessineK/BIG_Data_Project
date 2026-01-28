# Architecture Big Data Distribuée pour l’Analyse des Risques Médicamenteux

📌 Présentation du projet

Ce projet Big Data vise à analyser les risques médicamenteux à partir des données publiques de l’API OpenFDA.
Il permet d’identifier les effets indésirables, les médicaments à haut risque et d’extraire des indicateurs d’aide à la décision pour les professionnels de santé.

Projet réalisé dans le cadre du module Architecture Big Data Distribuée à la Faculté des Sciences de Sfax.



🎯 Objectifs

Surveiller les tendances des effets indésirables

Identifier les médicaments à haut risque

Exploiter des données massives en temps réel et batch

Fournir des tableaux de bord interactifs

📊 Données

Source : API OpenFDA

Type : JSON semi-structuré

Contenu :

Médicaments

Effets secondaires

Gravité des événements

Dates de déclaration


🏗️ Architectures mises en œuvre

#🔹Pipeline Big Data Complet

Kafka + Spark + Hadoop + Hive + Superset

Pipeline :

Kafka : ingestion temps réel des données OpenFDA

Spark Streaming (YARN) : traitement distribué

HDFS : stockage distribué

Hive : entrepôt analytique (Parquet)

Superset : visualisation et dashboards

📌 Architecture haute disponibilité (3 nœuds, réplication, tolérance aux pannes)


🧩 Technologies utilisées

Apache Kafka

Apache Spark (Streaming + Batch)

Apache Hadoop (HDFS, YARN)

Apache Hive

Apache Doris

Apache Superset

Python (Kafka Producer, Spark Consumer)

🗂️ Structure du projet
├── producer/
│   └── producer.py        # Ingestion OpenFDA → Kafka
├── consumer/
│   └── consumer.py        # Spark Streaming → Hive
├── kafka/
│   ├── server.properties
│   └── start-kafka.sh
├── spark/
│   ├── spark-env.sh
│   └── spark-defaults.conf
├── hadoop/
│   ├── core-site.xml
│   ├── hdfs-site.xml
│   └── yarn-site.xml
├── hive/
│   └── hive-site.xml
└── README.md

▶️ Exécution  

Démarrer Hadoop & YARN

start-dfs.sh
start-yarn.sh


Démarrer Kafka + ZooKeeper

./start-kafka.sh


Lancer le Producer Kafka

python producer.py


Lancer le Consumer Spark

spark-submit consumer.py


Accéder à Superset

http://huemaster:8088

📈 Résultats

Détection d’effets secondaires fréquents

Identification de médicaments à risque élevé

 
