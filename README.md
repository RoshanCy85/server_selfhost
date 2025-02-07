**# server_selfhost



 Installing Ubuntu Server



Head to the official Ubuntu website and download the latest version of Ubuntu Server. You’ll get an .iso file, which is what you'll use to install Ubuntu.
Use a tool like Rufus (on Windows) or Etcher (cross-platform) to create a bootable USB drive with the Ubuntu Server ISO you just downloaded.
Insert the bootable USB drive into your server machine and restart it. During startup, press the key (usually F12, Esc, or Del) to access the boot menu, then select your USB drive to boot from it.
Once the system boots, you’ll see the Ubuntu Server installation screen. Choose the language, keyboard layout, and then follow the on-screen instructions.
When prompted, choose the option to install Ubuntu Server. You’ll go through configuring the time zone, setting up a username and password, and partitioning your hard drive.
It’s a good idea to choose the “Install the minimal server” option if you’re just setting up a personal server.
During the installation, Ubuntu will ask about networking. You can configure your network settings here. You’ll want to set a static IP address to make sure your server always has the same IP on the local network (which is important for access and consistency).
The installation will continue, and once completed, it will prompt you to reboot. Make sure to remove the USB drive during the reboot to prevent booting from it again.
3. Setting Up a Static IP

To set up a static IP address, follow these steps after the installation is complete:

To configure a static IP, open the Netplan configuration file using:
sudo nano /etc/netplan/00-installer-config.yaml

Find the section that starts with eth0 (or the name of your network interface, such as enp3s0 or ens33).
Modify it to look like this:
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4


Replace 192.168.1.100 with the static IP you want for your server and 192.168.1.1 with your router's IP address

Save the file and exit (Ctrl + X, then Y to confirm changes).
Run the following command to apply the configuration
sudo netplan apply
Check if the static IP is set correctly by running:
ip a
4. Dynamic DNS Setup (DuckDNS)

The problem with home internet? My public IP changes randomly unless pay for a static one. That’s where Dynamic DNS (DDNS) helps — it gives me a fixed domain name that updates automatically when my IP changes.

Sign up at DuckDNS and create a subdomain (myserver.duckdns.org).
Get my token from the DuckDNS website.
Install the DuckDNS update script on my server:
sudo apt update && sudo apt install curl -y
Create a script to auto-update my IP:
sudo nano /etc/duckdns.sh
Add this inside (replace with my DuckDNS token and subdomain):
echo url="https://www.duckdns.org/update?domains=myserver&token=MYTOKEN&ip=" | curl -k -o ~/duckdns.log -K -
Make the script executable:
chmod +x /etc/duckdns.sh
Set up a cron job to update every 5 minutes:
crontab -e

Add this at the bottom:

*/5 * * * * /etc/duckdns.sh >/dev/null 2>&1

Now, u can always access my server using myserver.duckdns.org instead of worrying about my changing IP!

5. Setting Up Cloud

When setting up my personal cloud, I came across tons of options — Nextcloud, Immich, OwnCloud, Seafile, and more. After some digging, I decided to go with Nextcloud since I needed a solution for backing up all types of files, not just photos and videos.

But if I were only looking to back up photos and videos, I’d definitely go with Immich instead. It’s faster, works more like Google Photos, and has automatic mobile uploads. That being said, Nextcloud can feel a bit slow compared to Immich, especially when handling large media libraries.

Setting Up Nextcloud:

First, I needed to install Apache, MySQL, PHP, and the required modules:

sudo apt update && sudo apt upgrade -y
sudo apt install apache2 mysql-server php php-mysql libapache2-mod-php php-xml php-mbstring php-zip php-gd php-curl php-intl unzip -y
Then, I downloaded and extracted Nextcloud:

cd /var/www
sudo wget https://download.nextcloud.com/server/releases/latest.zip
sudo unzip latest.zip
sudo mv nextcloud /var/www/html/
I set the correct permissions:

sudo chown -R www-data:www-data /var/www/html/nextcloud
sudo chmod -R 755 /var/www/html/nextcloud
Finally, I restarted Apache:

sudo systemctl restart apache2
By default, Nextcloud stores files on the server’s local drive, but I wanted to use an external hard drive to avoid filling up my main storage.

To mount an external drive:

sudo mkdir /mnt/nextcloud-data
sudo mount /dev/sdb1 /mnt/nextcloud-data
I also updated /etc/fstab to make the mount permanent:

echo "/dev/sdb1 /mnt/nextcloud-data ext4 defaults 0 2" | sudo tee -a /etc/fstab
Then, I set Nextcloud’s data directory to use the mounted drive by modifying the config.php file:

sudo nano /var/www/html/nextcloud/config/config.php
And changed:

'datadirectory' => '/mnt/nextcloud-data',
Now that Nextcloud is set up, you can access it by enter your server’s local IP address in a browser:

If you’re on the same network as your server, just enter your server’s local IP address in a browser:

http://your-server-ip/nextcloud
5. Remote Access to the Server

Being able to access my server remotely is a must. Whether I need to manage files, run updates, or troubleshoot issues, I don’t want to be stuck at home just to do it. Here’s how I set up secure remote access to my server.

There are two main ways to access my server remotely:
SSH (Secure Shell) — Best for command-line access.
VPN — Best for full network access like a local device.


TUNNELING


Install Tailscale on the server:
curl -fsSL https://tailscale.com/install.sh | sh
Start Tailscale and log in:
sudo tailscale up
Accessing My Server Remotely

Find the Tailscale IP of my server by running:

tailscale status
It looks something like 100.x.x.x.

Open a browser and enter:

http://100.x.x.x
Now, I can access my Nextcloud dashboard or any other service I’ve set up, as if I’m at home!

