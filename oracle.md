# ⛓️ Oracle

> **⚠️ DANGER:** This installation is exclusive to those who installed it from me. Those who install it elsewhere may get errors.

### ➡️ Install Oracle

```bash
cd $HOME
rm -rf slinky
git clone https://github.com/skip-mev/slinky.git
cd slinky
git checkout v1.0.12
make build
mv build/slinky /usr/local/bin/
cd
```

or

```bash
cd $HOME
rm -rf connect
git clone https://github.com/warden-protocol/connect
cd connect
git checkout v1.3.0
make build
mv build/slinky /usr/local/bin/
cd
```

### ➡️ Create a service

```bash
echo "export W_PORT="119"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```bash
sudo tee /etc/systemd/system/slinkyd.service > /dev/null <<EOF
[Unit]
Description=Warden Slinky Oracle
After=network-online.target

[Service]
User=$USER
ExecStart=$(which slinky) --market-map-endpoint 127.0.0.1:${W_PORT}90 --port ${W_PORT}80 --metrics-prometheus-address 127.0.0.1:${W_PORT}02
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### ➡️ Let's start

```bash
sudo systemctl daemon-reload
sudo systemctl enable slinkyd
sudo systemctl start slinkyd
```

### ➡️ Logs

```bash
journalctl -fu slinkyd --no-hostname
```

### ➡️ Warden oracle settings

```bash
nano $HOME/.warden/config/app.toml
```

> **⚠️ DANGER:** At the bottom of app.toml, your oracle settings should be as shown below.

```toml
###############################################################################
###                                  Oracle                                 ###
###############################################################################
[oracle]
# Enabled indicates whether the oracle is enabled.
enabled = "true"

# Oracle Address is the URL of the out of process oracle sidecar. This is used to
# connect to the oracle sidecar when the application boots up. Note that the address
# can be modified at any point, but will only take effect after the application is
# restarted. This can be the address of an oracle container running on the same
# machine or a remote machine.
oracle_address = "127.0.0.1:11980"

# Client Timeout is the time that the client is willing to wait for responses from 
# the oracle before timing out. The recommended timeout is 3 seconds (3000ms).
client_timeout = "2s"

# MetricsEnabled determines whether oracle metrics are enabled. Specifically
# this enables instrumentation of the oracle client and the interaction between
# the oracle and the app.
metrics_enabled = "true"

# PriceTTL is the maximum age of the latest price response before it is considered stale. 
# The recommended max age is 10 seconds (10s). If this is greater than 1 minute (1m), the app
# will not start.
price_ttl = "0s"

# Interval is the time between each price update request. The recommended interval
# is the block time of the chain. Otherwise, 1.5 seconds (1500ms) is a good default. If this
# is greater than 1 minute (1m), the app will not start.
interval = "0s"
```

```bash
sudo systemctl daemon-reload && sudo systemctl restart wardend && sudo systemctl restart slinkyd
```

```bash
sudo journalctl -u wardend -f -o cat
```

```bash
sudo journalctl -u slinkyd -f -o cat
```

### ➡️ Check

```bash
curl localhost:${W_PORT}80/slinky/oracle/v1/prices | jq
```