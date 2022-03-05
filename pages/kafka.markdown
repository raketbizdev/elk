---
layout: page
title: Kafka
permalink: /kafka/
nav_order: 6
---

# Kafka Installation in On-Premise RHEL

## Prerequisite
* vim or nano
* Netcat or Telnet (for testing)
* ssh access
* jumphost access
* tar
* vpn access token
* Java

## Port open
* 22
* 9092
* 9093

## Instruction

Before you install Kafka you need to install [java](/java/). To check if java is installed run this command

```
java --version
```

1. Download kafka tar package. If youre using jumphost download the rpm package then upload to the jumphost server.
    ```
    wget https://dlcdn.apache.org/kafka/3.1.0/kafka_2.13-3.1.0.tgz
    wget https://downloads.apache.org/kafka/3.1.0/kafka_2.12-3.1.0.tgz.sha512
    ```
2. After download verify by running command below.

    ```
    shasum -a 512 kafka_2.12-3.1.0.tgz.sha512
    ```

3. Before unpacking kafka package make sure the file is  in the `/opt` directory.

    ```
    mv kafka_2.13-3.1.0.tgz /opt
    ```

4. Unpack kafka using tarball.

    ```
     sudo tar -xvf kafka_2.13-3.1.0.tgz
    ```
5. Create symlink 

    ```
    ln -s /opt/kafka_2.13-3.1.0 /opt/kafka
    ```

5. Create kafka user and change kafka permission.

    ```
    sudo useradd kafka
    sudo chown -R kafka:kafka /opt/kafka*
    ```

6. Create a kafka service

    ```
    touch /etc/systemd/system/kafka.service
    ```

    Open the  kafka service created from above

    ```
    vi /etc/systemd/system/kafka.service
    ```
    Then Add the code below to `kafka.service` file.

    ```
    [Unit]
    Description=Apache Kafka
    Requires=zookeeper.service
    After=zookeeper.service

    [Service]
    Type=simple

    User=kafka
    Group=kafka

    ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
    ExecStop=/opt/kafka/bin/kafka-server-stop.sh

    [Install]
    WantedBy=multi-user.target
    ```
7. Create a zookeeper service

    ```
    touch /etc/systemd/system/zookeeper.service
    ```

    Open the `zookeeper.service` created from above

    ```
    vi /etc/systemd/system/zookeeper.service
    ```

    Then Add the code below to `zookeeper.service` file.

    ```
    [Unit]
    Description=zookeeper
    After=syslog.target network.target

    [Service]
    Type=simple

    User=kafka
    Group=kafka

    ExecStart=/opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties
    ExecStop=/opt/kafka/bin/zookeeper-server-stop.sh

    [Install]
    WantedBy=multi-user.target
    ```

8. After everything is put in place time to run the `kafka` and `zookeeper`
    ```
    systemctl daemon-reload
    systemctl enable zookeeper
    systemctl enable kafka

    systemctl start zookeeper
    systemctl stop kafka
    systemctl start kafka

    ```

9. Once kafka started test the kafka if working correctly.

    ```
    sudo systemctl status kafka
    ```
    Output should be something like this.
    ![kafka status](/images/kafka_status.png)

10. Test also if Zookeeper is working

    ```
    sudo systemctl status zookeeper
    ```
    Output should be something like this.
    ![zookeeper status](/images/zookeeper_status.png)

## Creating your first topic

1. Create your first topic

    ``` 
    /opt/kafka/bin/kafka-topics.sh --create --partitions 1 --replication-factor 1 --topic test-events --bootstrap-server localhost:9092
    ```
2. listing topic

    ```
    /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
    ```
3. Run the producer topic

    ```
    /opt/kafka/bin/kafka-console-producer.sh --topic test-events --bootstrap-server localhost:9092
    ```

4. To test the message you have to run the consumer

    ```
    /opt/kafka/bin/kafka-console-consumer.sh --topic test-events --from-beginning --bootstrap-server localhost:9092
    ```

## Sent Data or push data from Kafka to Logstash

Now that you have the topics created and running. Time for us to send this data to **logstash** to **elasticsearch** to **kibana**. But before that you need to check if the kafka port is accessible form logstash server. Test connectivity using netcat.

```
nc -zv your.ip.addr.ss 9092
```


1. Open `server.properties` in the `/opt` directory

    If you are using kafka cluster
    ```
    vi /opt/kafka/config/server.properties
    ``

    If you are using stand-alone kafka 
    ```
    vi /opt/kafka/config/connect-standalone.properties
    ``
2. Add IP address server properties

    If you are using kafka cluster 
    ```
    From:
    31 #listeners=PLAINTEXT://localhost:9092
    36 #advertised.listeners=PLAINTEXT://localhost:9092
    40 #listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
    #bootstrap.servers=localhost:9092

    To:
    31 #listeners=PLAINTEXT://your.ip.addr.ss:9092
    36 #advertised.listeners=PLAINTEXT://your.ip.addr.ss:9092
    40 #listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
    ```

    If you are using stand-alone kafka
    ```
    From:
    #bootstrap.servers=localhost:9092

    To:
    bootstrap.servers=your.ip.addr.ss:9092
    ```

## Connect Kafka to Logstash

1. First Create a topic
2. Go to Logstash server and create config files. 
3. Under input add the kakfka settings

    ```
    input {
        kafka{
            codec => json
            bootstrap_servers => "kaf.ka.ip.add:9092"
            topics => ["pjl-prosperna-log"]
        }
    }
    output {
        stdout {
            codec => rubydebug
        }
    }
    ```
4. Run the logstash command to test

    ```
    /usr/share/logstash/bin/logstash -f your-logstash.conf
    ```
5. Go back to Kafka server and run the Kafka event.

     ```
    /opt/kafka/bin/kafka-console-producer.sh --topic test-events --bootstrap-server localhost:9092
    ```
6. Start typing any text and check logstash for the results.

