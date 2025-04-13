# ELK Stack - Filebeat Django App (AWS EC2 Deployment)
Description
Ce projet déploie une application Django dans un environnement AWS EC2 en utilisant la stack ELK (Elasticsearch, Logstash, Kibana) pour collecter, analyser et visualiser les logs d'application en temps réel. L'application Django envoie les logs à Elasticsearch via Filebeat, et Kibana est utilisé pour afficher ces logs.

Prérequis
Avant de commencer, vous devez avoir installé ou configuré les éléments suivants :

AWS EC2 : Une instance EC2 exécutant Ubuntu ou une autre distribution Linux compatible.

SSH : Vous devez avoir un accès SSH à votre instance EC2.

Python 3.12+ et Django 4.x installés.

Elasticsearch, Logstash, et Kibana installés manuellement sur des instances EC2 (hors Docker).

Filebeat installé pour envoyer les logs de l'application Django à Elasticsearch.

UFW (Uncomplicated Firewall) ou un autre pare-feu pour configurer l'accès aux ports nécessaires.

Architecture
Serveur EC2 avec Django : Application Django sur une instance EC2.

Elasticsearch : Instance EC2 dédiée à Elasticsearch.

Kibana : Instance EC2 dédiée à Kibana pour visualiser les logs.

Filebeat : Filebeat installé sur l'instance EC2 pour envoyer les logs de Django à Elasticsearch.

Étapes de Déploiement


1. Préparer les serveurs EC2
a. Lancer les instances EC2
Lancez une instance EC2 avec Ubuntu 20.04 ou supérieur.

Ouvrez les ports nécessaires (9200 pour Elasticsearch, 5601 pour Kibana, et 5044 pour Filebeat).

Configurez votre sécurité et votre clé SSH pour vous connecter à votre instance EC2.

b. Se connecter via SSH
Connectez-vous à votre instance EC2 avec SSH :
ssh -i /path/to/your-key.pem ubuntu@<EC2_PUBLIC_IP>


2. Installer la stack ELK (Elasticsearch, Logstash, Kibana)
a. Installer Elasticsearch
Ajoutez la clé GPG d'Elastic 
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
Ajoutez le dépôt Elastic à vos sources APT :
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
Mettez à jour les sources et installez Elasticsearch :
sudo apt update
sudo apt install elasticsearch -y
Configurez Elasticsearch pour qu'il démarre automatiquement et démarrez le service :
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
Vérifiez qu'Elasticsearch fonctionne en accédant à l'URL :
curl -X GET "localhost:9200/"

b. Installer Kibana
Installez Kibana :
sudo apt install kibana -y
Activez Kibana pour qu'il démarre automatiquement :
sudo systemctl enable kibana
Démarrez le service Kibana :
sudo systemctl start kibana
Accédez à Kibana via le navigateur :
http://<EC2_PUBLIC_IP>:5601

c. Vérifiez les services
Assurez-vous que les services sont correctement installés et fonctionnent :
sudo systemctl status elasticsearch
sudo systemctl status kibana




3. Installer et Configurer Filebeat
a. Installer Filebeat
Installez Filebeat sur votre instance EC2 :
sudo apt-get install filebeat -y
Configurez Filebeat pour envoyer les logs de Django vers Elasticsearch en modifiant le fichier /etc/filebeat/filebeat.yml :
sudo nano /etc/filebeat/filebeat.yml
Ajoutez la configuration suivante pour envoyer les logs de Django à Elasticsearch (remplacez <your_elasticsearch_ip> par l'adresse IP de votre instance Elasticsearch) :
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /path/to/django/logs/*.log

output.elasticsearch:
  hosts: ["http://<your_elasticsearch_ip>:9200"]

b. Copier la configuration de Filebeat dans le répertoire du projet
cp /etc/filebeat/filebeat.yml /home/ubuntu/elk-filebeat-djangoApp/elk-stack/filebeat/filebeat.yml

c. Démarrer Filebeat
Activez et démarrez Filebeat :
sudo systemctl enable filebeat
sudo systemctl start filebeat
Vérifiez que Filebeat fonctionne correctement en vérifiant son état :
sudo systemctl status filebeat


4. Configurer Django pour les logs
Dans votre application Django, vous devez configurer les logs afin qu'ils soient stockés dans un répertoire lisible par Filebeat. Modifiez settings.py dans votre projet Django pour configurer le logging :

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'DEBUG',
            'class': 'logging.FileHandler',
            'filename': '/path/to/django/logs/django.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}


5. Vérification
a. Accéder à Kibana
Après avoir configuré Filebeat et Django, vous pouvez visualiser les logs dans Kibana :
http://<EC2_PUBLIC_IP>:5601
Créez un index dans Kibana pour afficher les logs envoyés par Filebeat.

b. Vérifier les logs dans Elasticsearch
Vous pouvez également vérifier si les logs sont envoyés correctement dans Elasticsearch via :
curl -X GET "http://<your_elasticsearch_ip>:9200/filebeat-*/_search?pretty"


Dépannage
1. Problème avec Kibana
Si Kibana ne démarre pas, vous pouvez vérifier les logs avec :
sudo journalctl -u kibana -f
Si vous voyez une erreur liée à OpenSSL, vous pouvez la résoudre en suivant les instructions de la documentation d'Elastic.

2. Problème avec Filebeat
Si les logs ne sont pas envoyés correctement à Elasticsearch, vous pouvez vérifier l'état de Filebeat avec :sudo systemctl status filebeat
Et consultez les logs de Filebeat avec :sudo journalctl -u filebeat -f

Conclusion
Ce projet déploie avec succès une application Django avec la stack ELK sur AWS EC2. Vous avez configuré Elasticsearch pour l'indexation des logs, Kibana pour la visualisation, et Filebeat pour envoyer les logs de l'application Django.

Pour plus de détails, consultez la documentation officielle de Django, Elasticsearch, Kibana et Filebeat.
