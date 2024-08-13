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
- Buka ip:8080 atau localhost:8080 di browser (Masukkan domain jika menggunakan domain) buat user admin dan install gitea.
