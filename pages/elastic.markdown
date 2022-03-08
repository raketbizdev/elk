---
layout: page
title: Elasticsearch
permalink: /elasticsearch/
grand_parent: Home
nav_order: 1
---

# Installing Elasticsearch in On-Premise RHEL

## Prerequisite
* vim or nano
* Netcat or Telnet (for testing)
* ssh access
* jumphost access
* vpn access token

## Port open
* 22
* 9200
* 9300

## Instruction

1. Download elasticsearch RPM package. If youre using jumphost download the rpm package then upload to the jumphost server.
    ```
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.0.0-x86_64.rpm
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.0.0-x86_64.rpm.sha512
    ```
2. After download verify by running command below.

    ```
    shasum -a 512 -c elasticsearch-8.0.0-x86_64.rpm.sha512 
    ```
3. Install elasticsearch.

    ```
    sudo rpm --install elasticsearch-8.0.0-x86_64.rpm
    ```
4. Run the following command to installed to start elasticsearch.

    ```
    sudo systemctl daemon-reload
    sudo systemctl enable elasticsearch
    sudo system start elasticsearch
    ```
5. Once elasticsearch started test the elasticsearch if working correctly.

    ```
    curl -XGET "localhost:9200"
    ```
    output should be something like this.
    ```
        {
        "name" : "User.local",
        "cluster_name" : "elasticsearch_admin",
        "cluster_uuid" : "sI5IbibXSxOLLgQnj_Qx2w",
        "version" : {
            "number" : "8.0.0",
            "build_flavor" : "default",
            "build_type" : "tar",
            "build_hash" : "6fc81662312141fe7691d7c1c91b8658ac17aa0d",
            "build_date" : "2021-12-02T15:46:35.697268109Z",
            "build_snapshot" : false,
            "lucene_version" : "8.10.1",
            "minimum_wire_compatibility_version" : "6.8.0",
            "minimum_index_compatibility_version" : "6.0.0-beta1"
        },
        "tagline" : "You Know, for Search"
        }
    ```
    Also check the browser if its accessible. If not make sure port `9200` is open.
    ![elasticsearch](/elk_nehr/images/elasticsearch.png)

## How to Setup elasticsearch after installation

1. Goto elasticsearch directory
    ```
    cd /etc/elasticsearch
    ```
2. Open `elasticsearch.yml` file.

    ```
    vi elasticsearch.yml
    ```
3. Uncomment the following settings inside `elasticsearch.yml`.

    ```
    #http.port: 9200
    #cluster.name: my-application
    #node.name: node-1
    #network.host: 192.168.0.1
    #discovery.seed_hosts: ["host1", "host2"]
    #cluster.initial_master_nodes: ["node-1", "node-2"]
    ```
4.  Uncomment the `cluster.name` if you have multiple nodes there is only on cluster.
5.  Uncomment the `network.host` and change the value to `[_local_, _site_]`
6.  Uncomment the `node.name` Node name is the name of your current node.
7.  Uncomment the  `discovery.seed_hosts` Add the main`IP` you use for the cluster.
8.  Uncomment the  `cluster.initial_master_nodes` then add the value of your `node.name` and every node in the cluster

## Set the Security Settings

1. Add the following setting at the bottom of `elasticsearch.yml`

    ```
    # ---------------------------------- Security -----------------------------------
    #
    xpack.security.enabled: true
    xpack.security.transport.ssl.enabled: true

    ```
2. Restart the elasticsearch

    ```
    sudo systemctl restart elasticsearch
    ```
3. Run the command below.

    ```
    sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
    ```

    output should be something like below. Copy and save it somewhere safe as you need this when you want to connect to Kibana.

    ```
    Changed password for user apm_system
    PASSWORD apm_system = ZuqOHaAD2VFIufhu6Cs0

    Changed password for user kibana_system
    PASSWORD kibana_system = DEq9IOwqammi4PTFdVKB

    Changed password for user kibana
    PASSWORD kibana = DEq9IOwqammi4PTFdVKB

    Changed password for user logstash_system
    PASSWORD logstash_system = Aiqh4d1lVSU3mMq2iesj

    Changed password for user beats_system
    PASSWORD beats_system = UcMtL27OuXThWQqvt4M7

    Changed password for user remote_monitoring_user
    PASSWORD remote_monitoring_user = M7HuB1qkYOBzi4W1ECjH

    Changed password for user elastic
    PASSWORD elastic = MUXNVnepRoMSq1qU66ke
    ```
4. Then retest again with slightly different approach.
    
    ```
    curl -XGET "your.ip.addr.ss:9200" -u username:password
    ```
    
    where `username` us the elastic and `password` is the password of elastic