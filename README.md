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

