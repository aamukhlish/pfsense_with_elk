# Integrating pfSense with ELasticsearch, Logstash, and Kibana (ELK Stack)

**Prerequisites:**
1. Ubuntu server 16.04
2. pfSense
3. Elasticsearch
4. Kibana 
5. Logstash
6. Nginx
7. Java

**Install Java**

Login as root and install java
```
sudo -i
apt-get update
apt-get -y install oracle-java8-installer
```

**Elasticsearch**

Import the Elasticsearch public GPG key into APT
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
Add the Elastic source list to the sources.list.d directory, where APT will look for new sources
```
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```
After that, update the package lists so APT will read the new Elastic source
```
apt-get update
```
Then, elasticsearh can be installed with this command:
```
apt-get install elasticsearch -y
```
After installed, edit the main configuration file. allow only localhost that can access the elasticsearch by uncomment the network.host and replace the value with localhost
```
network.host: localhost
```
Next, start and enable the elasticsearch service
```
systemctl start elasticsearch
systemctl enable elasticsearch
```
**Logstash**

Install logstash using this command
```
apt-get install logstash
```
Create a patterns directory for Geo_IP
```
cd /etc/logstash/conf.d
mkdir patterns
```
Create pfsense grok file, and fill with the pfsense2-2.grok file
```
cd /etc/logstash/conf.d/patterns
nano pfsense2-2.grok
```
Download and unzip the GEO IP Database
```
cd /etc/logstash
curl -O "http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz"
gunzip GeoLiteCity.dat.gz
```
Then, continue to create the logstash conf files
1. Input file (fill with the 02-syslog-input.conf file)
```
nano /etc/logstash/02-syslog-input.conf
```
2. Syslog filter file (fill with the 02-syslog-filter.conf file)
```
nano /etc/logstash/02-syslog-input.conf
```
3. Syslog filter file (fill with the 20-syslog-filter.conf file)
```
nano /etc/logstash/20-syslog-filter.conf
```
4. pfSense filter file (fill with the 81-pfsense-filter.conf file)
```
nano /etc/logstash/81-pfsense-filter.conf
```
5. Elasticsearch output file (fill with the 99-elasticsearch-output.conf file)
```
nano /etc/logstash/99-elasticsearch-output.conf
```

**Kibana**

Install Nginx using this command :
```
apt-get install nginx
```
Install kibana using this command :
```
apt-get install kibana
```
After successfully installed, enable and start the kibana service
```
systemctl enable kibana
systemctl start kibana
```
Add authentication that stored in the htpasswd.users file
```
echo "your_user:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users
```
After that, create the Nginx server block files
```
nano /etc/nginx/sites-available/example.com
```
Link the file to sites-enabled directory to enable the new configuration
```
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```
After that, restart the Nginx service and allow firewall connection to Nginx
```
systemctl restart nginx
ufw allow 'Nginx Full'
```

**pfsense**

1. Login to pfsense
2. Go to Status -> System Logs -> Settings
3. Fill the server address int the Remote log servers. Use the 5140 port
4. Mark the Firewall Events in the Remote Syslog Contents
5. Save
6. Adjust firewall setting to allow and blokc connection

**Final steps**
Restart the ELK Service
```
systemctl restart elasticsearch.service
systemctl restart logstash.service
systemctl restart kibana.service
```
Login to Kibana by accessing the server address from browser, and configure the dashboard

**Credit to :**
1. https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elastic-stack-on-ubuntu-16-04
2. http://pfelk.3ilson.com/
3. https://www.reddit.com/r/PFSENSE/comments/5cppf0/elk_with_pfsense_23_working/