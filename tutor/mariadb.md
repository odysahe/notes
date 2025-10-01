# My Exprience Mariadb
## Installations
- `sudo apt update && sudo apt upgrade -y && sudo apt install mariadb-server mariadb-client`
## Secure Installations
- `sudo mysql_secure_installation`
- Change the password root if you need.
## Using REMOTE DATABASE
- `sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf`
- find `bind-address` and change to `0.0.0.0` or your spesial ip
- and last save it.
## Create New User (recommend)
```
  CREATE USER 'dbuser'@'%' IDENTIFIED BY 'passwordku';
  GRANT ALL PRIVILEGES ON *.* TO 'dbuser'@'%' WITH GRANT OPTION;
  FLUSH PRIVILEGES;
```
- And test remote your database. enjoy
