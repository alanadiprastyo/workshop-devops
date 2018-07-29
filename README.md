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

# Running Docker container pertama anda dari private registry
```
docker container run -ti docker-registry:5000/ubuntu bash
```

---
# Page 47 - Lab: 1st  time Playing w/ Docker
# Running Docker container pertama anda dari private registry
```
docker run -t docker-registry:5000/centos bash
docker images --all
docker ps
docker exec -it <CONTAINER-ID> bash
```
---
# Page 49 - docker run - Run a container
```
docker run docker-registry:5000/centos /bin/hostname
docker run docker-registry:5000/centos /bin/hostname
docker run docker-registry:5000/centos date +%H:%M:%S
docker run docker-registry:5000/centos true ; echo $?
docker run docker-registry:5000/centos false ; echo $?
```

---
# Page 50 - docker run - Foreground mode vs. Detached mode
```
docker run docker-registry:5000/centos date
docker run -d docker-registry:5000/centos date
docker logs <CONTAINER-ID>
```

---
# Page 52 - docker run - Set the container name
```
docker run -d -t docker-registry:5000/debian
docker run -d -t --name blahblah docker-registry:5000/debian
docker ps 
docker stop blahblah focused_raman
```

---
# Page 55 - Lab: Docker commit example
```
docker run --name my-container -t -i docker-registry:5000/debian
cat >> /etc/bash.bashrc <<EOF
> echo 'hello!'
> EOF

exit
docker start --attach my-container
exit
docker diff my-container
docker commit my-container hello
docker stop my-container; docker rm my-container
docker run --rm -t -i hello
docker images --all
```

---
# Page 58 - Lab: Mount examples
```
docker run --rm -t -i -v /tmp/persistent:/persistent docker-registry:5000/debian
echo "test persisten storage" >/persistent/foo
exit
cat /tmp/persistent/foo
docker run --rm -t -i -v /tmp/persistent:/persistent docker-registry:5000/debian
cat /persistent/foo
test persisten storage

mkdir /tmp/inputs
echo hello > /tmp/inputs/bar
docker run --rm -t -i -v /tmp/inputs:/inputs:ro docker-registry:5000/debian
cat /inputs/bar
touch /inputs/foo
```
---
# Page 59 - Lab: Mount examples continue 
```
mkfifo /tmp/fifo
docker run -d -v /tmp/fifo:/fifo docker-registry:5000/debian sh -c 'echo blah blah> /fifo'
cat /tmp/fifo

docker run --rm -t -i -v /dev/log:/dev/log docker-registry:5000/debian
logger blah blah blah
exit
cat /var/log/messages | grep blah
```
---
# Page 60 - docker run-inter-container links (legacy links )
```
docker run --name my-server docker-registry:5000/debian sh -c 'hostname -i && sleep 500' &
docker run --rm -t -i --link my-server:srv debian
ping srv
```
---
# Page 62 - User-defined networks (since v1.9.0)
```
docker network create NETWORK
docker run -t --name test-network --net=NETWORK debian
docker inspect test-network | grep -i NETWORK
docker network list
docker network connect NETWORK test-network
docker network connect bridge test-network
docker network disconnect NETWORK test-network
```
---
# Page 65 - Publish TCP Port
```
docker run -d -p 80:80 docker-registry:5000/nginx
docker inspect <CONTAINER-ID> | grep IPAddress
wget -nv http://localhost/
wget -nv http://172.17.0.2/

docker run -d -p 127.0.0.1:80:80 docker-registry:5000/nginx
wget -nv http://localhost/
wget http://172.17.0.2/
```
---
# Page 91 - Lab: Image creation from a container
```
docker run -ti  docker-registry:5000/ubuntu bash
apt-get update -y ; apt-get install figlet
exit

docker ps -a
docker commit <CONTAINER-ID>
```
---
# Page 92 - Lab: Image creation from a container
```
docker images ls
docker image tag <CONTAINER-ID> tag-intra
docker images ls
docker container run tag-intra figlet hello
```

---
# Page 107,108,109 - Lab: Dockerfile example
```
---------------start dari sini------------------
# Dockerfile to build nginx container images
# Based on debian latest version---##

# base image: last debian release
FROM docker-registry:5000/debian:latest

# name of the maintainer of this image
MAINTAINER alan@routecloud.net

# install the latest upgrades
#RUN apt-get update && apt-get -y dist-upgrade && echo alan-test > /tmp/test
RUN apt-get update && echo alan-test > /tmp/test

# install nginx
RUN apt-get -y install nginx

# set the default container command
# −> run nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]

# Tell the docker engine that there will be somenthing listening on the tcp port 80
EXPOSE 80
---------------stop sampai sini---------------
```

```
docker build -t nginx_alan .
docker run --name my_first_nginx_instance -i -t nginx_alan bash
cat /tmp/test
```
---
# Page 112,115 - Lab: Docker Compose Ubuntu, php7-fpm, Nginx and MariaDB Example
```
git clone https://github.com/alanadiprastyo/docker-php7
cd docker-php7 ; yum -y install epel-release ; yum install -y python-pip ; pip install docker-compose
docker-compose up
```
---
# Page 123 - Installing Portainer.io
```
docker volume create portainer_data
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data docker-registry:5000/portainer/portainer:latest
```
---
# Page 196 - Installing Openshift Origin
```
cat /etc/hosts | grep docker
yum install -y centos-release-openshift-origin
yum install -y wget git net-tools bind-utils iptables-services bridge-utils bash-completion 
yum -y install origin-clients origin
```

- Command di bawah ini untuk mengambil openshift image versi terbaru yaitu 3.6.0 dari private registry
- Sehingga tidak perlu download ke internet
- Perintah dibawah ini dilakukan hanya untuk environment Workshop ini
```
docker pull docker-registry:5000/openshift/origin:v3.9.0
docker pull docker-registry:5000/openshift/origin-deployer:v3.9.0
docker pull docker-registry:5000/openshift/origin-pod:v3.9.0
docker pull docker-registry:5000/openshift/origin-sti-builder:v3.9.0
docker pull docker-registry:5000/openshift/origin-haproxy-router:v3.9.0
docker pull docker-registry:5000/openshift/origin-docker-registry:v3.9.0
```

```
docker image tag docker-registry:5000/openshift/origin:v3.9.0 openshift/origin:v3.9.0
docker image tag docker-registry:5000/openshift/origin-deployer:v3.9.0 openshift/origin-deployer:v3.9.0
docker image tag docker-registry:5000/openshift/origin-pod:v3.9.0 openshift/origin-pod:v3.9.0
docker image tag docker-registry:5000/openshift/origin-sti-builder:v39.0 openshift/origin-sti-builder:v3.9.0
docker image tag docker-registry:5000/openshift/origin-haproxy-router:v3.9.0 openshift/origin-haproxy-router:v3.9.0
docker image tag docker-registry:5000/openshift/origin-docker-registry:v3.9.0 openshift/origin-docker-registry:v3.9.0
```

- Tambahkan rules firewall agar dapat mengakses Openshift

```
iptables -I INPUT 1  -p tcp --dport 8443 -j ACCEPT
iptables -I INPUT 1  -p udp --dport 53 -j ACCEPT
iptables -I INPUT 1  -p tcp --dport 53 -j ACCEPT
iptables -I INPUT 1  -p tcp --dport 443 -j ACCEPT
iptables -I INPUT 1  -p tcp --dport 80 -j ACCEPT
```
---
# Page 198 - Installing OpenShift – oc cluster up
```
oc cluster up --public-hostname=<IP-ADDRESS-ANDA>
oc cluster up  --public-hostname=192.168.1.171

oc login -u system:admin
oadm policy add-cluster-role-to-user cluster-admin admin
```
---
# Page 203 - Creating project
# Page 204,205,208 - Origin 1st App Deployment
- Pulling Docker image yang dibutuhkan dari Private registry
```
docker pull docker-registry:5000/centos/php-70-centos7
docker pull docker-registry:5000/centos/postgresql-95-centos7
docker pull docker-registry:5000/centos/mysql-57-centos7
docker pull docker-registry:5000/openshift/jenkins-1-centos7
docker pull docker-registry:5000/openshiftdemos/oc

docker tag docker-registry:5000/centos/php-70-centos7 centos/php-70-centos7 
docker tag docker-registry:5000/centos/postgresql-95-centos7 centos/postgresql-95-centos7
docker tag docker-registry:5000/centos/mysql-57-centos7 centos/mysql-57-centos7
docker tag docker-registry:5000/openshift/jenkins-1-centos7 openshift/jenkins-1-centos7
docker tag docker-registry:5000/openshiftdemos/oc openshiftdemos/oc
```


```
oc login -u system:admin
oc new-project test-project
oc new-app openshift/deployment-example
oc status
oc get all
docker ps | grep deployment-example
curl <IP-POD-APP>:8080
```

---
# Page 210 - Origin 2nd App Deployment
```
oc new-app https://github.com/openshift/ruby-hello-world -o yaml > myapp.yaml
cat myapp.yaml 
```

---
# Additional Lab - Continuous integration / continuous delivery (cicd)
```
docker pull docker-registry:5000/openshiftdemos/gogs
docker pull docker-registry:5000/openshiftdemos/sonarqube
docker pull docker-registry:5000/openshiftdemos/nexus
docker pull docker-registry:5000/openshiftdemos/oc

docker tag docker-registry:5000/openshiftdemos/gogs openshiftdemos/gogs
docker tag docker-registry:5000/openshiftdemos/sonarqube openshiftdemos/sonarqube
docker tag docker-registry:5000/openshiftdemos/nexus openshiftdemos/nexus
docker tag docker-registry:5000/openshiftdemos/oc openshiftdemos/oc

git clone https://github.com/isnuryusuf/openshift-cd-demo
cd openshift-cd-demo
oc new-project dev --display-name="Tasks - Dev"
oc new-project stage --display-name="Tasks - Stage"
oc new-project cicd --display-name="CI/CD"

oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n dev
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n stage

oc process -f cicd-template.yaml | oc create -f -

```

- Jika proses deploy fail karena timeout pada saat menunggu downloading docker images, lakukan perintah di bawah ini
```
oc delete pod install-gogs
oc delete pods -l app=gogs
oc process -f cicd-template.yaml | oc create -f -
```
- Nexus Repository
user admin / pass admin123

- Sonarqube 
user admin / pass admin


---
# Additional Lab - OpenShift "oc cluster up" Wrapper script
```
git clone https://github.com/openshift-evangelists/oc-cluster-wrapper
echo 'PATH=$HOME/oc-cluster-wrapper:$PATH' >> $HOME/.bash_profile
echo 'export PATH' >> $HOME/.bash_profile
sudo $HOME/oc-cluster-wrapper/oc-cluster completion bash > /etc/bash_completion.d/oc-cluster.bash
source .bash_profile
oc-cluster list
oc-cluster up devops --public-hostname=192.168.1.178
oc-cluster list
oc-cluster status 
```



##########################################################################################
################ END #########################################
##########################################################################################
---
```
yum install -y ansible pyOpenSSL python-cryptography python-lxml
git clone https://github.com/isnuryusuf/installcentos.git
vim installcentos/inventory.erb
git clone https://github.com/openshift/openshift-ansible
systemctl stop docker ; mv /var/lib/docker /var/lib/docker.bak
yum -y remove docker-ce ; yum -y install docker ; systemctl start docker ; systemctl enable docker
hostnamectl set-hostname console.docker-host.lan
vim /etc/hosts 
add:
<IP-VM-ANDA>	console.docker-host.lan
ansible-playbook -i installcentos/inventory.erb ./openshift-ansible/playbooks/byo/config.yml
```
```
