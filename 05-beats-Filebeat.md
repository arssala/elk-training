Filebeat : installation and configuration
==
Cet atelier décrit comment démarrer rapidement avec la collecte des logs. Vous apprendrez à:

* installer Filebeat sur chaque système que vous souhaitez surveiller
* spécifier l'emplacement de vos fichiers journaux
* analyser les données du journal dans des champs et les envoyer à Elasticsearch
* visualiser les données du journal dans Kibana

## Étape 1: Installez Filebeat

Installez Filebeat sur tous les serveurs que vous souhaitez surveiller.

Pour télécharger et installer Filebeat, utilisez les commandes qui fonctionnent avec votre système:

Debian:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.11.0-amd64.deb
sudo dpkg -i filebeat-7.11.0-amd64.deb
```
RHEL
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.11.0-x86_64.rpm
sudo rpm -vi filebeat-7.11.0-x86_64.rpm
```

## Étape 2: Connectez-vous à Elastic Stack

Des connexions à Elasticsearch et Kibana sont nécessaires pour configurer Filebeat.

Définissez les informations de connexion dans filebeat.yml.

Définissez l'hôte et le port sur lesquels Filebeat peut trouver l'installation d'Elasticsearch. Par exemple:
```
output.elasticsearch:
  hosts: ["localhost:9200"]
```
## Étape 3: Collectez les données du journalÉditer
Il existe plusieurs façons de collecter des données de journal avec Filebeat:

* Modules de collecte de données - simplifient la collecte, l'analyse et la visualisation des formats de journaux courants
* ECS loggers - structure et formatage des journaux d'application dans un JSON compatible ECS
* Configuration manuelle de Filebeat

### Activer et configurer les modules de collecte de données
1. Identifiez les modules que vous devez activer. Pour voir une liste des modules disponibles , exécutez:
```
filebeat modules list
```
2. Depuis le répertoire d'installation, activez un ou plusieurs modules. Par exemple, la commande suivante active la configuration des modules system, nginx et mysql:
```
filebeat modules enable system apache mysql
```
3. Dans la configuration du module sous `modules.d`, modifiez les paramètres du module pour qu'ils correspondent à votre environnement.

Par exemple, les emplacements des journaux sont définis en fonction du système d'exploitation. Si vos journaux ne sont pas dans les emplacements par défaut,
définissez la variable `paths`:
```
- module: apache
  access:
    var.paths: ["/var/log/apache2/access.log*"]
```

Dans le cas où, filebeat réside dans un serveur distant, il faut changer la configuration de Elasticsearch pour supporter les connexions sur son adresse IP réseau.
N'oubliez pas d'activer la Discovery à partir de l'adresse IP du serveur distant:

```
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: "192.168.33.10"
#
# Set a custom port for HTTP:
#
http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.seed_hosts: ["192.168.33.20"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
#cluster.initial_master_nodes: ["node-1", "node-2"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
```

Faites attention si il faut aussi changer la configuration de Kibana.
