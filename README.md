# Personal setup for ownCloud on Arch Linux, using Apache and MariaDB.
I used Banana Pi M1 for this setup but you can use it to Raspberry Pi or other Pi boards. I used Arch Linux as basic distro, Apache and MariaDB. Of cource there are Lighttpd, SQLite and other servers, but I would like to use those for bigger installations.

I assume you know how to install Arch Linux. I followed the [official wiki] (https://wiki.archlinux.org/index.php/OwnCloud).

This tutorial has the following sections:

- 1. Install needed extra software (including AUR)
- 2. Setup static ip
- 3. Setup no-ip or inadyn client
- 4. Setup MariaDB
- 5. Setup Apache
- 6. Setup ownCloud

---

### 1. Install needed extra software (including AUR)

First of all, setup the language.

```
$ nano /etc/locale.gen
```
And uncomment en_US.UTF-8 (I uncomment el_GR.UTF-8 for Greek).

Then open the conf file.

```
$ nano /etc/locale.conf
```

Copy and Paste the following.
```
LANG="en_US.UTF-8"
LC_COLLATE="C"
LC_TIME="el_GR.UTF-8"
```

Now run the following.
```
$ locale-gen
```
### Extra software

Install the programs that we'll use next.

```
$ pacman -S apache mariadb owncloud php-apcu php-apache php-intl php-mcrypt exiv2 ntp mc make wget fakeroot sudo packer htop ntfs-3g pkg-config
```

#### AUR

After some changes on pacman, there are some workarounds on how to install AUR.

Add users to a group with the gpasswd command as root:

```gpasswd -a alarm wheel```

Go to sudoers

```nano /etc/sudoers```

and change the wheel line

```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

Reboot the system and give the follwing commands as user

```
$ wget https://aur.archlinux.org/cgit/aur.git/snapshot/package-query.tar.gz
$ tar -xvzf package-query.tar.gz
$ cd package-query
$ makepkg -si
```

You can do the same with yaourt

```
$ wget https://aur.archlinux.org/cgit/aur.git/snapshot/yaourt.tar.gz
$ tar -xvzf yaourt.tar.gz
$ cd yaourt
$ makepkg -si
```

### 2. Setup static ip

Edit the file eth0.network. Systemd-networkd uses *.network files.

```$ nano /etc/systemd/network/eth0.network```

paste the following (for static IP `192.168.1.100`)

```
[Match]
Name=eth0

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
DNS=8.8.8.8
DNS=8.8.4.4
```

You will then need to disable netcl. To find out what is enabled that is netctl related, run this:

```$ systemctl list-unit-files```

Once you identify all netctl related stuff. Go through and disable all netctl related stuff. You may have more to disable than just the below:

```$ systemctl disable netctl@eth0.service```

You will then need systemd-networkd enabled:

```$ systemctl enable systemd-networkd```

Login with 

```ssh alarm@192.168.1.100```


### 3. Setup no-ip or inadyn client

If you installed AUR, here is how you can setup no-ip service.

Install No-IP Client with yaourt.

```
yaourt -S noip
``` 

*Configure No-IP Back-end*

Create the configuration file.
```
noip2 -C
```

Enter the relevant details when prompted. All settings can be modified individually later.
 
Modify the update interval.
```
noip2 -U 30
```

Interval that the IP will be updated is set with -U option in minutes. Default is 30 minutes.

 
Modify No-IP username.
```
noip2 -u username
``` 

Modify No-IP password.
```
noip2 -p password
``` 

Start noip service.

```
systemctl start noip2
```

Enable noip serive.
``` 
systemctl enable noip2
``` 

### 4. Setup MariaDB

After MariaDB is installed, setup the database.

```mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql```

Then change default password.

```
mysqld_safe --skip-grant-tables &
```

```
mysql_secure_installation
```

Create database for ownCloud.
```
mysql -u root -p

CREATE DATABASE owncloudb;

GRANT ALL ON owncloudb.* TO ocuser@localhost IDENTIFIED BY 'dbpass';
```

exit the environement.

You added the following (change those as you wish):
```
DATABASE: owncloudb
USER: ocuser
PASSWORD: dbpass
HOST: localhost
```

Start the service and enable it.
```
systemctl start mysqld.service
systemctl enable mysqld.service
```

### 5. Setup Apache

Check the [ownCloud Arch wiki] (https://wiki.archlinux.org/index.php/OwnCloud#Apache_configuration)

First of all copy the owncloud.conf file.

```
cp /etc/webapps/owncloud/apache.example.conf /etc/httpd/conf/extra/owncloud.conf
```

Then open nano and change the file.

```
nano /etc/httpd/conf/httpd.conf
```

And add
```Include conf/extra/owncloud.conf```

Start and enable the serivce.
```
systemctl start httpd.service
systemctl enable httpd.service
```

### 6. Extras

Start Network Time Protocol daemon.
```
systemctl start ntpd.service
systemctl enable ntpd.service
```

Enable [Caching] (https://wiki.archlinux.org/index.php/OwnCloud#Caching) 

Make sure you have the package php-apcu installed. Open the file apcu.ini.
```
nano /etc/php/conf.d/apcu.ini
```

and enable the extension
```
extension=apcu.so
```

Then open php.ini.
```
nano /etc/php/php.ini
```

And add
```
[apc]
 apc.enable_cli=1
``` 

Finally, open the file config.php.
```
nano /etc/webapps/owncloud/config/config.php
```

and add
```
'memcache.local' => '\OC\Memcache\APCu',
```

### 7. Setup ownCloud

mkdir /mnt/owncloud_data
chmod -R 0770 /mnt/owncloud_data
chown -R http:http /mnt/owncloud_data
