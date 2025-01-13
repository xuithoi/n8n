# N8N + LLM + SD in one shot!!!

**Quick Install Command (Non-IT)**

```bash
- VPS Có GPU: curl -L https://bit.ly/n8n_install | sh
- VPS Không GPU: curl -L https://bit.ly/n8n_install_noai | sh
- Chạy lại n8n có Ngrok: sh <(curl -L https://bit.ly/n8n_with_ngrok) (chỉ chạy sau khi chạy 1 trong 2 lệnh trên)
```

- Host by: [https://localai.io/basics/container/](https://localai.io/basics/container/)

```bash
#!/bin/bash
echo "--------- 🟢 Start install docker -----------"
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
apt-cache policy docker-ce
sudo apt install -y docker-ce
echo "--------- 🔴 Finish install docker -----------"
echo "--------- 🟢 Start install nvidia support 4 docker -----------"
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
| sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
| sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
| sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
echo "--------- 🔴 Finish install nvidia support 4 docker -----------"
echo "--------- 🟢 Start creating folder -----------"
cd ~
mkdir vol_localai
mkdir vol_n8n
sudo chown -R 1000:1000 vol_localai
sudo chmod -R 755 vol_localai
sudo chown -R 1000:1000 vol_n8n
sudo chmod -R 755 vol_n8n
echo "--------- 🔴 Finish creating folder -----------"
echo "--------- 🟢 Start docker compose up  -----------"
wget https://raw.githubusercontent.com/thangnch/MIAI_n8n_dockercompose/refs/heads/main/compose.yaml -O compose.yaml
export EXTERNAL_IP=http://"$(hostname -I | cut -f1 -d' ')"
docker compose up -d
echo "--------- 🔴 Finish! Wait a few minutes and test in browser at url $EXTERNAL_IP for n8n UI -----------"
```

```yaml

services:
	svr_localai:
    image: localai/localai:latest-gpu-nvidia-cuda-12
    container_name: cont_localai
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - /root/vol_localai:/build/models
    environment:
      - MODELS_PATH=/build/models
    command:
      - dreamshaper
      - llama-3.2-3b-instruct:q8_0
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            device_ids: ['0']
            capabilities: [gpu]
    
  svr_n8n:
    image: n8nio/n8n:1.71.3
    container_name: cont_n8n
    environment:
      - N8N_SECURE_COOKIE=false
      - GENERIC_TIMEZONE=Asia/Ho_Chi_Minh
      - N8N_EDITOR_BASE_URL=${EXTERNAL_IP}
    ports:
      - "80:5678"
    volumes:
      - /root/vol_n8n:/home/node/.n8n
```

1. n8n only

```bash
#!/bin/bash
echo "--------- 🟢 Start install docker -----------"
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
apt-cache policy docker-ce
sudo apt install -y docker-ce
echo "--------- 🔴 Finish install docker -----------"
echo "--------- 🟢 Start creating folder -----------"
cd ~
mkdir vol_n8n
sudo chown -R 1000:1000 vol_n8n
sudo chmod -R 755 vol_n8n
echo "--------- 🔴 Finish creating folder -----------"
echo "--------- 🟢 Start docker compose up  -----------"
wget https://raw.githubusercontent.com/thangnch/MIAI_n8n_dockercompose/refs/heads/main/compose_noai.yaml -O compose.yaml
export EXTERNAL_IP=http://"$(hostname -I | cut -f1 -d' ')"
docker compose up -d
echo "--------- 🔴 Finish! Wait a few minutes and test in browser at url $EXTERNAL_IP for n8n UI -----------"
```

```bash

services:
  svr_n8n:
    image: n8nio/n8n:1.73.1
    container_name: cont_n8n
    environment:
      - N8N_SECURE_COOKIE=false
      - GENERIC_TIMEZONE=Asia/Ho_Chi_Minh
      - N8N_EDITOR_BASE_URL=${EXTERNAL_IP}
    ports:
      - "80:5678"
    volumes:
      - /root/vol_n8n:/home/node/.n8n
```

1. Ngrok

```bash
#!/bin/bash                                                                                sngrok.sh                                                                                                  
echo "--------- 🟢 Start Docker compose down  -----------"
docker compose down
echo "--------- 🔴 Finish Docker compose down -----------"
echo "--------- 🟢 Start Ngrok setup -----------"
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
sudo tar xvzf ./ngrok-v3-stable-linux-amd64.tgz -C /usr/local/bin
sudo apt install -y jq
echo "🔴🔴🔴 Please login into ngrok.com and paste your token here:"
read token
ngrok config add-authtoken $token
ngrok http 80 > /dev/null &
echo "🔴🔴🔴 Please wait Ngrok to start...."
sleep 5
export EXTERNAL_IP="$(curl http://localhost:4040/api/tunnels | jq ".tunnels[0].public_url")"
echo Got Ngrok URL = $EXTERNAL_IP
echo "--------- 🔴 Finish Ngrok setup -----------"
echo "--------- 🟢 Start Docker compose up  -----------"
docker compose up -d
echo "--------- 🔴 Finish! Wait a few minutes and test in browser at url $EXTERNAL_IP for n8n UI -----------"
```
