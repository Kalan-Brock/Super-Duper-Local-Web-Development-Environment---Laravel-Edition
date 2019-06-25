# -*- mode: ruby -*-
# vi: set ft=ruby :

@script = <<SCRIPT
# Fix for https://bugs.launchpad.net/ubuntu/+source/livecd-rootfs/+bug/1561250
if ! grep -q "ubuntu-xenial" /etc/hosts; then
    echo "127.0.0.1 ubuntu-xenial" >> /etc/hosts
fi

echo -e "\n--- $(tput -T xterm setaf 1)One Super Duper Local Web Development Environment coming right up!$(tput -T xterm sgr0) ---\n"
sudo timedatectl set-ntp on > /dev/null 2>&1
apt-get update > /dev/null 2>&1

echo -e "--- Installing base packages, Vim, Curl, and Git ---"
apt-get -y install vim curl build-essential python-software-properties git > /dev/null 2>&1

echo -e "\n--- Adding repositories to update our box ---"
add-apt-repository ppa:ondrej/php > /dev/null 2>&1

#mariadb repo
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8 > /dev/null 2>&1
sudo add-apt-repository "deb [arch=amd64,arm64,i386,ppc64el] http://nyc2.mirrors.digitalocean.com/mariadb/repo/10.3/ubuntu xenial main"  > /dev/null 2>&1

apt-get -qq update

echo -e "\n--- Installing Apache and PHP  ---"
apt-get install -y apache2 php7.2 php7.2-bcmath php7.2-bz2 php7.2-cli php7.2-curl php7.2-mysql php7.2-intl php7.2-json php7.2-mbstring php7.2-opcache php7.2-soap php7.2-sqlite3 php7.2-xml php7.2-xsl php7.2-zip libapache2-mod-php7.2 memcached php-memcached > /dev/null 2>&1
rm -rf /var/www/html/
echo -e "\n--- Configuring PHP for Development Environment ---"
echo 'echo "error_reporting = E_ALL" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s
echo 'echo "display_errors = On" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s
echo 'echo "upload_max_filesize = 64M" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s
echo 'echo "post_max_size = 10M" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s
echo 'echo "max_execution_time = 700" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s
echo 'echo "always_populate_raw_post_data = -1" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s
mkdir /var/www/public > /dev/null 2>&1

echo -e "\n--- Configuring Apache for both http and https ---"
echo "<VirtualHost *:80>
    DocumentRoot /var/www/public/public
    AllowEncodedSlashes On

    <Directory /var/www/public>
        Options +Indexes +FollowSymLinks
        DirectoryIndex index.php index.html
        Order allow,deny
        Allow from all
        AllowOverride All
    </Directory>

    ErrorLog /var/www/error.log
    CustomLog /var/www/access.log combined
</VirtualHost>" > /etc/apache2/sites-available/000-default.conf
a2enmod rewrite > /dev/null 2>&1

openssl genrsa -out /etc/ssl/private/apache-selfsigned.key 1024 > /dev/null 2>&1
touch openssl.cnf
cat >> openssl.cnf <<EOF
[ req ]
prompt = no
distinguished_name = req_distinguished_name

[ req_distinguished_name ]
C = US
ST = Chicago
L = Illinois
O = Me Incorporated
OU = Web Development
CN = 192.168.33.10
emailAddress = noreply@me.com
EOF

openssl req -config openssl.cnf -new -key /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.csr > /dev/null 2>&1
openssl x509 -req -days 1024 -in /etc/ssl/certs/apache-selfsigned.csr -signkey /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt > /dev/null 2>&1

echo '<IfModule mod_ssl.c>
          <VirtualHost _default_:443>
              ServerAdmin noreply@me.com
              ServerName 192.168.33.10

              DocumentRoot /var/www/public/public

              ErrorLog /var/www/error.log
              CustomLog /var/www/access.log combined

              SSLEngine on
              SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
              SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

              <FilesMatch "\.(cgi|shtml|phtml|php)\$">
                SSLOptions +StdEnvVars
              </FilesMatch>

              <Directory /usr/lib/cgi-bin>
                SSLOptions +StdEnvVars
              </Directory>
          </VirtualHost>
      </IfModule>' > /etc/apache2/sites-available/default-ssl.conf

a2enmod rewrite > /dev/null 2>&1
a2enmod headers > /dev/null 2>&1
a2ensite default-ssl > /dev/null 2>&1
sed -i "s/www-data/vagrant/" /etc/apache2/envvars
sudo rm -rf /var/www/html/
service apache2 restart > /dev/null 2>&1
sed -i "s/-m 64/-m 1024/g" /etc/memcached.conf
service memcached restart

echo -e "\n--- Installing MariaDB ---"
debconf-set-selections <<< 'mariadb-server-10.3 mysql-server/root_password password root'
debconf-set-selections <<< 'mariadb-server-10.3 mysql-server/root_password_again password root'
apt-get install -y mariadb-server mariadb-client  > /dev/null 2>&1
mysql -uroot -proot -e "GRANT ALL PRIVILEGES ON * . * TO 'root'@'localhost' IDENTIFIED by 'root'" > /dev/null 2>&1
mysql -uroot -proot -e "CREATE USER 'root'@'%' IDENTIFIED BY 'root'" > /dev/null 2>&1
mysql -uroot -proot -e "GRANT ALL PRIVILEGES ON * . * to 'root'@'%' identified by 'root'" > /dev/null 2>&1
mysql -uroot -proot -e "GRANT GRANT OPTION ON * . * to 'root'@'%'" > /dev/null 2>&1
mysql -uroot -proot -e "FLUSH PRIVILEGES" > /dev/null 2>&1
sed -i 's/^bind-address/#bind-address/' /etc/mysql/my.cnf > /dev/null 2>&1
sed -i 's/^skip-external-locking/#skip-external-locking/' /etc/mysql/my.cnf > /dev/null 2>&1
sudo service mysql restart > /dev/null 2>&1

mysql -u root -proot -e "create database super_duper;" > /dev/null 2>&1

echo -e "\n--- Installing Composer ---"
curl --silent https://getcomposer.org/installer | php > /dev/null 2>&1
mv composer.phar /usr/local/bin/composer

echo -e "\n--- Installing NodeJS and NPM ---"
apt-get -y install nodejs > /dev/null 2>&1
apt-get -y install npm > /dev/null 2>&1
sudo ln -s /usr/bin/nodejs /usr/bin/node

echo -e "\n--- Installing Sass, Gulp, Bower, and Yarn ---"
npm install -g sass gulp bower yarn > /dev/null 2>&1

echo -e "\n--- Installing Redis ---"
sudo apt-get install -y redis-server > /dev/null 2>&1
sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.old
sudo cat /etc/redis/redis.conf.old | grep -v bind > /etc/redis/redis.conf
echo "bind 0.0.0.0" >> /etc/redis/redis.conf
sudo update-rc.d redis-server defaults > /dev/null 2>&1
sudo /etc/init.d/redis-server start > /dev/null 2>&1

echo -e "\n--- Installing MailHog ---"
wget --quiet -O /home/vagrant/mailhog https://github.com/mailhog/MailHog/releases/download/v1.0.0/MailHog_linux_amd64
chmod +x /home/vagrant/mailhog
touch /etc/systemd/system/mailhog.service
echo '
[Unit]
Description=MailHog Service
After=network.service vagrant.mount
[Service]
Type=simple
ExecStart=/usr/bin/env /home/vagrant/mailhog > /dev/null 2>&1 &
[Install]
WantedBy=multi-user.target
' > /etc/systemd/system/mailhog.service

sudo systemctl enable mailhog > /dev/null 2>&1
sudo systemctl start mailhog > /dev/null 2>&1

echo -e "\n--- Updating All The Things... ---"
sudo apt-get update > /dev/null 2>&1
sudo apt-get -y upgrade > /dev/null 2>&1

echo -e "\n--- Setting up bash profile with our custom commands ---\n"

##Backup Script
sudo touch /home/vagrant/backup.sh
cat > /home/vagrant/backup.sh <<- "EOF"
#!/bin/bash

echo "Creating your backup, one moment..."
mkdir /var/www/backups > /dev/null 2>&1
mkdir /var/www/backups/databases > /dev/null 2>&1
mkdir /var/www/backups/files > /dev/null 2>&1
databases=`mysql --user=root -proot -e "SHOW DATABASES;" | grep -Ev "(Database|information_schema|performance_schema)"`

for db in $databases; do
  mysqldump --force --opt --user=root -proot --databases $db | gzip > "/var/www/backups/databases/$db_$(date +%F).gz"
done
tar -czf /var/www/backups/files/files_$(date +%F).gz /var/www/public/ > /dev/null 2>&1
echo "Backup of project databases and files created @ /var/www/backups."
EOF

if ! grep -q "cd /var/www" /home/vagrant/.profile; then
    echo "cd /var/www" >> /home/vagrant/.profile
    echo -e "\nalias projectbackup='bash /home/vagrant/backup.sh'" >> /home/vagrant/.profile
fi

sudo touch /etc/update-motd.d/99-custom-header
echo "#!/bin/sh" > /etc/update-motd.d/99-custom-header
echo '
cat << "EOF"
*** Super Duper Local Web Development Environment (Laravel Edition)
*** https://github.com/Kalan-Brock/Super-Duper-Local-Web-Development-Environment---Laravel-Edition
-------------------------------------------------------------------------------
## To Create a Site File Backup: $ projectbackup
-------------------------------------------------------------------------------
Host Name:  | superduperlaravel
Http:       | http://192.168.33.10
Https:      | https://192.168.33.10
MySQL:      | Port: 3306  User: root  Password: root (SSH not needed)
Redis:      | Port: 6379 (test with redis-cli ping)
MailHog:    | http://192.168.33.10:8025
Error Log:  | /var/www/error.log
Access Log: | /var/www/access.log
user.ini    | /etc/php/7.2/apache2/conf.d/user.ini

EOF' >> /etc/update-motd.d/99-custom-header
sudo chmod +x /etc/update-motd.d/99-custom-header

echo -e "\n--- Grabbing a Fresh Copy of Laravel (prefer-dist) ---\n"
composer create-project --prefer-dist laravel/laravel /var/www/public > /dev/null 2>&1

echo -e "\n--- Linking Laravel Storage (public/storage via artisan) ---\n"
cd /var/www/public
php artisan storage:link > /dev/null 2>&1

echo -e "\n--- Correcting Directory and File Permissions for Laravel (storage and cache) ---\n"
sudo chmod -R ug+rwx /var/www/public/storage /var/www/public/bootstrap/cache

composer require imliam/laravel-env-set-command:^1.0.0 > /dev/null 2>&1

php artisan env:set app_debug true > /dev/null 2>&1
php artisan env:set db_username root > /dev/null 2>&1
php artisan env:set db_password root > /dev/null 2>&1
php artisan env:set db_database super_duper > /dev/null 2>&1
php artisan env:set app_name "Super Duper Web Development Environment" > /dev/null 2>&1
php artisan env:set app_url http://192.168.33.10 > /dev/null 2>&1

echo -e "\n--- Setting Up Laravel Debugbar ---\n"
composer require barryvdh/laravel-debugbar --dev > /dev/null 2>&1
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider" > /dev/null 2>&1
php artisan env:set debugbar_enabled true > /dev/null 2>&1

echo -e "\n\n--- Super Duper!  Visit your box at http://192.168.33.10 or https://192.168.33.10 ---\n\n"

SCRIPT

@cron = <<SCRIPT
echo -e "\n--- Setting Cron For Laravel Task Scheduling ---\n"
touch laravelcron > /dev/null 2>&1
echo "* * * * * cd /var/www/public && php artisan schedule:run >> /dev/null 2>&1" >> laravelcron
crontab laravelcron
rm laravelcron

SCRIPT

Vagrant.configure("2") do |config|
    config.vm.box = 'ubuntu/xenial64'
    config.vm.network "private_network", ip: "192.168.33.10"
    config.vm.hostname = "superduperlaravel"
    config.vm.network "forwarded_port", guest: 3306, host: 3306
    config.vm.network "forwarded_port", guest: 6379, host: 6379
    config.vm.network "forwarded_port", guest: 27017, host: 27017
    config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=777", "fmode=666"]
    config.vm.provision 'shell', inline: @cron, privileged: false
    config.vm.provision 'shell', inline: @script

    config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
    end
end