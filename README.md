# Setup Raspberry-pi 3B+
## [1. Turn on SSH with Pi Server](https://phoenixnap.com/kb/enable-ssh-raspberry-pi)

#### Enable SSH using the raspi-config too
```bash
sudo raspi-config
```

#### Use systemctl to Enable SSH
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

#### Check Raspberry Pi’s local IP address
```bash
hostname -I
```

#### Connect via SSH
```bash
ssh pi@[raspberrypi_ip_address]
```

## [2. Increasing Swap on a Raspberry Pi](https://pimylifeup.com/raspberry-pi-swap-file/)

#### Stop the operating system from using the current swap file
```bash
sudo dphys-swapfile swapoff
```

#### Modify the swap file configuration file
```bash
sudo nano /etc/dphys-swapfile
```

#### Increase our swap size to 2GB
```bash
CONF_SWAPSIZE=2048
```

#### Re-initialize the Raspberry Pi’s swap file
```bash
sudo dphys-swapfile setup
```

#### Start the operating systems swap system
```bash
sudo dphys-swapfile swapon
```

#### Reboot
```bash
sudo reboot
```

## [3. Installing Docker](https://pimylifeup.com/raspberry-pi-docker/)

#### Upgrade all existing packages
```bash
sudo apt update
sudo apt upgrade
```

#### Download and run the official Docker setup script
```bash
curl -sSL https://get.docker.com | sh
```

#### Add current user to the docker group.
```bash
sudo usermod -aG docker $USER
```

#### Log out
```bash
logout
```

#### Verify that the docker group
```bash
groups
```

#### Test docker working
```bash
docker run hello-world
```

## [4. Installing Docker compose](https://linuxhint.com/install-docker-compose-raspberry-pi/)

#### Upgrade all existing packages
```bash
sudo apt update
sudo apt upgrade
```

#### Install Docker Compose
```bash
sudo apt install docker-compose -y
```

#### Confirm Docker Compose Version
```bash
docker-compose version
```

#### Test Docker Compose
```bash
sudo docker compose up -d
```

#### Remove Docker Compose
```bash
sudo apt remove --autoremove docker-compose
```


## [5. Installing Portainer](https://pimylifeup.com/raspberry-pi-portainer/)

#### Upgrade all existing packages
```bash
sudo apt update
sudo apt upgrade
```

#### Portainer is available as a Docker container on the official Docker hub
```bash
sudo docker pull portainer/portainer-ce:latest
```

#### Start up Portainer
```bash
sudo docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

#### Check Raspberry Pi’s local IP address
```bash
hostname -I
```

#### Aaccess Portainer
```bash
http://[PIIPADDRESS]:9000
```


## [6. Installing OpenMediaVault](https://pimylifeup.com/raspberry-pi-openmediavault/)

#### Upgrade all existing packages
```bash
sudo apt update
sudo apt upgrade
```

#### Download the OpenMediaVault install script and pipe it directly through to bash
```bash
wget -O - https://raw.githubusercontent.com/OpenMediaVault-Plugin-Developers/installScript/master/install | sudo bash
```

#### Restart Raspberry Pi
```bash
sudo reboot
```

#### Check Raspberry Pi’s local IP address
```bash
hostname -I
```

#### Aaccess Portainer
```bash
http://[PIIPADDRESS]:80
```


## 7. Set DDNS Google with API Python Script

#### Create file setddns.py
```bash
nano setddns.py
```

#### Copy code
```python
#!/usr/bin/python3
import requests
import logging
import time
from urllib.request import urlopen

logging.basicConfig(filename='ddns.log', level=logging.DEBUG, filemode='a', format='%(asctime)s - %(message)s')

username = 'youruserddns'
password = 'yourpasswordddns'
hostname = 'subdomain.yourdomain.com'
old_ip = ''

while True:
	try:
		my_ip = urlopen('https://domains.google.com/checkip').read() 
	except:
		logging.debug('CATCHED AN ERROR... RETRYING IN 10 SECONDS')
		time.sleep(10)
	else:
		if my_ip != old_ip:
			url = 'https://{}:{}@domains.google.com/nic/update?hostname={}'.format(username, password, hostname)
			response = requests.post(url)
			output = response.content.decode('utf-8')
			if 'good' in output or 'nochg' in output:
				old_ip = my_ip
			logging.debug('-- OUTPUT FOR UPDATE: '+ hostname +' --')
			logging.debug('Response from DDNS update: '+ output)
		time.sleep(10)
```

#### Set file setddns.py auto run when start
```bash
sudo -i
nano /etc/rc.local
```

#### Fix file
```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi
python3 /path/to/file/setddns.py
exit 0

```

#### Reboot
```bash
sudo reboot
```

## [8. Mount a USB Drive](https://thesecmaster.com/how-to-partition-and-format-the-hard-drives-on-raspberry-pi/)

#### Filesystem Types
1. NTFS: This file system is developed by Microsoft in the early 90s. All new versions of Windows operating systems will support this file system. Theoretically, NTFS can support hard drives up to just under 16 EB. The individual file size is capped at just under 256 TB, at least in Windows 8 and Windows 10, as well as in some newer Windows Server versions. When it comes to supporting, this file system is universally supported. Although it’s developed by Microsoft, it is supported by most Linux distributions and Mac.
2. EXT4: This file system is developed based on the older Minix filesystem, A file system being used by Linux systems for ages. The higher maximum volume size it supports is 1 EB. That’s, again, a mathematical number. I know all these numbers don’t bother you like many of us. After all, who is going to use such a gigantic drive at home with Raspberry Pi!

#### List out the connected drives and use ‘print all’ to read the drive information
```bash
sudo parted
(parted) print all
```

#### Select the drive to partition
Select the drive to format and create new partitions. Type the ‘select’ command with the drive path.
```bash
(parted) select /dev/sda
```

#### Create a fresh GPT partition table
Create a fresh GPT partition table by typing ‘mklabel gpt’ command. You will get a warning to wipe out all your drive. Type ‘yes’ to continue. Please bear in mind, that it’s just a partition table, not the partitions.
```bash
(parted) mklabel gpt
```

#### Type ‘print’ to make sure the new GPT partition is created
```bash
(parted) print
```

#### Create new partitions
Create three partitions on this drive using ‘mkpart’ command: data-nas, and data-all.
Type ‘mkpart’ command to create a new partition. It asks four simple questions to create a partition. Your partition will be created upon supplying the answers. Just pay attention to the commands we used to create three partitions. You can create partitions in a single line command as well, as we show in the below screenshot.
```bash
(parted) mkpart data-nas ext4 0% 50%
(parted) mkpart data-all ext4 50% 100%
```

#### Exit from parted
```bash
(parted) q
```

#### Format the partitions
You can’t use the partitions until you format them. Let’s use mkfs commands to format the partitions. Different versions of mkfs commends are there to format NTFS and EXT4 file systems. In this command -L specifies the label of the drive, and -Q specifies quick format, which takes the partition name as a parameter. Note: EXT4 doesn’t take -Q as it doesn’t support the quick format.
```bash
sudo mkfs.ext4 -L data-nas /dev/sda1
sudo mkfs.ext4 -L data-all /dev/sda2
```

#### Reboot the Raspberry Pi
```bash
sudo reboot
```

#### Remount these partitions under /mnt
Change the directory to /mnt.
```bash
cd /mnt
```
Create two directories named ‘data-nas’ and ‘data-all’ under /mnt.
```bash
sudo mkdir data-nas
sudo mkdir data-all
```
Let’s now give our pi user ownership of this folder by running the command below.
```bash
sudo chown -R pi:pi /mnt/data-nas
sudo chown -R pi:pi /mnt/data-all
```
Mount the partitions using the mount command. Note: This is just a temp mount. It is not going to work after reboot.
```bash
sudo mount /dev/sda1 /mnt/data-nas
sudo mount /dev/sda2 /mnt/data-all
```
Do you remember the fstab? It’s a file system table. This is where you can mount a partition forever. Open the /etc/fstab file and see how it looks. You can only see the SD card at this time.
```bash
cat /etc/fstab
```

#### Take the PARTUUID value of the two partitions
You need to add those two partitions to /etc/fstab to mount permanently. Before that, make a note of PARTUUID value of the two partitions.
```bash
sudo blkid
```

#### Create a permanent mount
Use your choice of a text editor to edit and add the partition information in the /etc/fstab. You can add the two lines representing each line for a partition. Write this information separated by TAB.
```bash
sudo blkid
```

```bash
PARTUUID=VALUE
Mount path
File System
default or default, notime: The word ‘notime’ just tells us to keep track of the access time along with created and modified time.
0
0
```
Open fstab
```bash
sudo nano /etc/fstab
```

Add the partition information. Examples
```bash
PARTUUID=8c8816b4-13d5-4345-8410-85263b3c5891	/mnt/data-gitea	ext4	defaults,noatime	0	0
PARTUUID=6d56b35f-d771-4be0-9ae2-d4faf98e2326	/mnt/data-all	ext4	defaults,noatime	0	0
```

#### Reboot the Raspberry Pi
```bash
sudo reboot
```

## [9. Generating SSH Keys on Linux based systems](https://pimylifeup.com/raspberry-pi-ssh-keys/)

#### Generate SSH Keys
```bash
ssh-keygen
```

#### View ssh public key
```bash
cat ~/.ssh/id_rsa.pub
```