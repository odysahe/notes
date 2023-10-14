# My Exprience about PHP
## Change php version Method 1
- `sudo a2dismod php7.2`
- `sudo a2enmod php5.6`
- `sudo update-alternatives --set php /usr/bin/php5.6`
## Change php version Method 2
- `sudo update-alternatives --config php`
- `sudo update-alternatives --set phar /usr/bin/phar5.6 (change extention)`
- `sudo sysmtctl restart apache2 nginx` > what do you use
- `php -v`
## Change php version Method 3
ketika sudah update alternative tapi `phpinfo();` tetap menampilkan versi php yang lain jalankan perintah berikut:
``` 
- untuk disable
$ sudo a2disconf php8.0-fpm
- untuk enable
$ sudo a2enconf php7.4-fpm
```
jika `php*.*-fpm` tidak ditemukan/error maka install php fpm terlebih dahulu (pastikan repository [ondrej php](https://launchpad.net/~ondrej/+archive/ubuntu/php/)  sudah ditambahkan):

> [!WARNING]
> Sesuaikan versi php dengan kebutuhan

`sudo apt install php8.0-fpm php8.0-common php8.0-mysql php8.0-xml php8.0-xmlrpc php8.0-curl php8.0-gd php8.0-imagick php8.0-cli php8.0-dev php8.0-imap php8.0-mbstring php8.0-soap php8.0-zip php8.0-bcmath php8.0-apcu -y `