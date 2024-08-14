# My Exprience about Installation Gitea
> [!TIP]
> Saya menjalankan semua perintah ini sebagai `root`, jadi jika anda masih login sebagai user biasa diharap login sebagai `root` terlebih dahulu.
## Update and Upgrade Repository
`apt update && apt upgrade -y`
## Install git python3-certbot (SSL)
`apt install git python3-certbot-nginx -y`
## Add `git` User
```
adduser \
   --system \
   --shell /bin/bash \
   --gecos 'Git Version Control' \
   --group \
   --disabled-password \
   --home /home/git \
   git
  ```
> [!IMPORTANT]
> Catat UID dan GUID yang dihasilkan dari membuat user `git` 
## Install Docker
Untuk memastikan docker di install dengan bersih, harap clear installation docker dengan menjalankan :
```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
Dan Panduan installasi docker bisa diikuti dari dokumentasi resmi docker [Disini](https://docs.docker.com/engine/install/ubuntu/#installation-methods)

## Install Gitea
- Pertama kita login sebagai user `git` yang sudah kita buat di awal dengan cara `su git`.
- Buat direktori dan file `docker-compose.yml` : <br>
  `mkdir /home/git/gitea && /home/git/gitea/nano docker-compose.yml`
- Copy dan Paste code dibawah ini pada file `docker-compose.yml`, Jangan lupa untuk mengganti `USER_UID` dan `USER_GUID`, ganti dengan UID dan GUID dari hasil kita membuat user `git` diatas <br>
```
version: "3"

networks:
  gitea:
    external: false
volumes:
  gitea:
    driver: local
  data:
services:
  server:
    image: gitea/gitea:1
    container_name: gitea
    environment:
      - USER_UID=UID
      - USER_GID=GUID/GID
      - GITEA__database__DB_TYPE=mysql
      - GITEA__database__HOST=db:3306
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=giteaZr
    restart: always
    networks:
      - gitea
    volumes:
      - gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "8080:3000"
      - "2221:22"
    depends_on:
      - db

  db:
    image: mariadb
    restart: always
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_bin
    environment:
      - MYSQL_ROOT_PASSWORD=giteaZr
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=giteaZr
      - MYSQL_DATABASE=gitea
    networks:
      - gitea
    volumes:
      - data:/var/lib/mysql
    ports:
      - "3306:3306"
```
> [!CAUTION]
> Jika installasi anda dengan tujuan *_Productions_* diharap untuk merubah `environment` pada kode diatas.<br>
- Lalu kita jalankan kode diatas untuk membuat Container pada docker dengan cara : <br>
  `cd /home/git/gitea/ && docker compose up -d`
- Buka `ip:8080` atau `localhost:8080` di browser (Masukkan domain jika menggunakan domain) buat user admin dan install gitea.
- Installasi Gitea selesai.

## Install Docker Manager
Docker manager ini bertujuan agar tidak perlu menggunakan CLI lagi, tidak perlu khawatir didalam Docker Manager ini juga terdapat fitur CLI, docker manager yang saya gunakan adalah `dockge` yang dapat anda temukan [Disini](https://github.com/louislam/dockge) Dan untuk Installasinya ada [Disini](https://github.com/louislam/dockge?tab=readme-ov-file#basic) Atau bisa langsung mengikuti code ini :<br>
> [!NOTE]
> Saya sarankan tetap rujuk [repositori aslinya](https://github.com/louislam/dockge) untuk mendapat update terbaru
```
# Create directories that store your stacks and stores Dockge's stack
mkdir -p /opt/stacks /opt/dockge
cd /opt/dockge

# Download the compose.yaml
curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml

# Start the server
docker compose up -d

# If you are using docker-compose V1 or Podman
# docker-compose up -d
```
Jika anda tidak mengubah port pada file `compose.yaml` maka dockge akan berjalan di `IP:5001` Atau `localhost:5001`. Buat username dan password(harus unik) untuk admin dan klik tombol Create.
- Installasi dockge selesai.
## (Opsional) Install NGINX Sebagai `reverse_proxy`
> [!NOTE]
> Jika anda tidak memiliki domain bisa skip langkah ini.<br>

Installasi nginx di server langsung sebenarnya tidak di rekomendasikan, demi keamanan lebih baik install di docker sebagai `reverse_proxy`. Dalam hal ini saya akan menggunakan tool Nginx Proxy Manager yang dapat anda temukan [Disini](https://github.com/NginxProxyManager/nginx-proxy-manager)  <br>

### Langkah installasi :
   - Login ke `dockge` yang baru anda install
   - Klik tombol `+ Compose` atau `+ Menyusun`
   - Copy dan Paste code dibawah ini pada kolom `compose.yaml` diatas kolom `.env` : <br>
   ```
   services:
        app:
          image: 'docker.io/jc21/nginx-proxy-manager:latest'
          restart: unless-stopped
          ports:
            - '80:80'
            - '81:81'
            - '443:443'
          volumes:
            - ./data:/data
            - ./letsencrypt:/etc/letsencrypt
   ```
   - Klik Tombol `Deploy` atau `Terapkan`
   - Lalu buka di browser `IP:81` atau `localhost:81` Email dan Password Dockge biasanya `admin@example.com` dengan pass `changeme`, Sekali lagi baca dokumentasi resmi dari repository [Nginx Proxy Manager](ttps://github.com/NginxProxyManager/nginx-proxy-manager).
### Ujicoba Domain
   - Buat domain anda mengarah ke IP public anda saya disini memakai contoh `domain.com` sebagai root domain.
   - Setelah anda berhasil masuk ke Nginx Proxy Manager Klik Menu `Host`->`Proxy Hosts`->`Add Proxy Host`.
   - Di Tab `Details` isikan Domain Names dengan subdomain yang anda buat contoh `dockge.domain.com`. Pada kolom `Forward Hostname / IP` isikan `localhost`, Pada kolom `Forward Port` isikan port dari installasi `dockge` yaitu di port `5001`. Aktifkan `Cache Assets` dan `Block Common Exploits`.
   - Pindah ke Tab `SSL` pilih `Request a new SSL Certificate`, Aktifkan `Force SSL`, `HTTP/2 Support`, `HSTS Enabled`, `I Agree to the Let's Encrypt Terms of Service`. dan jangan lupa isikan email.
   - Klik simpan dan tunggu proses selesai, lalu buka domain `dockge.domain.com`.
   - Done
