---
title: Zabbix Alerts and ServiceNow Event Management
---

# Summary
[https://docs.servicenow.com/bundle/washingtondc-it-operations-management/page/product/event-management/task/t_EMConfigureZabbixConnector.html](https://docs.servicenow.com/bundle/washingtondc-it-operations-management/page/product/event-management/task/t_EMConfigureZabbixConnector.html)

Zabbix is a common agent/server on-premise monitoring product that is often used for virtual machine or web monitoring. ServiceNow can be configured to  **pull**  Alerts from Zabbix, including the ability to auto clear Alerts when Events are updated.

**Other notable points**

-   Zabbix also  [develops a push integration](https://www.zabbix.com/integrations/servicenow) to push into Incident records using the Table API.
-   It is also possible to  [develop a custom webhook](https://www.zabbix.com/documentation/current/en/manual/config/notifications/media/webhook) in Zabbix and take advantage of ServiceNow's inbound push connector API.

# Creating a Demo Zabbix Server with Docker-Compose

Zabbix provides Docker images in their  [github repo](https://github.com/zabbix/zabbix-docker). We can run a temporary Zabbix server on a Linux host with Docker installed.

Starting from a Linux host:
1.  Install Docker and Docker compose
```
######Add Docker's official GPG key
sudo apt-get update
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
#####Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install git docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
#####Add current user to docker group and re-evaluate session 
sudo usermod -aG docker $USER
su - $USER
```

2.  Download Zabbix Docker Repo and run compose file
```
git clone https://github.com/zabbix/zabbix-docker.git
cd zabbix-docker
docker compose -f ./docker-compose_v3_alpine_mysql_latest.yaml --profile full up -d
```
3.  Login to host on http://hostname. Default username/password is Admin/zabbix.
4.  Test connectivity to the Zabbix API.  **THIS IS IMPORTANT!** [Zabbix has recently changed their API but not their documentation.](https://www.zabbix.com/forum/zabbix-troubleshooting-and-problems/464787-zabbix-api-end-point-zabbix-api_jsonrpc-php-gives-file-not-found)Traditionally the API was found at  **hostname/zabbix/api_jsonrpc.php**, however in the current 7.x release it is found at  **hostname/api_jsonrpc.php.**  
      
    
    You can test with curl to ensure you get a response back.      
```
curl -H "Content-Type: application/json" -X POST http://hostname/api_jsonrpc.php -d'
{
"jsonrpc": "2.0",
"method": "user.login",
"params": {
"username": "Admin",
"password": "zabbix"},
"id": 1
}'
```	

Now that we have a running Zabbix host we can connect ServiceNow to it. The current supported method is pull and requires a MID Server that has connectivity to the Zabbix API and a user with a role that includes Zabbix API access.

# Setting up ServiceNow to retrieve Zabbix Problem Alerts

[https://docs.servicenow.com/bundle/washingtondc-it-operations-management/page/product/event-management/task/t_EMConfigureZabbixConnector.html](https://docs.servicenow.com/bundle/washingtondc-it-operations-management/page/product/event-management/task/t_EMConfigureZabbixConnector.html)

1.  Install Event Management Core/sn_em_ai plugin  
2.  Install the Event Management Connectors/sn_em_connector plugin
3.  Open discovery_credentials.list, and create a new "Basic Auth Credential" that holds the Zabbix API access user/pass.
4.  Open, All > Event Management > Integrations > Connector Instances.
    -   name: Zabbix Server
    -   connector definition: zabbix_v2 (newest at the time of this writing)
    -   host IP: IP of zabbix server
    -   credential: Zabbix Basic Auth Credential created earlier
    -   add your mid server in the list at the bottom
        
5.  Save record
6.  In the connector instance values, update protocol to http since we are not using https in the docker zabbix example
7.  Save record

Now you can try testing using the Test Connector UI action, but I expect it will fail with a 404. This goes back to Zabbix updating their API path.To adjust this in ServiceNow, open the  **ecc_agent_script_include**  table and update  **line 669** of the script include  **ZabbixV2_JS**  so that it does not use  **/zabbix** in the path.
![]({{ 'assets/images/zabbix_sn_fix.png' | relative_url }})
Now return to the connector and click "Test Connector".

# Creating an Event/Alert from Zabbix

To create an alert, stop the zabbix agent docker container and wait for 3 minutes. Alternatively you can configure an alert to trigger when a file is shows up - [https://aaronsaray.com/2020/zabbix-test-notification/](https://aaronsaray.com/2020/zabbix-test-notification/) 

```
docker stop zabbix-docker-zabbix-agent-1

#to later auto close the alert from Zabbix
#docker start zabbix-docker-zabbix-agent-1
```

**Result**
![]({{ 'assets/images/zabbix_sn_alert.png' | relative_url }})
