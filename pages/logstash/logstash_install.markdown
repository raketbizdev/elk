---
layout: page
title: Logstash Installation
permalink: /logstash/
parent: Logstash
nav_order: 1
---

# Logstash Installation in On-Premise RHEL

## Prerequisite
* vim or nano
* Netcat or Telnet (for testing)
* ssh access
* jumphost access
* vpn access token

## Port open
* 22
* 5044
* 9200
* 9092


## Instruction

1. Download logstash RPM package. If youre using jumphost download the rpm package then upload to the jumphost server.
    ```
    wget https://artifacts.elastic.co/downloads/logstash/logstash-8.0.0-x86_64.rpm
    ```
2. After download verify by running command below.

    ```
    shasum -a 512 logstash-8.0.0-x86_64.rpm 
    ```
3. Install logstash.

    ```
     sudo rpm --install logstash-8.0.0-x86_64.rpm
    ```
4. Run the following command to installed to start logstash.

    ```
    sudo systemctl daemon-reload
    sudo systemctl enable logstash
    sudo system start logstash
    ```
5. Once logstash started test the logstash if working correctly.

    ```
    sudo systemctl status logstash
    ```
    Output should be something like this.
    ![logstash status](/images/logstash_status.png)

## Connect Logstash to Elasticsearch and View Kibana Dashboard.

1. Go to the `conf.d` directory inside logstash

    ```
    cd /etc/logstash/conf.d
    ```
    Create a test file `logstash-sample.conf`

    ```
    touch logstash-sample.conf
    ```

    Add this sample file

    ```
    input {
        stdin {
        }
    }

    output {
        stdout {
            codec => rubydebug
        }
    }
    ```

    After creating test config run following command

    ```
    /usr/share/logstash/bin/logstash -f logstash-sample.conf
    ```

    Output should be something the image below check the highlight in blue and the last line of the logs says pipeline is running.
    ![Logstash Test](/images/logstash_test.png)

    At the bottom of the logs type anything there will be an output similar to the image below.
    ![logstash_hello](/images/logstash_hello.png)

2. To connect to Logstash to Elasticsearch check the elasticsearch if accessible using [netcat](/kibana/#setting-up-kibana) or telnet command then modify the `logstash-sample.conf` output

    ```
    output {
      elasticsearch {
        hosts => ["http://your.ip.addr.ss:9200"]
        index => "custom-index-name"
        #user => "elastic"
        #password => "change-me"
      }
    }
    ```
    Run the command again to test again.

    ```
    /usr/share/logstash/bin/logstash -f logstash-sample.conf
    ```

    Then Check kibana dashboard by going to `Stack Management Dasboard` then click the `Index management` under `Data`.
    ![kibana_index_management](/images/kibana_index_management.png)

    Create index pattern under kibana nav
    ![custom_index_pattern](/images/custom_index_pattern.png)

    After creating index patter visit analytics and then click `Discover`.
    ![kibana_discover](/images/kibana_discover.png)

    If everything is working according to the plan your ready to create a visualization dashboard.