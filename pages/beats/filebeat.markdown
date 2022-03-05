---
layout: page
title: Filebeat
permalink: /beats/
parent: Beats

nav_order: 1
---

# Filebeat Installations in RHEL environment


1. Download filebeat tar package

    ```
    wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.0.0-x86_64.rpm
    ```
2. Install the filebeat

    ```
    rpm --install filebeat-8.0.0-x86_64.rpm
    ```

    After install start the filebeat.

    ```
    sudo systemctl daemon-reload
    sudo systemctl enable filebeat
    sudo systemctl start filebeat
    ```

3. Test the Filebeat

    ```
    sudo systemctl status filebeat
    ```
    Output is something like this.
    ![Filebeat status](/images/filebeat_status.png)

## Setup Filebeat connect to Kafka.

1. Modify the `filebeat.yml` file

    ```
    cd /etc/filebeat/
    ```
    Open `filebeat.yml`
    ```
    vi filebeat.yml
    ```
2. Add kafka connection under line 204

    ```
    # ------------------------------ Kafka Output -------------------------------

    output.kafka:
    # initial brokers for reading cluster metadata
    hosts: ["your.ip.addr.ss:9092"]
    topic: "filebeat-54-173-199-118"
    # message topic selection + partition
    codec.format:
        string: "%{[@timestamp]} %{[message]}"
    partition.round_robin:
        reachable_only: false
    
    required_acks: 1
    compression: gzip
    max_message_bytes: 1000000
    ```
3. Activate Modules depends on app you want log to collect.
    ![Log Modules](/images/log_modules.png)
    
    If an application is not part of the module disable the module directory in `line 99` and add the custom path below `line 74`

4. Test Filebeat connectivity to kafka 

    ```
    filebeat test output
    ```
    
    Output should be something like this

    ```
    Kafka: 54.227.242.218:9092...
        parse host... OK
        dns lookup... OK
        addresses: 54.227.242.218
        dial up... OK
    ```
