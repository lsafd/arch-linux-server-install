# Arch Linux Server Install

### Prerequisites:

* Device to use as a server (Server)
* Device to SSH into server (Client)
* Router that can port forward (Router)
* Domain from domain name provider (Domain)
* [USB flash drive with Arch Linux](https://wiki.archlinux.org/index.php/USB_flash_installation_media)



### 0. Starting Point:

This guide assumes that your server has access to ethernet and that you want to install remotely from SSH. On the device that you plan to use as a server, boot into Arch Linux from your USB flash drive.

### 1. Initial SSH Setup:

##### On Server:

Check your network connection and note your IP and network interface:

```
ip a
```

Set the root password:

```
passwd
```

Start SSH:

```
systemctl start sshd
```

##### On Client:

SSH into your server (For IP `AAA.AAA.AAA.AAA`):

```
ssh root@AAA.AAA.AAA.AAA
```

### 2. System Install:

##### On Server:

Enable the system clock:

```
timedatectl set-ntp true
```

List device disks:

```
fdisk -l
```

Partition disks (Boot, System, Home, Swap) (For drive `BBB`):

```
fdisk /dev/BBB
```

Format disks (System, Home):

```
mkfs.ext4 /dev/BBB#
```

Format disk (Boot):

```
mkfs.fat -F32 /dev/BBB#
```

Format disk (Swap):

```
mkswap /dev/BBB#
```

Enable swap:

```
swapon /dev/BBB#
```

Mount System:

```
mount /dev/BBB# /mnt
```

Make Home directory:

```
mkdir /mnt/home
```

Make Boot directory:

```
mkdir /mnt/boot
```

Mount Home:

```
mount /dev/BBB# /mnt/home
```

Mount Boot:

```
mount /dev/BBB# /mnt/boot
```

Check mount:

```
mount |grep BBB
```

Edit package mirrors (To install nano run `pacman -Sy nano`):

```
nano /etc/pacman.d/mirrorlist
```

Install the system and other useful packages:

```
pacstrap /mnt base linux linux-firmware base-devel grub efibootmgr os-prober dhcpcd git openssh
```

Generate fstab:

```
genfstab -U /mnt >> /mnt/etc/fstab
```

Change root:

```
arch-chroot /mnt
```

Edit openssh config (Set "PermitRootLogin no"):

```
nano /etc/ssh/sshd_config
```

Enable openssh:

```
systemctl enable sshd
```

Enable dhcpcd (For network interface `CCC`):

```
systemctl enable dhcpcd@CCC
```

Change time zone (For region `DDD` and city `EEE`):

```
ln -sf /usr/share/zoneinfo/DDD/EEE /etc/localtime
```

Sync clock:

```
hwclock --systohc
```

Edit locales:

```
nano /etc/locale.gen
```
Remove the `#` in front of your locale.

Generate locale:

```
locale-gen
```

Create `/etc/locale.conf`:

```
nano /etc/locale.conf
```

Add your locale (Ex: `LANG=en_US.UTF-8`).

Create hostname:

```
nano /etc/hostname
```

Add your hostname (Ex: `FFF`).

Edit hosts:

```
nano /etc/hosts
```
Add the following lines:

```
127.0.0.1 localhost
::1       localhost
127.0.1.1 FFF.localdomain	FFF
```


Add another user (For user `GGG`):

```
useradd -m -G wheel GGG
```

Set a password for the new user:

```
passwd GGG
```

Edit sudoers:

```
EDITOR=nano visudo
```

Remove the `%` in front of `%wheel ALL=(ALL) ALL`.

Install Grub:

```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --recheck
```

Generate Grub config:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

Exit chroot:

```
exit
```

Shutdown:

```
poweroff
```

### 3. Nextcloud Install:

Remove the USB flash drive and boot.

##### On Client:

SSH into your server (For user `GGG` and IP `AAA.AAA.AAA.AAA`):

```
ssh GGG@AAA.AAA.AAA.AAA
```

##### On Server:

Update packages:

```
sudo pacman -Syyyu
```

Set root password to expire:

```
sudo passwd -l root
```

Clone the [yay](https://github.com/Jguer/yay) repo:

```
git clone https://aur.archlinux.org/yay.git
```

Move into the cloned folder:

```
cd yay/
```

Make the package:

```
makepkg -si
```

Install yay:

```
sudo pacman -U yay-*.pkg.tar.xz
```

Install necessary packages:

```
sudo pacman -S nginx-mainline mariadb php-fpm php-gd php-intl ddclient certbot certbot-nginx nextcloud
```

Start and enable nginx:

```
sudo systemctl enable nginx --now
```

##### On Client:

Check the server's ip in a browser. You should see a nginx page.

##### On Server:

Initalize mariadb:

```
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

Start and enable mariadb:

```
sudo systemctl enable mysqld --now
```

Install mysql (There is not yet a sql root password):

```
sudo mysql_secure_installation
```


Edit the nginx config:

```
sudo nano /etc/nginx/nginx.conf
```

Remove the `#`s in front of `location ~ \.php... fastcgi_params;}` and change `html` to `/usr/share/nginx/html` and change `127.0.0.1:9000` to `unix:/run/php-fpm/php-fpm.sock` and change `/scripts` to `$document_root`.

Edit php extensions:

```
sudo nano /etc/php/php.ini
```

Remove the `#`s in front of `extension=mysqli` `extension=intl` `extension=zip` `extension=curl` `extension=gd` and `extension=pdo_mysql`.

Start and enable php-fpm:

```
sudo systemctl enable php-fpm --now
```

##### On Client:

Generate an SSH key:

```
ssh-keygen -t ecdsa -b 521
```

Copy the public key to your server (For key `HHH` and user `GGG` and IP `AAA.AAA.AAA.AAA`):

```
cat ~/.ssh/HHH.pub | ssh GGG@AAA.AAA.AAA.AAA "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

SSH into your server (For user `GGG` and IP `AAA.AAA.AAA.AAA`):

```
ssh GGG@AAA.AAA.AAA.AAA
```

##### On Server:

Edit SSH config:

```
sudo nano /etc/ssh/sshd_config
```

Set `PasswordAuthentication no`.

Restart SSH:

```
sudo systemctl restart sshd
```

##### On Router:

Set a static IP for server.

Setup port forwarding rules from WAN to LAN for ports 69, 80, 443.

##### On Client:

SSH using your router's public IP adress (For user `GGG` and public IP `III.III.III.III`):

```
ssh GGG@III.III.III.III
```

##### On Server:

Edit ddclient config:

```
sudo nano /etc/ddclient/ddclient.conf
```

Add the following lines (For domain `JJJ.JJJ` and password `LLL`) (This applies only to Namecheap domains. You can try to find information for your provider):

```
use=web, web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
server=dynamicdns.park-your-domain.com
login=JJJ.JJJ
password=LLL
@
```

##### On Domain:

Add new record (`Host:"@", Ip Adress: "127.0.0.1", TTL: "1 min"`).

##### On Server:

Start and enable ddclient:

```
sudo systemctl enable ddclient --now
```

Check ddclient status:

```
sudo systemctl status ddclient
```

If didn't work remove cache or reset password:

```
sudo rm /var/cache/ddclient/ddclient.cache
```

##### On Client:

Check the domain in a browser. You should see a nginx page.

##### On Server:

Change the owner of the nextcloud folder:

```
sudo chown http:http /usr/share/webapps/nextcloud -R
```

Change nextcloud folder permissions:

```
sudo chmod 750 /usr/share/webapps/nextcloud
```

Make nextcloud data directory:

```
sudo mkdir /var/nextcloud
```

Change the owner of the nextcloud data directory:

```
sudo chown http:http /var/nextcloud
```

Change nextcloud data directory permissions:

```
sudo chmod 750 /var/nextcloud
```

Edit php-fpm permissions:

```
sudo systemctl edit php-fpm
```

Add the following lines;

```
[Service]
ReadWritePaths = /usr/share/webapps/nextcloud/apps
ReadWritePaths = /etc/webapps/nextcloud/config
ReadWritePaths = /var/nextcloud
```

Restart php-fpm:

```
sudo systemctl restart php-fpm
```

Login to mysql console:

```
sudo mysql -u root -p
```

Create nextcloud database:

```
CREATE DATABASE nextcloud DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_unicode_ci';
```

Create nextcloud user and grant privileges:

```
GRANT ALL PRIVILEGES ON nextcloud.* TO nextcloud@localhost IDENTIFIED BY 'III';
```

Flush privileges:

```
flush privileges;
```

Exit:

```
exit;
```

Make folder for nginx config file:

```
sudo mkdir /etc/nginx/conf.d
```

Make nextcloud config:

```
sudo nano /etc/nginx/conf.d/nextcloud.conf
```

Change `server_name nextcloud.your-domain.com;` to `server_name JJJ.JJJ;` (For domain `JJJ.JJJ`).


Edit nginx config:

```
sudo nano /etc/nginx/nginx.conf
```

Add the following line:

```  
    include    /etc/nginx/conf.d/*.conf;
```

Restart nginx:

```
sudo systemctl restart nginx
```

##### On Client:

Check the domain in a browser. You should see a Nextcloud page.

##### On Server:

Setup certbot:

```
sudo certbot --nginx -d JJJ.JJJ
```

Restart nginx:

```
sudo systemctl restart nginx
```

##### On Client:

Check the domain in a browser. You should see a https page.

##### On Server:

Further optional setup:

```
yay -S redis php-redis php-apcu php-igdbinary
```
