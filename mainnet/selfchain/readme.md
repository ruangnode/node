### How To Install Full Node Selfchain Mainnet

## Konfigurasi
```
BINARY_NAME="selfchaind"
CHAIN_ID="self-1"
DOWNLOAD_BINARY="https://github.com/hotcrosscom/Self-Chain-Releases/releases/download/mainnet-v1.0.1/selfchaind-linux-amd64"
GENESIS_URL="https://server-1.ruangnode.com/snap-mainnet/selfchain/genesis.json"
ADDRBOOK_URL="https://server-1.ruangnode.com/snap-mainnet/selfchain/addrbook.json"
MIN_GAS_PRICE="0.005uslf"
PEERS=""
SEEDS=""
NODENAME="<Your_Nodename_Moniker>"
WALLET="wallet"
GO_VERSION="1.22.4"
SELFCHAIN_SERVICE="/etc/systemd/system/selfchaind.service"
```

## Setting up vars
Your Nodename (validator) that will show in explorer:
```
NODENAME=<Your_Nodename_Moniker>
```

Save variables to system:
```
echo "export NODENAME=<Your_Nodename_Moniker>" >> $HOME/.bash_profile
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export CHAIN_ID=self-1" >> $HOME/.bash_profile
echo "export GO_VERSION=1.22.4" >> $HOME/.bash_profile
echo "export BINARY_NAME=selfchaind" >> $HOME/.bash_profile
echo "export SELFCHAIN_SERVICE=/etc/systemd/system/selfchaind.service" >> $HOME/.bash_profile
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
  wget "https://golang.org/dl/go1.22.4.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go1.22.4.linux-amd64.tar.gz"
  rm "go1.22.4.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
cd $HOME
wget https://github.com/hotcrosscom/Self-Chain-Releases/releases/download/mainnet-v1.0.1/selfchaind-linux-amd64 -O selfchaind-linux-amd64
chmod +x selfchaind-linux-amd64
mv selfchaind-linux-amd64 $HOME/go/bin/selfchaind
```

## Init app
```
selfchaind init <Your_Nodename_Moniker> --chain-id self-1
```

## Download configuration
```
cd $HOME
wget -O $HOME/.self-1/config/genesis.json https://server-1.ruangnode.com/snap-mainnet/selfchain/genesis.json
wget -O $HOME/.self-1/config/addrbook.json https://server-1.ruangnode.com/snap-mainnet/selfchain/addrbook.json
```

## Disable indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.self-1/config/config.toml
```

## Config pruning
```
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.self-1/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.self-1/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "19"|g' $HOME/.self-1/config/app.toml
```

## Set minimum gas price
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.005uslf"|g' $HOME/.self-1/config/app.toml
```

## Create service
```
sudo tee /etc/systemd/system/selfchaind.service > /dev/null <<EOF2
[Unit]
Description=Selfchain Node
After=network-online.target

[Service]
User=github
ExecStart=$(which selfchaind) start
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
sudo systemctl enable selfchaind
sudo systemctl restart selfchaind && sudo journalctl -u selfchaind -f -o cat
```
