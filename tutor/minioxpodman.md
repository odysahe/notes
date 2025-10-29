# My Exprience Minio X Podman
## Installations
- `sudo apt update && sudo apt upgrade -y && sudo apt update && sudo apt install -y podman`
## Make Directory
```
sudo mkdir -p /opt/minio/data
sudo mkdir -p /opt/minio/config
sudo chown -R $USER:$USER /opt/minio
```
make directory for file `compose.yaml`
```
mkdir -p ~/minio-oss/data
cd ~/minio-oss
nano compose.yaml
```
and paste this:
```
version: "3.9"

services:
  minio:
    image: quay.io/minio/minio
    container_name: minio
    restart: always
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin123
    volumes:
      - ./data:/data
      - ./config:/root/.minio
    command: server /data --console-address ":9001"
```
change the `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD`. <br>
If you try running `podman-compose up -d` it will run perfectly, but when you end the ssh session you might notice that the minio server will shut down. To solve this problem, I created `systemd` to automatically run minio.<br>
Make new file in `systemd`:
```
sudo nano /etc/systemd/system/minio-oss.service
```
And paste this:
```
[Unit]
Description=MinIO (via Podman Compose)
Requires=network-online.target
After=network-online.target

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/ubuntu/minio-oss
ExecStart=/usr/bin/podman-compose up -d
ExecStop=/usr/bin/podman-compose down
Restart=on-failure
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target
```
Reload and Activate service
```
sudo systemctl daemon-reload
sudo systemctl enable --now minio-oss
sudo systemctl status minio-oss
```
Enjoy running!
( Optional ) If running in productions add this:
```
StandardOutput=append:/var/log/minio-compose.log
StandardError=append:/var/log/minio-compose.log
```
## Setting in Laravel 12
- Install package laravel s3:
  ```
  composer require league/flysystem-aws-s3-v3
  ```
  and edit `.env` file like this:
  ```
  FILESYSTEM_DISK=s3
  ....
  AWS_ACCESS_KEY_ID=minioadmin
  AWS_SECRET_ACCESS_KEY=minioadmin123
  AWS_DEFAULT_REGION=us-east-1
  AWS_BUCKET=your-bucket-name
  AWS_USE_PATH_STYLE_ENDPOINT=true
  AWS_ENDPOINT=https://to.domain.api
  ```
- And test enjoy
