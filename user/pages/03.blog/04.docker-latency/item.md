---
title: 'Docker-latency'
date: '2020-08-11 20:42'
taxonomy:
    tag: []
---

## [](<https://github.com/lorenzogirardi/docker-latency#aka-the-network-blaming-tool>)aka the network blaming tool

So again another grafana stack with docker  
Well yes but with a precise scope

In this period we are almost all working from home,  
the blaming topic is usually the connection with our offices or the datacenters.

Is not so rare for a network Administrator hear people that sais ,  
_the vpn is slow_ ,  _i cannot connect to ... $something_ , bla bla bla

In my experience this is usually due to the quality of the provider,  
sometimes is also a problem on route path on T2/T3 providers

### [](<https://github.com/lorenzogirardi/docker-latency#how-we-can-undestand-if-our-network-is-really-slow->)HOW we can undestand if our network is really slow ?

The idea is to start a grafana stack ready-made to handle the basics statistics of our internet connection.  
We need to choose some endpoints to monitor, example , your vpn endpoint , your datacenter/office public ip , the main dns servers and so on

#### [](<https://github.com/lorenzogirardi/docker-latency#requirements>)Requirements

  * Docker
  * Docker Compose



#### [](<https://github.com/lorenzogirardi/docker-latency#stack>)Stack

  * Influxdb
  * Grafana
  * Telegraf



#### [](<https://github.com/lorenzogirardi/docker-latency#tree>)Tree
    
    
    ├── .env
    ├── Makefile
    ├── README.md
    ├── docker
    │   ├── grafana
    │   │   ├── Dashboard-PING.json
    │   │   ├── dashboard.yaml
    │   │   └── datasource.yaml
    │   ├── influxdb
    │   │   ├── influxdb.conf
    │   │   
    │   └── telegraf
    │       └── telegraf.conf
    ├── docker-compose.yml
    

Makefile is ... well a makefile , commands allowed  
 _up , down, dev, down, logs, clean_  
up is to startup the stack  
down to shutdown clean is done to remove also the storage saved for influxdb and grafana

.env contains the grafana and influxdb credentials (yes the default password is quite complicated)  
Since this tool is hosted in your laptop (could be everywhere), never mind the  _security_
    
    
    GRAFANA_USER=admin
    GRAFANA_PASSWORD=EQyFJpjxvJG8k2K8
    INFLUXDB_DOMAIN=influxdb
    INFLUXDB_DATABASE=ping
    

### [](<https://github.com/lorenzogirardi/docker-latency#configuration>)Configuration

We just need to choose the endpoints we'd like to monitor from our internet connection This could be done editing  _telegraf.conf_
    
    
    [global_tags]
    [agent]
      interval = "10s"
      round_interval = true
      metric_batch_size = 1000
      metric_buffer_limit = 10000
      collection_jitter = "0s"
      flush_interval = "10s"
      flush_jitter = "0s"
      precision = ""
      hostname = "local-telegraf"
      omit_hostname = false
    [[outputs.influxdb]]
       urls = ["http://127.0.0.1:8086"]
       database = "ping"
    [[inputs.ping]]
    urls = ["1.1.1.1", "8.8.8.8", "208.67.222.222", "test1.velocable.com"]
    count = 7
    ping_interval = 1.0
    

Edit  _urls =_ adding / modify the endpoints  
(in this example, Cloudflare dns , Google dns, opendns, and a server in Madrid used for speedtest)

The configuration is collecting information every 10 seconds , and run a ping command 7 time each with 1 second delay.

### [](<https://github.com/lorenzogirardi/docker-latency#startup>)Startup

Inside the main folder run

`make up`

output:
    
    
    docker-latency$ make up
    docker-compose -f docker-compose.yml up -d
    Creating network "docker-latency_default" with the default driver
    Creating grafana  ... done
    Creating influxdb ... done
    Creating telegraf ... done
    

login to:  
`http://localhost:3000/ `admin/EQyFJpjxvJG8k2K8

you will see

[![grafana_home](/user/images/docker-latency/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313538353634313235322f6d6973632f67726166616e615f686f6d652e706e67)](<https://camo.githubusercontent.com/56bea14df7a89b07f4e319b45ad4d27c39ad865f/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313538353634313235322f6d6973632f67726166616e615f686f6d652e706e67>)

than , checking for the only board present -->  _internet latency_

you will have all details about the endpoint chosen , packet loss especially

[![grafana_ping](/user/images/docker-latency/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313538353539353832342f6d6973632f67726166616e615f70696e672e706e67)](<https://camo.githubusercontent.com/5fd87b371e34b4497018003fcd41cd2e09d23c72/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313538353539353832342f6d6973632f67726166616e615f70696e672e706e67>)

100% packet loss simulated disabling network card for few seconds.  
The dashboard is using variables in order to create 1 row for each endpoint.

### [](<https://github.com/lorenzogirardi/docker-latency#conclusion>)Conclusion

Now we have data, so we know what is going on in our internet connection and we can probably  
have more details about the  _infomagic_ words like ...  _is slow_
