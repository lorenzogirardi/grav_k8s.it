---
title: 'Kubernetes-nstats'
date: '2021-02-22 18:18'
taxonomy:
    tag:
            - docker
            - grafana
            - iftop
            - influxdb
            - kubernetes
            - network
            - observability
            - pod
            - sidecar
---

Here we go ... another weird sidecar container

## Motivations

  
  
I've always been interested in the observability area , there are many aspect that improve performances and fix bugs  
One of the most interested aspect is te network usage.

This is not related to "network issue"  
[![network_issue](/user/images/kubernetes-nstats/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3234302f76313631333938303238392f6d6973632f6e6574776f726b69737375652e706e67)](<https://camo.githubusercontent.com/2f639fd02add51a9e1529a68b56a830b12ea328791f0ae671771c5db650dcccd/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3234302f76313631333938303238392f6d6973632f6e6574776f726b69737375652e706e67>)  
  


It's related to the usages

You are probably habit to see something like this for your vms  
[![vm_net](/user/images/kubernetes-nstats/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938303835362f6d6973632f766d5f6e65742e706e67)](<https://camo.githubusercontent.com/722f34ecd7e011074527d76ce43b8c3eb28dba06876c311d8900ae1d9f518a17/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938303835362f6d6973632f766d5f6e65742e706e67>)  
That is showing the traditional IN and OUT  
  
  


An now ... with kubernetes you can have the same related to your pod  
[![pod_net](/user/images/kubernetes-nstats/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938313333382f6d6973632f706f645f6e65742e706e67)](<https://camo.githubusercontent.com/7051fc44a11f2cc78bd547e91bfba0218d9dd7aba2082de32ddb8475cdb1ad0f/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938313333382f6d6973632f706f645f6e65742e706e67>)  
And again you have the IN and OUT  
  
  


But **where** this bandwidth is be used ?

Answer is not easy , i mean

  * you can profile the application
  * you can profile the vm/pod
  * you can have a dedicated APM
  * you can have installed a [service mesh](https://github.com/lorenzogirardi/kubernetes-servicemesh)  
  




Service mesh is a good tool but for the reasons explained in the link ... you should promote it in the right way ... it's able to cover my question but let's assume it's a overengineering for my porpose.

APM ... well it depends on your company/money capability

What is missing ?  
Well even if we are in 2021 i'm habit to use **iftop** to understand the usage , the limit is that is working in rountime and i miss a long vision term.  
[![iftop](/user/images/kubernetes-nstats/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313631333938373038352f6d6973632f6966746f702e706e67)](<https://camo.githubusercontent.com/72309f9a4ee7e1f4c6b64f784c375d764d6b73385ea107d01e132e55b1f7e7a9/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313631333938373038352f6d6973632f6966746f702e706e67>)

  
  
  
  


## GOALs

  
  


  * Monitor a kubernetes pod network with a sidecar container
  * Be able to know src-dst of the pod connections
  * Use it as sidecar
  * Try to imagine a win-win solution (aka quick and dirty)



  
  
  
  


## Implementation

Project https://github.com/lorenzogirardi/Kubernetes-nstats  
  
My colleagues has done an amazing work with a GO container able to have this kind of observability.  
I tried to imagine a prototype with a win-win solution and i started evaluation this interesting [project](https://github.com/scottmsilver/iftop-telegraf-influx),  
where most the work as already done with the following steps:

  * Create an iftop static dump
  * Filter the results in a matrix
  * Build an influxdb layout to POST directly to the database



so ... let's share some evidence

Kubernetes-nstats  
|-- Dockerfile  
|-- README.md  
|-- cron.sh  
|-- crontab  
|-- format.py  
`-- parse.awk

### Dockerfile
    
    
    FROM debian:stretch-slim
    MAINTAINER lgirardi <[[email protected]](</cdn-cgi/l/email-protection>)>
    
    RUN apt-get -y update && apt-get -yq install \
    	iftop \
    	python3 \
    	cron \
    	curl
    
    
    RUN touch /var/log/cron.log
    RUN mkdir /code
    WORKDIR /code
    ADD . /code/
    RUN chmod +x /code/cron.sh
    COPY crontab /etc/crontab
    RUN crontab /etc/crontab
    CMD env > /code/env.sh ; cron -f
    

CRON ?!?!?! ... yes it's a prototype and for this scope k8s cronjob are not effective. The most interesting part is `env > /code/env.sh` that is used to create an environment file based on the docker environment variables.

### Code

cron.sh
    
    
    #!/bin/bash
    /usr/sbin/iftop -nNb -i $(grep IFACE /code/env.sh |cut -d= -f2) -s 10 -o 10s -t -L 100 2>/dev/null |/usr/bin/awk -f /code/parse.awk |/usr/bin/python3 /code/format.py |/usr/bin/curl -i -XPOST 'http://'"$(grep INFLUX /code/env.sh |cut -d= -f2)"'/write?db='"$(grep IDB /code/env.sh |cut -d= -f2)"'' --data-binary @-
    

parse.awk
    
    
    #!/bin/awk -f
    BEGIN {
    	numlist = 0
    	nblines = 15
    }
    {
    	if ( numlist == 1 && $1 == "--------------------------------------------------------------------------------------------" ) {
    		exit
    	}
    
    	if ( numlist == 0 && $1 == "--------------------------------------------------------------------------------------------" ) {
    		numlist = 1
    		next
    	}
    
    	if ( numlist == 1 ) {
    		if ( $0 ~ "=>" && nblines > 0 ) {
    			SENDER = $2
    			STX = pfFormat($5)
    			getline
    			RECEIVER = $1
    			RTX = pfFormat($4)
    			printf "%s,%s,%s,%s\n", SENDER, RECEIVER, RTX, STX
    			nblines--
    			if ( nblines < 1 ) {
    				exit
    			}
    		}
    		next
    	}
    }
    END {
    }
    
    function pfFormat(str) {
     	sub("b","",str)
    	return str
    }
    

format.py
    
    
    #!/usr/local/bin/python3
    
    import csv
    import socket
    import sys
    import re
    
    def getHostName(ipAddress):
    	hostName = ipAddress
    
    	try:
    		hostName = socket.gethostbyaddr(ipAddress.strip())[0]
    	except socket.herror:
    		pass
    
    	return hostName
    
    def prefixToMultiplier(prefix):
    	multiplier = {
    		'K': 1000,
    		'M': 1000000,
        'G': 1000000000
    	}
    
    	return multiplier.get(prefix, 1)
    
    
    def expandBitRate(bitRate):
    	groups = re.match(r"(\d+\.?\d*)(?:(K|M|G)?)", bitRate).groups()
    	multiplier = 1.0
    	if len(groups) > 1:
    		multiplier = prefixToMultiplier(groups[1])
    
    	value = float(groups[0])
    	return value * multiplier
    
    host = socket.gethostname()
    
    with sys.stdin as csvfile:
    	csvReader = csv.reader(csvfile)
    	for row in csvReader:
    		(senderIp, receiverIp, receiveRate, sendRate) = (row[0], row[1], expandBitRate(row[2]), expandBitRate(row[3]))
    		sender = getHostName(senderIp)
    		receiver = getHostName(receiverIp)
    		print("nstat,hosts=" + host +",sender=" + sender + ",receiver=" + receiver + " sendRate=" + str(sendRate) + ",receiveRate=" + str(receiveRate))
    

crontab
    
    
    SHELL=/bin/bash
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
    * * * * *  sh -x /code/cron.sh >> /var/log/cron.log 2>&1
    #
    

  
  
So that this stuff is doing ?

`/usr/sbin/iftop -nNb -i $(grep IFACE /code/env.sh |cut -d= -f2) -s 10 -o 10s -t -L 100 2>/dev/null` on this part is defined a 10 second of dump sorted on the last 10second column.

[![iftop_dump](/user/images/kubernetes-nstats/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938373739312f6d6973632f6966746f705f64756d702e706e67)](<https://camo.githubusercontent.com/ec94b65472cf70ce7e0e6024aa4433bee5eccb53f02c6a4c69554598958fa5cc/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938373739312f6d6973632f6966746f705f64756d702e706e67>)

at this point the awk parsing `| /usr/bin/awk -f /code/parse.awk`

[![iftop_awk](/user/images/kubernetes-nstats/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938373739312f6d6973632f6966746f705f61776b2e706e67)](<https://camo.githubusercontent.com/ca6cc436234b92705223f0d117a4b01a3bca8f109258e229b8bee0ceeced85fa/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938373739312f6d6973632f6966746f705f61776b2e706e67>)

the format part done by python script `| /usr/bin/python3 /code/format.py`  
[![iftop_format](/user/images/kubernetes-nstats/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938373739312f6d6973632f6966746f705f666f726d61742e706e67)](<https://camo.githubusercontent.com/2159b3ceb19d51de4e7a8cc95fc909cc9b3e65f7476129209e3a9f5c60b28784/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313631333938373739312f6d6973632f6966746f705f666f726d61742e706e67>)

and finally we ship the metrics to influxdb with `| /usr/bin/curl -i -XPOST 'http://$IP/write?db=$DB' --data-binary @-`

  
  
  
  


## Results

You can build an run locally `docker build -t nstats .`  
`docker run -d -e IFACE=eth0 -e INFLUX=192.168.1.28:8086 -e IDB=test nstats`

or add into kubernetes in an existing pod  
it is quite simple and doesn't need any refactoring
    
    
          containers:
          - image: lgirardi/py-test-backend
            imagePullPolicy: Always
            name: pytbak
        etc etc etc ....
          - env:
            - name: IFACE
              value: eth0
            - name: INFLUX
              value: 192.168.1.28:8086
            - name: IDB
              value: test
            image: lgirardi/nstats
            imagePullPolicy: Always
            name: nstats
    

  
  
What you should have on your grafana reflect the network usages

[![iftop_grafana](/user/images/kubernetes-nstats/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313631343031333731352f6d6973632f6966746f705f67726166616e612e706e67)](<https://camo.githubusercontent.com/4613ad0a5c58a0c87db124ead36ff00cad611599b58314fb61cea6442163bbf0/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313631343031333731352f6d6973632f6966746f705f67726166616e612e706e67>)
