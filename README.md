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


### Reboot system [reboot system and wait a few minutes] 
```
reboot
```

### Let's tune system limit for high performance Node
```
cat << 'EOF' | sudo tee /etc/security/limits.d/99-tempo.conf
*    soft    nofile    1048576
*    hard    nofile    1048576
EOF
```
<img width="3472" height="496" alt="image" src="https://github.com/user-attachments/assets/00aba8d9-5b37-4765-b885-9e71eb8e813d" />

### Reboot system aggain 
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
  * Binary
### In this guide, I will be `Building Tempo from Source` 





