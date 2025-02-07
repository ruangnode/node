### How To Install Full Node Atomone Mainnet

## Setting up vars
Your Nodename (validator) that will shows in explorer
```
NODENAME=<Your_Nodename_Moniker>
```

Save variables to system
```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export ATOMONE_CHAIN_ID=atomone-1" >> $HOME/.bash_profile
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

## Install go
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.21.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
cd $HOME
git clone https://github.com/atomone-hub/atomone.git
cd atomone
git checkout v1.0.1
make install
```

## Init app
```
atomoned init $NODENAME --chain-id $ATOMONE_CHAIN_ID
```

### Download configuration
```
wget https://server-1.ruangnode.com/snap-mainnet/atomone/genesis.json -O $HOME/.atomone/config/genesis.json
wget https://server-1.ruangnode.com/snap-mainnet/atomone/addrbook.json -O $HOME/.atomone/config/addrbook.json
```

## Set the minimum gas price and Peers, Filter peers/ MaxPeers
```
SEEDS=""
PEERS="ed0e36c57122184ab05b6c635b2f2adf592bfa0c@atomone-mainnet-peer.itrocket.net:61657,5d913650738a081aa02631a7f108dc7812330f0b@37.27.129.24:13656,706a835221dcc171afa14429fac536d6b5a3736d@63.250.54.71:62656,4ef48d2cc03b332f9a711fc65dc0453839f9040d@8.52.153.92:61656,752bb5f1c914c5294e0844ddc908548115c1052c@65.108.236.5:14556,6c4b686add2ae26aad617a15e4db012e7496eee1@154.91.1.108:62656,d3adcf9eee8665ee2d3108f721b3613cdd18c3a3@23.227.223.49:62656,8391dab9a9ece4e3f80e06512bdd1a84af5f257f@95.217.36.103:14556,61b7861a468dfa84532526afd98bea81bf41a874@121.78.247.244:16656,42c384bdf78ea2a2e7fc0c4e1716ef94951fca16@95.214.52.233:36656,37201c92625df2814a55129f73f10ab6aa2edc35@185.16.39.137:27396"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.atomone/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0001uatone\"/;" ~/.atomone/config/app.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.atomone/config/config.toml
```

## Disable indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.atomone/config/config.toml
```

## Config pruning
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="19" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.atomone/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.atomone/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.atomone/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.atomone/config/app.toml
```

## Create service
```
sudo tee /etc/systemd/system/atomoned.service > /dev/null <<EOF
[Unit]
Description=atomone
After=network-online.target

[Service]
User=$USER
ExecStart=$(which atomoned) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable atomoned
sudo systemctl restart atomoned && sudo journalctl -u atomoned -f -o cat
```
