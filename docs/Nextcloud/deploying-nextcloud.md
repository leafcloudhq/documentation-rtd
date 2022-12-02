# Deploying Nextcloud

This tutorial wil guide you through the steps to deploy a Nextcloud server.

## Create the instance 

**Note: you can find tutorials on how to set up an instance [here](https://docs.leaf.cloud/en/latest/Getting-Started/Launching-an-instance/).** 

Create an Ubuntu 22.04 instance with the following settings: 

- volume: 70GB minimum if you want to use S3 compatible storage 
- Network: external 
- Flavor: ec1.large recommended 
- Security Group: ports 80 and 443 (http & https)

* create a dns record for the ip assigned to your instance, later on you will need a domain name in the configuration

## Installing dependencies
* SSH into the instance using 'ubuntu@<your.domain>` and the key pair selected when creating the image.
* Update the instance: 
```
sudo apt update && 
sudo apt -y upgrade && 
sudo apt -y dist-upgrade && 
sudo apt -y autoremove && 
sudo apt -y autoclean
``` 
* Install PHP & dependencies 
```
sudo apt-get install -y php8.1 php8.1-cli php8.1-common php8.1-imap php8.1-redis php8.1-snmp php8.1-xml php8.1-zip php8.1-mbstring php8.1-curl php8.1-gd php8.1-mysql unzip mysql-server snapd apache2 &&
sudo snap install core && 
sudo snap refresh core &&
sudo snap install --classic certbot
```
* Enable and start the apache2 service `sudo systemctl enable apache2 && sudo systemctl start apache2`
* start the mysql service 
`sudo systemctl start mysql.service`
* Reboot the server and SSH back in
## Create a SQL database
* open mysql 
`sudo mysql`
* enter the following command and replace <password> with your desired root password:
`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<password>';`
* exit mysql 
`exit;`
* start the secure installation
`sudo mysql_secure_installation`
* follow the steps of the secure installation  
* reopen mysql as the root user
`sudo mysql -u root -p`
* create a database for nextcloud
`create database nextcloud;`
* create a user
`CREATE USER 'nextcloud'@'localhost' IDENTIFIED WITH mysql_native_password BY '<password>';`
replace <password> with your desired password
* grant privileges on the database to the nextcloud user
`GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' WITH GRANT OPTION;`
* Apply the changes
`flush privileges;`
* exit mysql
`exit;`

## Install Nextcloud
* move to the following directory 
`cd /var/www`
* remove the html folder
`sudo rm -R -v html`
* download nextcloud
`sudo wget https://download.nextcloud.com/server/releases/latest.zip`
* unzip and remove the zip file
`sudo unzip latest.zip && sudo rm latest.zip`
* rename the 'nextcloud' folder to 'html'
`sudo mv nextcloud html` 
* change ownership to the folder for the right permission
`sudo chown -R www-data:www-data /var/www/html/`


* create an apache virtual host file
`sudo nano /etc/apache2/sites-available/nextcloud.conf`
* add the following to the file, replacing <your_domain> with the DNS record made earlier:
`<VirtualHost *:80>`
`ServerName <your_domain>`
`DocumentRoot /var/www/html/`

`<Directory /var/www/html/>`
 `Require all granted`
 `Options FollowSymlinks MultiViews`
 `AllowOverride All`
 `<IfModule mod_dav.c>`
 `Dav off`
 `</IfModule>`
`</Directory>`

`ErrorLog /var/log/apache2/<your_domain>.error_log`
`CustomLog /var/log/apache2/<your_domain>.access_log common`
`</VirtualHost>`
* edit the apache config
`sudo nano /etc/apache2/apache2.conf`
* below the line `#global configuration` add 'ServerName' followed by your domain
`ServerName <your_domain>`
* enable the config 
`sudo a2ensite nextcloud.conf && sudo a2enmod rewrite`
* Open your apache2 php configuration
`sudo nano /etc/php/8.1/apache2/php.ini`
    * Make the following changes to the file
        * memory limit: set this to a value that represent the flavor you chose in the first step (in this case 6G for the 8G available on a `ec1.large` flavor.
        * post max size: Maximum size of POST data that PHP will accept, 16G is fine in most cases.
    ```
    memory_limit = 6G
    post_max_size = 16G
    ``` 
* reload and restart apache
`sudo systemctl reload apache2 && sudo systemctl restart apache2`
* Check if apache2 is running
`apachectl -t`
the output should be 'Syntax OK'

## Configuring Nextcloud
* Visit your url in your browser
* Create an admin account by picking a username and password
* configure the database you made earlier in mysql
   * both user and database name are nextcloud 
   * the password is the user password, not the mysql root password
![image.png](/.attachments/image-41618fda-0410-43e7-a2c4-c0b77084eba0.png)
* hit 'install'

## Getting a SSL certificate

* link directories to make certbot work
`sudo ln -s /snap/bin/certbot /usr/bin/certbot`
* run certbot for apache and follow the steps on screen
`sudo certbot --apache`
* check if the SSL certificate is valid in your browser by hitting refresh and clicking the lock icon next to the URL.
  You might have to clear your browsers cache if you don't see the certificate straight away.
* You now have a working Nextcloud instance

**Note: If you want to implement Object Stoarge for your Nextcloud server, please following the 'Implementing S3 as primary storage' tutorial BEFORE you use start using your Nextcloud instance.**