cryptsetup (LearnLinuxTV LVM with Encryption)
Do I need to forward more ports
Making nextcloud config (nextcloud.conf)
Check wiki to add new requirements



##Arch Linux Server Install


##Check network
ip a
##Set root password
passwd
##Start ssh
systemctl start sshd
##Check ssh status
systemctl status sshd
##Enable system clock
timedatectl set-ntp true
##Check clock status
timedatectl status
##View disks
fdisk -l
##Partition disks (EFI, System, Home, Swap)
fdisk /dev/XXX
##Format disks (System, Home)
mkfs.ext4 /dev/XXX#
##Format disk (EFI)
mkfs.fat -F32 /dev/XXX#
##Format disk (Swap)
mkswap /dev/XXX#
##Turn swap on
swapon /dev/XXX#
##Mount System
mount /dev/XXX# /mnt
##Make Home directory
mkdir /mnt/home
##Make EFI directory
mkdir /mnt/boot
##Mount Home
mount /dev/XXX# /mnt/home
##Mount EFI
mount /dev/XXX# /mnt/boot
##Check mount
mount |grep XXX
##Install prefered text editor
pacman -Sy nano
##Edit package mirrors
nano /etc/pacman.d/mirrorlist
##Install system
pacstrap /mnt base linux linux-firmware
##Generate fstab
genfstab -U /mnt >> /mnt/etc/fstab
##Change root
arch-chroot /mnt
##Install useful packages
pacman -Sy base-devel grub efibootmgr os-prober dosfstools nano dhcpcd git openssh
##Edit openssh config (Set "PermitRootLogin no" and "Port 69")
nano /etc/ssh/sshd_config
##Enable openssh
systemctl enable sshd
##Enable dhcpcd
systemctl enable dhcpcd@YYY
##Change time zone
ln -sf /usr/share/zoneinfo/America/ZZZ /etc/localtime
##Sync clock
hwclock --systohc
##Edit locales
nano /etc/locale.gen
##Generate locale
locale-gen
##Create /etc/locale.conf (Add "LANG=en_US.UTF-8")
nano /etc/locale.conf
##Create hostname (Add "AAA")
nano /etc/hostname
##Edit hosts
nano /etc/hosts


127.0.0.1	localhost
::1		localhost
127.0.1.1	AAA.localdomain	AAA


##Add another user
useradd -m -G wheel BBB
##Set a password for the new user
passwd BBB
##Give the user account sudo privlidges (Uncomment "%wheel ALL=(ALL) ALL")
EDITOR=nano visudo
##Grub installation
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --recheck
##Generate grub config
grub-mkconfig -o /boot/grub/grub.cfg
##Edit grub boot config (Add "nomodeset")
nano /boot/grub/grub.cfg
##Exit chroot
exit
##Unmount partitions
umount -R /mnt
##Shutdown
shutdown
##Remove install drive and boot
##Check network status
ip a
##Update packages
sudo pacman -Syyyu
##Set root password to expire
sudo passwd -l root
##Clone the git repo
git clone https://aur.archlinux.org/yay.git
##Go to the cloned folder
cd yay/
##Make the package
makepkg -s
##Install yay
sudo pacman -U yay-CCC.pkg.tar.xz
##Install necessary packages
sudo pacman -S nginx-mainline mariadb php-fpm php-gd php-intl ddclient certbot certbot-nginx nextcloud
##Start and enable nginx
sudo systemctl enable nginx --now

##On other computer
##Check server ip in a browser

##Initalize mariadb
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
##Start and enable mariadb
sudo systemctl enable mysqld --now
##Install mysql (There is not yet a sql root password)
sudo mysql_secure_installation


##Edit the nginx conig (Uncomment "location ~ \.php... fastcgi_params;}" and "html" to "/usr/share/nginx/html" and "127.0.0.1:9000" to "unix:/run/php-fpm/php-fpm.sock" and "/scripts" to "$document_root")
sudo nano /etc/nginx/nginx.conf


##Edit php extensions (Uncomment "extension=mysqli"extension=intl" "extension=zip" "extension=curl" "extension=gd" and "extension=pdo_mysql")
sudo nano /etc/php/php.ini
##Start and enable php-fpm
sudo systemctl enable php-fpm --now

##On other computer
##Generate ssh key
ssh-keygen -t ecdsa -b 521
##Copy the public key
cat ~/.ssh/DDD.pub | ssh BBB@EEE.EEE.EEE.EEE -p 69 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys"
##Check ssh
ssh BBB@EEE.EEE.EEE.EEE -p 69

##On server 
##Edit ssh config (Set "PasswordAuthentication no")
sudo nano /etc/ssh/sshd_config
##Restart ssh
sudo systemctl restart sshd

##On router
##Set port forwarding rules (Forward 69, 80, 443)
##Set static ip for server

##On other computer
##SSH using public ip adress
ssh BBB@FFF.FFF.FFF.FFF -p 69

##On server
##Edit ddclient config
sudo nano /etc/ddclient/ddclient.conf


use=web, web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
server=dynamicdns.park-your-domain.com
login=GGG.GGG
password=HHH
@



##On domain provider
##Add new record (Host:"@", Ip Adress: "127.0.0.1", TTL: "1 min")

##On server
##Start and enable ddclient
sudo systemctl enable ddclient --now
##Check ddclient status
sudo systemctl status ddclient
##If didn't work remove cache or reset password
sudo rm /var/cache/ddclient/ddclient.cache

##On other computer
##Check domain for nginx page

##On server
##Change owner of nextcloud folder
sudo chown http:http /usr/share/webapps/nextcloud -R
##Change nextcloud folder permissions
sudo chmod 750 /usr/share/webapps/nextcloud
##Make nextcloud data directory
sudo mkdir /var/nextcloud
##Change nextcloud data directory owner
sudo chown http:http /var/nextcloud
##Change nextcloud data directory permissions
sudo chmod 750 /var/nextcloud
##Edit php-fpm permissions
sudo systemctl edit php-fpm


[Service]
ReadWritePaths = /usr/share/webapps/nextcloud/apps
ReadWritePaths = /etc/webapps/nextcloud/config

# Replace the following path with the Nextcloud data directory
ReadWritePaths = /var/nextcloud


##Restart php-fpm
sudo systemctl restart php-fpm
##Login to mysql console
sudo mysql -u root -p
##Create nextcloud database
CREATE DATABASE nextcloud DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_unicode_ci';
##Create nextcloud user and grant privileges
GRANT ALL PRIVILEGES ON nextcloud.* TO nextcloud@localhost IDENTIFIED BY 'III';
##Flush privileges
flush privileges;
##Exit
exit;
##Make folder for nginx config file
sudo mkdir /etc/nginx/conf.d
##Go to the folder
cd /etc/nginx/conf.d
##Make and edit nextcloud conig (Change "server_name nextcloud.your-domain.com;" to "server_name JJJ.JJJ;")
sudo nano nextcloud.conf
##Change directory
cd /
##Edit nginx config (add "    include    /etc/nginx/conf.d/*.conf;")
sudo nano /etc/nginx/nginx.conf
##Restart nginx
sudo systemctl restart nginx

##On other computer
##Check domain for nextcloud page

##On server
##Setup certbot
sudo certbot --nginx -d JJJ.JJJ
##If didn't work edit nginx mime.types (In line "application/vnd.geocube+xml" change ending to just "g3")
sudo nano /etc/nginx/mime.types
##Restart nginx
sudo systemctl restart nginx
##Setup certbot
sudo certbot --nginx -d JJJ.JJJ

##On other computer
##Check domain for https


##Further setup
yay -S redis php-redis php-apcu php-igdbinary