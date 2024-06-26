

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

## Init app
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
PEERS="e3ac92ce5b790c76ce07c5fa3b257d83a517f2f6@178.18.251.146:30656,2692225700832eb9b46c7b3fc6e4dea2ec044a78@34.126.156.141:26656,2a574706e4a1eba0e5e46733c232849778faf93b@84.247.137.184:53456,40d3f977d97d3c02bd5835070cc139f289e774da@168.119.10.134:26313,1f6633bc18eb06b6c0cab97d72c585a6d7a207bc@65.109.59.22:25756,4a988797d8d8473888640b76d7d238b86ce84a2c@23.158.24.168:26656,e3679e68616b2cd66908c460d0371ac3ed7795aa@176.34.17.102:26656,d2a8a00cd5c4431deb899bc39a057b8d8695be9e@138.201.37.195:53456,329227cf8632240914511faa9b43050a34aa863e@43.131.13.84:26656,517c8e70f2a20b8a3179a30fe6eb3ad80c407c07@37.60.231.212:26656,07632ab562028c3394ee8e78823069bfc8de7b4c@37.27.52.25:19656,028999a1696b45863ff84df12ebf2aebc5d40c2d@37.27.48.77:26656,3c44f7dbb473fee6d6e5471f22fa8d8095bd3969@185.219.142.137:53456,8db320e665dbe123af20c4a5c667a17dc146f4d0@51.75.144.149:26656,c424044f3249e73c050a7b45eb6561b52d0db456@158.220.124.183:53456,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,edcc2c7098c42ee348e50ac2242ff897f51405e9@65.109.34.205:36656,140c332230ac19f118e5882deaf00906a1dba467@185.219.142.119:53456,4eb031b59bd0210481390eefc656c916d47e7872@37.60.248.151:53456,ff9dbc6bb53227ef94dc75ab1ddcaeb2404e1b0b@178.170.47.171:26656,ffb9874da3e0ead65ad62ac2b569122f085c0774@149.28.134.228:26656" && \

SEEDS="2eaa272622d1ba6796100ab39f58c75d458b9dbc@34.142.181.82:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25756" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.initia/config/config.toml
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
## Lanjut download SNAPSHOT, Ikuti step dibawah ini

## Stop the service and reset the data
```
sudo systemctl stop initiad.service
cp $HOME/.initia/data/priv_validator_state.json $HOME/.initia/priv_validator_state.json.backup
rm -rf $HOME/.initia/data
```
## Download latest snapshot
```
curl -L https://snapshots.kzvn.xyz/initia/initiation-1_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.initia
mv $HOME/.initia/priv_validator_state.json.backup $HOME/.initia/data/priv_validator_state.json
```
## Restart the service and check the log
```
sudo systemctl start initiad.service && sudo journalctl -u initiad.service -f --no-hostname -o cat
```
## Sambil menunggu SYNCED, Kalian buat / restore wallet initia dulu

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
## Setelah SYNCED, Jalankan validator (pastikan sudah mempunyai faucet)

### Running Validators 
```
initiad tx mstaking create-validator \
--amount 1000000uinit \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--pubkey $(initiad tendermint show-validator) \
--moniker "NAMA KALIAN" \
--identity "PGP KEYBASE" \
--website "LINK FB / TWITTER / IG" \
--details "BEBAS" \
--chain-id initiation-1 \
--gas auto --fees 80000uinit \
-y
```
-----------------------------------------------------------------------------------------------
# CATATAN

## Check Version
```
initiad version
```
## Check Wallet
```
initiad keys list
```
## Check Balance
```
initiad q bank balances $(initiad keys show $WALLET -a)
```
## Reset blockchain data
```
sudo systemctl stop initia
cp $HOME/.initia/data/priv_validator_state.json $HOME/.initia/priv_validator_state.json.backup
rm -rf $HOME/.initia/data
```
## Backup state data
```
mv $HOME/.initia/priv_validator_state.json.backup $HOME/.initia/data/priv_validator_state.json
```
## Restart node
```
sudo systemctl restart initiad.service
```
## Check blocks
```
initiad status 2>&1 | jq .sync_info
```
OR
```
initiad status | jq

local_height=$(initiad status | jq -r .sync_info.latest_block_height); network_height=$(curl -s https://rpc-initia-testnet.trusted-point.com/status | jq -r .result.sync_info.latest_block_height); blocks_left=$((network_height - local_height)); echo "Your node height: $local_height"; echo "Network height: $network_height"; echo "Blocks left: $blocks_left"
```
## Check logs
```
sudo journalctl -fu initiad -o cat
```
## Check Validator name / Moniker
```
initiad q mstaking validator $(initiad keys show $WALLET --bech val -a) --output json | jq .description.moniker
```
## Check Wallet Validator
```
initiad keys show $WALLET --bech val -a
```
## Edit Your Validator
```
initiad tx mstaking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id initiation-1 \
--commission-rate 0.05 \
--from $WALLET \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.15uinit \
-y
```
## Delete Node
```
cd $HOME
sudo systemctl stop initiad.service
sudo systemctl disable initiad.service
sudo rm /etc/systemd/system/initia.service
sudo systemctl daemon-reload
rm -f $(which initiad)
rm -rf $HOME/.initia
rm -rf $HOME/initia
```

Source for snapshot: https://docs.kzvn.xyz/cosmos/initia/snapshot
