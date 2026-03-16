
# ➡️ v1.0.0 Update

## BEFORE THE UPGRADE [BLOCK](https://explorer.mictonode.com/Warden-Mainnet/block/5418000)

### Preparing for the Upgrade

#### 1. Create Upgrade Directories and Download the Binary

```bash
cd $HOME
mkdir -p $HOME/.warden/cosmovisor/upgrades/v1.0.0/bin
wget https://github.com/warden-protocol/wardenprotocol/releases/download/v1.0.0/wardend-v1.0.0-linux-amd64 -O $HOME/.warden/cosmovisor/upgrades/v1.0.0/bin/wardend
chmod +x $HOME/.warden/cosmovisor/upgrades/v1.0.0/bin/wardend
cd $HOME
```

#### 2. Check Version

```bash
$HOME/.warden/cosmovisor/upgrades/v1.0.0/bin/wardend version
```

## AFTER THE UPGRADE [BLOCK](https://explorer.mictonode.com/Warden-Mainnet/block/5418000)

#### 1. Stop the Service and Download the Binary

```bash
sudo systemctl stop wardend
cd $HOME
mkdir -p $HOME/.warden/cosmovisor/upgrades/v1.0.0/bin
wget https://github.com/warden-protocol/wardenprotocol/releases/download/v1.0.0/wardend-v1.0.0-linux-amd64 -O $HOME/.warden/cosmovisor/upgrades/v1.0.0/bin/wardend
chmod +x $HOME/.warden/cosmovisor/upgrades/v1.0.0/bin/wardend
cd $HOME
```

#### 2. Activate the New Version

```bash
sudo ln -sfn $HOME/.warden/cosmovisor/upgrades/v1.0.0 $HOME/.warden/cosmovisor/current
sudo ln -sfn $HOME/.warden/cosmovisor/current/bin/wardend /usr/local/bin/wardend
```

#### 3. Restart and Monitor the Service

```bash
sudo systemctl restart wardend
sudo journalctl -fu wardend -o cat
```