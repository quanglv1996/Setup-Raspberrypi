# Setup Raspberry-pi 3B+
## [1.Increasing Swap on a Raspberry Pi](https://pimylifeup.com/raspberry-pi-swap-file/)

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

## [2.Installing Docker](https://pimylifeup.com/raspberry-pi-docker/)

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

## [3.Installing Docker compose](https://linuxhint.com/install-docker-compose-raspberry-pi/)

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


## [4.Installing Portainer](https://pimylifeup.com/raspberry-pi-portainer/)

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


## [5.Installing OpenMediaVault](https://pimylifeup.com/raspberry-pi-openmediavault/)

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


## 6. Set DDNS Google with API Python Script

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