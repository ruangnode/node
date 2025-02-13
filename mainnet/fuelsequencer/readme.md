Save variables to system:
```
echo "export NODENAME=<Your_Nodename_Moniker>" >> $HOME/.bash_profile
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export CHAIN_ID=seq-mainnet-1" >> $HOME/.bash_profile
echo "export GO_VERSION=1.21.3" >> $HOME/.bash_profile
echo "export BINARY_NAME=fuelsequencerd" >> $HOME/.bash_profile
echo "export SELFCHAIN_SERVICE=/etc/systemd/system/fuelsequencerd.service" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux net-tools ccze -y
```

## Install Go
```
if ! command -v go &> /dev/null; then
  cd $HOME
  wget "https://golang.org/dl/go1.21.3.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go1.21.3.linux-amd64.tar.gz"
  rm "go1.21.3.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
cd $HOME
wget https://github.com/FuelLabs/fuel-sequencer-deployments/releases/download/seq-mainnet-1.2-improved-sidecar/fuelsequencerd-seq-mainnet-1.2-improved-sidecar-linux-amd64 -O fuelsequencerd-linux-amd64
chmod +x fuelsequencerd-linux-amd64
mv fuelsequencerd-linux-amd64 $HOME/go/bin/fuelsequencerd
```

## Init app
```
fuelsequencerd init <Your_Nodename_Moniker> --chain-id seq-mainnet-1
```

## Download configuration
```
cd $HOME
wget -O $HOME/.seq-mainnet-1/config/genesis.json https://server-1.ruangnode.com/snap-mainnet/selfchain/genesis.json
wget -O $HOME/.seq-mainnet-1/config/addrbook.json https://server-1.ruangnode.com/snap-mainnet/selfchain/addrbook.json
```

## Disable indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.seq-mainnet-1/config/config.toml
```

## Config pruning
```
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.seq-mainnet-1/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.seq-mainnet-1/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "19"|g' $HOME/.seq-mainnet-1/config/app.toml
```

## Set minimum gas price
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "10fuel"|g' $HOME/.seq-mainnet-1/config/app.toml
```

## Create service
```
sudo tee /etc/systemd/system/fuelsequencerd.service > /dev/null <<EOF2
[Unit]
Description=Selfchain Node
After=network-online.target

[Service]
User=github
ExecStart=$(which fuelsequencerd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF2
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable fuelsequencerd
sudo systemctl restart fuelsequencerd && sudo journalctl -u fuelsequencerd -f -o cat
```
