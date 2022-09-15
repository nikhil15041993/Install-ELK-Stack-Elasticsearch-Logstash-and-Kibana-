# Install-ELK-Stack-Elasticsearch-Logstash-and-Kibana


The ELK stack is a set of applications for retrieving and managing log files.

It is a collection of three open-source tools, Elasticsearch, Kibana, and Logstash. The stack can be further upgraded with Beats, a lightweight plugin for aggregating data from different data streams.

## Step 1: Install Dependencies
Install Java
The ELK stack requires Java 8 to be installed. Some components are compatible with Java 9, but not Logstash.

Note: To check your Java version, enter the following:

java -version

If you don’t have Java 8 installed, install it by opening a terminal window and entering the following:

```
sudo apt-get install openjdk-8-jdk
```


## Step 2: Add Elastic Repository
Elastic repositories enable access to all the open-source software in the ELK stack. To add them, start by importing the GPG key.

1. Enter the following into a terminal window to import the PGP key for Elastic:

```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

2. Next, install the apt-transport-https package:
```
sudo apt-get install apt-transport-https
```

3. Add the Elastic repository to your system’s repository list:
```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee –a /etc/apt/sources.list.d/elastic-7.x.list
```

## Step 3: Install Elasticsearch

1. Prior to installing Elasticsearch, update the repositories by entering:

```
sudo apt-get update
```

2. Install Elasticsearch with the following command:

```
sudo apt-get install elasticsearch
```
### Configure Elasticsearch

1. Elasticsearch uses a configuration file to control how it behaves. Open the configuration file for editing in a text editor of your choice. We will be using nano:

```
sudo nano /etc/elasticsearch/elasticsearch.yml
```

2. You should see a configuration file with several different entries and descriptions. Scroll down to find the following entries:

```
#network.host: 192.168.0.1

#http.port: 9200
```

3. Uncomment the lines by deleting the hash (#) sign at the beginning of both lines and replace 192.168.0.1 with localhost.

It should read:

```
network.host: server ip

http.port: 9200
```

4. Just below, find the Discovery section. We are adding one more line, as we are configuring a single node cluster:

```
discovery.type: single-node
```

5. By default, JVM heap size is set at 1GB. We recommend setting it to no more than half the size of your total memory. Open the following file for editing:

```
sudo nano /etc/elasticsearch/jvm.options
```

6. Find the lines starting with -Xms and -Xmx. In the example below, the maximum (-Xmx) and minimum (-Xms) size is set to 512MB.

### Start Elasticsearch
1. Start the Elasticsearch service by running a systemctl command:

```
sudo systemctl start elasticsearch.service
```

It may take some time for the system to start the service. There will be no output if successful.

2. Enable Elasticsearch to start on boot:

```
sudo systemctl enable elasticsearch.service
```
### Test Elasticsearch
Use the curl command to test your configuration. Enter the following:

```
curl -X GET "localhost:9200"
```
## Step 4: Install Kibana

It is recommended to install Kibana next. Kibana is a graphical user interface for parsing and interpreting collected log files.

1. Run the following command to install Kibana:

```
sudo apt-get install kibana
```

2. Allow the process to finish. Once finished, it’s time to configure Kibana.

Configure Kibana
1. Next, open the kibana.yml configuration file for editing:

```
sudo nano /etc/kibana/kibana.yml
```

2. Delete the # sign at the beginning of the following lines to activate them:

```
#server.port: 5601

#server.host: "your-hostname"

#elasticsearch.hosts: ["http://localhost:9200"]
```

The above-mentioned lines should look as follows:

```
server.port: 5601

server.host: "0.0.0.0"

elasticsearch.hosts: ["http://<elk ip>:9200"]

```

### Start and Enable Kibana

1. Start the Kibana service:

```
sudo systemctl start kibana
```

There is no output if the service starts successfully.

2. Next, configure Kibana to launch at boot:

```
sudo systemctl enable kibana
```

### Test Kibana
To access Kibana, open a web browser and browse to the following address:

```
http://localhost:5601
```

## Step 5: Install Logstash

Logstash is a tool that collects data from different sources. The data it collects is parsed by Kibana and stored in Elasticsearch.

Install Logstash by running the following command:

```
sudo apt-get install logstash
```

Start and Enable Logstash
1. Start the Logstash service:

```
sudo systemctl start logstash
```
2. Enable the Logstash service:

```
sudo systemctl enable logstash
```

3. To check the status of the service, run the following command:

```
sudo systemctl status logstash
```

## Step 6: Install Filebeat

Filebeat is a lightweight plugin used to collect and ship log files. It is the most commonly used Beats module. One of Filebeat’s major advantages is that it slows down its pace if the Logstash service is overwhelmed with data.

Install Filebeat by running the following command:

```
sudo apt-get install filebeat
```

### Configure Filebeat

Filebeat, by default, sends data to Elasticsearch. Filebeat can also be configured to send event data to Logstash.

1. To configure this, edit the filebeat.yml configuration file:

```
sudo nano /etc/filebeat/filebeat.yml
```

2. Under the Elasticsearch output section, comment out the following lines:

```
# output.elasticsearch:
   # Array of hosts to connect to.
   # hosts: ["localhost:9200"]
```

3. Under the Logstash output section, remove the hash sign (#) in the following two lines:

```
# output.logstash
     # hosts: ["localhost:5044"]
```
It should look like this:
```
output.logstash
     hosts: ["localhost:5044"]
```
4. Next, enable the Filebeat system module, which will examine local system logs:

```
sudo filebeat modules enable system
```

The output should read Enabled system.

5. Next, load the index template:

```
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
```

## Download and install Filebeat on Remote machine

### Download and install Filebeat


```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.17.0-amd64.deb
sudo dpkg -i filebeat-7.17.0-amd64.deb
```

### Edit the configuration

Modify /etc/filebeat/filebeat.yml to set the connection information:

```
output.elasticsearch:
  hosts: ["<es_url>"]
  username: "elastic"
  password: "<password>"
setup.kibana:
  host: "<kibana_url>"
  
```
Where password is the password of the elastic user, <es_url> is the URL of Elasticsearch, and <kibana_url> is the URL of Kibana

### enable and configure the  module

```
sudo filebeat modules enable <module name>
```

Modify the settings in the /etc/filebeat/modules.d/apache.yml file.
   
### Start Filebeat

The setup command loads the Kibana dashboards. If the dashboards are already set up, omit this command.

```
sudo filebeat setup
sudo service filebeat start   
```   

To test the filebeat connection use ``` sudo filebeat test output ```


## Enable user authentication

Install the required packages.


```
apt-get update
apt-get install curl mlocate jq
```
Test your communication with the ElasticSearch server.


```
curl -X GET "http://192.168.100.7:9200/_xpack/license"
```

Here is the command output.

```
Copy to Clipboard
{
  "name" : "elasticsearch.local",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "w5CUwsjPQPqW4Ne_04wuRg",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
Verify the license installed on the ElasticSearch server.

```
curl -X GET "http://192.168.100.7:9200/_xpack/license"
```
Here is the command output.

```
{
  "license" : {
    "status" : "active",
    "uid" : "9f3d50e7-4d3c-47ec-8011-6f6b1d1167c0",
    "type" : "basic",
    "issue_date" : "2020-04-22T00:46:28.831Z",
    "issue_date_in_millis" : 1587516388831,
    "max_nodes" : 1000,
    "issued_to" : "elasticsearch",
    "issuer" : "elasticsearch",
    "start_date_in_millis" : -1
  }
}
```

In our example, we have a basic license installed on the ElasticSearch server.

Enable the trial license on the ElasticSearch server.

```
curl -X POST "http://192.168.100.7:9200/_license/start_trial?acknowledge=true&pretty"
```

Here is the command output.

```
{
  "acknowledged": true,
  "trial_was_started": true,
  "type": "trial"
}
```
Verify the license installed on the ElasticSearch server.

```

curl -X GET "http://192.168.100.7:9200/_xpack/license"
```
The user authentication is not available on the ElasticSearch basic license.

Tutorial ElasticSearch - Configure the user authentication
Stop the ElasticSearch service.

```
systemctl stop elasticsearch
```
Edit the ElasticSearch configuration file named: elasticsearch.yml

```
vi /etc/elasticsearch/elasticsearch.yml
```
Add the following lines at the end of the file.

```
xpack.security.enabled: true
```
Here is the original file, before our configuration.


Start the ElasticSearch service.

```
systemctl start elasticsearch
```
Test your communication with the ElasticSearch server.

```
curl -X GET "http://192.168.100.7:9200/?pretty"
```
Here is the command output.

```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "security_exception",
        "reason" : "missing authentication credentials for REST request [/?pretty]",
        "header" : {
          "WWW-Authenticate" : "Basic realm=\"security\" charset=\"UTF-8\""
        }
      }
    ],
    "type" : "security_exception",
    "reason" : "missing authentication credentials for REST request [/?pretty]",
    "header" : {
      "WWW-Authenticate" : "Basic realm=\"security\" charset=\"UTF-8\""
    }
  },
  "status" : 401
}
```
The ElasticSearch server is requiring user authentication.

Set the password for the ElasticSearch internal accounts.

```
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```
Here is the command output.

```
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y
​
​
Enter password for [elastic]:
Reenter password for [elastic]:
Enter password for [apm_system]:
Reenter password for [apm_system]:
Enter password for [kibana]:
Reenter password for [kibana]:
Enter password for [logstash_system]:
Reenter password for [logstash_system]:
Enter password for [beats_system]:
Reenter password for [beats_system]:
Enter password for [remote_monitoring_user]:
Reenter password for [remote_monitoring_user]:
Changed password for user [apm_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```
Test your communication with the ElasticSearch server using the ELASTIC user account.

```
curl --user elastic -X GET "http://192.168.100.7:9200/?pretty"

```
Here is the command output.

```
{
  "name" : "elasticsearch.local",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "w5CUwsjPQPqW4Ne_04wuRg",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
You may choose to enter the user password on the command-line.

```
curl --user elastic:elastic123 -X GET "http://192.168.100.7:9200/?pretty"
```
