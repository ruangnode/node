## Snapshot
```
sudo systemctl stop atomoned
cp $HOME/.atomone/data/priv_validator_state.json $HOME/.atomone/priv_validator_state.json.backup
rm -rf $HOME/.atomone/data
curl https://server-1.ruangnode.com/snap-mainnet/atomone/atomone-2025-02-07-1693160.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.atomone
mv $HOME/.atomone/priv_validator_state.json.backup $HOME/.atomone/data/priv_validator_state.json
sudo systemctl restart atomoned && sudo journalctl -u atomoned -f
```
