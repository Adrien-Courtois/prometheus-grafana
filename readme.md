# Sommaire

- [Sommaire](#sommaire)
- [Installations](#installations)
  - [Prometheus](#prometheus)
    - [Installation des outils](#installation-des-outils)
    - [Configuration des users](#configuration-des-users)
    - [Configuration du service](#configuration-du-service)
  - [Grafana](#grafana)
    - [Installation](#installation)
    - [Configuration du service](#configuration-du-service-1)
  - [Node exporter](#node-exporter)
    - [Installation](#installation-1)
    - [Configuration du service](#configuration-du-service-2)
- [Relevé de metrics](#relevé-de-metrics)
  - [Prometheus](#prometheus-1)
  - [Grafana](#grafana-1)


# Installations

## Prometheus

Prometheus permet de stocker les metrics récupérer par des exporters (tels que node_exporter pour le hardware) voici la [liste](https://prometheus.io/docs/instrumenting/exporters/) des exporters reconnu par Prometheus

### Installation des outils

Création des dossiers de prometheus
```
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
```

Téléchargement de la dernière version de prometheus
```
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -i -
```

Ou d'une version spécifique
```
wget https://github.com/prometheus/prometheus/releases/download/v2.31.0/prometheus-2.31.0.linux-amd64.tar.gz

```

Extraction de l'archive
```
tar -xvf prometheus*.tar.gz
cd prometheus*/
```

Déplacement des fichiers
```
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
cd ..
rm -r prometheus*
```

Vérification de l'installation
```
prometheus --version
```

### Configuration des users

Création du groupe prometheus

```
sudo groupadd --system prometheus
```

Création du user prometheus

```
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

Changement de permission
```
sudo chown -R prometheus:prometheus /etc/prometheus/ /var/lib/prometheus/
sudo chmod -R 775 /etc/prometheus/ /var/lib/prometheus/
```

### Configuration du service

Création du fichier systemd
```
sudo nano /etc/systemd/system/prometheus.service
```

Copier le contenu suivant 
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=always
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target
```

Sauvegarder puis démarrer le service
```
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

Vous devriez voir apparaître dans votre navigateur WEB sur le port 9090 l'interface prometheus

## Grafana

Grafana permet de créer des dashboard afin de visualiser les données de façon clair et simple

### Installation

Récupération des keyrings
```
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
```

Installation du repository de grafana (stable)
```
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Installation de grafana
```
sudo apt-get update
sudo apt-get install -y grafana
```

### Configuration du service

Démarrage du service
```
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

Grafana est maintenant disponible via le navigateur WEB sur le port 3000 (les identifiants du premier login sont admin - admin).

Une fois arrivé sur le dashboard de Grafana nous pouvons ajouter la source de données qu'est prometheus, pour cela il suffit de cliquer sur le lien "Add your first data source", puis indiqué Prometheus et l'url vers le serveur Prometheus.

## Node exporter

Node exporter est un exporter de metrics utilisé dans prometheus, il permet d'avoir des metrics sur le hardware, d'autre exporters sont disponible [ici](https://prometheus.io/docs/instrumenting/exporters/), les exporters peuvent aussi être créés manuellement.

### Installation

Récupérer la dernière version de node exporter
```
wget https://github.com`curl -s https://github.com/prometheus/node_exporter/releases/ | grep "linux-amd64.tar.gz" | head -n 1 | cut -d '"' -f 2`
```

Décompresser l'archive
```
tar -xzf node_exporter-*.tar.gz
```

Déplacer le binaire
```
sudo mv node_exporter-*/node_exporter /usr/local/bin/
rm -r node_exporter-*
```

### Configuration du service

Éditer le fichier sytemd de node_exporter
```
sudo nano /etc/systemd/system/node_exporter.service
```

Copier le contenu suivant
```
[Unit]
Description=Node_Exporter
Wants=network-online.target
After=network-online.target

[Service]
Restart=always
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Lancer le service
```
sudo systemctl start node_exporter.service
sudo systemctl enable node_exporter.service
sudo systemctl status node_exporter.service
```

Par défaut node exporter tourne sur le port 9100

# Relevé de metrics

## Prometheus

Il faut maintenant indiquer à prometheus où aller chercher les metrics

```
sudo nano /etc/prometheus/prometheus.yml
```

Ajouter les lignes suivantes à la fin du fichier dans le bloc "scrape_configs" :
```
- job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]
```

Relancer le service
```
sudo systemctl restart prometheus.service
```

Maintenant vous devriez voir 2 targets sur l'url suivante [http://localhost:9090/targets](http://localhost:9090/targets) (localhost est à remplacer par l'IP ou le nom de domaine de votre serveur)

## Grafana

Il ne reste plus qu'à afficher les metrics de façon plus accessible

Pour cela il faut se rendre sur l'interface WEB de grafana puis sur le paneau gauche il faut survoler `Dashboard > + Import`. Pour le test j'uilise le dashboard avec l'ID 1860.

Une fois importer les metrics s'affichent sur le dashboard.