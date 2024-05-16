

## Update Packages
```
sudo apt update && sudo apt upgrade -y
```

## Install Dependecies
```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## Install Golang
```
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```
## Build Binaries
```
cd $HOME
rm -rf initia/
git clone https://github.com/initia-labs/initia.git
cd initia
git checkout v0.2.11
make install
```

# Init app
```
initiad init (yourname)  --chain-id initiation-1
```

> yourname ( change to your name )

## Download Genesis
```
curl -Ls https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json > $HOME/.initia/config/genesis.json
```
## Settings Gas Fee
```
# setting minimum-gas-prices = "0.15uinit,0.01uusdc"
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.15uinit,0.01uusdc\"|" $HOME/.initia/config/app.toml
```

## Settings Peers 
```
SEEDS="cd69bcb00a6ecc1ba2b4a3465de4d4dd3e0a3db1@initia-testnet-seed.itrocket.net:51656"
PEERS="aee7083ab11910ba3f1b8126d1b3728f13f54943@initia-testnet-peer.itrocket.net:11656,7dc968e489e4b8e1b7754703947689b914204884@75.119.153.251:26656,8db26137b760df77c181b939100cdc5ec37c6879@84.46.242.223:15656,394ecb5ad08b02c5063b7a8b61046a5e1261792a@185.227.135.229:51656,50de06f67aaca8c01c7c15e3ddc04c287d327bae@158.220.87.67:22656,1d7d2d2cdb62df2a59aae536047d17f554e58bc3@154.38.181.13:656,1b0843bb3dce9c91115906305b698dc507bf138e@89.117.51.191:51656,ca6731bf7c3ba8c1d8545feb26e3a74adf889f16@38.242.134.10:24656,d25922f0a64e2cbce813d321ac007543e4741f94@137.184.113.189:26656,5a8e5f65179acb3d759099faccfe7752ca4ba536@178.18.248.75:26656,49da32b984143181ae5cae6564aba3a150624d7d@194.180.176.225:26656"
sed -i -e "s/^seeds =./seeds = "$SEEDS"/; s/^persistent_peers =./persistent_peers = "$PEERS"/" $HOME/.initia/config/config.toml
```

## Create Service
```
tee /etc/systemd/system/initiad.service > /dev/null << EOF
[Unit]
Description=Initiad Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which initiad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
## Register Service
```
sudo systemctl daemon-reload && \
sudo systemctl enable initiad && \
sudo systemctl start initiad && sudo journalctl -fu initiad -o cat
```
## TUNGGU SAMPAI SYNCED
-------------------------------------------------------------------------------------------
## ADDITIONAL

### FOR USE SNAPSHOT, FOLLOW THIS INSTRUCTION

## Reset blockchain data

```
sudo systemctl stop initia
cp $HOME/.initia/data/priv_validator_state.json $HOME/.initia/priv_validator_state.json.backup
rm -rf $HOME/.initia/data
```
## Download snapshot
```
curl https://testnet-files.itrocket.net/initia/snap_initia.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.initia
```
## Backup state data
```
mv $HOME/.initia/priv_validator_state.json.backup $HOME/.initia/data/priv_validator_state.json
```
## Restart node
```
sudo systemctl restart initiad.service
```
# BUT IF YOU FIND ERROR LIKE THIS :
failed to initialize database: file missing
initiad.service: Main process exited, code=exited, status=1/FAILURE
initiad.service: Failed with result 'exit-code'.
# FOLLOW THE STEPS BELOW
```
sudo systemctl stop initiad
```
```
lz4 -d -c ./latest_snapshot.tar.lz4 | tar -xf - -C $HOME/.initia
```
## MOVE TO YOUR SCREEN AND PASTE THIS COMMAND
```
sudo systemctl restart initiad && sudo journalctl -u initiad -f -o cat
```

### Syncing blocks
```
initiad status 2>&1 | jq .sync_info
```

### Check logs
```
sudo journalctl -fu initiad -o cat
```

## We Will Give You 2 Option [ Create Wallet or Import Private Key ]

### to create a new wallet, use the following command. don’t forget to save the mnemonic
```
initiad keys add $WALLET
```

### to restore exexuting wallet, use the following command
```
initiad keys add $WALLET --recover
```

### to get current list your address 
```
initiad keys list
```

### Running Validators 
```
initiad tx mstaking create-validator \
--amount 1000000uinit \
--pubkey $(initiad tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id initiation-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.15uinit \
-y
```

### Delete Node
```
sudo systemctl stop initiad
sudo systemctl disable initiad
sudo rm -rf /etc/systemd/system/initiad.service
sudo rm $(which initiad)
sudo rm -rf $HOME/.initia
sed -i "/INITIA_/d" $HOME/.bash_profile
```

Source for snapshot: https://docs.nodex.one/networks/testnet/initia/snapshot
