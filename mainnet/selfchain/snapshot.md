# Snapshot Guide

## Snapshot
To restore a snapshot, follow the steps below:

```bash
sudo systemctl stop selfchaind
cp $HOME/.selfchain/data/priv_validator_state.json $HOME/.selfchain/priv_validator_state.json.backup
rm -rf $HOME/.selfchain/data
curl https://server-1.ruangnode.com/snap-mainnet/selfchain/selfchain-2025-02-07-1693160.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.selfchain
mv $HOME/.selfchain/priv_validator_state.json.backup $HOME/.selfchain/data/priv_validator_state.json
sudo systemctl restart selfchaind && sudo journalctl -u selfchaind -f
```

