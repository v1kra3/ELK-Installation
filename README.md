# ELK-Installation 8.6

**The Elastic Stack** — formerly known as the ELK Stack — is a collection of open-source software produced by Elastic which allows you to search, analyze, and visualize logs generated from any source in any format, a practice known as centralized logging. Centralized logging can be useful when attempting to identify problems with your servers or applications as it allows you to search through all of your logs in a single place. It’s also useful because it allows you to identify issues that span multiple servers by correlating their logs during a specific time frame.

**The Elastic Stack has four main components:**

**Elasticsearch**: A distributed search and analytics engine built on Apache Lucene. Elasticsearch has quickly become the most popular search engine and is commonly used for log analytics, full-text search, security intelligence, business analytics, and operational intelligence use cases

**Logstash**: Logstash is an open-source data ingestion tool that allows you to collect data from a variety of sources, transform it, and send it to your desired destination.

**Kibana**: Kibana is a data visualization and exploration tool used for log and time-series analytics, application monitoring, and operational intelligence use cases. It offers powerful and easy-to-use features such as histograms, line graphs, pie charts, heat maps, and built-in geospatial support.

**Beats**: lightweight, single-purpose data shippers that can send data from hundreds or thousands of machines to either Logstash or Elasticsearch.

## Prerequisites

- A Linux system running Ubuntu 22.04 server with 4GB RAM and 2 CPU
- Access to a terminal window/command line (Search > Terminal)
- A user account with sudo or root privileges
- Installation of OpenJDK 11
- Recommend you have a static IP set for your system

Note that the amount of CPU, RAM, and storage that your Elasticsearch server will require depends on the volume of logs that you expect.


# Installation and configuration
## Add Elastic Repository

1. Enter the following into a terminal window to import the PGP key for Elastic:

```
wget  -qO -  https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add
```
2. Install the apt-transport-https package from APT repository:
```
sudo apt-get install apt-transport-https
```
3. Add the Elastic repository to your system’s repository list:
```
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee –a /etc/apt/sources.list.d/elastic-8.x.list
```
## Installing and configuring Elasticsearch
1. Prior to installing Elasticsearch, update the repositories by entering:
```
sudo apt-get update
```
2. Install Elasticsearch with the following command:
```
sudo apt-get install elasticsearch
```
3. After the installation, copy the elastic built-in superuser password: 

![image](https://user-images.githubusercontent.com/121095320/211773111-f6be06ea-a8dc-4b5d-8cb7-039526b20b74.png)

4. Elasticsearch is now installed and ready to be configured using nano:
```
sudo nano /etc/elasticsearch/elasticsearch.yml
```
5. You should see a configuration file with several different entries and descriptions. Scroll down to find the following entries in network portion:
```
#network.host: 
#http.port: 9200
```
  Uncomment the lines by deleting the hash (#) sign at the beginning of both lines and replace the IP:
```
network.host: localhost (or system IP)
http.port: 9200
```
  Paste the beow xpack modules in Begin security and auto configuration and save the yml file:
```  
xpack.monitoring.collection.enabled: true
xpack.security.audit.enabled: true
xpack.license.self_generated.type: "basic"
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```
![image](https://user-images.githubusercontent.com/121095320/212024739-37918298-393c-4e38-8ac5-c5d678b19775.png)


6. Start the Elasticsearch service by running a systemctl command:
```
sudo systemctl start elasticsearch.service
```
*It may take some time for the system to start the service. There will be no output if successful.*

7. Enable Elasticsearch to start on boot:
```
sudo systemctl enable elasticsearch.service
```
8. Check the status of Elasticsearch:
```
sudo systemctl status elasticsearch.service
```
**Open elasticsearch in browser using https**
```
username: elastic
Password: (which we copied elastic built-in superuser password) 
```
***Elasticsearch has been installed and configured***

## Installing and configuring Kibana

1. Install Kibana with the following command:
```
sudo apt-get install kibana
```
2. Kibana is now installed and ready to be configured using nano:
```
sudo nano /etc/kibana/kibana.yml
```
3. Delete the # sign at the beginning of the following lines and enter the details to activate them and save the yml file:
```
#server.port: 5601
#server.host: "Your-system-IP"
#serverpublicBaseUrl: "https://Your-system-IP:5601"
#sercer.name: "hostname"
```
4. Start the kibana service by running a systemctl command:
```
sudo systemctl start kibana.service
```
*It may take some time for the system to start the service. There will be no output if successful.*

5. Enable kibana to start on boot:
```
sudo systemctl enable kibana.service
```
6. Check the status of kibana:
```
sudo systemctl status kibana.service
```
7. Create an enrollment token using below command and copy the token:
```
cd /usr/share/elasticsearch/bin/
sudo ./elasticsearch-create-enrollment-token -s kibana
```
**Open kibana in browser using http and paste the enrollment token and click configure elastic**

![image](https://user-images.githubusercontent.com/121095320/212025124-3983e360-afdd-4d3b-b873-07b9ac52d9c5.png)

8. Generate Verification code using below command:
```
cd /usr/share/kibana/bin
sudo ./kibana-verification-code
```
9. Enter the generated the verification code and login using:
```
username: elastic
Password: (which we copied elastic built-in superuser password) 
```
![image](https://user-images.githubusercontent.com/121095320/212025373-b62e1851-7a19-45df-97ed-dff1bf71d522.png)

10. TLS communication between browser to kibana(follow the below command):
```
cd /usr/share/elasticsearch/
sudo ./bin/elasticsearch-certutil csr -name (Name) -dns (Name), System IP
Ex sudo ./bin/elasticsearch-certutil csr -name elk -dns elk, 192.168.10.89
sudo ls  you will able to see csr-bundle
sudo unzip csr-bundle.zip
sudo ./bin/elasticsearch-certutil ca -pem
sudo unzip elastic-stack-ca.zip
```
**Move certificate to kibana folder and configure.(Follow below command)**
```
sudo mkdir /etc/kibana/certs
sudo cp /usr/share/elasticsearch/ca/* /etc/kibana/certs/
sudo ls /etc/kibana/certs/   -verify wheather the file is available
```
11. Generate kibana encryption key using below command:
```
cd /usr/share/kibana/bin/
sudo ./kibana-encryption-keys generate
```
*Note: Copy the all xpack key which will be located under settings* 

12. Kibana configuration command:
```
sudo nano /etc/kibana/kibana.yml
```
Find the System(Kibana server) and delete the hash(#) and mention the certificate path
```   
server.ssl.enabled: false (Change to true)
server.ssl.certificate: /etc/kibana/certs/ca.crt (path)
server.ssl.key: /etc/kibana/certs/ca.key (Path)

paste the xpack key in bottom of the kibana.yml
```
![image](https://user-images.githubusercontent.com/121095320/211802787-33a566c5-47a2-4a4a-b5fa-fc6540c77624.png)
12. Restart/Enable kibana to start on boot
```
sudo systemctl restart kibana.service
sudo systemctl enable kibana.service
```
***Kibana has been installed and configured***

## Installing and configuring Beats
The Elastic Stack uses several lightweight data shippers called Beats to collect data from various sources and transport them to Logstash or Elasticsearch. Here are the Beats that are currently available from Elastic:

- [Filebeat](https://www.elastic.co/beats/filebeat) : collects and ships log files.
- [Metricbeat](https://www.elastic.co/beats/metricbeat) : collects metrics from your systems and services.
- [Packetbeat](https://www.elastic.co/beats/packetbeat) : collects and analyzes network data.
- [Winlogbeat](https://www.elastic.co/beats/winlogbeat) : collects Windows event logs.
- [Auditbeat](https://www.elastic.co/beats/auditbeat) : collects Linux audit framework data and monitors file integrity.
- [Heartbeat](https://www.elastic.co/beats/heartbeat) : monitors services for their availability with active probing.

In this tutorial we will use Filebeat to forward local logs to our Elastic Stack

1. Install filebeat with the following command:
```
sudo apt-get install filebeat
```
2. Configure Filebeat with the following command:
```
sudo nano /etc/filebeat/filebeat.yml
```
3. You should see a configuration file with several different entries and descriptions. Scroll down to find the following entries:
```
#setup.dashboards.enabled: false  (In Dashboard portion)
output.elasticsearch:             (In Elasticsearch portion)
  hosts: ["localhost:9200"]
  #protocol: "https"
 
  #username: "elastic"
  #password: "password"
```
 uncomment the lines by deleting the hash (#) sign at the beginning and change the dashboard.enabled true.
 And Edit the following in Elasticsearch portion.
```
setup.dashboards.enabled: true  (In Dashboard portion)
output.elasticsearch:           (In Elasticsearch portion)
  hosts: ["Configured - IP:9200"]
  protocol: "https"
 
  username: "elastic"
  password: "Password of elasticsearch"
  ssl.verification_mode: "none"
```
  Edit and add the following command in Kibana portion in filebeat.yml file.
```
  host: ["https://configuredIP:5601"]       (In kibana portion) 
  username: "elastic"
  password: "Password of elasticsearch"
  ssl.verification_mode: "none"
```
4. Filebeat logs check command:
```
sudo filebeat setup -e
```
5. To check the modules in filebeat:
```
sudo filebeat modules list
```
***Note: Based on your requirement, you can enable the modules. The functionality of Filebeat can be extended with*** [Filebeat modules](https://www.elastic.co/guide/en/beats/filebeat/8.6/filebeat-modules.html).

6. In this tutorial we will use the system module, which collects and parses logs created by the system logging service
```
filebeat modules enable system
nano /etc/filebeat/modules.d/system.yml
Enable the syslog and authentication status from false to true
```
7. Start and enable Filebeat and check the status using:
```
sudo systemctl start filebeat
sudo systemctl enable filebeat
sudo systemctl status filebeat
```
***filebeat has been installed and configured***

## Installing and configuring Logstash
1. Install Logstash by running the following command:
```
sudo apt-get install logstash
```
2. Start and enable logstash and check the status using:
```
sudo systemctl start logstash
sudo systemctl enable logstash
sudo systemctl status logstash
```
3. Configure Logstash: 
```
Logstash is a highly configurable component of the ELK stack. 
Once installed, configure its INPUT, FILTERS, and OUTPUT pipelines to your specific needs.
/etc/logstash/conf.d/ contains all custom Logstash configuration files.
```
![image](https://user-images.githubusercontent.com/121095320/212022816-1c011e2e-0f78-4d06-a664-1ab3c55fb415.png)

For more information kindly check [logstash](https://www.elastic.co/guide/en/logstash/8.6/setup-logstash.html)

# Conclusion
**You now know how to set up the Elastic Stack to gather and examine system logs. We advise identifying your demands so that you can begin customising ELK**

***Additional Reference Materials***

Fore more detailed information on ELK, visit the Elastic configuration guides below:

- [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)
- [Kibana](https://www.elastic.co/guide/en/kibana/current/install.html)
- [Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html)
- [Beats](https://www.elastic.co/guide/en/beats/libbeat/current/beats-reference.html)
