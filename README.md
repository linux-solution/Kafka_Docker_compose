# Kafka_Docker_compose
```
yml configuration file for kafka docker composer
```

## Docker Install Step
### Uninstall Docker Completely
```
dpkg -l | grep -i docker
sudo apt-get purge -y docker-engine docker docker.io docker-ce  
sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce 
sudo rm -rf /var/lib/docker
sudo rm /etc/apparmor.d/docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

### Install Docker
```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add –
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce
sudo systemctl status docker
```

### Set user for Docker
```
sudo usermod -aG docker administrator
su – administrator
id –nG
```

### Install kafkacat
```
brew install kafkacat
```

## Run kafka
### Run kafka full-stack
```
docker-compose -f full-stack.yml up
```
** Note that when you run kafka full-stack yml, navigate to correct directory where the full-stack.yml file is placed. **

### Stop kafka full-stack
```
docker-compose -f full-stack.yml down
```

### Run Producer in other terminal by kafkacat
```
Kafkacat –P –b localhost:9092 –t test
```

### Run Customer in other terminal by kafkacat
```
Kafkacat –C –b localhost:9092 –t test
```

## What was the issue?

• In kafkacat command there is an option –b.

This option is for listening/binding address and for our testing environment, the binding address is localhost or 127.0.0.1

• And no need to create specified Topic.

The topic is automatically created when we create the Producer with option –t and Customer can connect to this topic.

• When we run the “docker-compose –f full-stack.yml up”, need to make sure that previously downed this composer.

So, when we can see an error in composer log after run “docker-compose –f full-stack.yml up”, you can repair this by simply run “docker-compose –f full-stack.yml down”.

* The most important thing is to install Docker correctly.*

Please don’t miss to install docker dependency packages correctly and as important as User for the Docker.
Explained at the top steps above.
 
## New Issue
• Can not read records in Topics in Kafka UI

When customer connect on UI via port 8000, if click one topic, can not read records in that topic even a few records are there as well as a lot of topic.

### Reason:
Because kafka Rest Proxy has a bug.

Actually we request up to MAX_BYTES from REST Proxy, so it will not return more messages than needed.

You can also tweak RECORD_POLL_TIMEOUT because for a large topic, the REST Proxy will take too long to return entries from the start of the topic.

The bug you hit though, probably comes from REST Proxy itself as it has a default timeout of just 1000ms, which is way too small.

In order to fix this, you should set consumer.request.timeout.ms=30000 in the REST Proxy. Why 30000? Because REST Proxy has a bug (we have reported it upstream) and treats the consumer.request.timeout.msthe same as request.timeout.ms which are actually two different values in the REST Proxy context.

Have a look here as well: https://github.com/Landoop/kafka-topics-ui/tree/master/docker#kafka-rest-proxy-configuration

Also you can test your kafka cluster response times yourself, using the kafka-console-consumer command:

time kafka-console-consumer --bootstrap-server PLAINTEXT://broker.url:9092 --topic topic.name --from-beginning --max-messages 100

### Fix:
Add some parameters in the configuration file “full-stack.yml”.

We need to add some environmental variables to fix the Kafka Rest Proxy bug.

In kafka-rest-proxy:
```
KAFKA_REST_CONSUMER_REQUEST_TIMEOUT_MS: 30000
```
* This is for the case of kafka rest proxy is not respoding *
In kafka-topic-ui:
```
MAX_BYTES: 50000
   		RECORD_POLL_TIMEOUT: 5000
		This is the case of there are a lot of kafka records.
```
In Addition:

We run the kafka very simply using the full-stack.yml configuration file.

All config data are stored in this yml file so that we can modify to change the kafka.
```
docker-compose -f full-stack.yml down
Sudo nano full-stack.yml
	*** edit ***
docker-compose -f full-stack.yml up –d
```

### Reference
https://github.com/Landoop/kafka-topics-ui/issues/107
https://github.com/Landoop/kafka-topics-ui/issues/72
https://github.com/Landoop/kafka-topics-ui/issues/86
