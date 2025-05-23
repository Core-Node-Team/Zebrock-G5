




<h1 align="center"> Zenrock


![image](https://github.com/user-attachments/assets/6874e9ae-034d-4d9b-b93b-aab60756234e)


</h1>


 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>


### Explorer And Public Rpc Api

https://explorer.corenodehq.xyz/Zenrock-Testnet/staking

https://zenrock-g5-api.corenodehq.xyz

https://zenrock-g5-rpc.corenodehq.xyz

## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	8|
| RAM	| 16+ GB |
| Storage	| 400 GB SSD |



### 🚧Gerekli kurulumlar
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip gcc clang cmake build-essential -y
```

### 🚧 Go kurulumu
```
cd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### 🚧Dosyaları çekelim
```
echo "export ZENROCK_CHAIN_ID="gardia-5"" >> $HOME/.bash_profile
echo "export ZENROCK_PORT="59"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
cd $HOME
wget -O zenrockd.zip https://github.com/Zenrock-Foundation/zrchain/releases/download/v6.3.3/zenrockd.zip
unzip zenrockd.zip
rm zenrockd.zip
chmod +x $HOME/zenrockd
```
```
mkdir -p $HOME/.zrchain/cosmovisor/genesis/bin
cp -r $HOME/zenrockd $HOME/.zrchain/cosmovisor/genesis/bin/zenrockd
```
```
mkdir -p $HOME/.zrchain/cosmovisor/upgrades/v6rev1/bin
sudo mv $HOME/zenrockd $HOME/.zrchain/cosmovisor/upgrades/v6rev1/bin/zenrockd
```
### 🚧System link
```
sudo ln -s $HOME/.zrchain/cosmovisor/genesis $HOME/.zrchain/cosmovisor/current -f
sudo ln -s $HOME/.zrchain/cosmovisor/current/bin/zenrockd /usr/local/bin/zenrockd -f
```
### 🚧Cosmovisor indirelim
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### 🚧Servis oluşturalım
```
sudo tee /etc/systemd/system/zenrockd.service > /dev/null << EOF
[Unit]
Description=zenrock node service
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor run start --home $HOME/.zrchain
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=/root/.zrchain"
Environment="DAEMON_NAME=zenrockd"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/root/.zrchain/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target

EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable zenrockd.service
```
### 🚧İnit
NOT: node adınızı yazınız.
```
zenrockd init $MONIKER --chain-id $ZENROCK_CHAIN_ID
zenrockd config set client chain-id $ZENROCK_CHAIN_ID
zenrockd config set client node tcp://localhost:${ZENROCK_PORT}657
```
### 🚧Genesis addrbook 
```
wget -O $HOME/.zrchain/config/genesis.json https://raw.githubusercontent.com/Core-Node-Team/Zebrock-G5/refs/heads/main/genesis.json
wget -O $HOME/.zrchain/config/addrbook.json https://raw.githubusercontent.com/Core-Node-Team/Zebrock-G5/refs/heads/main/addrbook.json
```

### 🚧Gas pruning ayarı
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.zrchain/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.zrchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.zrchain/config/app.toml

sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0urock"|g' $HOME/.zrchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.zrchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.zrchain/config/config.toml
```

### 🚧Port Ayarları

```
sed -i.bak -e "s%:1317%:${ZENROCK_PORT}317%g;
s%:8080%:${ZENROCK_PORT}080%g;
s%:9090%:${ZENROCK_PORT}090%g;
s%:9091%:${ZENROCK_PORT}091%g;
s%:8545%:${ZENROCK_PORT}545%g;
s%:8546%:${ZENROCK_PORT}546%g;
s%:6065%:${ZENROCK_PORT}065%g" $HOME/.zrchain/config/app.toml

```
```
sed -i.bak -e "s%:26658%:${ZENROCK_PORT}658%g;
s%:26657%:${ZENROCK_PORT}657%g;
s%:6060%:${ZENROCK_PORT}060%g;
s%:26656%:${ZENROCK_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ZENROCK_PORT}656\"%;
s%:26660%:${ZENROCK_PORT}660%g" $HOME/.zrchain/config/config.toml
```
### 🚧Seed
```
SEEDS=""
PEERS="6ef43e8d5be8d0499b6c57eb15d3dd6dee809c1e@sentry-1.gardia.zenrocklabs.io:26656,1dfbd854bab6ca95be652e8db078ab7a069eae6f@sentry-2.gardia.zenrocklabs.io:36656,63014f89cf325d3dc12cc8075c07b5f4ee666d64@sentry-3.gardia.zenrocklabs.io:46656,12f0463250bf004107195ff2c885be9b480e70e2@sentry-4.gardia.zenrocklabs.io:56656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.zrchain/config/config.toml

```
### 🚧Snap
```
zenrockd tendermint unsafe-reset-all --home $HOME/.zrchain
if curl -s --head curl http://37.120.189.81/zenrock_testnet/zenrock_testnet_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl http://37.120.189.81/zenrock_testnet/zenrock_testnet_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.zrchain
    else
  echo "no snapshot found"
fi
```
### 🚧Başlatalım   
```
sudo systemctl daemon-reload
sudo systemctl restart zenrockd
```
### 🚧Log
```
sudo journalctl -u zenrockd.service -f --no-hostname -o cat
```
### 🚧Cüzdan oluşturma
NOT: cüzdan adınızı yazınız
```
zenrockd keys add cuzdan-adini-yaz
```
- Eski cüzdan import ederkene bele
```
zenrockd keys add wallet --recover
```

### 🚧Validator oluşturma

```
cd $HOME
```
```
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(zenrockd comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000urock\",
    \"moniker\": \"test\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"CR\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json
# Create a validator using the JSON configuration
zenrockd tx validation create-validator validator.json \
    --from $WALLET \
    --chain-id gardia-5 \
	--fees 500000urock
```
### Delete
```
sudo systemctl stop zenrockd
sudo systemctl disable zenrockd
sudo rm -rf /etc/systemd/system/zenrockd.service
sudo rm $(which zenrockd)
sudo rm -rf $HOME/.zrchain
sed -i "/ZENROCK_/d" $HOME/.bash_profile
```
