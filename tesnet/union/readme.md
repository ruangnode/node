#Setup Validator Env
```
echo "export NODENAME=<Your_Nodename_Moniker>" >> $HOME/.bash_profile
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export CHAIN_ID=union-testnet-9" >> $HOME/.bash_profile
echo "export GO_VERSION=1.21.4" >> $HOME/.bash_profile
echo "export BINARY_NAME=uniond" >> $HOME/.bash_profile
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
  wget "https://golang.org/dl/go1.21.4.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go1.21.4.linux-amd64.tar.gz"
  rm "go1.21.4.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
cd $HOME
wget https://github.com/unionlabs/union/releases/download/uniond%2Fv0.25.0/uniond.x86_64-linux.tar.gz -O uniond-linux-amd64
chmod +x uniond-linux-amd64
mv uniond-linux-amd64 $HOME/go/bin/uniond
```

## Init app
```
alias uniond='uniond --home=$HOME/.union/'
uniond init <Your_Nodename_Moniker> --chain-id union-testnet-9
uniond config set client keyring-backend test
uniond config set client node tcp://localhost:26657
```

## Download configuration
```
cd $HOME
wget -O $HOME/.union/config/genesis.json https://server-1.ruangnode.com/testnet/union/genesis.json
wget -O $HOME/.union/config/addrbook.json https://server-1.ruangnode.com/testnet/union/addrbook.json
```

## Disable indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.union/config/config.toml
```

## Config pruning
```
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.union/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.union/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "19"|g' $HOME/.union/config/app.toml
```

## Set minimum gas price
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0muno"|g' $HOME/.union/config/app.toml
```

## Create service
```
sudo tee /etc/systemd/system/uniond.service > /dev/null <<EOF2
[Unit]
Description=union Node
After=network-online.target

[Service]
User=USER
ExecStart=$(which uniond) start --home $HOME/.union
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
sudo systemctl enable uniond
sudo systemctl restart uniond && sudo journalctl -u uniond -f -o cat
```
