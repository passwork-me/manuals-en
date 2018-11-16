# How to install password manager on Ubuntu 16.04

**1. Get root privileges and reload local package database.**

```
sudo -i 
apt-get update
```


Change server hostname to "passwork".

```
hostnamectl set-hostname passwork
```


Automatic local network configuration and discovery.
```
apt-get install -y avahi-daemon libnss-mdns
```


Change `AVAHI_DAEMON_DETECT_LOCAL` from 1 to 0.

```
nano /etc/default/avahi-daemon
```


Set:

```
AVAHI_DAEMON_DETECT_LOCAL = 0
```

Restart:

```
service avahi-daemon restart
```


**2. Install Git and Apache2.**

```
apt-get install -y git apache2
```


**3. Install MongoDB.**

Import the public key used by the package management system.

```
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5
```


Create a /etc/apt/sources.list.d/mongodb-org-3.6.list file for MongoDB.

```
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.6 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-3.6.list
```


Reload local package database.

```
apt-get update
```


Install the latest stable version of MongoDB.

```
apt-get install -y mongodb-org
```


Start MongoDB Service.

```
service mongod start
```


Enable mongod service start on the system boot.

```
systemctl enable mongod.service
```


**4. Install PHP7.**

**Add the PPA.**

```
apt-get install python-software-properties
add-apt-repository ppa:ondrej/php
```


**Install PHP and additional extensions.**

```
apt-get update
apt-get install -y php7.0 php7.0-json php7.0-mcrypt php7.0-dev php7.0-ldap php7.0-xml php7.0-bcmath php7.0-mbstring
```


**5. Install PHP Mongo driver.**

```
apt-get install -y pkg-config
pecl install mongodb

echo "extension=mongodb.so" | tee /etc/php/7.0/apache2/conf.d/20-mongodb.ini
```


**6. Install Phalcon PHP framework.**

```
git clone --depth=1 "git://github.com/phalcon/cphalcon.git"
cd cphalcon/build
./install
echo "extension=phalcon.so" | tee /etc/php/7.0/apache2/conf.d/20-phalcon.ini
service apache2 restart
```


**7. Download and install Passwork.**

Clone the repository using your login and password.

```
cd /var/www
git init
git remote add origin https://passwork.download/passwork/passwork.git
git pull origin master
```


Enter the username and password to get access to the repository.

Create config file and set up permissions for the files.

```
cp /var/www/app/config/config.example.ini /var/www/app/config/config.ini
find /var/www/ -type d -exec chmod 755 {} \;
find /var/www/ -type f -exec chmod 644 {} \;
chown -R www-data:www-data /var/www/
```


Restore a MongoDB database.

```
mongorestore /var/www/dump/
```


**Configure your Apache2.**

Open Apache’s configuration file.

```
nano /etc/apache2/sites-enabled/000-default.conf
```


Make it look like this:

```
<VirtualHost *:80>
    #ServerName .....
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/public
    <Directory /var/www/public>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```


Enable rewrite module and restart Apache.

```
a2enmod rewrite
service apache2 restart
```


**License installation.**

Extract archive with registration keys and move "demo.openssl.lic" and "reginfo.php" to "/var/www/app/keys/" directory.


**Done.**

Open [http://passwork.local](http://passwork.local) to access website.


**Use default account to sign in:**

login: `admin@passwork.me`

pass: `DemoDemo`


**8. Create a SSL certificate.**

First, enable the Apache SSL module.

```
a2enmod ssl
```


Activate the default Apache website.

```
a2ensite default-ssl
```


Restart Apache to put these changes into effect.

```
service apache2 restart
```


Create a new directory where we can store the private key and certificate.

```
mkdir /etc/apache2/ssl
```


Generate a new certificate and a private key to protect it.

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```


Invoking this command will result in a series of prompts.

* Common Name: Specify your server's IP address or hostname. This field matters, since your certificate needs to match the domain (or IP address) for your website.

* Fill out all other fields at your own discretion.

Example answers are shown below.

```
Interactive
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
——
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Example
Locality Name (eg, city) []:Example
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Example Inc
Organizational Unit Name (eg, section) []:Example Dept
Common Name (e.g. server FQDN or YOUR name) []:passwork.local
Email Address []:test@passwork.local
```


Set the file permissions to protect your private key and certificate.

```
chmod 600 /etc/apache2/ssl/*
```


Your certificate and the private key that protects it are now ready for Apache to use.


**9. Configure Apache to use SSL.**

Open the server configuration.

```
nano /etc/apache2/sites-enabled/default-ssl.conf
```


Locate the section that begins with `<VirtualHost _default_:443>` and make the following changes.

* Add a line with your server name directy below the `ServerAdmin` email line. This can be your domain name or IP address:

```
ServerAdmin webmaster@localhost
ServerName passwork.local:443
```


* Add “Directory” directive next to the “ServerName”.

```
    <Directory /var/www/public>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
```


* Find the following two lines, and update the paths to match the locations of the certificate and key we generated earlier. If you purchased a certificate or generated your certificate elsewhere, make sure the paths here match the actual locations of your certificate and key:

```
SSLCertificateFile /etc/apache2/ssl/apache.crt
SSLCertificateKeyFile /etc/apache2/ssl/apache.key
```


Once these changes have been made, check that your virtual host configuration file matches the following.

```
<IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerAdmin webmaster@localhost
        ServerName passwork.local:443
        DocumentRoot /var/www/public
        <Directory /var/www/public>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Order allow,deny
            allow from all
        </Directory>
        SSLEngine on
        SSLCertificateFile /etc/apache2/ssl/apache.crt
        SSLCertificateKeyFile /etc/apache2/ssl/apache.key
    </VirtualHost>
</IfModule>
```


Restart Apache to apply the changes.

```
service apache2 restart
```


Check SSL connection by going to [https://passwork.local](https://passwork.local).


**10. Installing Postfix.**

Install Postfix with the following command:

```
apt-get install -y postfix
```


During the installation, a prompt will appear asking for your General type of mail configuration. Select Internet Site.

![alt text](./images/Ubuntu16.04_01.png)

Enter the fully qualified name of your domain, passwork.

![alt text](./images/Ubuntu16.04_02.png)

Once the installation is finished, open the /etc/postfix/main.cf.

```
nano /etc/postfix/main.cf
```


Make sure that the myhostname parameter is configured with your server’s FQDN:

```
myhostname = passwork
```


**Configuring SMTP usernames and passwords.**

Open or create the /etc/postfix/sasl_passwd.

```
nano /etc/postfix/sasl_passwd
```


Add your destination (SMTP Host), username, and password in the following format:

```
[mail.isp.example] username:password
```


If you want to specify a non-default TCP Port (such as 587), then use the following format:

```
[mail.isp.example]:587 username:password
```


**for Gmail it is:**
```
[smtp.gmail.com]:587 username:password
```


Create the hash db file for Postfix by running the postmap command:

```
postmap /etc/postfix/sasl_passwd
```


If all went well, you should have a new file named sasl_passwd.db in the /etc/postfix/ directory.


**Securing your password and hash database files.**

The /etc/postfix/sasl_passwd and the /etc/postfix/sasl_passwd.db files created in the previous steps contain your SMTP credentials in plain text.
For security reasons, you should change their permissions so that only the root user can read or write to the file. 

Run the following commands to change the ownership to root and update the permissions for the two files:

```
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```


**Configuring the relay server.**

Open the /etc/postfix/main.cf.

```
nano /etc/postfix/main.cf
```


Update the relayhost parameter to show your external SMTP relay host. Important: If you specified a non-default TCP port in the sasl_passwd file, then you must use the same port when configuring the relayhost parameter.

specify SMTP relay host:

```
relayhost = [mail.isp.example]:587
```


**for Gmail it is:**

```
relayhost = [smtp.gmail.com]:587
```


At the end of the file, add the following parameters to enable authentication:
```
# enable SASL authentication
smtp_sasl_auth_enable = yes
# disallow methods that allow anonymous authentication.
smtp_sasl_security_options = noanonymous
# where to find sasl_passwd
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
# Enable STARTTLS encryption
smtp_use_tls = yes
# where to find CA certificates
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```


Save your changes.

Restart Postfix:

```
service postfix restart
```