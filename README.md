Larrode Alexis <br>
Cruz Julien

# Projet Docker 1: WordPress et MariaDB avec Docker Compose

## Description
Ce projet Docker vise à déployer une instance WordPress avec une base de données MariaDB. Le déploiement est facilité par l'utilisation de Docker Compose.

## Composants
1. **MariaDB**: Une image Docker personnalisée basée sur `mariadb:latest`, configurée avec un fichier `my.cnf` pour des réglages spécifiques comme la gestion de la journalisation et des paramètres de la base de données.

2. **WordPress**: Utilisation d'une image WordPress personnalisée (`fanswerss/wordpress:1.0`) qui se connecte à la base de données MariaDB.

3. **Docker Compose**: Gère la coordination entre le service WordPress et le service MariaDB, en définissant les réseaux, les volumes, et les variables d'environnement nécessaires.

## Configuration
- **Dockerfile (MariaDB)**:
  ```Dockerfile
  FROM mariadb:latest

  COPY my.cnf /etc/mysql/my.cnf

  EXPOSE 3306
  ```
- **my.cnf**:
  ```ini
  [mysqld]
  character-set-server=utf8mb4
  collation-server=utf8mb4_unicode_ci

  key_buffer_size=64M
  innodb_buffer_pool_size=256M

  log_error=/var/log/mysql/error.log
  general_log=ON
  general_log_file=/var/log/mysql/general.log

  skip-networking
  ```
- **.env**: Contient des variables d'environnement pour les services WordPress et MariaDB, incluant les informations d'authentification et de configuration de la base de données.
- **docker-compose.yml**: Configure les services, les réseaux et les volumes nécessaires pour le déploiement de WordPress et MariaDB.

## Déploiement avec Docker Compose
1. **Construire et Démarrer les Services**:
   Exécutez `docker-compose up -d` pour démarrer les services en mode détaché.

2. **Accès à WordPress**:
   WordPress sera accessible à l'adresse `http://localhost:8080`.

3. **Gestion de la Base de Données**:
   MariaDB est accessible sur le port `3306`. Les fichiers de la base de données sont stockés dans un volume Docker pour la persistance des données.

## Sécurité et Maintenance
- Changez les mots de passe par défaut dans le fichier `.env` avant le déploiement en production.
- Assurez-vous que les journaux de MariaDB (`general.log` et `error.log`) sont surveillés et gérés correctement.



# Projet Docker 2: Elasticsearch et Kibana avec Docker Compose

## Description
Ce projet Docker vise à configurer et déployer un cluster Elasticsearch avec une interface Kibana pour l'analyse et la visualisation des données. Le déploiement est géré par Docker Compose, facilitant la mise en place et la configuration du cluster.

## Composants
1. **Elasticsearch**: Un cluster Elasticsearch composé de trois nœuds (`es01`, `es02`, `es03`) pour la gestion et l'indexation des données.

2. **Kibana**: Un service Kibana pour l'interface utilisateur, permettant la visualisation et l'analyse des données stockées dans Elasticsearch.

## Configuration
- **elasticsearch.yml**: Configurations du cluster Elasticsearch, y compris les réglages de réseau et les paramètres du cluster.
  ```yml
  # Contenu du fichier elasticsearch.yml
xpack.security.enabled: false
cluster.name: my-elasticsearch-cluster
node.name: node-1
network.host: 0.0.0.0
discovery.seed_hosts: ["es01", "es02", "es03"]
cluster.initial_master_nodes: ["es01", "es02", "es03"]

  ```
- **kibana.yml**: Configuration de Kibana, incluant la connexion au cluster Elasticsearch.
  ```yml
  # Contenu du fichier kibana.yml
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://es01:9200", "http://es02:9201", "http://es03:9203"]

  ```
- **.env**: Contient les variables d'environnement, notamment la version d'Elasticsearch.
  ```env
  ES_VERSION=8.11.3
  ```
- **docker-compose.yml**: Configure les services, les réseaux, et les volumes pour Elasticsearch et Kibana.
  ```yml
  version: '3.8'

services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:${ES_VERSION}
    container_name: es01
    environment:
      - node.name=es01
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - cluster.name=my-elasticsearch-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata01:/usr/share/elasticsearch/data
      - ./es-cluster/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    networks:
      - elastic
    ports:
      - "9200:9200"

  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:${ES_VERSION}
    container_name: es02
    environment:
      - node.name=es02
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - cluster.name=my-elasticsearch-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata02:/usr/share/elasticsearch/data
      - ./es-cluster/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    networks:
      - elastic
    ports:
      - "9201:9200"

  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:${ES_VERSION}
    container_name: es03
    environment:
      - node.name=es03
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - cluster.name=my-elasticsearch-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata03:/usr/share/elasticsearch/data
      - ./es-cluster/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    networks:
      - elastic
    ports:
      - "9202:9200"

  kibana:
    image: docker.elastic.co/kibana/kibana:${ES_VERSION}
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://es01:9200
    networks:
      - elastic

networks:
  elastic:
    driver: bridge

volumes:
  esdata01:
    driver: local
  esdata02:
    driver: local
  esdata03:
    driver: local

  ```

## Déploiement avec Docker Compose
1. **Construire et Démarrer les Services**:
   Exécutez `docker-compose up -d` pour démarrer le cluster Elasticsearch et Kibana en mode détaché.

2. **Accès à Kibana**:
   Kibana sera accessible à l'adresse `http://localhost:5601`.

3. **Gestion du Cluster Elasticsearch**:
   Le cluster Elasticsearch est accessible sur les ports `9200`, `9201`, et `9202` pour les différents nœuds.

## Sécurité et Maintenance
- Assurez-vous de configurer des mesures de sécurité appropriées, en particulier si le cluster est accessible sur Internet.
- Surveillez l'état et les performances du cluster Elasticsearch et de Kibana.
