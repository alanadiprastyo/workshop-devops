---
# Initial Configuration for LAB
# Setting hostname otomatis
```
IPGW=`ifconfig | grep inet | grep 255$ | cut -d " " -f 10`

if grep -Ri "docker-host" /etc/hosts
then
	echo "Host file sudah tersetting"
	cat /etc/hosts | grep  docker-host
else
	echo "$IPGW  docker-host" >> /etc/hosts
	echo "Setting hostname"
	hostnamectl set-hostname docker-host
	hostname
fi
```

# Setting hostname manual
```
cat /etc/hosts | grep docker-host
hostnamectl set-hostname docker-host
hostname
```
---
# Page 44 - Lab: Installing Docker - PreSetup
```
yum -y install net-tools vim vim-enhanced vim-common wget git net-tools bind-utils iptables-services bridge-utils bash-completion
```

# Page 44 - Lab: Installing Docker - PreSetup
# Install Docker dan konfigurasi Selinux untuk Docker
```
curl -fsSL https://get.docker.com/ | sh
systemctl is-active docker ; systemctl enable docker ; systemctl restart docker 
```
---
# Page 45 - Lab: Installing Docker - PreSetup
# Konfigurasi Docker insecure Network untuk docker private registry
```
vim /usr/lib/systemd/system/docker.service
Edit
ExecStart=/usr/bin/dockerd
to
ExecStart=/usr/bin/dockerd --insecure-registry 172.30.0.0/16 --insecure-registry 192.168.1.0/24

systemctl daemon-reload ; systemctl restart docker
```