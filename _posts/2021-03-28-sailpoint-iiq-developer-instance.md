---
title: SailPoint IIQ Developer Instance
---

# Summary
Many Platform as a service products offer free developer instances like ServiceNow and Salesforce, however Sailpoint IIQ is not there today. So if you need to do some discovery or development, you have to roll your own. Fortunately there are some easy options.

# Option 1 - identity works repo
The repo found [here](https://git.identityworksllc.com/pub/sailpoint-docker)  makes it pretty simple to build out a full development instance. First you have to head over to [https://community.sailpoint.com/](https://community.sailpoint.com/)  to download the software (identityiq-8.1.zip & identityiq-8.1p2.jar & 1_ssb-v6.1.zip). This one is ideal for a shared developer space on a server.

The script below outlines how to build it using the 8.x release of IIQ
```
###Example on Ubuntu 20

#install prereqs
apt install docker.io docker-compose git smbclient unzip ant openjdk-8-jdk git-lfs -y

#copy files from windows share to needed directories
cd /opt
mkdir iiqbuild
cd iiqbuild
smbclient '//cifshare/e$' -c 'lcd .; cd sailpoint; get identityiq-8.1.zip' -U administrator%password123
smbclient '//cifshare/e$' -c 'lcd .; cd sailpoint; get identityiq-8.1p2.jar' -U administrator%password123
smbclient '//cifshare/e$' -c 'lcd .; cd sailpoint; get ssb-v6.1.zip' -U administrator%password123

#clone repo
git clone https://git.identityworksllc.com/pub/sailpoint-docker.git

#optionally modify the docker-compose.yml file to:
#-use mysql 
#-newer release of mssql 2019
#-exclude unwanted components (openldap/phpldapadmin/unused db)

cd sailpoint-docker
./build.sh -z ../identityiq-8.1.zip 

## other options
# patch 8.1 does not like mssql - https://community.sailpoint.com/t5/IdentityIQ-Server-Software/IdentityIQ-8-1p2/ta-p/182114
#./build.sh -z ../identityiq-8.1.zip -p ../identityiq-8.1p2.jar

#bring up containers
docker-compose up -d

#to watch the server come up
docker logs sailpoint-docker_iiq_1 --follow

#to access sailpoint iiq spadmin/admin
#http://ipaddress:8080/identityiq

#to access traefik load balancer
#http://ipaddress:28080/dashboard/

#to access mailhog
#http://ipaddress:8025

#to connect to the db
#point your sql client to ipaddress:3306 or ipaddress:1433

#to shut it down and remove the containers
docker-compose down

```
# Option 2 - UberEther repo
The repo found [here](https://github.com/UberEther/standalone-docker-sailpoint-iiq): 
First you have to head over to [https://community.sailpoint.com/](https://community.sailpoint.com/) to download the software (identityiq-8.1.zip & identityiq-8.1p2.jar & 1_ssb-v6.1.zip). This one is ideal fora personal desktop but could also be used for a shared space if you don't mind hardcoding host files.

The script below outlines how to build it using the 8.x release of IIQ
```
requirement
#https://community.sailpoint.com/ to download the software (identityiq-8.1.zip & identityiq-8.1p2.jar & 1_ssb-v6.1.zip)
###Example on Ubuntu 20
apt install docker.io docker-compose git smbclient unzip openjdk-8-jdk  git-lfs -y

cd /opt
sudo git clone https://github.com/UberEther/standalone-docker-sailpoint-iiq.git

#copy files from windows share to needed directories
cd standalone-docker-sailpoint-iiq
sudo smbclient '//cifshare/e$' -c 'lcd ssb/components/iiq8.1/base/ga; cd sailpoint; get identityiq-8.1.zip' -U administrator%password123
sudo smbclient '//cifshare/e$' -c 'lcd ssb/components/iiq8.1/base/patch; cd sailpoint; get identityiq-8.1p2.jar' -U administrator%password123
sudo smbclient '//cifshare/e$' -c 'lcd ssb/components/ssb-v6.1; cd sailpoint; get ssb-v6.1.zip' -U administrator%password123

#test
cd ssb
sudo ./build.sh

#####create containers###
cd ../uedocker/

#to fix create script
sudo sed -i 's/^DB_CONTAINER_NAME.*/DB_CONTAINER_NAME=$(docker-compose ps -q db)/g' create.sh
sudo sed -i 's/^APP_CONTAINER_NAME.*/APP_CONTAINER_NAME=$(docker-compose ps -q app)/g' create.sh

#permissions on volume
chown -R 1000:1000 volumes

#load it up
sudo ./bootstrap.sh

#reminder
echo "###Remember to set your host file to 192.168.1.x dev.icam.local###"

#to access sailpoint iiq spadmin/admin
#https://dev.icam.local/identityiq

#########to cleanup###################
#docker kill $(docker ps -q)
#docker container prune
#docker image prune --all
#or
#cd ../uedocker/
#docker-compose down
```
