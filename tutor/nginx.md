# My Exprience About Nginx
## Configuration Laravel With nginx 
```
server {
    listen 443 ssl default_server;

    root /var/www/laravel/public/;
    index index.php;
    ssl_certificate /path/to/cert;
    ssl_certificate_key /path/to/key;

    location / {
         try_files $uri $uri/ /index.php$is_args$args; # IMPORTANT
    }

    # pass the PHP scripts to FastCGI server listening on /var/run/php-fpm.sock
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm.sock;
        fastcgi_index index.php; # NOT THAT IMPORTANT
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; # NOT THAT IMPORTANT
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_split_path_info ^(.+?\.php)(/.+)$; # NOT THAT IMPORTANT
        include fastcgi_params; # NOT THAT IMPORTANT
    }
}
```
## Hide Header nginx 
From this [Source](https://github.com/openresty/headers-more-nginx-module#installation) not working, but i try install `apt-get install libnginx-mod-http-headers-more-filter` everything is work
