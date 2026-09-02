# 📘 AMD AI Server with Ubuntu, Ollama and Open WebUI

Hardware used: [GMKtec Evo-X2 AI Mini-PC AMD Ryzen AI Max+ 395 (5.1GHz, 96GB LPDDR5X 8000MHz (12GB*8), 1TB PCIe 4.0 SSD](https://www.amazon.fr/dp/B0FK2299GS)

## 🔹 Step 1 : System Preparation

We assume the server already has Ubuntu Server installed.

1. **Modify the VRAM reserved in the BIOS** to free up maximum memory for the GPU (e.g., 512MB or 1GB).

2. **System update** :
```bash
sudo apt update && sudo apt dist-upgrade -y
```

## 🔹 Step 2 : AMD Driver Installation (ROCm)

1. **Download and install the AMDGPU package** :
```bash
wget https://repo.radeon.com/amdgpu-install/31.30/ubuntu/noble/amdgpu-install_31.30.313000-1_all.deb
sudo dpkg -i amdgpu-install_31.30.313000-1_all.deb
sudo apt update
```

2. **Install ROCm tools and graphics components** :
```bash
sudo amdgpu-install --usecase=graphics,rocm
```

3. **Add to `render` and `video` groups for GPU access** :
```bash
sudo usermod -aG render,video $LOGNAME
```

4. **Reboot the server** :
```bash
sudo reboot
```

## 🔹 Step 3 : GPU Memory Management Optimization

**Adjust memory page limit (VRAM management)** :
```bash
sudo nano /etc/modprobe.d/amdttm.conf
```

The file content depends on how much memory you want to allocate to the GPU (plan for a minimum of 44 GB):
- To allow 44 GB (ideal for 64 GB machines) : 44 * 1024 * 1024 / 4, specify `11534336` (4k pages)
- To allow 72 GB (ideal for 96 GB machines) : 72 * 1024 * 1024 / 4, specify `18874368` (4k pages)
- To allow 96 GB (ideal for 128 GB machines) : 96 * 1024 * 1024 / 4, specify `25165824` (4k pages)

For example for 72 GB :
```
options amdttm pages_limit=18874368
options amdttm page_pool_size=18874368
```

**Apply settings and reboot** :
```bash
sudo update-initramfs -u -k all
sudo reboot
```

## 🔹 Step 4 : Docker & Docker Compose Installation

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose-v2 -y
sudo systemctl enable --now docker
sudo usermod -aG docker $LOGNAME
```

## 🔹 Step 5 : Ollama Installation and Configuration

1. **Installation via the official script** :
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

2. **Modify the systemd service to expose the API and increase the context window** :
Create or edit the override file with :
```bash
sudo systemctl edit ollama
```
Insert the following content :
```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_KEEP_ALIVE=-1"
```

Expose Ollama on the local network, and enable keep-alive mode indefinitely (to limit wait times related to cold model startup).

3. **Reload and restart the service** :
```bash
sudo systemctl restart ollama.service
```

## 🔹 Step 6 : LLM Model Download

1. **Download Qwen3.6 and Gemma models via Ollama** :

```bash
ollama pull jetelain/Qwen3.6-35B-A3B
ollama pull jetelain/Qwen3.6-35B-A3B-Code
ollama pull jetelain/Gemma-4-26B-A4B
ollama pull jetelain/Qwen3.8-27B
ollama pull jetelain/Gemma-4-31B
```

2. **Add aliases with configuration adapted to each model** :

```bash
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B",
  "from": "jetelain/Qwen3.6-35B-A3B"
}'

curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B-Code",
  "parameters": {
    "num_ctx": 196608
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
    "num_ctx": 196608
  }
}'

curl http://localhost:11434/api/create -d '{
  "model": "Gemma-4-31B",
  "from": "jetelain/Gemma-4-31B"
}'
```

## 🔹 Step 7 : Open WebUI Deployment via Docker Compose

1. **Create the working directory** :
```bash
mkdir -p /opt/open-webui
cd /opt/open-webui
```

2. **Create the `docker-compose.yml` file** :

```bash
sudo nano docker-compose.yml
```

File content :
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

3. **Start the container** :
```bash
docker compose up -d
```

4. **Access the interface** :

Open a browser at `http://[server IP]:3000/` to configure the Open WebUI administrator account

## 🔹 Post-installation Verification and Monitoring

Run these commands to validate proper functioning :
```bash
# Ollama API status
ollama list
ollama ps

# GPU monitoring & ROCm memory allocation
rocm-smi --showpids
rocminfo | grep -i "Marketing Name:"

# General system monitoring
htop
df -h
free -h
```

## 🔹 Maintenance / Update

1. **Ubuntu** :
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