# Setting up Tempo Node
Tempo is Stripe and Paradigm's blockchain built for stablecoin payments. It runs on the Reth SDK and offers sub-second finality with fees paid directly in stablecoins.

<img width="7200" height="4050" alt="image" src="https://github.com/user-attachments/assets/e08f5f07-32bf-4d77-90b2-cd116c020f2c" />

---

## Tempo raised $500M at a $5B Valuation which includes Investors like Sequoia Capital, Ribbit Capital, Paradigm among few other Investors. 
<img width="4020" height="2362" alt="image" src="https://github.com/user-attachments/assets/d5637aec-15a6-4107-a73a-6022d214af29" />

---

### Type of Tempo Nodes: 
   * RPC Node:  Open to everyone. Serves API requests for dApps and wallets.
   * Validator Node: This is Permissioned as it requires Tempo team approval to produce blocks.
<img width="2918" height="410" alt="image" src="https://github.com/user-attachments/assets/16b6df62-774f-4693-841d-e18b6b264fc8" />




### Update package: 
```
sudo apt-get update && sudo apt-get upgrade -y 
```
<img width="3700" height="1020" alt="image" src="https://github.com/user-attachments/assets/5f2cf5f1-4598-46bb-a728-1d016e88c71c" />


### Install Build Tool 
```
sudo apt install -y curl git build-essential pkg-config libssl-dev clang lz4 jq htop
```

### Let's tune system limit for high performance Node
```
cat << 'EOF' | sudo tee /etc/security/limits.d/99-tempo.conf
*    soft    nofile    1048576
*    hard    nofile    1048576
EOF
```
<img width="3472" height="496" alt="image" src="https://github.com/user-attachments/assets/00aba8d9-5b37-4765-b885-9e71eb8e813d" />

### Reboot system  
```
reboot
```

### Create Working Directories 
```
mkdir -p $HOME/tempo-node/data
cd $HOME/tempo-node
```

Confirm with `df -h $HOME/tempo-node`
  * Make sure there is enough space >500gb preferably as Official docs say 100GB minimum, but actual snapshot is >200GB compressed
  * After extraction, you'll need 300GB+ actual space
  * For RPC nodes: 500GB+ NVMe SSD is preferred. 

---

## GETTING STARTED: Tempo is written in RUST, let's get started to Installation. 

### Install RUST 
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
```
<img width="3828" height="696" alt="image" src="https://github.com/user-attachments/assets/87283c5e-0637-41f9-80fb-fa428b35be8e" />

### Reload SHELL: 
```
source "$HOME/.cargo/env"
```

### Verify RUST installation
```
rustc --version && cargo --version
```
<img width="1992" height="304" alt="image" src="https://github.com/user-attachments/assets/afea0bde-a2cd-4477-b959-33eb6f8d6eea" />


---
There are differnet ways of which we can install Tempo and running the Node, 
  * Pre-build Binary
  * Build from Source
  * Docker
### In this guide, I will be using `Pre-Build` 


## Install Docker 

```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER

sudo systemctl enable docker
sudo systemctl start docker
```

### Log out and back in for docker group to take effect.
```
reboot
```

### Verify Docker works:
```
docker --version
docker run hello-world
```

### Install UFW
```
apt update
apt install -y ufw
```

### Allow SSH 
```
ufw allow ssh
```
<img width="3700" height="720" alt="image" src="https://github.com/user-attachments/assets/73b25596-6de3-4167-8ced-c3dad9a946d0" />


### Allow Tempo P2P ports
Tempo (Reth-based) needs both TCP and UDP.
```
ufw allow 30303/tcp
ufw allow 30303/udp
```

### Enable UFW 
Reply with `y` and proceed 
```
ufw enable
```


### CD into Directory
```
cd $HOME/tempo-node
```

## Pre-built Binary 
```
curl -L https://tempo.xyz/install | bash
tempo --version
```

<img width="3748" height="2256" alt="image" src="https://github.com/user-attachments/assets/a4170011-2534-49ad-bfad-78b4b0d81e58" />


### Reload Source
```
source /root/.bashrc
```
---
### Install and Open new screen 
```
apt update
apt install -y screen
screen -S tempo
```
- Detach screen:    Ctrl + A, then D
- List screens:     screen -ls
- Reattach screen:  screen -r tempo
- Force reattach:   screen -D -r tempo
- Kill screen:      exit   (from inside the screen)

---
### Start Tempo Node: 
```
tempo node \
  --follow \
  --http --http.port 8545 \
  --http.api eth,net,web3,txpool,trace
```

<img width="3760" height="2216" alt="image" src="https://github.com/user-attachments/assets/1dd8a5c4-c4e7-429a-a9ca-f6ca913ce9ab" />


** You should see sync logs and peer connections. Press Ctrl+C to stop once confirmed working. ** 


## Production Setup (Systemd)
### Create a [service file](https://docs.tempo.xyz/guide/node/rpc#example-systemd-service) so the node runs in background and auto-restarts:

```
sudo tee /etc/systemd/system/tempo-rpc.service > /dev/null << EOF
[Unit]
Description=Tempo RPC Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=$USER
WorkingDirectory=$HOME/tempo-node
Environment=RUST_LOG=info
ExecStart=/usr/local/bin/tempo node \\
    --datadir $HOME/tempo-node/data \\
    --port 30303 \\
    --discovery.addr 0.0.0.0 \\
    --discovery.port 30303 \\
    --http \\
    --http.addr 0.0.0.0 \\
    --http.port 8545 \\
    --http.api eth,net,web3,txpool,trace \\
    --ws \\
    --ws.addr 127.0.0.1 \\
    --ws.port 8546 \\
    --metrics 9000
Restart=always
RestartSec=5
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
EOF
```

### Activate it:
```
sudo systemctl daemon-reload
sudo systemctl enable tempo-rpc
sudo systemctl start tempo-rpc
```

### Check if running

```
sudo systemctl status tempo-rpc
```
<img width="3784" height="1044" alt="image" src="https://github.com/user-attachments/assets/2b136e24-dd36-4e87-a5ea-78e7ff240cb7" />


---

### Let's grab Snapshot 
Syncing from block zero takes forever. Instead, we wi;; grab from official snapshot 

```
cd $HOME/tempo-node
```

### Check [Tempo Snapshot](https://snapshots.tempoxyz.dev/) for the latest URL

The one I marked in below screenshot is the latest URL (old now) 
```
SNAP_URL="<paste_snapshot_url_here>"
curl -L "$SNAP_URL" | lz4 -d | tar -xvf - -C $HOME/tempo-node/data
```
<img width="4020" height="1914" alt="image" src="https://github.com/user-attachments/assets/9de1533d-3d3c-40ab-afb0-25a58744bfea" />

---
## What I did here,  I copied the latest snapshot link form above link/image and executed 
  ```
mkdir data
SNAP_URL="https://tempo-node-snapshots.tempoxyz.dev/tempo-42429-8380104-1767416422.tar.lz4"
curl -L "$SNAP_URL" | lz4 -d | tar -xvf - -C $HOME/tempo-node/data
  ```

** This Download is ~190gb currently, and extracting it also takes significant amount of space ** 

<img width="3752" height="1868" alt="image" src="https://github.com/user-attachments/assets/5ffe7cfd-eaf9-4ce3-a05b-01ae83f1d991" />


### After extraction, verify size:
```
du -sh $HOME/tempo-node/data
```
Should be 300GB+.



### Clone the Repo: 
```
cd $HOME/tempo-node
git clone https://github.com/tempoxyz/tempo.git
cd tempo
cargo build --release --bin tempo
```
<img width="2854" height="1062" alt="image" src="https://github.com/user-attachments/assets/17f60a56-a804-4a7a-8c33-4aa30e692b5d" />


### Move to system path
```
sudo cp target/release/tempo /usr/local/bin/
sudo chmod +x /usr/local/bin/tempo
```


# Confirm it works
You should see the below screenshot. 
```
tempo
```
<img width="2342" height="842" alt="image" src="https://github.com/user-attachments/assets/4d88fcbe-967a-486c-a112-e9e41d7f3886" />


---

### After extracting, verify file 
As the blocks keep increasing, so as file size increases 
```
du -sh $HOME/tempo-node/data
```

## Running RPC Node 
### RPC nodes don't need special permission. They sync the chain and serve API requests. 

```
tempo node \
    --datadir $HOME/tempo-node/data \
    --follow \
    --http \
    --http.addr 127.0.0.1 \
    --http.port 8545 \
    --http.api eth,net,web3,txpool,trace
```

---

## 📚 Documentation

Official Tempo node guide:  
https://docs.tempo.xyz/guide/node


##  Twitter
Follow updates and node operation posts on Twitter:  
https://twitter.com/mztacat

---

