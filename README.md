<div align="center">

# 🚀 **0G Storage Node Complete Setup Guide (Using `/mnt/data`)** 🚀

### *by Satyam Jha*

</div>

## ⚡ **Initial Setup Steps**

**Before You Begin:**

* Add the 0G Galileo Testnet: [Instructions here](https://docs.0g.ai/run-a-node/testnet-information)
* Request testnet tokens: [0G Faucet](https://faucet.0g.ai/)

---

## 🔧 **Install Required Dependencies**

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install -y curl iptables build-essential git wget lz4 jq make protobuf-compiler cmake gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip screen ufw
```

---

### **Install Rust**

```bash
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustc --version
```

---

### **Install Go**

```bash
wget https://go.dev/dl/go1.24.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.24.3.linux-amd64.tar.gz
rm go1.24.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

## 📂 **Clone & Build Storage Node in `/mnt/data`**

```bash
sudo mkdir -p /mnt/data/0g-storage-node
sudo chown -R $USER:$USER /mnt/data/0g-storage-node
git clone https://github.com/0glabs/0g-storage-node.git /mnt/data/0g-storage-node
cd /mnt/data/0g-storage-node
git checkout v1.1.0
git submodule update --init
cargo build --release
```

---

## ⚙️ **Configuration**

Create run & log directories:

```bash
mkdir -p /mnt/data/0g-storage-node/run/log
```

Download config file:

```bash
curl -o /mnt/data/0g-storage-node/run/config.toml \
https://raw.githubusercontent.com/Mayankgg01/0G-Storage-Node-Guide/main/config.toml
```

Edit config to add your **miner\_key** (without `0x`) and use absolute paths:

```bash
nano /mnt/data/0g-storage-node/run/config.toml
```

Change or add these lines:

```
data_dir   = "/mnt/data/0g-storage-node/run/data"
db_dir     = "/mnt/data/0g-storage-node/run/db"
log_dir    = "/mnt/data/0g-storage-node/run/log"
log_config = "/mnt/data/0g-storage-node/run/log4rs.yaml"
```

---

### **Create Logging Config**

```bash
tee /mnt/data/0g-storage-node/run/log4rs.yaml > /dev/null <<'YAML'
refresh_rate: 30 seconds
appenders:
  stdout:
    kind: console
  file:
    kind: file
    path: "/mnt/data/0g-storage-node/run/log/zgs.log"
root:
  level: info
  appenders: [stdout, file]
YAML
```

---

## 🛡️ **Systemd Service Setup**

```bash
sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
[Unit]
Description=ZGS Node
After=network.target

[Service]
User=root
WorkingDirectory=/mnt/data/0g-storage-node/run
ExecStart=/mnt/data/0g-storage-node/target/release/zgs_node --config /mnt/data/0g-storage-node/run/config.toml
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Reload & enable:

```
sudo systemctl daemon-reload
```
```
sudo systemctl enable zgs
```
```
sudo systemctl start zgs
```

---

## 🔎 **Monitoring & Logs**

Check status:

```bash
sudo systemctl status zgs
```

Follow logs:

```bash
tail -f /mnt/data/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)
```

Check sync and block status:

```bash
while true; do \
resp=$(curl -s -X POST http://localhost:5678 -H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"zgs_getStatus","params":[],"id":1}');
logSyncHeight=$(echo $resp | jq '.result.logSyncHeight');
connectedPeers=$(echo $resp | jq '.result.connectedPeers');
echo -e "logSyncHeight: \033[32m$logSyncHeight\033[0m, connectedPeers: \033[34m$connectedPeers\033[0m";
sleep 5; done
```

---

## 🛑 **If you want to Stop & Remove Node**

```bash
sudo systemctl stop zgs
sudo systemctl disable zgs
sudo rm /etc/systemd/system/zgs.service
sudo systemctl daemon-reload
sudo rm -rf /mnt/data/0g-storage-node
```

---

If you follow this version, your **entire node will run from `/mnt/data/0g-storage-node`**, and `/dev/vda1` won’t fill up.
Didn't setup /mnt/data ?? follow this [guide](https://github.com/satyamjhablockdev/DOmergeguide.git) and star 🌟 the repo.
