# Sao Testnet guide

![Sao](https://user-images.githubusercontent.com/44331529/223452452-434790b5-0079-4402-b6f5-9ddc878ac826.png)

[WebSite](https://www.sao.network/#/)\
[GitHub]( https://github.com/SaoNetwork)
=
[EXPLORER](https://explorer.stavr.tech/sao-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O sao https://raw.githubusercontent.com/obajay/nodes-Guides/main/Sao/sao && chmod +x sao && ./sao
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19.4
```python
ver="1.19.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 04.03.23
```python
cd $HOME
git clone https://github.com/SaoNetwork/sao-consensus.git
cd sao-consensus
git checkout testnet0
make install
```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`saod version --long`
- version: 0.0.9-28-g284ebe6
- commit: 284ebe63db9256dc83f745f861809859abec995e

```python
saod init STAVRguide --chain-id sao-testnet0
saod config chain-id sao-testnet0
```    

## Create/recover wallet
```python
saod keys add <walletname>
  OR
saod keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.sao/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Sao/genesis.json"

```
`sha256sum $HOME/.sao/config/genesis.json`
+ fbd400351e29ca405906937ee343f0be099903d506d7ae06249c23d8922d6794

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0sao\"/;" ~/.sao/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sao/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sao/config/config.toml
peers="557c49ddc6c98e5c5ac6030a93451ad5fcd54e34@164.90.147.133:20656,4a4c330115ed36bf8a5c8ffbc568d165ee91bd72@207.154.243.48:20656,e5d450435d3041e57400edb1cb65845f33be5b13@167.71.53.182:26656,244c464e3d500ee3f242fa3a10ae50d4cd02fc26@164.90.221.101:26656,a22a3ad8f847ab87bd64d0b9365b870750bde4e5@143.198.204.248:20656,395e1f7e7ea858fd9093de8832a25be67e7b6d9d@171.226.79.93:09656,a5298771c624a376fdb83c48cc6c630e58092c62@192.18.136.151:26656,59cef823c1a426f15eb9e688287cd1bc2b6ea42d@152.70.126.187:26656,18bd77a58bea85dce428e2bc0cd1eed461947d1c@89.163.215.6:55656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sao/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sao/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.sao/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.sao/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sao/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sao/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.sao/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Sao/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/saod.service > /dev/null <<EOF
[Unit]
Description=sao
After=network-online.target

[Service]
User=$USER
ExecStart=$(which saod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Sao Testnet
```python
SNAP_RPC=http://sao.rpc.t.stavr.tech:1077
peers="006e207a3f235a28bc0815001b76ee385ee4bda3@sao.peers.stavr.tech:1076"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sao/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.sao/config/config.toml
saod tendermint unsafe-reset-all --home /root/.sao --keep-addr-book
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.sao/config/app.toml
sudo systemctl restart saod && journalctl -u saod -f -o cat
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop saod
cp $HOME/.sao/data/priv_validator_state.json $HOME/.sao/priv_validator_state.json.backup
rm -rf $HOME/.sao/data
curl -o - -L http://sao.snapshot.stavr.tech:1025/sao/sao-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.sao --strip-components 2
mv $HOME/.sao/priv_validator_state.json.backup $HOME/.sao/data/priv_validator_state.json
wget -O $HOME/.sao/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Sao/addrbook.json"
sudo systemctl restart saod && journalctl -u saod -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable saod
sudo systemctl restart saod && sudo journalctl -u saod -f -o cat
```

### Create validator
```python
saod tx staking create-validator \
  --amount=1000000sao \
  --pubkey=$(saod tendermint show-validator) \
  --moniker="STAVRguide" \
  --details="" \
  --identity="" \
  --website="" \
  --chain-id="sao-testnet0" \
  --commission-rate="0.10" \
  --commission-max-rate="0.50" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=wallet \
  --node "tcp://localhost:1077" -y
```

## Delete node
```python
sudo systemctl stop saod && \
sudo systemctl disable saod && \
rm /etc/systemd/system/saod.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf sao-consensus && \
rm -rf .sao && \
rm -rf $(which saod)
```
#
### Sync Info
```python
saod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
saod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u saod -f -o cat
```
### Check Balance
```python
saod query bank balances saod...addressjkl1yjgn7z09ua9vms259j
```
