# startAddSoft
#Доп. софт при локально установке установке
#!/bin/bash

echo "create user"
echo -e "\e[31mEnter your user name\e[0m"
read nameuser
echo -e "\e[31mEnter the user password\e[0m"
read passwduser

useradd $nameuser
echo "$passwduser" | passwd --stdin "$nameuser"
cd /home/$nameuser

#echo "stop firewall"
localurl="http://192.168.1.199/mrepo"
echo "repo"
repos=/etc/yum.repos.d

echo "add net-repo and copy-bak"

cp -a $repos $repos.net

echo "add local repo"

rm -rf $repos
mkdir $repos
base=$repos/base.repo
epel=$repos/epel.repo
nux=$repos/nux.repo

echo "create repofiles"

touch $base 
echo '[base]' >> $base
echo 'name=CentOS-$releasever - Base' >> $base
echo "baseurl=$localurl/mirror.yandex.ru/centos/7/os/x86_64/" >> $base
echo 'gpgcheck=0' >> $base

touch $epel
echo '[epel]' >> $epel
echo 'name=CentOS-7-epel' >> $epel
echo "baseurl=$localurl/epel/" >> $epel
echo 'gpgcheck=0' >> $epel

touch $nux
echo '[nux]' >> $nux
echo 'name=CentOS-7-nux' >> $nux
echo "baseurl=$localurl/nux/" >> $nux
echo 'gpgcheck=0' >> $nux

yum repolist

echo "repos adding complete"

echo "install needing soft"

yum install mc net-tools nano wget -y



echo "add russian in console"
yum localinstall $localurl/mirror.yandex.ru/fedora/russianfedora/russianfedora/free/fedora/releases/19/Everything/x86_64/os/workaround-cyrillic-console-1.0-5.fc19.R.noarch.rpm -y
sed -i -e 's/KEYMAP="us"/KEYMAP="ru"/' /etc/vconsole.conf
sed -i -e 's/FONT="latarcyrheb-sun16"/FONT="cyr-sun16"/' /etc/vconsole.conf

sed -i -e 's/#HandleLidSwitch=suspend/HandleLidSwitch=ignore/' /etc/systemd/logind.conf

yum groupinstall "GNOME Desktop" -y

ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target

yum install openvpn -y  

yum install ntfs-3g fuse -y

modprobe fuse

echo "install keepass2, x2goclient"

yum install xdotool -y
yum install $localurl/add/keepass-2.27-1.fc20.x86_64.rpm -y

yum install NetworkManager-vpnc NetworkManager-pptp NetworkManager-openvpn NetworkManager-openvpn-gnome x2goclient -y

echo "install truecrypt"
yum install -y libfuse.so.2 

wget $localurl/add/truecrypt-7.1a-linux-console-x64.tar.gz
tar -xvf truecrypt-7.1a-linux-console-x64.tar.gz
./truecrypt-7.1a-setup-console-x64

homedir=/home/$nameuser
namefile=$homedir/start.sh
stopnamefile=$homedir/stop.sh
wget $localurl/add/start.png
wget $localurl/add/stop.png

pathtrue="$homedir/soft_usb"
touch $namefile && chmod +x $namefile
echo '#!/bin/sh' >> $namefile
echo "pathtrue=$pathtrue #заменить на нужный путь" >> $namefile
echo "homedir=$homedir #заменить на нужный путь" >> $namefile
echo 'truecrypt --keyfiles=$pathtrue/readme.txt --mount $pathtrue/tcp $homedir/true' >> $namefile
chown $nameuser:$nameuser $namefile
echo "file start.sh create)"


touch $stopnamefile && chmod +x $stopnamefile
echo '#!/bin/sh' >> $stopnamefile
echo "truecrypt -d" >> $stopnamefile
chown $nameuser:$nameuser $stopnamefile
chown -R $nameuser:$nameuser /$homedir

echo "%""$nameuser"" ALL=/usr/bin/truecrypt " >> /etc/sudoers

echo "return inet repo, local-bak"
cp -a $repos $repos.local
rm -rf $repos
cp -a $repos.net $repos
rpm -Uvh https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm 
rpm -ivh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
yum repolist

#cp $namefile /home/$nameuser/'Рабочий стол'/start.sh
url="http://192.168.1.166:8082/kee"
wget $url/$nameuser.kdbx
wget $url/tcp/$nameuser"_soft_usb".zip
unzip $nameuser"_soft_usb.zip"

mkdir $homedir/true

startfile=/$homedir/start.desktop
stopfile=/$homedir/stop.desktop

touch $startfile && chmod a+x $startfile
echo '[Desktop Entry]' >> $startfile
echo 'Name = Start' >> $startfile
echo "Exec = sh $namefile" >> $startfile
echo 'Terminal = true' >> $startfile
echo 'Type = Application' >> $startfile
echo "Icon = $homedir/start.png" >> $startfile

touch $stopfile && chmod a+x $stopfile
echo '[Desktop Entry]' >> $stopfile
echo 'Name = Stop' >> $stopfile
echo "Exec = sh $stopnamefile" >> $stopfile
echo 'Terminal = true' >> $stopfile
echo 'Type = Application' >> $stopfile
echo "Icon = $homedir/stop.png" >> $stopfile

chown $nameuser:$nameuser $startfile
chown $nameuser:$nameuser $stopfile

chown -R $nameuser:$nameuser $homedir


echo "winload"
#grub2-set-default "Windows (loader) (on /dev/sda1)"
sed -i -e 's/GRUB_TIMEOUT=5/GRUB_TIMEOUT=2/' /etc/default/grub
sed -i -e 's/GRUB_DEFAULT=saved/GRUB_DEFAULT=2/' /etc/default/grub

echo 'set menu_color_normal=red/black' >> /etc/grub.d/40_custom
echo 'set menu_color_highlight=yellow/black' >> /etc/grub.d/40_custom
echo 'set color_normal=yellow/black' >> /etc/grub.d/40_custom
echo ' ' >> /etc/grub.d/40_custom
echo 'ForexOS (exchange)' > /etc/system-release
grub2-mkconfig -o /boot/grub2/grub.cfg

echo "$nameuser"".remote" > /etc/hostname
sed -i -e 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
echo "install add soft - complete )"
echo "reboot )"
rm -rf asmr.sh
reboot
