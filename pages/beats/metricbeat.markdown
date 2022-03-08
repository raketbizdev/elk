---
layout: page
title: Metricbeat
permalink: /beats/metricbeat
parent: Beats
nav_order: 2
---


# Metricbeat Installations in RHEL environment


1. Download metricbeat tar package

    ```
    wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-8.0.0-x86_64.rpm
    ```
2. Install the metricbeat

    ```
    rpm --install metricbeat-8.0.0-x86_64.rpm
    ```

    After install start the metricbeat.

    ```
    sudo systemctl daemon-reload
    sudo systemctl enable metricbeat
    sudo systemctl start metricbeat
    ```

3. Test the metricbeat

    ```
    sudo systemctl status metricbeat
    ```
    Output is something like this.
    ![metricbeat status](/elk_nehr/images/filebeat_status.png)

## Setup metricbeat connect to Kafka.

1. Modify the `metricbeat.yml` file

    ```
    cd /etc/metricbeat/
    ```
    Open `metricbeat.yml`
    ```
    vi metricbeat.yml
    ```
2. Add kafka connection under line 204

    ```
    # ------------------------------ Kafka Output -------------------------------

    output.kafka:
    # initial brokers for reading cluster metadata
    hosts: ["your.ip.addr.ss:9092"]
    topic: "metricbeat-54-173-199-118"
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
    ![Log Modules](/elk_nehr/images/log_modules.png)
    
    If an application is not part of the module disable the module directory in `line 99` and add the custom path below `line 74`

