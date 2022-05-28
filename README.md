# stafihub-validator-TR
StaFiHub Validatör kurma testnet Türkçe rehberi

# Selamlar, hızlıca StaFiHub'ın validatör testnetini anlatıyorum, daha sonra form dolduruyoruz.

# Donanım Gereksinimleri, Minimal (En düşük VPS özellikleri):

```
4GB RAM
100GB SSD
2 vCPU

Recommended ( Önerilen VPS özellikleri)
8GB RAM
200GB SSD
4 vCPU
```
# Kurulum Adımları:

Önce yetkilendirme için hazırlık yapıyoruz ( Aşağıdaki kodları sırayla putty üzerinden sunucunuza girin)

```
cd $HOME
sudo apt update
sudo apt list --upgradable
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"
```
![image](https://user-images.githubusercontent.com/101149671/170845495-49341a4a-ab45-4be8-8954-c7e6461404f9.png)

# Şimdi Go kuruyoruz: (Son komutta go versiyonu resimdeki gibi olmalı sarı ile işaretlediğim tek satır olarak girilecek)
```
GO: https://go.dev/dl/
-------------------------------------------------------------------------------------
cd $HOME
wget -O go1.17.3.linux-amd64.tar.gz https://golang.org/dl/go1.17.3.linux-amd64.tar.gz
```
![image](https://user-images.githubusercontent.com/101149671/170845524-46ebad9b-5a35-44a3-b8fa-67b88efe4c5f.png)

```
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.17.3.linux-amd64.tar.gz && rm go1.17.3.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```
![image](https://user-images.githubusercontent.com/101149671/170845531-347d9ad6-5da3-4e26-a5b9-7dbbce016bc5.png)

# Şimdi Git Deposunu Klonluyoruz: ( Güncel versiyonuna göre komutu düzenledim yeni güncelleme olursa tekrar güncelleme duyurusu iletiriz)

```
git clone --depth 1 --branch public-testnet-v2 https://github.com/stafihub/stafihub
```
![image](https://user-images.githubusercontent.com/101149671/170845547-0e99051c-d0a5-4193-a863-46bcde436db0.png)

# Şimdi Kurulum Yapıyoruz:
```
cd $HOME/stafihub && make install
```
![image](https://user-images.githubusercontent.com/101149671/170845560-30e8bc04-6c01-4ed8-84f0-2f04726b2bdc.png)

# Genesis İndiriyoruz ( Aşağıdaki YOUR_NODE_NAME yazan yere kendi Validatör isminizi girin)
```
stafihubd init YOUR_NODE_NAME --chain-id stafihub-public-testnet-2
wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/s...stnets/stafihub-public-testnet-2/genesis.json"
stafihubd tendermint unsafe-reset-all --home ~/.stafihub
```
![image](https://user-images.githubusercontent.com/101149671/170845563-459473d1-ba3c-4aed-bed0-09883031a909.png)

# Node yapılandırıyoruz:
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
sed -i /\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
peers="4e2441c0a4663141bb6b2d0ea4bc3284171994b6@46.38.241.169:26656,79ffbd983ab6d47c270444f517edd37049ae4937@23.88.114.52:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
```

# Screen Kuruyoruz ve Screen içinde stafihub komutunu başlatacağız:
```
screen -S stafi
```
# Node arka planda çalıştırıyoruz:
```
stafihubd start
```
# Yada Node çalıştırmak için aşağıdaki hizmeti yükleyin (bunu tercih ediyorum)
```
echo "[Unit]
Description=StaFiHub Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which stafihubd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/stafihubd.service
sudo mv $HOME/stafihubd.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable stafihubd
sudo systemctl restart stafihubd
```
# Node loglarınızı kontrol edin:
```
journalctl -u stafihubd -f
```

# Cüzdan oluşturma ( Aşağıdaki YOUR_WALLET_NAME yazan yere kendi Cüzdan isminizi girin ve şifre belirleyin, mnemonic kelimeleri ve cüzdan adresinizi farklı bir yere kaydedin)
```
stafihubd keys add YOUR_WALLET_NAME --keyring-backend file
```
# Discord kanalından jeton talebinde bulunabilirsiniz.

Discord: https://discord.gg/FUdSUPTDHD

Aşağıdaki YOUR_WALLET_ADRESS yazan yere kendi Cüzdan adresinizi yazın ve discord üzerinden jeton talep edin.
Daha önce göstermiştim en altta faucet kanalından !faucet send + adresinizi girerek alıyorsunuz.

# Validatöre $FIS delege etmek : Aşağıdaki YOUR_NODE_NAME yazan yere kendi node adınızı ve YOUR_WALLET_NAME yazan yere kendi cüzdan adınızı yazın.
```
stafihubd tx staking create-validator -y --amount=1000000ufis --pubkey=$(stafihubd tendermint show-validator) --moniker=YOUR_NODE_NAME --commission-rate=0.10 --commission-max-rate=0.20 --commission-max-change-rate=0.01 --min-self-delegation=1 --from=YOUR_WALLET_NAME --chain-id=stafihub-public-testnet-2 --gas-prices=0.025ufis --keyring-backend file
```

# Aşağıdaki site üzerinden validatörünüzü izleyebilir ve cüzdan bilgilerinizi girerek $FIS delegasyon undelegasyon işlemlerini yapabilirsiniz.
```
https://testnet-explorer.stafihub.io/
```

--------------------------------------------------------------------------------------------------------------------------------------------

Şimdi gelelim node kuran ve kurmayan kişilere, https://testnet-explorer.stafihub.io/ üzerine gidip bizim topluluğumuzda ki validatörlere FIS stake etmek

Örnek, AhmKah55 ve forgottensemicolon bizim topluluğumuzdan, ekstra olarak bana da delege edebilirsiniz.

VALIDATOR KURMADIYSANIZ BİLE BU GÖREVİ KESİNLİKLE YAPIN, teşekkürler <3

![image](https://user-images.githubusercontent.com/101149671/170845711-89ff69a1-61b8-4aa6-aea8-54972678827e.png)
