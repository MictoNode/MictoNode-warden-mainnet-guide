# 🔌 Installation

## 1️⃣ Installation packages and dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
sudo apt install snapd -y
sudo snap install just --classic
```

### ➡️ Go Installation

```bash
cd $HOME
VER="1.25.0"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### ➡️ Set Vars

*You can replace 119 with anything you want. Please write 3 digits.*

```bash
echo "export W_PORT="119"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## 2️⃣ Install node

```bash
cd $HOME
mkdir -p $HOME/.warden/cosmovisor/upgrades/v0.7.2/bin
wget https://github.com/warden-protocol/wardenprotocol/releases/download/v0.7.3/wardend-v0.7.3-linux-amd64 -O $HOME/.warden/cosmovisor/upgrades/v0.7.2/bin/wardend
chmod +x $HOME/.warden/cosmovisor/upgrades/v0.7.2/bin/wardend
cd $HOME
```

```bash
sudo ln -sfn $HOME/.warden/cosmovisor/upgrades/v0.7.2 $HOME/.warden/cosmovisor/current
sudo ln -sfn $HOME/.warden/cosmovisor/current/bin/wardend /usr/local/bin/wardend
```

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```

### ➡️ Create a service

```bash
sudo tee /etc/systemd/system/wardend.service > /dev/null << EOF
[Unit]
Description=Warden Node Service MictoNode
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.warden"
Environment="DAEMON_NAME=wardend"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.warden/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable wardend
```

### ➡️ Initialize the node

```bash
wardend config set client chain-id warden_8765-1
wardend config set client keyring-backend test
wardend config set client node tcp://localhost:${W_PORT}57
wardend init "change-moniker" --chain-id warden_8765-1
```

### ➡️ Genesis

```bash
wget -O $HOME/.warden/config/genesis.json "https://files.mictonode.com/warden-mainnet/genesis/genesis.json"
```

### ➡️ Addrbook

```bash
wget -O $HOME/.warden/config/addrbook.json "https://files.mictonode.com/warden-mainnet/addrbook/addrbook.json"
```

### ➡️ Gas Settings

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "10award"|g' $HOME/.warden/config/app.toml
```

### ➡️ Port

```bash
sed -i.bak -e "s%:1317%:${W_PORT}17%g;
s%:8080%:${W_PORT}80%g;
s%:9090%:${W_PORT}90%g;
s%:9091%:${W_PORT}91%g;
s%:8545%:${W_PORT}45%g;
s%:8546%:${W_PORT}46%g;
s%:6065%:${W_PORT}65%g" $HOME/.warden/config/app.toml
```

```bash
sed -i.bak -e "s%:26658%:${W_PORT}58%g;
s%:26657%:${W_PORT}57%g;
s%:6060%:${W_PORT}60%g;
s%:26656%:${W_PORT}56%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${W_PORT}56\"%;
s%:26660%:${W_PORT}61%g" $HOME/.warden/config/config.toml
```

### ➡️ Peers and Seeds

```bash
SEEDS="7dbf2c58286b59aae1d9c121f1cee59fc21a59ef@54.220.127.230:26656,02810bc9ed25af587213a4ddb1fa4ab3a0e9978d@54.74.49.211:26656,e5ce023918478f61a3606e93b9642ca24e027328@63.33.179.20:26656,577bc08457d716f09def49b496952f3af4d60ba1@warden-mainnet-seed.itrocket.net:36656,27941ba20ad57cb665c7870d073a938e35e7d634@seed-warden.ibs.team:16663,96b4296e4d0ea9c7c569e8706886b1e40b990cd2@seed.warden.cros-nest.com:26656"
PEERS="bcd489ebc8a3e8d5b117c9e0080fe292b1394759@warden-mainnet-peer.itrocket.net:36656,7dbf2c58286b59aae1d9c121f1cee59fc21a59ef@54.220.127.230:26656,02810bc9ed25af587213a4ddb1fa4ab3a0e9978d@54.74.49.211:26656,e5ce023918478f61a3606e93b9642ca24e027328@63.33.179.20:26656,e8f50a175c1b50f556f794495692577ddd40e8a6@65.21.196.123:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.warden/config/config.toml
```

### ➡️ Pruning

```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.warden/config/config.toml
```

### ➡️ Other settings

```bash
cd $HOME/.warden/config
sed -i.bak 's|^\s*evm-chain-id\s*=.*|evm-chain-id = 8765|' app.toml
sed -i.bak 's|^\s*timeout_propose\s*=.*|timeout_propose = "1s"|' config.toml
sed -i.bak 's|^\s*timeout_propose_delta\s*=.*|timeout_propose_delta = "200ms"|' config.toml
sed -i.bak 's|^\s*timeout_prevote\s*=.*|timeout_prevote = "500ms"|' config.toml
sed -i.bak 's|^\s*timeout_prevote_delta\s*=.*|timeout_prevote_delta = "200ms"|' config.toml
sed -i.bak 's|^\s*timeout_precommit\s*=.*|timeout_precommit = "500ms"|' config.toml
sed -i.bak 's|^\s*timeout_precommit_delta\s*=.*|timeout_precommit_delta = "200ms"|' config.toml
sed -i.bak 's|^\s*timeout_commit\s*=.*|timeout_commit = "2s"|' config.toml
sed -i.bak 's|^\s*create_empty_blocks\s*=.*|create_empty_blocks = true|' config.toml
cd $HOME
```

### ➡️ Starter Snap

Check snapshot height
```bash
echo "Warden Mainnet Snapshot Height: $(curl -s https://files.mictonode.com/warden-mainnet/snapshot/block-height.txt)"
```

```bash
wardend comet unsafe-reset-all --home $HOME/.warden --keep-addr-book

SNAPSHOT_URL="https://files.mictonode.com/warden-mainnet/snapshot/"
LATEST_SNAPSHOT=$(curl -s $SNAPSHOT_URL | grep -oP 'warden-mainnet_\d+\.tar\.lz4' | sort -t_ -k2 -n | tail -n 1)

if [ -n "$LATEST_SNAPSHOT" ]; then
  FULL_URL="${SNAPSHOT_URL}${LATEST_SNAPSHOT}"
  if curl -s --head "$FULL_URL" | head -n 1 | grep "200" > /dev/null; then
    curl "$FULL_URL" | lz4 -dc - | tar -xf - -C $HOME/.warden
  else
    echo "Snapshot URL not accessible"
  fi
else
  echo "No snapshot found"
fi
```

### ➡️ Let's get started

```bash
sudo systemctl restart wardend
journalctl -fu wardend -o cat
```

### ➡️ Create wallet

*Don't forget change "wallet-name"*

```bash
wardend keys add wallet-name
```

### ➡️ Import wallet

```bash
wardend keys add wallet-name --recover
```

### ➡️ Create Validator

> **⚠️ Warning:** Save your pubkey

```bash
wardend comet show-validator
```

Create validator.json

```bash
nano /root/validator.json
```

```json
{
        "pubkey": "paste-your-pubkey",
        "amount": "1000000000000000000award",
        "moniker": "MictoNodeGuide",
        "identity": "optional identity signature (ex. UPort or Keybase)",
        "website": "validator's (optional) website",
        "security": "validator's (optional) security contact email",
        "details": "validator's (optional) details",
        "commission-rate": "0.1",
        "commission-max-rate": "0.2",
        "commission-max-change-rate": "0.01",
        "min-self-delegation": "1"
}
```

Send tx for create validator

```bash
wardend tx staking create-validator /root/validator.json \
    --from=wallet-name \
    --chain-id=warden_8765-1 \
    --gas auto --gas-adjustment 1.3 --fees 250000000000000award \
    -y
```

### ➡️ Delegation

```bash
wardend tx staking delegate $(wardend keys show wallet-name --bech val -a) amount000000000000000000award \
--chain-id warden_8765-1 \
--from "wallet-name" \
--gas auto --gas-adjustment 1.6 --fees 250000000000000award \
-y
```

### ➡️ Complete deletion

```bash
cd $HOME
sudo systemctl stop wardend
sudo systemctl disable wardend
sudo rm -rf /etc/systemd/system/wardend.service
sudo systemctl daemon-reload
sudo rm -f /usr/local/bin/wardend
sudo rm -f $(which wardend)
sudo rm -rf $HOME/.warden
sed -i "/W_PORT_/d" $HOME/.bash_profile
```

### ➡️ Block check

```bash
local_height=$(wardend status | jq -r .sync_info.latest_block_height); network_height=$(curl -s https://warden-mainnet-rpc.mictonode.com/status | jq -r .result.sync_info.latest_block_height); blocks_left=$((network_height - local_height)); echo "Your node height: $local_height"; echo "MictoNode height: $network_height"; echo "Blocks left: $blocks_left"
```

* **Your node height** - the current block of your node
* **MictoNode height** - the last block of the network
* **Blocks left** - how many blocks your node has left to sync