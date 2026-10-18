
# 📘 Serveur AMD AI avec Ubuntu, Ollama et Open WebUI

Matériel utilisé : [GMKtec Evo-X2 AI Mini-PC AMD Ryzen AI Max+ 395 (5,1GHz), 96GB LPDDR5X 8000MHz (12GB*8), 1TB PCIe 4.0 SSD](https://www.amazon.fr/dp/B0FK2299GS)

## 🔹 Étape 1 : Préparation du système

On suppose que le serveur dispose déjà d'Ubuntu Server.

1. **Modifier la VRAM réservée dans le BIOS** pour libérer un maximum de mémoire pour le GPU (ex: 512MB ou 1GB).

2. **Mise à jour du système** :
```bash
sudo apt update && sudo apt dist-upgrade -y
```

## 🔹 Étape 2 : Installation des pilotes AMD (ROCm)

1. **Téléchargement et installation du package AMDGPU** :
```bash
wget https://repo.radeon.com/amdgpu-install/31.30/ubuntu/noble/amdgpu-install_31.30.313000-1_all.deb
sudo dpkg -i amdgpu-install_31.30.313000-1_all.deb
sudo apt update
```

2. **Installation de l'outil ROCm et des composants graphiques** :
```bash
sudo amdgpu-install --usecase=graphics,rocm
```

3. **Ajout aux groupes `render` et `video` pour l'accès GPU** :
```bash
sudo usermod -aG render,video $LOGNAME
```

4. **Redémarrage du serveur** :
```bash
sudo reboot
```

## 🔹 Étape 3 : Optimisation de la gestion mémoire GPU

**Ajustement de la limite de pages mémoire (gestion de la VRAM)** :
```bash
sudo nano /etc/modprobe.d/amdttm.conf
```

Le contenu du fichier dépend de la quantité de mémoire que vous souhaitez accorder au GPU (prévoir au minimum 44 Go)
- Pour autoriser 44 Go (idéal pour les machines à 64 Go) : 44 * 1024 * 1024 / 4, spécifier `11534336` (pages de 4k)
- Pour autoriser 72 Go (idéal pour les machines à 96 Go) : 72 * 1024 * 1024 / 4, spécifier `18874368` (pages de 4k)
- Pour autoriser 96 Go (idéal pour les machines à 128 Go) : 96 * 1024 * 1024 / 4, spécifier `25165824` (pages de 4k)

Par exemple pour 72 Go :
```
options amdttm pages_limit=18874368
options amdttm page_pool_size=18874368
```

**Appliquer les paramètres et relancer la machine**
```bash
sudo update-initramfs -u -k all
sudo reboot
```

## 🔹 Étape 4 : Installation de Docker & Docker Compose

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose-v2 -y
sudo systemctl enable --now docker
sudo usermod -aG docker $LOGNAME
```

## 🔹 Étape 5 : Installation et configuration d'Ollama
1. **Installation via le script officiel** :
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

2. **Modification du service systemd pour exposer l'API et augmenter la fenêtre de contexte** :
Créez ou éditez le fichier de remplacement avec :
```bash
sudo systemctl edit ollama
```
Insérez le contenu suivant :
```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_KEEP_ALIVE=-1"
```

Expose Ollama sur le réseau local, et active le mode keep-alive indéfiniment (pour limiter les temps d'attente liés au démarrage à froid du modèle).

3. **Rechargement et redémarrage du service** :
```bash
sudo systemctl restart ollama.service
```

## 🔹 Étape 6 : Téléchargement des modèles LLM

1. **Téléchargement des modèles Qwen3.6 et Gemma via Ollama** :

```bash
ollama pull jetelain/Qwen3.6-35B-A3B
ollama pull jetelain/Qwen3.6-35B-A3B-Code
ollama pull jetelain/Gemma-4-26B-A4B
ollama pull jetelain/Qwen3.8-27B
ollama pull jetelain/Gemma-4-31B
```

2. **Ajout d'alias avec une configuration adaptée à chaque modèle** :

```bash
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B",
  "from": "jetelain/Qwen3.6-35B-A3B"
}'

curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B-Code",
  "parameters": {
    "num_ctx": 225280
  }
}'

curl http://localhost:11434/api/create -d '{
  "model": "Gemma-4-26B-A4B",
  "from": "jetelain/Gemma-4-26B-A4B"
}'

curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.8-27B",
  "from": "jetelain/Qwen3.8-27B",
  "parameters": {
    "num_ctx": 225280
  }
}'

curl http://localhost:11434/api/create -d '{
  "model": "Gemma-4-31B",
  "from": "jetelain/Gemma-4-31B"
}'
```

## 🔹 Étape 7 : Déploiement d'Open WebUI via Docker Compose
1. **Création du répertoire de travail** :
```bash
mkdir -p /opt/open-webui
cd /opt/open-webui
```

2. **Création du fichier `docker-compose.yml`** :

```bash
sudo nano docker-compose.yml
```

Contenu du fichier:
```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    ports:
      - "3000:8080"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      - OLLAMA_BASE_URL=http://host.docker.internal:11434
      - ENABLE_SIGNUP=false
    volumes:
      - ./data:/app/backend/data
```

3. **Lancement du conteneur** :
```bash
docker compose up -d
```

4. **Accès à l'interface** :

Ouvrir un navigateur sur `http://[ip du serveur]:3000/` pour configurer le compte administrateur d'Open WebUI

## 🔹 Vérification et surveillance post-installation
Exécutez ces commandes pour valider le bon fonctionnement :
```bash
# État de l'API Ollama
ollama list
ollama ps

# Monitoring GPU & Allocation mémoire ROCm
rocm-smi --showpids
rocminfo | grep -i "Marketing Name:"

# Surveillance système générale
htop
df -h
free -h
```

## 🔹 Maintenance / Mise à jour

1. **ubuntu** :
```bash
sudo apt update && sudo apt dist-upgrade -y
```

2. **Ollama** :
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

3. **Open WebUI** :
```bash
cd /opt/open-webui
docker compose up --pull always -d
```