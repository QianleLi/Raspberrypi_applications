# Preface

Device: RaspberryPi 3B with a 128 GB micro SD card.   
System: Raspibian installed by noobs.  
# Index

* [Assign a static IP address for the raspi](Raspi_Applications.md#Assign-a-static-IP-address-for-the-raspi)
* [Manage the raspi through your brouser](Raspi_Applications.md#Manage-the-raspi-through-your-brouser)
* [Monitor the status of Raspi](Raspi_Applications.md#Monitor-the-status-of-Raspi)
* [Install samba and share files with other devices within LAN](Raspi_Applications.md#Install-samba-and-share-files-with-other-devices-within-LAN)

# Assign a static IP address for the raspi

Edit the file dhcpcd.conf.  
`sudo nano /etc/dhcpcd.conf`  
Uncommand and change the following line(44):  
```
interface eth0
static ip_address=192.168.1.158    //Your own IP address
static routers=192.168.1.1
static domain_name_servers=114.114.114.114 8.8.8.8
```

# Manage the raspi through your brouser

**First install apache2.**  
`sudo apt-get install apache2`  
Check if it can work.  
`sudo systemctl start apache2`  
Open `THE IP ADDRESS OF RASPI` in your brouser, you should see the Apache Debian default page, which location is /var/www/html/, remove this page.  
```
cd /var/www/html/
sudo rm -f index.html
```
***
**Download KOD Cloud.**  
Note: In this example we only use the free version. There are several more powerful paid versions.  
```
sudo wget http://static.kodcloud.com/update/download/kodexplorer4.25.zip  
sudo unzip kodexplorer4.25.zip  
sudo rm -f kodexplorer4.25.zip
```
***
**Install php**  
`sudo apt-get install php`   
Restart the apache2.  
`sudo systemctl restart apache2`  
***
**Get rid of warnings and errors**  
Refresh the page and you should see warning messages:  
```
su -c 'setenforce 0'
chmod -R 777 /var/www/html/
```
One solution is: Ignore the first statement, and run the second statement in the terminal, then restart apache2 again.  
```
chmod -R 777 /var/www/html/
sudo systemctl restart apache2
```
However, when I tried the above solution, the warning was still there. Therefore, the following is the second solution:  
```
sudo chown -Rf www-data:www-data /var/www/html/
sudo systemctl restart apache2
```
***
Refresh the page, now you should get three new error messages:
```
error:
Php library missing curl
Php library missing mb_string
php GD is not enabled
```
Just install those missing parts:  
```
sudo apt-get install php-curl
sudo apt-get install php-mbstring
sudo apt-get install php-gd
```
Refresh the page and the kod explorer should be installed successfully. Setup the administer password and login.
```
admin
YOUR_PASSWORD
```
You can manage the file system through KOD explorer now!
***
**Several tips**:  
- You may add more users to the system and only log in the admin account when you have to.  
- Since my microSD card has a capacity of 128 GB, I have no need to mount a hard drive to store large files. If you need to do that, create a folder under `/var/www/html/User/[USER_NAME]/home`, and mount your hard drive directly to the folder. 
- Raspberry pi cannot provide enough power to run a hard drive, so you have to use a usb hub which can provide extra power to the hard drive.

# Monitor the status of Raspi

Under the directory `/var/www/html`, download and unzip the project:  
```
sudo wget https://codeload.github.com/spoonysonny/pi-dashboard/zip/master 
sudo mkdir pi
sudo cp master ./pi/
cd pi
sudo unzip master
cd pi-dashboard-master/
sudo mv * ../
cd ..
sudo rm -rf master
sudo rm -rf pi-dashboard-master/
cd ..
sudo rm -rf master
sudo systemctl restart apache2
```
Open `RASPI_IP_ADDRESS/pi/` in your brouser.

# Install samba and share files with other devices within LAN

**Install samba**  
```
cd /
sudo apt-get install samba  
```
***
**Change configuration of samba**  
`sudo vim /etc/samba/smb.conf`  
Comment home, printers and print$ tags, and comment all statements under those three tags. Modify statements under global tag. Add share tag and all statements like following.
```
[global]
	workgroup = WORKGROUP
	security = user
	log file = /var/log/samba/log.%m
	max log size = 1000
	server role = standalone server
	map to guest = bad user
	usershare allow guests = yes
[share]
	comment = share
	browseable = yes
	path = [Directory in which files you want to share locates]
	guest ok = yes
	create mask = 0777
	create mode = 0777  
	directory mask = 0777
	force create mode = 0777  
	directory mode = 0777  
	force directory mode = 0755  
	readonly = no
	writeable = yes
```
***
**Add user pi to the service**  
`sudo smbpasswd -a pi`  
*** 
**Start the service**  
```
sudo systemctl start smbd
sudo systemctl start nmbd
```
***
**On windows laptop:**  
Open task manager and open the address: `\\RASPI IP ADDRESS`
***
**On IOS devices:**  
Download ES File Explorer from app store.  
Service -> Available devices -> RASPBERRYPI
***


Wrote by 2020/8/12