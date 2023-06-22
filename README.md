# Stride Mainnet Türkçe Kurulum Rehberi

![Stride-1](https://user-images.githubusercontent.com/102043225/180435619-36ea3c1e-410d-4abb-b7c0-ec21152bbcfb.png)

##  Minimum Sistem Gereksinimleri
* 4vCPU
* 8GB RAM
* 200GB SSD

##  Önerilen Sistem Gereksinimleri
* 4vCPU
* 64GB RAM
* 1TB SSD/NVME

## Sistemi Güncelleme
```shell
sudo apt update && sudo apt upgrade -y
```

## Gerekli Kütüphanelerin Kurulması
```shell
sudo apt install curl make build-essential gcc tmux jq chrony htop -y < "/dev/null"
```

## Go Kurulumu
```shell
ver="1.19"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
rm -rf /usr/local/go
tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm -rf "go$ver.linux-amd64.tar.gz"
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

## Değişkenleri Yükleme
Aşağıda değiştirmeniz gereken yerleri yazıyorum.
* `$STRIDE_NODENAME` validator adınız. Örneğin; Bilge
* `$STRIDE_WALLET` cüzdan adınız. Örneğin; Bilge
```shell
echo "export STRIDE_NODENAME=$STRIDE_NODENAME"  >> $HOME/.bash_profile
echo "export STRIDE_WALLET=$STRIDE_WALLET" >> $HOME/.bash_profile
echo "export STRIDE_PORT=16" >> $HOME/.bash_profile
echo "export STRIDE_CHAIN_ID=stride-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
* Düzenleyeceğiniz kod aşağıdakine benzer olacak. Anlamanız için faydalı olacaktır. 
```shell
echo "export STRIDE_NODENAME=Bilge"  >> $HOME/.bash_profile
echo "export STRIDE_WALLET=Bilge" >> $HOME/.bash_profile
echo "export STRIDE_PORT=16" >> $HOME/.bash_profile
echo "export STRIDE_CHAIN_ID=stride-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Stride Kurulumu
```shell
cd $HOME
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout v10.0.0
make build
sudo cp $HOME/stride/build/strided /usr/local/bin
```

## Uygulamayı Yapılandırma
```shell
trided config chain-id $STRIDE_CHAIN_ID
strided config keyring-backend file
strided init $STRIDE_NODENAME --chain-id $STRIDE_CHAIN_ID
```

## Genesis ve addrbook Dosyasının İndirilmesi
```shell
curl https://raw.githubusercontent.com/Stride-Labs/testnet/infra-test/poolparty/infra/genesis.json > ~/.stride/config/genesis.json
curl https://raw.githubusercontent.com/koltigin/Stride-Mainnet-Turkce-Kurulum-Rehberi/main/addrbook.json > ~/.stride/config/addrbook.json

```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ustrd\"/" $HOME/.stride/config/app.toml
```

## SEED ve PEERS Ayarlanması
```shell
PEERS="3a6ea526f5f0857272c79448850abef71653d4df@83.136.249.85:26656,fbc05b4136a15b0b69a2b7f093731453f04ce2f4@192.168.50.57:26656,6795c4cd27d6132a547888ed5b7996aee454a025@172.25.0.2:26656,34c52450d2f107b7c164eb103641df9e45a322d4@65.21.192.108:26656,681803f48d5e9ef9870918e8330551513eccb31c@78.47.51.53:26656,0f2d7f17589e6e31691649ec04fe19561c0d12a6@10.138.0.7:26656,cb0b38aa612e8ac05f704d9b2feb7526607afb77@159.203.191.62:26656,55b446443f2bd68e06200c3f294c735c333722b0@162.251.235.252:26656,68fb634620e00a5a18f606360b6ca6d989da8ce6@65.108.106.131:26656,f56ddd6af02efaac4c47cc8053685d11c1065996@0.0.0.0:26656,f1721dd0324f29c108c1072b2e4fe9c64e63122d@192.168.86.25:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
SEEDS=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $HOME/.stride/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.stride/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.stride/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.stride/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stride/config/app.toml
```
## External Adres Ekleme
```shell
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.stride/config/config.toml
```

## Port Ayarlarını Yapılandırma
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${STRIDE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${STRIDE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${STRIDE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${STRIDE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${STRIDE_PORT}660\"%" $HOME/.stride/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${STRIDE_PORT}317\"%; s%^address = \":8080\"%address = \":${STRIDE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${STRIDE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${STRIDE_PORT}091\"%" $HOME/.stride/config/app.toml
```

## Zincir Verilerini Sıfırlama
```shell
strided tendermint unsafe-reset-all --home $HOME/.stride
```

## Servis Dosyası Oluşturma
```shell
tee <<EOF >/dev/null /etc/systemd/system/strided.service 
[Unit]
Description=stride
After=network-online.target
[Service]
User=$USER
ExecStart=$(which strided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## Servisi Başlatma ve Logları Kontrol Etme
```shell
sudo systemctl daemon-reload
sudo systemctl enable strided
sudo systemctl restart strided
journalctl -u strided -f -o cat
```  

### Yeni Cüzdan Oluşturma
`$STRIDE_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza isim belirledik.
```shell 
strided keys add $STRIDE_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
strided keys add $STRIDE_WALLET --recover
```  

* BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.

## Cüzdan Bakiyesini Kontrol Etme
```shell
strided query bank balances CUZDAN_ADRESINIZ --chain-id $STRIDE_CHAIN_ID
```  

## Senkronizasyonu Kontrol Etme
`false` çıktısı almaldıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
strided status 2>&1 | jq .SyncInfo
```

## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlittiğim yerler dışında bir değişikli yapmanız gerekmez;
  * `identity`  buraya `https://keybase.io` sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
  * `details` kendiniz hakkında bilgiler verebilir ya da `Rues Community Supporter` yazabilirsiniz.
  * `website`  Varsa bir siteniz yazabilirsiniz ya da `https://forum.rues.info` olarak bırakabilirsiniz.
  * `security-contact`  E-posta adresiniz.
 ```shell 
strided tx staking create-validator \
--amount=9900000ustrd \
--pubkey=$(strided tendermint show-validator) \
--moniker=$STRIDE_NODENAME \
--chain-id=$STRIDE_CHAIN_ID \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees=250ustrd \
--gas=200000 \
--from=$STRIDE_WALLET \
--details="Rues Community Supporter" \
--security-contact="E-POSTANIZ" \
--website="https://forum.rues.info" \
--identity="XXXX1111XXXX1111" \
-y
```

## FAYDALI KOMUTLAR

### Logları Kontrol Etme 
```shell
journalctl -fu strided -o cat
```

### Sistemi Başlatma
```shell
systemctl start strided
```

### Sistemi Durdurma
```shell
systemctl stop strided
```

### Sistemi Yeniden Başlatma
```shell
systemctl restart strided
```

### Node Senkronizasyon Durumu
```shell
strided status 2>&1 | jq .SyncInfo
```

### Validator Bilgileri
```shell
strided status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri
```shell
strided status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme
```shell
strided tendermint show-node-id
```

### Node IP Adresini Öğrenme
```shell
curl icanhazip.com
```

### Peer Adresinizi Öğrenme
```shell
echo $(strided tendermint show-node-id)@$(curl ifconfig.me)16656
```

### Cüzdanların Listesine Bakma
```shell
strided keys list
```

### Cüzdanı İçeri Aktarma
```shell
strided keys add $WALLET --recover
```

### Cüzdanı Silme
```shell
strided keys delete CUZDAN_ADI
```

### Cüzdan Bakiyesine Bakma
```shell
strided query bank balances CUZDAN_ADRESI
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma
```shell
strided tx bank send CUZDAN_ADRESI GONDERILECEK_CUZDAN_ADRESI 100000000ustrd
```

### Proposal Oylamasına Katılma
```shell
strided tx gov vote 1 yes --from $WALLET --chain-id=CHAIN_ID 
```

### Validatore Stake Etme  Delegate Etme
```shell
strided tx staking delegate $VALOPER_ADDRESS 100000000ustrd --from=$WALLET --chain-id=$CHAIN_ID  --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme  Redelegate Etme
```shell
strided tx staking redelegate MevcutValidatorAdresi StakeEdilecekYeniValidatorAdresi 100000000ustrd --from=WALLET --chain-id=CHAIN_ID  --gas=auto
```

### Ödülleri Çekme
```shell
strided tx distribution withdraw-all-rewards --from=$WALLET --chain-id=CHAIN_ID  --gas=auto
```

### Komisyon Ödüllerini Çekme
```shell
strided tx distribution withdraw-rewards VALIDATOR_ADRESI --from=$WALLET --commission --chain-id=CHAIN_ID 
```

### Validator İsmini Değiştirme
```shell
strided tx staking edit-validator 
--moniker=YENI_NODE_ADI 
--chain-id=$CHAIN_ID  
--from=$WALLET
```

### Validatoru Jail Durumundan Kurtarma 
```shell
strided tx slashing unjail 
  --broadcast-mode=block 
  --from=$WALLET 
  --chain-id=$CHAIN_ID  
  --gas=auto
```

### Node'u Tamamen Silme 
```shell
sudo systemctl stop strided && 
sudo systemctl disable strided && 
rm etc/systemd/system/stride.service && 
systemctl daemon-reload && 
cd $HOME && rm .stride -rf && rm stride -rf 
rm $(which strided) -rf 
```

# Hesaplar

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
