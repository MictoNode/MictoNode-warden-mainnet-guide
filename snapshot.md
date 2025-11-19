# ♻️ Snapshot

> **Info**  
> Updated every 12 hours.  
> Pruning: custom | 100/0/10 | indexer: null

Check snapshot height
```bash
echo "Warden Mainnet Snapshot Height: $(curl -s https://files.mictonode.com/warden-mainnet/snapshot/block-height.txt)"
```

```bash
sudo systemctl stop wardend
cp $HOME/.warden/data/priv_validator_state.json $HOME/.warden/priv_validator_state.json.backup
rm -rf $HOME/.warden/data

SNAPSHOT_URL="https://files.mictonode.com/warden-mainnet/snapshot/"
LATEST_SNAPSHOT=$(curl -s $SNAPSHOT_URL | grep -oP 'warden-mainnet_\d+\.tar\.lz4' | sort -t_ -k2 -n | tail -n 1)

if [ -n "$LATEST_SNAPSHOT" ]; then
  FULL_URL="${SNAPSHOT_URL}${LATEST_SNAPSHOT}"
  if curl -s --head "$FULL_URL" | head -n 1 | grep "200" > /dev/null; then
    curl "$FULL_URL" | lz4 -dc - | tar -xf - -C $HOME/.warden
    
    mv $HOME/.warden/priv_validator_state.json.backup $HOME/.warden/data/priv_validator_state.json
    
    sudo systemctl restart wardend && sudo journalctl -fu wardend -o cat
  else
    echo "Snapshot URL is not accessible"
  fi
else
  echo "No snapshot found"
fi
```