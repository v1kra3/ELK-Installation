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
  Paste the beow xpack modules in Begin security and auto configuration:
```  
xpack.monitoring.collection.enabled: true
xpack.security.audit.enabled: true
xpack.license.self_generated.type: "basic"
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

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
3. Delete the # sign at the beginning of the following lines to activate them:
```
#server.port: 5601
#server.host: "Your-system-IP"
#serverpublicBaseUrl: "https://Your-system-IP:5601"
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
./elasticsearch-create-enrollment-token -s kibana
```
**Open kibana in browser using http and paste the enrollment token**

8. Generate Verification code using below command:
```
cd /usr/share/kibana/bin
./kibana-verification-code
```
9. Enter the verification code and login:
```
username: elastic
Password: (which we copied elastic built-in superuser password) 
```
10. TLS communication between browser to kibana(follow the below command):
```
cd /usr/share/elasticsearch/
./bin/elasticsearch-certutil csr -name (Name) -dns (Name), System IP
Ex ./bin/elasticsearch-certutil csr -name elk -dns elk, 192.168.10.89
ls  you will able to see csr-bundle
unzip csr-bundle.zip
./bin/elasticsearch-certutil ca -pem
unzip elastic-stack-ca.zip
```
**Move certificate to kibana folder and configure.(Follow below command)**
```
mkdir /etc/kibana/certs
cp /usr/share/elasticsearch/ca/* /etc/kibana/certs/
```
11. Generate kibana encryption key using below command:
```
cd /usr/share/kibana/bin/
./kibana-encryption-keys generate
```
*Note: Copy the all xpack key which will be located under settings* 

12. Kibana configuration command:
```
nano /etc/kibana/kibana.yml
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
systemctl restart kibana.service
systemctl enable kibana.service
```
***Kibana has been installed and configured***
