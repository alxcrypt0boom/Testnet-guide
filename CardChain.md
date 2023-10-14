# Cardchain
## Packages
```bash
sudo apt update && sudo apt upgrade -y
```
## Dependencies
```bash
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```
## Go
```bash
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```
## Moniker, whrite some name
```bash
NODENAME=Cosmo-gem
```
## Make your custom ports. You can chose from 10 to 65
```bash
CARD_PORT=29
```
## Save and import variables

```bash
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export CARD_CHAIN_ID=cardtestnet-4" >> $HOME/.bash_profile
echo "export CARD_PORT=${CARD_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Binaries

```bash
cd $HOME
git clone https://github.com/DecentralCardGame/Testnet
wget https://github.com/DecentralCardGame/Cardchain/releases/download/v0.9.0/Cardchaind
chmod +x Cardchaind
sudo mv Cardchaind /usr/local/bin/Cardchaind
```
## Config app

```bash
Cardchaind config chain-id $CARD_CHAIN_ID
Cardchaind config node tcp://localhost:${CARD_PORT}657
```
## For your own risk 
```bash
Cardchaind config keyring-backend file
```

## Init app

```bash
Cardchaind init $NODENAME --chain-id $CARD_CHAIN_ID
```
## Seeds and peers

```bash
SEEDS=""
PEERS=""; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.Cardchain/config/config.toml
```
## Genesis

```bash
wget http://45.136.28.158:3000/genesis.json -O $HOME/.Cardchain/config/genesis.json
```

## Custom ports 

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CARD_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CARD_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CARD_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CARD_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CARD_PORT}660\"%" $HOME/.Cardchain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CARD_PORT}317\"%; s%^address = \":8080\"%address = \":${CARD_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CARD_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CARD_PORT}091\"%" $HOME/.Cardchain/config/app.toml
```
## Pruning

```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain/config/app.toml
```
## Minimum gas price

```bash
sed -i 's/minimum-gas-prices = "0ubpf"/minimum-gas-prices = "0.0001ubpf"/g' $HOME/.Cardchain/config/app.toml
```

## Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.Cardchain/config/config.toml
```
## Reset chain data

```bash
Cardchaind tendermint unsafe-reset-all --home $HOME/.Cardchain 
```
## Addrbook
```bash
wget -O $HOME/.Cardchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crowd_Control/addrbook.json"
```

## Snap

```
SNAP_RPC="http://lxgr.xyz:26657"
```
```
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.Cardchain/config/config.toml
```

# Service

```bash
sudo tee /etc/systemd/system/Cardchaind.service > /dev/null <<EOF
[Unit]
Description=Cardchaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which Cardchaind) start --home $HOME/.Cardchain
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
## Register and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable Cardchaind
sudo systemctl restart Cardchaind && sudo journalctl -u Cardchaind -f -o cat
```
