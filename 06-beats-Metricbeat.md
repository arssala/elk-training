Metricbeat: installation et configuration

Metricbeat vous aide à surveiller vos serveurs et les services qu'ils hébergent en collectant des métriques à partir du système d'exploitation et des services.

Ce guide explique comment démarrer rapidement avec la collecte de métriques. Vous apprendrez à:

* installer Metricbeat sur chaque système que vous souhaitez surveiller
* spécifier les métriques que vous souhaitez collecter
* envoyer les métriques à Elasticsearch
* visualiser les données métriques dans Kibana

![](images/metricbeat-system-dashboard.png)

## Étape 1: Installez Metricbeat
Installez Metricbeat aussi près que possible du service que vous souhaitez surveiller. Par exemple, si vous avez quatre serveurs avec MySQL en cours d'exécution, il est recommandé d'exécuter Metricbeat sur chaque serveur. Cela permet à Metricbeat d'accéder à votre service à partir de l'hôte local et ne provoque aucun trafic réseau supplémentaire ni n'empêche Metricbeat de collecter des métriques en cas de problèmes de réseau. Les métriques de plusieurs instances Metricbeat seront combinées sur le serveur Elasticsearch.

Pour télécharger et installer Metricbeat, utilisez les commandes qui fonctionnent avec votre système:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.11.0-amd64.deb
sudo dpkg -i metricbeat-7.11.0-amd64.deb
```

## Étape 2: Connectez-vous à Elastic Stack
Des connexions à Elasticsearch et Kibana sont nécessaires pour configurer Metricbeat.

Définissez les informations de connexion dans metricbeat.yml.

1. Définissez l'hôte et le port sur lesquels Metricbeat peut trouver l'installation d'Elasticsearch. Par exemple:
```
output.elasticsearch:
  hosts: ["192.168.33.10:9200"]
```

2. Si vous prévoyez d'utiliser les tableaux de bord Kibana prédéfinis, configurez le point de terminaison Kibana. Ignorez cette étape si Kibana s'exécute sur le même hôte qu'Elasticsearch.
```
setup.kibana:
   host: "192.168.33.10:5601"
```
## Étape 3: Activer et configurer les modules de collecte de métriques
Metricbeat utilise des modules pour collecter des métriques. Chaque module définit la logique de base pour collecter les données d'un service spécifique, tel que Redis ou MySQL. Un module se compose d'ensembles de métriques qui récupèrent et structurent les données.

1. Identifiez les modules que vous devez activer.
```
metricbeat modules list
```

2. Depuis le répertoire d'installation, activez un ou plusieurs modules. Si vous acceptez la configuration par défaut sans activer de modules supplémentaires, Metricbeat collecte uniquement les métriques système.

La commande suivante active les configurations apache et mysql dans le répertoire modules.d:

```
metricbeat modules enable apache mysql
```

3. Dans la configuration du module sous modules.d, modifiez les paramètres du module pour qu'ils correspondent à votre environnement.


Pour tester votre fichier de configuration, passez dans le répertoire où le binaire Metricbeat est installé et exécutez Metricbeat au premier plan avec les options suivantes spécifiées: ./metricbeat test config -e.

## Étape 4: configurer les assets
Metricbeat est fourni avec des ressources prédéfinies pour l'analyse, l'indexation et la visualisation de vos données. Pour charger ces éléments:
```
metricbeat setup -e
```
Cette étape charge le modèle d'index recommandé pour l'écriture dans Elasticsearch et déploie les exemples de tableaux de bord pour visualiser les données dans Kibana.

## Étape 5: Démarrez Metricbeat
```
sudo systemctl start metricbeat
```

## Étape 6: Affichez vos données dans KibanaÉditer
Metricbeat est livré avec des tableaux de bord et des interfaces utilisateur Kibana prédéfinis pour visualiser les données de journal. Vous avez chargé les tableaux de bord plus tôt lorsque vous avez exécuté la commande `setup`.

Pour ouvrir les tableaux de bord:

1. Lancez Kibana:

Dans la navigation latérale, cliquez sur **Discover**. Pour afficher les données Metricbeat, assurez-vous que le modèle d'index prédéfini **metricbeat-\*** est sélectionné.

Si vous ne voyez pas de données dans Kibana, essayez de changer le filtre temporel sur une plage plus large. Par défaut, Kibana affiche les 15 dernières minutes.

2. Dans la navigation latérale, cliquez sur Tableau de bord , puis sélectionnez le tableau de bord que vous souhaitez ouvrir.
Les tableaux de bord sont fournis à titre d'exemples. Nous vous recommandons de les personnaliser pour répondre à vos besoins.
