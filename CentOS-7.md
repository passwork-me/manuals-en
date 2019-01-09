# How to install Passwork on CentOS 7

**1. Get root privileges and reload local package database.**

```
su
cd ~
yum makecache
```


Change server hostname to "passwork".

```
hostnamectl set-hostname passwork
/etc/init.d/network restart
```


**2. Install Git and Apache2, add firewall rules.**

```
yum -y install git httpd avahi
systemctl start httpd
systemctl enable httpd
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=5353/udp
firewall-cmd --reload
systemctl restart avahi-daemon
```


**3. Install MongoDB.**

Configure the package management system (yum).

Create a /etc/yum.repos.d/mongodb-org-3.6.repo file so that you can install MongoDB directly, using yum.

```
yum -y install nano
nano /etc/yum.repos.d/mongodb-org-3.6.repo
```


Make it look like this:

```
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```


To install the latest stable version of MongoDB, issue the following command:

```
yum -y install mongodb-org
```


Set SELinux to disabled mode in /etc/selinux/config by setting the `SELINUX` setting to disabled.

```
nano /etc/selinux/config
```


Set:

```
SELINUX=disabled
```


You must reboot the system for the changes to take effect.

Start MongoDB Service.

```
su
cd ~
service mongod start
```


Enable mongod service start on the system boot.

```
systemctl enable mongod.service
```


**4. Install PHP7.**

**Install the Remi repository configuration package.**

```
yum -y install wget yum-utils
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum-config-manager --enable remi-php70
```


**Install PHP and additional extensions.**

```
yum -y install php php-json php-mcrypt php-ldap php-xml php-bcmath php-mbstring
```


**5. Install PHP Mongo driver.**

```
yum -y install gcc php-pear php-devel openssl-devel
pecl install mongodb
echo "extension=mongodb.so" | tee /etc/php.d/20-mongodb.ini
systemctl restart httpd
```


**6. Install Phalcon PHP framework.**

```
yum -y install php-mysql libtool pcre-devel
git clone --branch 3.4.x  --depth=1 "git://github.com/phalcon/cphalcon.git"
cd cphalcon/build
./install
echo "extension=phalcon.so" | tee /etc/php.d/50-phalcon.ini
systemctl restart httpd
```


**7. Download and install Passwork.**

Clone the repository using your login and password.

```
cd /var/www
git init
git remote add origin http://passwork.download/passwork/passwork.git
git fetch
git checkout php7
```


Enter the username and password to get access to the repository.

Create config file and set up permissions for the files.

```
cp /var/www/app/config/config.example.ini /var/www/app/config/config.ini
find /var/www/ -type d -exec chmod 755 {} \;
find /var/www/ -type f -exec chmod 644 {} \;
chown -R apache:apache /var/www/
```


Restore a MongoDB database.

```
mongorestore /var/www/dump/
```


**Configure your Apache2.**

Create non-ssl configuration file.

```
nano /etc/httpd/conf.d/non-ssl.conf
```


change directives accordingly the entries below.

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/public
    <Directory /var/www/public>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
    ErrorLog logs/error_log
    TransferLog logs/access_log
    LogLevel warn
</VirtualHost>
```


Restart Apache.

```
systemctl restart httpd
```


**License installation.**

Extract archive with registration keys and move `.lic` and `reginfo.json` to "/var/www/app/keys/" directory.


**Done.**

Open [http://passwork.local](http://passwork.local) or [http://127.0.0.1](http://127.0.0.1) to access website.


**Use default account to sign in:**

Login: `admin@passwork.me`

Password: `DemoDemo`


**8. Create a SSL certificate.**

First, install the Apache SSL module.

```
yum -y install mod_ssl
```


Create a new directory to store our private key (the /etc/ssl/certs directory is already available to hold our certificate file):

```
mkdir /etc/ssl/private
```


Set correct permissions:

```
chmod 700 /etc/ssl/private
```


Generate a new certificate and a private key to protect it.

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
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


Create a strong Diffie-Hellman group.

```
openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```


Append the generated file to the end of our self-signed certificate.

```
cat /etc/ssl/certs/dhparam.pem | tee -a /etc/ssl/certs/apache-selfsigned.crt
```


**9. Configure Apache to use SSL.**

Open Apache's SSL configuration file in your text editor.

```
nano /etc/httpd/conf.d/ssl.conf
```


Locate the section that begins with `<VirtualHost _default_:443>` and make the following changes.

* First, uncomment the `DocumentRoot` line and edit the address in quotes to the location of your site's document root. By default, this will be in /var/www/html.

* Next, uncomment the `ServerName` line and replace `www.example.com` with your domain name or server IP address (whichever one you put as the common name in your certificate):

```
DocumentRoot /var/www/public
ServerName passwork.local:443
```


* Add “Directory” directive next to the “ServerName”.

```
    <Directory /var/www/public>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        Allow from all
		Require all granted
    </Directory>
```


* Next, find the SSLProtocol and SSLCipherSuite lines and either delete them or comment them out. The configuration we be pasting in a moment will offer more secure settings than the default included with CentOS's Apache:

```
# SSLProtocol all -SSLv2
# SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5:!SEED:!IDEA
```


Find the SSLCertificateFile and SSLCertificateKeyFile lines and change them to the directory we made at /etc/httpd/ssl:

```
SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
```


Once these changes have been made, check that your virtual host configuration file matches the following.

```
<VirtualHost _default_:443>
    DocumentRoot /var/www/public
    ServerName passwork.local:443
    <Directory /var/www/public>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        Allow from all
		Require all granted
    </Directory>
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>
```


We're now done with the changes within the actual VirtualHost block. Save changes (Ctr+O) and exit (Ctr+X).

Restart Apache to apply the changes.

```
systemctl restart httpd
```


Check SSL connection by going to [https://passwork.local](https://passwork.local).


**10. Installing Postfix.**

Install Postfix with the following command:

```
yum -y install postfix cyrus-sasl-plain mailx
systemctl restart postfix
```


Set Postfix to start on boot.

```
systemctl enable postfix
```


Open the /etc/postfix/main.cf.

```
nano /etc/postfix/main.cf
```


and add the following lines to the end of the file.

```
myhostname = passwork
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
```


Save the main.cf file and close the editor.


**Configure Postfix SASL credentials.**

The Gmail credentials must now be added for authentication. Create a /etc/postfix/sasl_passwd file.

``` 
nano /etc/postfix/sasl_passwd
```


and add following line:

```
[smtp.gmail.com]:587 username:password
```


The username and password values must be replaced with valid Gmail credentials. The sasl_passwd file can now be saved and closed.

A Postfix lookup table must now be generated from the sasl_passwd text file by running the following command.

```
postmap /etc/postfix/sasl_passwd
```


Access to the sasl_passwd files should be restricted.

```
chown root:postfix /etc/postfix/sasl_passwd*
chmod 640 /etc/postfix/sasl_passwd*
```


Lastly, reload the Postfix configuration.

```
systemctl reload postfix
```
