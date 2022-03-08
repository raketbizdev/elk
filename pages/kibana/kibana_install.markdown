---
layout: page
title: Kibana Installation
permalink: /kibana/
parent: Kibana
nav_order: 1
---

# Kibana Installation in On-Premise RHEL

## Prerequisite
* vim or nano
* Netcat or Telnet (for testing)
* ssh access
* jumphost access
* vpn access token
* nginx

## Port open
* 22
* 80
* 5601

## Instruction

1. Download Kibana RPM package. If youre using jumphost download the rpm package then upload to the jumphost server.
    ```
    wget https://artifacts.elastic.co/downloads/kibana/kibana-8.0.0-x86_64.rpm
    ```
2. After download verify by running command below.

    ```
    shasum -a 512 kibana-8.0.0-x86_64.rpm 
    ```
3. Install Kibana.

    ```
     sudo rpm --install kibana-8.0.0-x86_64.rpm
    ```
4. Run the following command to installed to start Kibana.

    ```
    sudo systemctl daemon-reload
    sudo systemctl enable Kibana
    sudo system start Kibana
    ```
5. Once Kibana started test the Kibana if working correctly.

    ```
    sudo systemctl status kibana
    ```
    Output should be something like this.
    ![kibana status](/elk_nehr/images/kibana_status.png)


## Install Nginx so Kibana can be viewed in the browser

Make sure your'e in the `root` directory before downloading 

    sudo su


1. Download Nginx RPM package. If your'e using jump host download the rpm package then upload to the jump host server.
    
    ```
    wget https://nginx.org/packages/rhel/7/x86_64/RPMS/nginx-module-njs-1.20.2%2B0.7.2-1.el7.ngx.x86_64.rpm

    ```

2. Install Nginx

    ```
    sudo rpm --install nginx-module-njs-1.20.2%2B0.7.2-1.el7.ngx.x86_64.rpm
    ```
3.  After installation setup the nginx config
    
    ```
    cd /etc/nginx/conf.d
    ```
    Create config file

    ```
    touch kibana.conf

    ```
    paste the command settings below

    ```
    server {
            listen 80;
            server_name your.ip.addr.ss;
            location / {
                proxy_pass http://localhost:5601;
                proxy_http_version 1.1;
                proxy_set_header Upgrade \$http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host \$host;
                proxy_cache_bypass \$http_upgrade;        
            }
        }
    ```

4. After the setup test the nginx if work correctly

    ```
    sudo nginx -t
    sudo systemctl start nginx
    ```
    Then check the nginx status by running the command below

    ```
    sudo systemctl status nginx
    ```
    Output should be something like this

    ![Nginx Status](/elk_nehr/images/nginx_status.png)

## Setting up Kibana

Before setting up kibana to be accessible by browser make sure that you test if port `9200` is open and the elasticsearch server is accessible by testing connectivity using netcat or telnet. 

Testing using netcat
```
nc -zv your.ip.addr.ss 9200
```

The Output should be something like this

```
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Connected to your.ip.addr.ss:9200.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.
```

If theres no error proceed to setup the `kibana.yml` file


1. Open `kibana.yml` file kibana file is located at the `/etc/kibana` directory.

    ```
    vi /etc/kibana/kibana.yml
    ```
2. Change or uncomment the following code

    ```
    #server.port: 5601
    #server.host: "localhost"
    #server.basePath: ""
    #elasticsearch.hosts: ["host"]
    #server.name: "acn-elk"
    #elasticsearch.username: "kibana"
    #elasticsearch.password: "change-me"
    ```
    Uncomment `server.port` in line 2 keep the port 5601
    ```
    server.port: 5601
    ```

    Uncomment and modify the `server.host` in line 7. 

    ```
    From"
    #server.host: "localhost"

    To:
    server.host: "your.ip.addr.ss"
    ```
    Uncomment `server.basePath` in line 13 and add the dns or ip with the http protocol

    ```
    From:
    #server.basePath: ""

    To:
    server.basePath: "http://your.ip.addr.ss"
    ```
    


    Uncomment `#server.name` and add the server name in line 29.

    ```
    From:
    #server.name: "your-hostname"

    To:
    server.name: "your-new-hostname"
    ```

    Connect the elasticsearch server to kibana in line 32

    ```
    From:
    #elasticsearch.hosts: ["host"]

    To:
    elasticsearch.hosts: ["your.ip.addr.ss"] 
    #If you are operating in a elasticsearch cluster add all the Ip addresses in the hosts.

    ```

    Then also change the `elasticsearch.username` and `elasticsearch.password`  in lin3 45 and 46 from the username and password generated from elasticsearch see [Elasticsearch settings](/elasticsearch/#set-the-security-settings)


    ```
    #elasticsearch.username: "elastic"
    #elasticsearch.password: "change-me"
    ```

3. After the kibana setup restart kibana and check status if its working.

    ```
    sudo systemctl restart kibana
    ```

4. Test kibana if viewable in the terminal.

    ```
    http://your.ip.addr.ss
    ```

    Output should be similar to this
    ![Kibana Dashboard](/elk_nehr/images/kibana.png)

5. Then enter the the elasticsearch username and password provide you save from setting up elasticsearch.