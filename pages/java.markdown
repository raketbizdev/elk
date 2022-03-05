---
layout: page
title: Java
permalink: /java/
nav_order: 7
---

# Installing Java in On-Premise RHEL


Latest Elasticsearch, Logstash and Kibana dont need Java as they have built in java package installed however Kafka need latest java version.

1. make sure you have port 22 access to your server.
2. Download the java dependency first and install

   1. If your server has access to the internet.
      ```
      yum install java-17-openjdk -y
      ```
   2. If you are using on-prem and use jumphost you have to download manually the java package.
      ```
      wget http://mirror.centos.org/centos/8-stream/AppStream/x86_64/os/Packages/java-17-openjdk-devel-17.0.1.0.12-2.el8_5.x86_64.rpm

      dnf install java-17-openjdk
      ```
   3. Once Java installed you need to test java
      ```
      java --version

      output: 
        openjdk 17.0.2 2022-01-18 LTS
        OpenJDK Runtime Environment 21.9 (build 17.0.2+8-LTS)
        OpenJDK 64-Bit Server VM 21.9 (build 17.0.2+8-LTS, mixed mode, sharing)
      ```