# Drosera Node Kurulum Rehberi (SystemD, Docker'sız)

Bu rehber, sistem gereksinimi düşük olan @DroseraNetwork node'unu kolayca kurabilmeniz için hazırlanmıştır. Boşta duran VPS’lerinizde veya ücretsiz sunucularda çalıştırabilirsiniz.

## Sistem Gereksinimleri

| Gereksinim              | Detaylar                                 |
|------------------------|------------------------------------------|
| CPU Mimarisi           | amd64 veya arm64                         |
| RAM                    | En az 4 GB RAM                          |
| Depolama               | En az 20 GB SSD                         |
| İşletim Sistemi        | Ubuntu 20.04 veya üzeri                 |
| Cüzdan Gereksinimi     | Holesky ağı üzerinde ETH bulunan EVM cüzdan |

## Sunucu Önerileri

**Ücretsiz:**
- https://www.digitalocean.com/ → Kredi veriyor, kredi ile sunucu kiralayabilirsiniz.

**Ücretli:**
- https://contabo.com/en/vps/cloud-vps-4c/ → 6$ civarı, en ucuzunu seçebilirsiniz.

## Kurulum Öncesi Gereklilikler

- Yeni bir MetaMask cüzdanı oluşturun.
- Bu cüzdana Holesky ETH yükleyin.

📌 Faucet bağlantısı: https://www.alchemy.com/faucets

## 1- Sunucuya Bağlanma

```bash
ssh root@[Sunucu_IP]
```

## 2- Gerekli Güncellemeleri ve Araçları Yükleyin

```bash
sudo apt-get update && sudo apt-get upgrade -y && sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## 3- CLI Kurulumları

```bash
curl -L https://app.drosera.io/install | bash
source /root/.bashrc
droseraup
```
```bash
curl -L https://foundry.paradigm.xyz | bash![Go18yj6WMAAHvfj](https://github.com/user-attachments/assets/11009cc2-4d78-46ec-b8d2-6f3d0caaf0c1)

source /root/.bashrc
foundryup
```
```bash
curl -fsSL https://bun.sh/install | bash
```
```bash
source /root/.bashrc
```

## 4- Trap Oluşturma

```bash
mkdir my-drosera-trap && cd my-drosera-trap
```
```bash
git config --global user.email "mail-adresini-gir"
git config --global user.name "github-kullanici-adi"
```
- "mail-adresini-gir" kısmına githup mail adresinizi girin.
- "github-kullanici-adi" kısmına githup kullanıcı adınızı girin.

## 5- Template'i Çekip Derleme

```bash
forge init -t drosera-network/trap-foundry-template
```
```bash
curl -fsSL https://bun.sh/install | bash
bun install
```
```bash
forge build
```

## 6- Deploy Etme

```bash
DROSERA_PRIVATE_KEY=kendi-privatekey drosera apply
```
- Yukarıdaki "kendi-privatekey" kısmına Holesky ETH bulunan MetaMask cüzdanınızın private key’ini girin.
- Deploy sırasında sizden onay istenirse ofc yazıp Enter'a basın.

## 7- Kontrol Et ve Bloom Boost İşlemi

- https://app.drosera.io adresine girin
- Cüzdanınızı bağlayın
- "Traps Owned" sekmesinden trap'e tıklayın
- "Send Bloom Boost" butonuna basın ve bir miktar Holesky ETH yatırın

## 8- Whitelist Ayarı

```bash
cd ~/my-drosera-trap
nano drosera.toml
```

```toml
private_trap = true
whitelist = ["0xSeninPublicCuzdanAdresin"]
```
⚠ Fazladan private ve whitelist'i silin.

```bash
DROSERA_PRIVATE_KEY=kendi-privatekeyin drosera apply
```
- Yukarıdaki "kendi-privatekeyin" kısmına Holesky ETH bulunan MetaMask cüzdanınızın private key’ini girin.

## 9- Operatör CLI Kurulumu

```bash
cd ~
curl -LO https://github.com/drosera-network/releases/releases/download/v1.16.2/drosera-operator-v1.16.2-x86_64-unknown-linux-gnu.tar.gz
tar -xvf drosera-operator-v1.16.2-x86_64-unknown-linux-gnu.tar.gz
sudo cp drosera-operator /usr/bin
drosera-operator --version
```

## 10- Alchemy RPC Alma

1. https://dashboard.alchemy.com adresine git
2. Ethereum → Holesky ağı için bir uygulama oluştur
3. Uygulama sayfasından özel RPC linkini kopyala

## 11- Portları Aç:

```bash
sudo ufw allow ssh && \
sudo ufw allow 22 && \
sudo ufw allow 31313/tcp && \
sudo ufw allow 31314/tcp && \
sudo ufw --force enable
```

## 12- SystemD Servisi Oluşturma

```bash
sudo tee /etc/systemd/system/drosera.service > /dev/null <<EOF
[Unit]
Description=drosera node service
After=network-online.target

[Service]
User=$USER
Restart=always
RestartSec=15
LimitNOFILE=65535
ExecStart=$(which drosera-operator) node   --db-file-path $HOME/.drosera.db   --network-p2p-port 31313   --server-port 31314   --eth-rpc-url Oluşturduğun_RPC   --eth-backup-rpc-url https://1rpc.io/holesky   --drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8   --eth-private-key Private_Keyin   --listen-address 0.0.0.0   --network-external-p2p-address VPS_IP_Adresin   --disable-dnr-confirmation true

[Install]
WantedBy=multi-user.target
EOF
```
- "Private_Keyin" kısmına Deploy ettiğiniz cüzdanın private key’i girin.
- "VPS_IP_Adresin" kısmına VPS sunucunuzun IP adresini girin.
- "Oluşturduğun_RPC" kısmına Alchemy'den aldığınız özel Holesky RPC adresini girin.

## 13- Node'u Başlatma ve Kontrol Etme

```bash
sudo systemctl daemon-reload
sudo systemctl enable drosera
```
```bash
sudo systemctl start drosera
```
```bash
journalctl -u drosera.service -f
```

## ✅ Aşağıdaki gibi loglar almaya başladıysanız, kurulumunuzu başarılı bir şekilde tamamladınız demektir,
![Ekran görüntüsü 2025-04-18 201619](https://github.com/user-attachments/assets/5d9b6c89-a2ca-49c8-bdf4-f91e77368dfa)

## 14- Opt in Tx:

- https://app.drosera.io adresine git.
- "Traps Owned"a git ve deploy ettiğin trap'e tıkla.
- "Opt in" tuşuna bas ve cüzdanından gelen onaylamaları tamamla.

## Sağlıklı Bloklar İçin Durdur ve Tekrar Başlat:

```bash
# Node'u Durdur
sudo systemctl stop drosera
```
```bash
# Tekrar Başlat
sudo systemctl restart drosera
```

- [Bu sayede node’unuzun çalışıp çalışmadığını kolaylıkla kontrol edebilirsiniz.](https://app.drosera.io adresine girerek node’unuzun yeşil bloklar ürettiğini görebilirsiniz.
Bu sayede node’unuzun çalışıp çalışmadığını kolaylıkla kontrol edebilirsiniz.)

- ✅ "CTRL + C" yaparak sunucudan çıkış yapabilirsiniz. Node’unuz arka planda çalışmaya devam edecektir.
![Go16e2MXYAAL2QX](https://github.com/user-attachments/assets/bf58d2da-b57b-476c-bee2-0f4c47a22465)

---

Sosyal Medya Hesabım: https://x.com/UfukDegen
