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

### Configuration du service prometheus

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