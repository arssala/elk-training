Analyse des journaux avec Filebeat
==
Nous allons créer un pipeline Logstash qui utilise Filebeat pour prendre les journaux Web Apache en entrée, analyser ces journaux pour créer des champs nommés spécifiques à partir des journaux et écrire les données analysées dans un cluster Elasticsearch.

Plutôt que de définir la configuration du pipeline sur la ligne de commande, nous définirons le pipeline dans un fichier de configuration.

Pour commencer, cliquez [ici](https://download.elastic.co/demos/logstash/gettingstarted/logstash-tutorial.log.gz) pour télécharger l'exemple de jeu de données ( logstash-tutorial.log.gz ) utilisé dans cet exemple. Décompressez le fichier.

## Configuration de Filebeat pour envoyer des lignes de journal à Logstash
Avant de créer le pipeline Logstash, nous souhaitons configurer Filebeat pour envoyer des lignes de log à Logstash.

Le client Filebeat est un outil léger et convivial qui collecte les journaux des fichiers sur le serveur et les transmet à notre instance Logstash pour traitement.

Filebeat est conçu pour la fiabilité et une faible latence. Filebeat a une faible empreinte de ressources sur la machine hôte, de sorte que le plug-in d'entrée Beats minimise les demandes de ressources sur l'instance Logstash.

L'installation par défaut de Logstash comprend le plug-in d'entrée Beats .

Le plug-in d'entrée Beats permet à Logstash de recevoir des événements du framework Elastic Beats, ce qui signifie que tout Beat écrit pour fonctionner avec le framework Beats, tel que Packetbeat et Metricbeat, peut également envoyer des données d'événement à Logstash.

Pour installer Filebeat sur notre machine source de données, téléchargez le package approprié à partir de la [page du produit Filebeat](https://www.elastic.co/downloads/beats/filebeat).

Pour télécharger et installer Filebeat sur Ubuntu 16.04, utilisez les commandes suivantes:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.11.0-amd64.deb
sudo dpkg -i filebeat-7.11.0-amd64.deb
```

Avant de démarrer Filebeat, nous devons examiner les options de configuration dans le fichier de configuration, par exemple /etc/filebeat/filebeat.yml . Pour plus d'informations sur ces options, consultez [Options de configuration](https://www.elastic.co/guide/en/beats/filebeat/1.2/filebeat-configuration-details.html).

nous voulons utiliser Logstash pour effectuer un traitement supplémentaire sur les données collectées par Filebeat, nous devons configurer Filebeat pour utiliser Logstash.

Pour ce faire, nous éditons le fichier de configuration Filebeat pour désactiver la sortie Elasticsearch en la commentant et nous activons la sortie Logstash en décommentant la section logstash ( /etc/filebeat/filebeat.yml ):
```
# ================================== Outputs ===================================

# Configure what output to use when sending the data collected by the beat.

# ---------------------------- Elasticsearch Output ----------------------------
#output.elasticsearch:
  # Array of hosts to connect to.
  # hosts: ["localhost:9200"]

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  #username: "elastic"
  #password: "changeme"

# ------------------------------ Logstash Output -------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key

```

Dans cette configuration, les hôtes spécifient le serveur Logstash et le port (5044) sur lequel Logstash est configuré pour écouter les connexions Beats entrantes.

Notez que nous définissons des chemins pour pointer vers l'exemple de fichier log Apache, logstash-tutorial.log , que nous avons téléchargé précédemment.

Pour garder la configuration simple, nous n'avons pas spécifié les paramètres TLS / SSL comme nous le ferions dans un scénario réel.

Pour tester notre fichier de configuration, exécutez Filebeat au premier plan avec les options suivantes spécifiées:
```
sudo filebeat test config -e -c /etc/filebeat/filebeat.yml
```

Pour utiliser cette configuration, nous devons également configurer Logstash pour recevoir les événements de Beats.

## Configurer Logstash pour recevoir un événement de Beat
Dans cette configuration, le Beat envoie des événements à Logstash. Logstash reçoit ces événements à l'aide du [plug-in d'entrée Beats](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-beats.html) pour Logstash, puis envoie la transaction à Elasticsearch à l'aide du plug-in de sortie Elasticsearch pour Logstash.

Le plugin de sortie Elasticsearch utilise l'API en masse, ce qui rend l'indexation très efficace.

Pour configurer Logstash:
1. Assurez-vous que la dernière version compatible du plug-in d'entrée Beats pour Logstash est installée. Pour installer le plugin requis, exécutez la commande suivante:
```
sudo /usr/share/logstash/bin/logstash-plugin install logstash-input-beats
```
```
Validating logstash-input-beats
Installing logstash-input-beats
Installation successful
```
2. Configurez Logstash pour écouter sur le port 5044 les connexions Beats entrantes et pour indexer dans Elasticsearch. Nous devons configurer Logstash en créant un fichier de configuration. Par exemple, nous pouvons enregistrer l'exemple de configuration suivant dans un fichier appelé /etc/logstash/conf.d/beats.conf:
```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => "localhost:9200"
    manage_template => false
    index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
    document_type => "%{[@metadata][type]}"
  }
}
```
### Mise à jour du plug-in d'entrée Beats pour Logstash (Facultatif)
Logstash utilise cette configuration pour indexer les événements dans Elasticsearch de la même manière que le Beat, mais nous bénéficions d'une mise en mémoire tampon supplémentaire et d'autres fonctionnalités fournies par Logstash.

Mise à jour du plug-in d'entrée Beats pour Logstash
Les plugins ont leur propre cycle de publication et sont souvent publiés indépendamment du cycle de publication principal de Logstash. Pour vous assurer que nous disposons de la dernière version du plug-in d'entrée Beats pour Logstash, exécutez la commande suivante:
```
sudo /usr/share/logstash/bin/logstash-plugin update logstash-input-beats
```
Gardez à l'esprit que nous pouvons mettre à jour vers la dernière version du plugin sans avoir à passer à une version plus récente de Logstash. Plus de détails sur l'utilisation des plugins d'entrée dans Logstash sont disponibles ici .

Lancez Logstash:
```
sudo systemctl restart logstash
```
## Query into Elasticsearch
Vérifions si Logstash a indexé le journal de filebeat avec succès et le mettons dans Elasticsearch:
```
curl -XGET 'http://localhost:9200/beats-2015.01.04/_search?pretty&q=geoip.city_name=Buffalo'
```
