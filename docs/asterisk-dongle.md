---
tags: 
  - asterisk
  - dongle
---

# On Ubuntu 22 Asterisk 18 FreePBX 16

    FreePBX 16 OS VERSION="22.04.3 LTS (Jammy Jellyfish)"
    Asterisk 18.19.0 FreePBX 16.0.33 

I combined all the required files and sources Except OS in one tar file.
[Descarga aquí](https://drive.google.com/file/d/1hEt13Y6iALR6aRrM83UD497G_E43LQV0/view?usp=sharing)

You will find all of these steps in install.sh file inside the tar file it is recommended to copy paste the commands not to run the script, since it is not prepared for autoinstall Install OS ubuntu 18 and enable root on ssh access then follow the steps

Move all files using winscp to root directory

## Server update packages and add repos then reboot

    curl -o /etc/apt/trusted.gpg.d/mariadb_release_signing_key.asc 'https://mariadb.org/mariadb_release_signing_key.asc'
---
    sh -c "echo 'deb https://ftp.yz.yamagata-u.ac.jp/pub/dbms/mariadb/repo/10.10/ubuntu jammy main' >>/etc/apt/sources.list"
---
    LC_ALL=C.UTF-8add-apt-repositoryppa:ondrej/php

    apt update
    apt upgrade
    reboot
---

### Install packages

    apt-get install unzip git gnupg2 curl libnewt-dev libssl-dev libncurses5-dev subversion libsqlite3-dev build-essential libjansson-dev libxml2-dev uuid-dev subversion apt-transport-https curl sox mpg123 unixodbc unixodbc-dev libmariadb3 libmariadb-dev
    apt-get install apache2
    apt-get install mariadb-server
    apt-get install php7.4 libapache2-mod-php7.4 php7.4-cgi php7.4-common php7.4-curl php7.4-mbstring php7.4-gd php7.4-mysql php7.4-bcmath php7.4-zip php7.4-xml php7.4-imap php7.4-json php7.4-snmp
    apt-get install php-pear
    apt-get install nodejsnpm
    apt-get install python3-certbot-apache

### Asterisk build

    tar zxf asterisk-18-current.tar.gz
    cd asterisk-18.*/
    ./contrib/scripts/get_mp3_source.sh
    ./contrib/scripts/install_prereq install
    ./configure
    make menuselect

### select app_macro module, format mp3

    make -j2
    make install install-headers install-logrotate
    make samples
    make config
    ldconfig
    groupadd asterisk
    useradd -r -d /var/lib/asterisk -g asterisk asterisk 
    usermod -aG audio,dialout asterisk 
    chown -R asterisk.asterisk /etc/asterisk
    chown -R asterisk.asterisk /var/{lib,log,spool}/asterisk
    chown -R asterisk.asterisk /usr/lib/asterisk

### fix asterisk

    cd
    echo 'AST_USER="asterisk"' >>/etc/default/asterisk
    echo 'AST_GROUP="asterisk"' >>/etc/default/asterisk
    sed -i 's/;\[radius\]/[radius]/g' /etc/asterisk/cdr.conf
    echo 'radiuscfg => /etc/radcli/radiusclient.conf' >>/etc/asterisk/cdr.conf
    sed -i 's";radiuscfg => /usr/local/etc/radiusclient-ng/radiusclient.conf"radiuscfg => /etc/radcli/radiusclient.conf"g'/etc/asterisk/cel.conf
    systemctl restart asterisk

### Fix webserver

    a2enmod rewrite 
    rm -rf /var/www/html
    sed -i 's/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf
    sed -i 's/^\(User\|Group\).*/\1 asterisk/' /etc/apache2/apache2.conf
    sed -i 's/\(^upload_max_filesize = \).*/\12000M/' /etc/php/7.4/cli/php.ini
    sed -i 's/\(^upload_max_filesize = \).*/\12000M/' /etc/php/7.4/apache2/php.ini
    sed -i 's/\(^memory_limit = \).*/\1512M/' /etc/php/7.4/apache2/php.ini
    systemctl restart apache2

### FreePBX install

    tar -xvzf freepbx-16.0-latest.tgz
    cd freepbx 
    ./install -n
    fwconsole ma downloadinstall certman
    fwconsole ma install pm2

### Install dongles driver

    cd
    echo 'KERNEL=="ttyUSB*", MODE="0666", OWNER="asterisk", GROUP="uucp"'>/etc/ udev/rules.d/92-dongle.rules
    echo 'rungroup = dialout'>>/etc/asterisk/asterisk.conf
    mv dongle.conf /etc/asterisk/dongle.conf
    chown asterisk.asterisk /etc/asterisk/dongle.conf
    chmod 664 /etc/asterisk/dongle.conf
    mv extensions_custom.conf /etc/asterisk/extensions_custom.conf
    chown asterisk.asterisk /etc/asterisk/extensions_custom.conf
chmod 664 /etc/asterisk/extensions_custom.conf

### Build the driver

    unzip asterisk-chan-dongle-master* 
    cd asterisk-chan-dongle-master/
    autoupdate
    aclocal
    autoconf
    automake -a ./bootstrap
    ./configure  --with-astversion=18.19.0
    make clean
    make
    make install
    chmod 755 /usr/lib/asterisk/modules/chan_dongle.so
    systemctl restart asterisk

### Secure mysql database and add root password

    mysql_secure_installation

### Reboot

    apt autoremove
    reboot

### Navigate through web page to server

#### Set admin username and password  
