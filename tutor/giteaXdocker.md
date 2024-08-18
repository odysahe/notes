# My Exprience about Installation Gitea
> [!NOTE]
> Gitea yang saya gunakan `Gitea v1.22.1`, Server `VPS Hostinger` -> OS `Ubuntu 22.04`, Domain Management `Cloudflare`.<br>
> Anda harus, bahkan WAJIB! mengerti cara mengoperasikan [Git](https://git-scm.com/).<br>
> Akan sangat berguna jika anda sudah paham [Github Actions](https://docs.github.com/en/actions).<br>
> Saya menjalankan semua perintah ini sebagai `root`, jadi jika anda masih login sebagai user biasa diharap login sebagai `root` terlebih dahulu.
## Update and Upgrade Repository
   `apt update && apt upgrade -y`
## Install git
   `apt install git`
## Add `git` User
   ```bash
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
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
Dan Panduan installasi docker bisa diikuti dari dokumentasi resmi docker [Disini](https://docs.docker.com/engine/install/ubuntu/#installation-methods)

## Install Gitea
- Pertama kita login sebagai user `git` yang sudah kita buat di awal dengan cara `su git`.
- Buat direktori dan file `docker-compose.yml` : <br>

  ```bash
  mkdir /home/git/gitea && nano /home/git/gitea/docker-compose.yml
  ```
- Copy dan Paste code dibawah ini pada file `docker-compose.yml`, Jangan lupa untuk mengganti `USER_UID` dan `USER_GUID`, ganti dengan UID dan GUID dari hasil kita membuat user `git` diatas <br>

   ```yaml
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
       # command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_bin
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

  ```bash
  exit && cd /home/git/gitea/ && docker compose up -d
  ```
- Buka `ip:8080` atau `localhost:8080` di browser (Masukkan domain jika menggunakan domain) buat user admin dan install gitea.
- Installasi Gitea selesai.

## Install Docker Manager
Docker manager ini bertujuan agar tidak perlu menggunakan CLI lagi, tidak perlu khawatir didalam Docker Manager ini juga terdapat fitur CLI, docker manager yang saya gunakan adalah `dockge` yang dapat anda temukan [Disini](https://github.com/louislam/dockge) Dan untuk Installasinya ada [Disini](https://github.com/louislam/dockge?tab=readme-ov-file#basic) Atau bisa langsung mengikuti code ini :<br>
> [!NOTE]
> Saya sarankan tetap rujuk [repositori aslinya](https://github.com/louislam/dockge) untuk mendapat update terbaru

   ```bash
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
> Jika anda tidak memiliki domain bisa lewati langkah ini.<br>

Installasi nginx di server langsung sebenarnya tidak di rekomendasikan, demi keamanan lebih baik install di docker sebagai `reverse_proxy`. Dalam hal ini saya akan menggunakan tool Nginx Proxy Manager yang dapat anda temukan [Disini](https://github.com/NginxProxyManager/nginx-proxy-manager)  <br>

### Langkah installasi :
   - Login ke `dockge` yang baru anda install
   - Klik tombol `+ Compose` atau `+ Menyusun`
   - Copy dan Paste code dibawah ini pada kolom `compose.yaml` diatas kolom `.env` : <br>
   
      ```yaml
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
   - Buat juga subdomain untuk `gitea`, contoh `git.domain.com`, ikuti dari awal langkah [Uji Domain](https://github.com/odysahe/notes/blob/main/tutor/giteaXdocker.md#ujicoba-domain)
   - Done

## Gitea `act_runner`
> [!WARNING]
> Jika anda menggunakan Domain sebelum memulai `act_runner` pastikan url dari `gitea` anda sudah benar, masuk ke halaman admin pada web gitea.
> Pada saat anda login sebagai admin di gitea web akan ada peringatan ketidak sesuaian ROOT_URL, Edit ROOT_URL yang semula `http://git.domain.com/` menjadi `https://git.domain.com/` dengan cara ikuti langkah [ini](https://docs.gitea.com/installation/install-with-docker#customization), atau langsung dibawah ini: <br>
> ```bash
> nano /var/lib/docker/volumes/gitea_gitea/_data/gitea/conf/app.ini
> # restart container gitea
> docker container restart gitea
> ```
Gitea `act_runner` adalah pelari actions atau library untuk menguji aplikasi agar bisa di deploy ke docker. `act_runner` dapat anda temukan [disini](https://gitea.com/gitea/act_runner), installasi yang saya gunakan dari [sini](https://gitea.com/gitea/act_runner#run-with-docker), sebagai contoh dari saya : <br>

   - Buka terminal dan login sebagai `root`
   - Jalankan perintah berikut :
     
      ```bash
        docker run -e GITEA_INSTANCE_URL=https://git.domain.com -e GITEA_RUNNER_REGISTRATION_TOKEN=<your_token> -v /var/run/docker.sock:/var/run/docker.sock --name my_runner gitea/act_runner:nightly
      ```
> [!NOTE]
> - `GITEA_INSTANCE_URL` -> Diisi url ke git anda
> - `GITEA_RUNNER_REGISTRATION_TOKEN` -> Diisi dengan token yang didapat dari <br> `git.domain.com/user/settings/actions/runners` -> `Create New Runner` (Untuk alasan keamanan disarankan untuk tidak mengambil dari admin web karna semua pengguna git tanpa terkecuali akan bisa menggunakan `act_runner`)
> - `--name my_runner` Nama Container docker

   - Selanjutnya lihat ke halaman web gitea `git.domain.com/user/settings/actions/runners` jika pada kolom status tidak `idle` maka jalankan perintah berikut: <br>
      ```bash
      docker container start my_runner 
      ```
   - Lihat kembali pada halaman runner jika sudah berstatus idle maka runners sudah siap digunakan.
   - Buat dan Clone Repository anda
   - Jika meminta username dan password masukkan username dan password saat anda akan login ke gitea.
   - Setelah berhasil clone repository anda, buat folder `nama-repositori/.gitea/workflows` dan buat file berektensi `.yaml` contoh: `myrepository/.gitea/workflows/runner.yaml` dan isi file `.yaml` dengan text :
     
     ```yaml
      name: Gitea Actions Demo
      run-name: ${{ gitea.actor }} is testing out Gitea Actions 🚀
      on: [push]
      
      jobs:
        Explore-Gitea-Actions:
          runs-on: ubuntu-latest
          steps:
            - run: echo "🎉 The job was automatically triggered by a ${{ gitea.event_name }} event."
            - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by Gitea!"
            - run: echo "🔎 The name of your branch is ${{ gitea.ref }} and your repository is ${{ gitea.repository }}."
            - name: Check out repository code
              uses: actions/checkout@v4
            - run: echo "💡 The ${{ gitea.repository }} repository has been cloned to the runner."
            - run: echo "🖥️ The workflow is now ready to test your code on the runner."
            - name: List files in the repository
              run: |
                ls ${{ gitea.workspace }}
            - run: echo "🍏 This job's status is ${{ job.status }}."
      
     ```
  - Simpan dan push ke repository anda kemudian lihat di halaman actions di repository anda, maka akan tampil runner yang sedang berjalan.
  - Done, sekarang `Gitea Actions` siap digunakan.
> [!NOTE]
> Jika anda sudah menginstall `act_runner` di docker tidak perlu menjalankan perintah `docker run` lagi cukup jalankan:<br>
>  ```bash
>   docker exec my_runner act_runner register --instance https://git.domain.com --token <your_token> --no-interactive --name name_of_runner
>  ```
### Test deploy VueJs App
   - Buat aplikasi [VueJs](https://vuejs.org/) dengan contoh nama folder `myvue`, Langka-langkah pembuatan aplikasi VueJs bisa ditemukan [disini](https://vuejs.org/guide/quick-start.html#creating-a-vue-application).
   - Buat folder `/myvue/.gitea/wokflows/` dan isi dengan file `runner.yaml`.
   - Copy dan Paste kode dibawah ini (Ganti `git.domain.com` dengan domain anda sendiri atau IP Lokal anda):

     ```yaml
      name: Build And Push
      run-name: ${{ gitea.actor }} is runs ci pipeline
      on: [ push ]
      
      jobs:
        build-push:
          runs-on: ubuntu-latest
          env:
            RUNNER_TOOL_CACHE: /toolcache
          steps:
            - name: Checkout
            - uses: https://github.com/actions/checkout@v4
            - name: Use Node.js
              uses: https://github.com/actions/setup-node@v4
              with:
                node-version: '20.15.0'
            - run: npm install
            - run: npm run build
              env:
                 NODE_OPTIONS: --max_old_space_size=4096
            - name: Set up Docker Buildx
              uses: https://github.com/docker/setup-buildx-action@v3
              with:
                config-inline: |
                  [registry."git.domain.com"]
                    http = true
                    insecure = true            
            - name: Login to DockerHub
              uses: docker/login-action@v2
              with:
                registry: git.domain.com
                username: ${{ secrets.DOCKER_USERNAME }}
                password: ${{ secrets.DOCKER_PASSWORD }}
                
            - name: Get Meta
              id: meta
              run: |
                echo REPO_NAME=$(echo ${GITHUB_REPOSITORY} | awk -F"/" '{print $2}') >> $GITHUB_OUTPUT                            
            - name: Build and push Docker image
              uses: https://github.com/docker/build-push-action@v5
              with:
                context: .
                file: ./Dockerfile
                push: true
                tags: "git.domain.com/${{ secrets.DOCKER_USERNAME }}/${{steps.meta.outputs.REPO_NAME}}:latest"
     ```
   - Simpan file `runner.yaml` tadi.
   - Kemudian buat file bernama `Dockerfile` tanpa ekstensi di folder `/myvue/`, Copy dan Paste kode dibawah ini:

     ```Dockerfile
     # build stage
      FROM node:lts-alpine as build-stage
      WORKDIR /app
      COPY package*.json ./
      RUN npm install
      COPY . .
      RUN npm run build
      
      # production stage
      FROM nginx:stable-alpine as production-stage
      COPY --from=build-stage /app/dist /usr/share/nginx/html
      EXPOSE 80
      CMD ["nginx", "-g", "daemon off;"]
     ```
   - Simpan file `Dockerfile` tadi.
   - Sebelum menjalankan push ke repository anda silahkan pergi ke Setting repository anda<br> `git.domain.com/{nama-user}/{nama-repository}/settings/actions/secrets` dan setting variable secrets untuk mengisi environment pada file `runner.yaml` anda, klik `add secret` isi:
     Name | Value |
     --- | --- |
     DOCKER_USERNAME| Username_Login_Gitea |
     DOCKER_PASSWORD| Password_Login_Gitea |
   - Jika sudah disetting silahkan push repository anda.
   - Kemudian Silahkan cek halaman `git.domain.com/{nama-user}/{nama-repository}/actions` maka akan tampil halaman `act_runner` berjalan membuild `Docker Container`.
   - Jika Gitea Actions telah berhasil silahkan lihat ke halaman `git.domain.com/{nama-user}/-/packages`.
   - Klik package yang sudah di deploy tadi kemudian copy setelah text  `docker pull` menjadi contoh `git.domain.com/human/reponame:latest`.
   - Login ke [dockge](https://github.com/odysahe/notes/blob/main/tutor/giteaXdocker.md#install-docker-manager) yang sudah di buat di awal, paste pada kolom `run docker` tambahkan text yang di copy tadi menjadi<br> `docker run -d --name mycontainer --publish 8088:80 git.domain.com/human/reponame:latest` lalu klik `change to stack` lalu klik `deploy`
   - Jika proses deploy sudah berhasil silahkan cek ke browser dengan alamat `IP:8088` atau `localhost:8088`, Jika membuat nama domain yang mengarah ke aplikasi VueJs yang anda buat tadi ikuti langkah [Uji Coba Domain](https://github.com/odysahe/notes/blob/main/tutor/giteaXdocker.md#ujicoba-domain) diatas.
   - Done.
## Kustomisasi Gitea Web Dan Logo

> [!NOTE]
> Kustomisasi ini ditujukan untuk Gitea yang terinstall di docker sebagai container.<br>
> Jika anda menginstall tanpa docker lokasi foldernya ada di `/var/lib/gitea/custom` silahkan baca dokumentasi lengkap gitea tentang [Customizing Gitea](https://docs.gitea.com/administration/customizing-gitea)

- Langkah pertama login ssh ke server anda.
- Install text editor `nano` di container gitea:

  ```bash
  # Masuk bash container gitea sebagai root
  docker exec -it gitea bash

  # Update repo
  apk update

  # Install Nano
  apk add nano

  # Keluar dari user root
  exit
  ```
- Masuk ke `bash` Container gitea sebagai user `git` dengan cara:

  ```bash
  docker exec -u git -it gitea bash
  ```
- Jika konfigurasi `volume` pada saat installasi gitea tidak diubah maka seharusnya direktori yang digunakan adalah `data/gitea` silahkan arahkan terminal ke direktori `data/gitea`:
  
  ```bash
  cd data/gitea
  ```
- Buat direktori baru untuk gambar dan template website gitea yang akan digunakan untuk kustomisasi halaman gitea web:

  ```bash
  mkdir public && mkdir public/assets && mkdir public/assets/img && mkdir templates && cd public/assets/img
  ```
- Pastikan anda sudah berada di dalam direktori `data/gitea/public/assets/img` dan download gambar untuk dijadikan logo, di sini saya memakai contoh logo [Github](https://github.com/fluidicon.png):

  ```bash
  wget https://github.com/fluidicon.png
  ```
- Jika sudah di download silahkan coba akses gambar tadi dari web gitea anda dengan cara buka url `https://git.domain.com/assets/img/fluidicon.png`, Maka akan terlihat logo yang sudah di download tadi.
- Untuk mengubah logo pada landing page web gitea, anda harus mengunduh file template yang sudah disediakan oleh gitea melalui [link ini](https://github.com/go-gitea/gitea/tree/main/templates).
- Klik pada file [home.tmpl](https://github.com/go-gitea/gitea/blob/main/templates/home.tmpl), lalu Klik tombol `raw` di pojok kanan atas pada halaman editor github dan setelah terbuka copy urlnya gunakan `wget` untuk mendownload file template tadi.
- Sebelum mengunduh pastikan anda sudah berada dalam directori `data/gitea/templates` lalu jalankan perintah berikut:

  ```bash
  wget https://raw.githubusercontent.com/go-gitea/gitea/main/templates/home.tmpl
  ```
- Edit file `home.tmpl` yang sudah di download tadi:

  ```bash
  nano home.tmpl
  ```
- Cari tag `<img>` dan ganti nama file image dengan yang anda download tadi contoh `fluidicon.png` menjadi :

  ```html
  <img class="logo" width="220" height="220" src="{{AssetUrlPrefix}}/img/fluidicon.png" alt="{{ctx.Locale.Tr "logo"}}">
  ```
- Jika sudah simpan dan restart container gitea :

  ```bash
  docker container restart gitea
  ```
- Kemudian reload web gitea, Done gambar sudah berubah.
>[!NOTE]
> Untuk merubah halaman lain atau navbar dsb. sesuaikan dengan struktur folder yang ada di [github gitea templates](https://github.com/go-gitea/gitea/tree/main/templates).<br>
> file template untuk icon navbar adalah `base/head_navbar.tmpl`<br>
> file template untuk tab icon adalah `base/head.tmpl`

Selain semua installasi diatas anda juga dapat menambahkan `autoupdate container` untuk membuat container selalu update jika ada perubahan pada file container yang ada di gitea, dan juga `autobackuper`.<br><br>
***Sekian Terimakasih***
# Selesai
