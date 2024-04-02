### Deploying wordpress

## create an instance
- Follow the "Getting Started" instructions [here](https://docs.leaf.cloud/en/latest/) to create an Ubuntu instance, keypair, security group, and to connect to your instance using SSH. 
- Create an instance that fits your website's resource demand. For this example, we will use a small instance (ec1.small with 20GB). Depending on the number of concurrent visitors, this is sufficient for most small/medium websites.
- For your Security groups, the ports used by WordPress are 80 & 443 (http & https)
- Create a DNS A record to point your domain to your assigned IP, this might take some time and is needed later for SSL/HTTPS

## setting up requirements
- update your instance
```
sudo apt update &&
sudo apt -y upgrade && 
sudo apt -y dist-upgrade &&
sudo apt -y autoremove &&
sudo apt -y autoclean &&
sudo snap refresh
```

- Download requirements
```
sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```
* reboot your instance (in case of kernel changes)
`sudo reboot`
* ssh back into your instance

## Create an SQL database
* set a root password for mysql
```
sudo mysql
```
replace "my_supersecure_pwd" with your preferred pasword
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'my_supersecure_pwd';
```
* exit mysql: 
```
exit;
```
* run the secure installation procedure:
```
sudo mysql_secure_installation
```
* enter the root password you set before (excluding the '')
* Choose wether you want the 'VALIDATE PASSWORD COMPONENT'
this is adviced, however for this tutorial we choose not to enable it
* Choose whether you want to keep or change the root password
* Choose 'Y' to remove the anonymous user
* Since you are connecting over SSH choose 'N' for disallowing remote root login
* Choose 'Y' for removing the test database
* And a final 'Y' to reload the privilege tables
* login to mysql:
```
mysql -u root -p
```
* create a database for your site, replacing 'MY_WEBSITE' with your preferred database name
```
CREATE DATABASE MY_WEBSITE DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```
* create a separate mysql user & user password by replacing "MY_USER" and "MY_PWD"
```
CREATE USER 'MY_USER'@'%' IDENTIFIED WITH mysql_native_password BY 'MY_PWD';
```
* Grant your new user access to your database, make sure you replace the fields
```
GRANT ALL ON MY_WEBSITE.* TO 'MY_USER'@'%';
```
* apply privileges and exit mysql:
```
FLUSH PRIVILEGES;

exit;
```

## Configure apache2
* enable your site in apache2
```
a2ensite 000-default
```
enable the rewrite module
```
sudo a2enmod rewrite
```
restart apache2
```
sudo systemctl restart apache2
```
## download wordpress
* go to your temporary folder 
```
cd /tmp
```
* download wordpress
```
wget https://wordpress.org/latest.tar.gz
```
* extract the package
```
tar xzvf latest.tar.gz
```
* create some files and folders to avoid permission issues later
```
touch /tmp/wordpress/.htaccess
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```
```
mkdir /tmp/wordpress/wp-content/upgrade
```
* copy the wordpress folder to your webroot
```
sudo cp -a /tmp/wordpress/. /var/www/html
```
* change ownership of your webroot to the 'www-data' user and group
```
sudo chown -R -v www-data:www-data /var/www/html
```
* use the wordpress keygen to create some secure keys for your config file by opening the following page in your browser:
```
https://api.wordpress.org/secret-key/1.1/salt/
```
* copy these keys & edit the config file
```
sudo nano /var/www/html/wp-config.php
```
* Scroll down until you find the example keys. Remove the examples and copy your newly generated ones in their place.
* Now scroll up until you find the 'Database settings' section and fill in the variables you entered earlier in the mysql section of this tutorial
```
define( 'DB_NAME', 'MY_WEBSITE' );

/** Database username */
define( 'DB_USER', 'MY_USER' );

/** Database password */
define( 'DB_PASSWORD', 'MY_PWD' );
```
* Close the config file with 'ctrl+x' followed by 'y' and 'enter'
* remove the apache2 default index file
```
sudo rm /var/www/html/index.html
```

## finish wordpress installation
* in your browser visit your URL and follow the installation steps

## Get a certificate
* in your SSH session install certbot
```
sudo snap install --classic certbot
```
* make sure certbot can work outside it's snap
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
* get a certificate by running this command and following the steps
```
sudo certbot --apache
```

## Finished!
Your wordpress site is now fully operational and can be reached at
<https://yourdomain>

