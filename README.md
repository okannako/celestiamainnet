![208](https://github.com/okannako/celestiamainnet/assets/73176377/a36d1b99-80be-47c6-869b-d32c2f5790dd)

* Celestia Mainnet Beta ağında bu kılavuz içerisinde adım adım ilerleyerek Mainnet Beta ağında Validator, Bridge, Full Storage ve Light Node çalıştırabilirsiniz.

* Kurulumn sırasında veya sonrasında bir şey sormak isterseniz bana Telegram, Mail ve Discord yoluyla ulaşabirsiniz.

## Validator Node Minimum Sistem Gereksinimleri

 - Memory: 8 GB RAM
 - CPU: 6 Cores
 - Disk: 500 GB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.22.0"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Dosyaları Yüklemek
```
cd $HOME 
rm -rf celestia-app 
git clone https://github.com/celestiaorg/celestia-app.git 
cd celestia-app/ 
APP_VERSION=v1.10.1
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
celestia version
```

### Inıt Node
```
celestia-appd init "Node Name" --chain-id celestia
```

### Genesis ve Addrbook İndirmek
```
wget -O $HOME/.celestia-app/config/genesis.json https://mainnet-files.itrocket.net/celestia/genesis.json
wget -O $HOME/.celestia-app/config/addrbook.json https://mainnet-files.itrocket.net/celestia/addrbook.json
```

### Seeds ve Peers Eklemek
```
SEEDS="12ad7c73c7e1f2460941326937a039139aa78884@celestia-mainnet-seed.itrocket.net:40656"
PEERS="d535cbf8d0efd9100649aa3f53cb5cbab33ef2d6@celestia-mainnet-peer.itrocket.net:40656,fed121cc9450f5518f2441ee9e1168392027a117@135.181.236.8:36656,905cecaaefc2ec59c0f383ff4e318baf9530e903@65.109.49.164:26000,5f7b67e8f41fec251f9b86045b1b648a2fba5988@37.120.245.61:26656,3e9edb7aa157894b498e75373e2148f7c22100b2@103.219.171.67:26656,140df92f010e78bd9d854e47d019468e10df7628@65.109.27.253:26001,e19f9cd1005fa5176ebf97c73694863f361a9d9b@95.216.71.19:11656,2a4cf74e21a28f4aa387ad00de3956af512d837b@65.108.123.161:26656,57307aa4cac3b00aace33d34fe0a7c52e290705f@80.190.129.50:27657,5e727b10ba178737a56f2891168a0e42380b6755@198.244.200.98:26664,9a179402b03fa08f4b635439a0cd857184c87979@65.21.69.230:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.celestia-app/config/config.toml
```

### Pruning Açmak (Zorunlu Değil)
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.celestia-app/config/app.toml
```

### Minimum Gas Price Eklemek
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.002utia"|g' $HOME/.celestia-app/config/app.toml
```

### Cüzdan Oluşturnmak
- Aşağıdaki kodu girdiğinizde ekranda size şifre koymanızı ister ve bundan sonra cüzdan adresinizi, cüzdanın gizli kelimelerini gösteriri. Bunları mutlaka kaydedin, herhangi bir geri getirme durumunda bu kelimeler gerekli.

```
celestia-appd keys add cuzdanadı
```

Not: Herhangi bir yerden TIA satın alıp ilgili cüzdana atıp validator oluşturabilirsiniz.

### Servis Oluşturmak
```
sudo tee /etc/systemd/system/celestia-appd.service > /dev/null <<EOF
[Unit]
Description=Celestia node
After=network-online.target
[Service]
User=root
ExecStart=$(which celestia-appd) start --home $HOME/.celestia-app
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Node Başlatmnak
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-appd
sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f
```

### Sisteme Eşitlenme Kontrolü
- Kodu girdiğinizde sonuç olarak false verdiğinde sisteme eşitlenmiş demektir.

```
celestia-appd status 2>&1 | jq 
```

### Validator Oluşturmak
- Sisteme eşitlendikten sonra aşağıdaki kod ile validator oluşturabilirsiniz. 1 adet test tokenına göre ayarlanmıştır.
```
celestia-appd tx staking create-validator \
--amount 1000000utia \
--from cuzdanismi \
--moniker "Monikerismi" \
--chain-id celestia \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(celestia-appd tendermint show-validator) \
--gas 500000 \
--fees 3000utia \
--identity "" \
--website "" \
--details "" \
-y
```

NOT: Ağda olan bütün durumları ve tx sonuçarı için [Explorer](https://celestia.explorers.guru/) için siteyi ziyaret edebilirsiniz.


## Bridge Node Minimum Sistem Gereksinimleri

 - Memory: 16 GB RAM (Minimum)
 - CPU: 6 Cores
 - Disk: 10 TB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.22.0"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Celestia-Node Yüklemek
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.13.7
make build 
make install 
make cel-key 
```

- Versiyon Kontrol ```celestia version``` >>> 0.13.7

### Celestia-App Yüklemek
```
cd $HOME 
rm -rf celestia-app 
git clone https://github.com/celestiaorg/celestia-app.git 
cd celestia-app/ 
APP_VERSION=v1.9.0
git checkout tags/$APP_VERSION -b $APP_VERSION 
make install
```

### Init İşlemi
```
celestia bridge init --core.ip rpc.celestia.pops.one
```

### Cüzdan Oluşturma
```
./cel-key list --node.type bridge --p2p.network celestia --keyring-backend test
```

NOT: Herhangi bir yerden TIA alıp cüzdana yollamanız gerekiyor.

### Servis Oluşturma
```
sudo tee /etc/systemd/system/celestia-bridge.service > /dev/null <<EOF
[Unit]
Description=celestia-bridge Cosmos daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/celestia bridge start --core.ip rpc.celestia.pops.one --core.rpc.port 26657 --core.grpc.port 9090 --keyring.accname my_celes_key --metrics.tls=true --metrics --metrics.endpoint otel.celestia.observer
Restart=on-failure
RestartSec=3
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```

### Bridge Node Başlatmak
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-bridge
sudo systemctl start celestia-bridge && sudo journalctl -u celestia-bridge -f
```

### Node Id Öğrenmek
```
AUTH_TOKEN=$(celestia bridge auth admin --p2p.network celestia)
```
```
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```

### Bridge Node Silmek
```
sudo systemctl stop celestia-bridge
sudo systemctl disable celestia-bridge
rm -rf $HOME/celestia-node  
rm -rf $HOME/.celestia-bridge-mocha-4
rm -rf $HOME/.celestia-app
```

## Full Storage Node Sistem Gereksinimleri

 - Memory: 16 GB RAM (Minimum)
 - CPU: Quad-Core Cores
 - Disk: 10 TB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.22.0"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Celestia-Node Yüklemek
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.13.7 
make build 
make install 
make cel-key 
```

### Init İşlemi
```
celestia full init --core.ip rpc.celestia.pops.one 
```

### Cüzdan Oluşturma
```
./cel-key list --node.type full --keyring-backend test --p2p.network celestia
```

### Servis Oluşturma
```
sudo tee /etc/systemd/system/celestia-full.service > /dev/null <<EOF
[Unit]
Description=celestia-bridge Cosmos daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/celestia full start --core.ip rpc.celestia.pops.one --core.rpc.port 26657 --core.grpc.port 9090 --keyring.accname my_celes_key --metrics.tls=true --metrics --metrics.endpoint otel.celestia.observer
Restart=on-failure
RestartSec=3
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```

### Node Başlatmak
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-full
sudo systemctl start celestia-full && sudo journalctl -u celestia-full -f
```

### Node Id Öğrenmek
```
AUTH_TOKEN=$(celestia full auth admin --p2p.network celestia)
```
```
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```

### Full Storage Node Silmek
```
sudo systemctl stop celestia-full
sudo systemctl disable celestia-full
rm -rf $HOME/celestia-node  
rm -rf $HOME/.celestia-full-mocha-4
rm -rf $HOME/.celestia-app
```

## Light Node Sistem Gereksinimleri

 - Memory: 500 MB RAM (Minimum)
 - CPU: Single Core
 - Disk: 100 GB SSD Storage
 - Bandwidth: 56 Kbps 

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.22.0"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Celestia-Node Yüklemek
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.13.7 
make build 
make install 
make cel-key 
```

### Cüzdan Oluşturma
```
./cel-key list --node.type light --p2p.network mocha --keyring-backend test
```

### Init İşlemi
```
celestia light init --core.ip rpc.celestia.pops.one --p2p.network celestia
```

### Servis Oluşturma
```
sudo tee /etc/systemd/system/celestia-light.service > /dev/null <<EOF
[Unit]
Description=celestia-bridge Cosmos daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/celestia light start --core.ip rpc.celestia.pops.one --core.rpc.port 26657 --core.grpc.port 9090 --keyring.accname my_celes_key --metrics.tls=true --metrics --metrics.endpoint otel.celestia.observer
Restart=on-failure
RestartSec=3
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```

### Node Başlatmak
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-full
sudo systemctl start celestia-full && sudo journalctl -u celestia-full -f
```

### Node Id Öğrenmek
```
AUTH_TOKEN=$(celestia light auth admin --p2p.network celestia)
```
```
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```

### Light Node Silmek
```
sudo systemctl stop celestia-light
sudo systemctl disable celestia-light
rm -rf $HOME/celestia-node  
rm -rf $HOME/.celestia-light-mocha
rm -rf $HOME/.celestia-app
```
