# Quicksilver  Testnet

![quick](https://user-images.githubusercontent.com/44331529/201520331-711f381d-89ab-4b8b-bab9-114c2b2521bd.png)


[WEBSITE](https://quicksilver.zone/)
=
[EXPLORER 1](https://explorer.stavr.tech/quicksilver/staking) \
[EXPLORER 2](https://testnet.manticore.team/quicksilver/staking)
=

# 1) Auto_install script
```python
wget -O quicktest https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quicksilver/Tetstnet/quicktest && chmod +x quicktest && ./quicktest
```

# 2) Manual installation

### Updating the repositories  
```python
sudo apt update && sudo apt upgrade -y
```
### Installing the necessary utilities
```python
udo apt install curl build-essential git wget jq make gcc tmux nvme-cli -y
```    
### GO 1.19 (one command)
```python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
### Node installation 07.02.23
```python
cd $HOME
wget https://github.com/ingenuity-build/quicksilver/releases/download/v1.4.0-rc4/quicksilverd-v1.4.0-rc4-amd64
chmod +x quicksilverd-v1.4.0-rc4-amd64
mv quicksilverd-v1.4.0-rc4-amd64 /root/go/bin/quicksilverd
```

*******🟢UPDATE🟢******* 07.02.23

```python
wget https://github.com/ingenuity-build/quicksilver/releases/download/v1.4.0-rc4/quicksilverd-v1.4.0-rc4-amd64
chmod +x quicksilverd-v1.4.0-rc4-amd64
mv quicksilverd-v1.4.0-rc4-amd64 /root/go/bin/quicksilverd
sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

`quicksilverd version --long`
+ version: v1.4.0-rc4
+ commit: 3c296c39b0a493c86a3c171e7c372f0ca23ab275

### Initialize the node
```java
quicksilverd config chain-id innuendo-5
quicksilverd init STAVRguide --chain-id innuendo-5
```

=
### Create wallet or restore
    quicksilverd keys add <name_wallet>
            or
    quicksilverd keys add <name_wallet> --recover

### Download Genesis
```python
wget -O $HOME/.quicksilverd/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quicksilver/Tetstnet/genesis.json"
```
`sha256sum ~/.quicksilverd/config/genesis.json`
 + fd19758d9b0b1c71e45599e52530c92442899fc4a4df650626e0659518833b65

### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0001uqck\"/;" ~/.quicksilverd/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.quicksilverd/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.quicksilverd/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.quicksilverd/config/config.toml
seeds="3f472746f46493309650e5a033076689996c8881@quicksilver-testnet.rpc.kjnodes.com:11659"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.quicksilverd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.quicksilverd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.quicksilverd/config/config.toml
```


### Setting up pruning with one command (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.quicksilverd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.quicksilverd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.quicksilverd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.quicksilverd/config/app.toml
```
### Disable indexing (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.quicksilverd/config/config.toml
```

### Download addrbook
```python
wget -O $HOME/.quicksilverd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quicksilver/addrbook.json"
```

### Create a service file
```python
sudo tee /etc/systemd/system/quicksilverd.service > /dev/null <<EOF
[Unit]
Description=quicksilver
After=network-online.target

[Service]
User=$USER
ExecStart=$(which quicksilverd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync 
```python
SNAP_RPC=http://quick.rpc.t.stavr.tech:21027
peers="1a178dec165fad14ab1b2fb6832dd092f6ab7a5b@quickt.peers.stavr.tech:21026"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.quicksilverd/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" ~/.quicksilverd/config/config.toml
quicksilverd tendermint unsafe-reset-all --home /root/.quicksilverd
systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

# SnapShot (~0.2GB) updated every 5 hours 
```python
cd $HOME
apt install lz4
sudo systemctl stop quicksilverd
cp $HOME/.quicksilverd/data/priv_validator_state.json $HOME/.quicksilverd/priv_validator_state.json.backup
rm -rf $HOME/.quicksilverd/data
curl -o - -L http://quickt.snapshot.stavr.tech:1016/quickt/quickt-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.quicksilverd --strip-components 2
mv $HOME/.quicksilverd/priv_validator_state.json.backup $HOME/.quicksilverd/data/priv_validator_state.json
wget -O $HOME/.quicksilverd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quicksilver/addrbook.json"
sudo systemctl restart quicksilverd && journalctl -u quicksilverd -f -o cat
```

```python    
sudo systemctl daemon-reload
sudo systemctl enable quicksilverd
sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

#### After synchronization, we go to the discord and we request coins

### Create a validator
```python
quicksilverd tx staking create-validator \
--chain-id innuendo-5 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.1 \
--min-self-delegation="1" \
--amount=1000000uqck \
--pubkey $(quicksilverd tendermint show-validator) \
--moniker "STAVRguide" \
--from=<name_wallet> \
--gas="auto" \
--fees 555uqck -y
```    

## Delete node
```python
sudo systemctl stop quicksilverd && \
sudo systemctl disable quicksilverd && \
rm /etc/systemd/system/quicksilverd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf quicksilver && \
rm -rf .quicksilverd && \
rm -rf $(which quicksilverd)
```

#

`Sync Info`
```python
quicksilverd status 2>&1 | jq .SyncInfo
```
`NodeINfo`
```python
quicksilverd status 2>&1 | jq .NodeInfo
```
`Check node logs`
```python
quicksilverd journalctl -u haqqd -f -o cat
```
`Check Balance`
```python
quicksilverd query bank balances quicksilver...addressdefund1yjgn7z09ua9vms259j
```
