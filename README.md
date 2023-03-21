# Nagios

AWS EC2 Instance Security Groups Ingress
https://stackoverflow.com/questions/21981796/cannot-ping-aws-ec2-instance



Nagios Main Server
https://www.digitalocean.com/community/tutorials/how-to-install-nagios-4-and-monitor-your-servers-on-ubuntu-18-04

https://www.vultr.com/docs/install-nagios-on-ubuntu-20-04/

sudo apt update
sudo apt install autoconf gcc make unzip libgd-dev libmcrypt-dev libssl-dev dc snmp libnet-snmp-perl gettext
sudo apt install wget unzip curl openssl build-essential libgd-dev libssl-dev libapache2-mod-php php-gd php apache2 -y

cd ~
curl -L -O https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.4.tar.gz
tar zxf nagios-4.4.4.tar.gz
cd nagioscore-nagios-4.4.4
./configure --with-httpd-conf=/etc/apache2/sites-enabled

make all
sudo make install-groups-users
sudo make install
sudo make install-daemoninit
sudo make install-commandmode
sudo make install-config
sudo make install-webconf

sudo a2enmod rewrite
sudo a2enmod cgi
sudo usermod -a -G nagios www-data
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
sudo systemctl restart apache2

cd ~
curl -L -O https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
tar zxf nagios-plugins-2.2.1.tar.gz
cd nagios-plugins-2.2.1
./configure
make
sudo make install

cd ~
curl -L -O https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-3.2.1/nrpe-3.2.1.tar.gz
tar zxf nrpe-3.2.1.tar.gz
cd nrpe-3.2.1
./configure
make check_nrpe
sudo make install-plugin


sudo nano /usr/local/nagios/etc/nagios.cfg

cfg_dir=/usr/local/nagios/etc/servers


sudo mkdir /usr/local/nagios/etc/servers

#sudo nano /usr/local/nagios/etc/objects/contacts.cfg

sudo nano /usr/local/nagios/etc/objects/commands.cfg

define command{
        command_name check_nrpe
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}

sudo systemctl start nagio



Host to be Monitored

sudo useradd nagios
sudo apt update
sudo apt install autoconf gcc libmcrypt-dev make libssl-dev wget dc build-essential gettext

cd ~
curl -L -O https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
tar zxf nagios-plugins-2.2.1.tar.gz
cd nagios-plugins-2.2.1
./configure
make
sudo make install

cd ~
curl -L -O https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-3.2.1/nrpe-3.2.1.tar.gz
tar zxf nrpe-3.2.1.tar.gz
cd nrpe-3.2.1
./configure

make nrpe
sudo make install-daemon
sudo make install-config
sudo make install-init

df -h /

sudo nano /usr/local/nagios/etc/nrpe.cfg

...
server_address=second_ubuntu_server_private_ip
...
allowed_hosts=127.0.0.1,::1,your_nagios_server_private_ip
...
command[check_xvda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/xvda1
sudo systemctl start nrpe.service
sudo systemctl status nrpe.service
sudo ufw allow 5666/tcp


From Nagios Main Server to test remote connection
/usr/local/nagios/libexec/check_nrpe -H second_ubuntu_server_ip

sudo nano /usr/local/nagios/etc/servers/ngclient.cfg
define host {
        use                             linux-server
        host_name                       your_monitored_server_host_name
        alias                           My client server
        address                         your_monitored_server_private_ip
        max_check_attempts              5
        check_period                    24x7
        notification_interval           30
        notification_period             24x7
}
define service {
        use                             generic-service
        host_name                       your_monitored_server_host_name
        service_description             Load average
        check_command                   check_nrpe!check_load
}
define service {
        use                             generic-service
        host_name                       your_monitored_server_host_name
        service_description             /dev/vda1 free space
        check_command                   check_nrpe!check_vda1
}
sudo systemctl restart nagios
Other Integration Pagerduty and Monitoring Services
https://www.youtube.com/watch?v=qwaMu6LlgWw
Ansible + Nagios - 
https://www.youtube.com/watch?v=vb5YpwanDms
https://www.youtube.com/watch?v=6vfhflwC_Wg
