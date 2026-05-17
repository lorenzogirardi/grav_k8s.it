---
title: 'Kubernetes api gateway'
date: '2020-11-08 13:57'
taxonomy:
    tag:
            - api key
            - api-gateway
            - apigw
            - auth
            - kong
            - konga
            - kubernetes
            - rate limit
            - token
---

It's time to talk about the api gateway

In a modern infrastructure , especially in a microservices environment you probably know what i'm referring to ,however it's better to clarify some points.

_An API gateway takes all API calls from clients, then routes them to the appropriate microservice with request routing, composition, and protocol translation. Typically it handles a request by invoking multiple microservices and aggregating the results, to determine the best path. It can translate between web protocols and web‑unfriendly protocols that are used internally._

_An e‑commerce site might use an API gateway to provide mobile clients with an endpoint for retrieving all product details with a single request. It invokes various services, like product info and reviews, and combines the results_

  
  


[![apigw](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343833393633312f6d6973632f61706967772e706e67)](<https://camo.githubusercontent.com/70d257f92412994f23cfce0b702ca359556a2b64b7835f8f3ef9b58a493a4d60/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343833393633312f6d6973632f61706967772e706e67>)

  
  


Some months ago i discussed about the service mesh  
<https://github.com/lorenzogirardi/kubernetes-servicemesh>

To better understand the differences on those products it is better to summarize the common areas

[![apigw_vs_servicemesh](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343730333838302f6d6973632f61706967775f76735f736572766963656d6573682e706e67)](<https://camo.githubusercontent.com/43086b5aea0afa64170f3dac4261da1ad4a2ae2645d428fee16b7ce2eb6484f1/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343730333838302f6d6973632f61706967775f76735f736572766963656d6573682e706e67>)

So forget about service mesh and have the focus only on the apigw.

  
  


## GOAL

Needed:

  * Api must be public
  * Users needs to have authentication
  * An user could be rate limited
  * A service could be rate limited
  * Backend applications should not be changed
  * We need to interact as a code with the apigw  
  
  




Important:

  * Analytics
  * Transformation
  * Versioning
  * Circuit Breaker
  * gRPC support
  * Caching
  * Documentation



  
  


## Software opportunity

Around the api gateway world we have some possibility, some of those are onprem, some other in cloud or hybrid.

The onprem scenario we have some historycal software and a "new generation" born in a kubernetes architecture, howevere here we will check for a tool that could work in kubernetes (with a plus for supporting vm)

From the most knew i'd like to mention:

  * Kong
  * Tyk
  * Nginx
  * Ambassador
  * 3scale



On the other side on the cloud scenario we have:

  * Aws api gateway
  * Mashery (from Tibco)
  * Google apigee
  * Akana (hybrid)
  * Azure api management

| nginx| kong| tyk| ambassador| aws api gw  
---|---|---|---|---|---  
basic| yes| yes| yes| yes| pay  
oauth| pay| yes| yes| pay| pay  
jwt| pay| yes| yes| pay| pay  
ip listing| yes| yes| yes| no| pay  
analytics| yes| yes| yes| yes| pay  
rate limiting| yes| yes| yes| yes| pay  
transformation| pay| yes| yes| yes| pay  
grpc| yes| yes| yes| yes| no  
websocket| yes| yes| yes| yes| pay  
service mesh| no| yes| yes| pay| no  
k8s ingress| yes| yes| yes| yes| no  
caching| yes| pay| yes| no| pay  
documentation| pay| yes| yes| no| pay  
admin ui| pay| trd party| yes| no| pay  
ease to setup| 3.5| 4| 2.5| 3| 5  
  
Cloud

[![apigw_cloud](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343736363830322f6d6973632f61706967775f636c6f75642e706e67)](<https://camo.githubusercontent.com/6f7e05d1971da2d537452717a8571e3fc13ac8a7471428550e0cf065ca2997e1/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343736363830322f6d6973632f61706967775f636c6f75642e706e67>)

The advantage/disadvantage of cloud solution is the pay per use, you don't need to manage the infrastrcture but only the configuration and you have the support that can drive you on the right configuration.  
Another valid point of view is the  _DDOS_ attack that will not land to your platform but to a cloud infrastructure that is supposed was designed to manage this scenario.

PRO:

  * managed infrastructure
  * support
  * attacks mitigation



CONS:

  * pricing
  * often not customizable  
  




Onprem

[![apigw_dc](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343736363830322f6d6973632f61706967775f64632e706e67)](<https://camo.githubusercontent.com/f650f08e78f1fa640565179fa79ccedcb5928fc85554118e8b2cbb720bbea6e6/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343736363830322f6d6973632f61706967775f64632e706e67>)

Those solutions should be hosted on your platform or in a cloud to let the datacenter  _unknown_ (anyway you have to pay per traffic "*" ).  
Your engineer team need to know how it's working the infrastructure but they can also improve the process with automations

PRO:

  * open source
  * customizable
  * awareness on the complete funnel



CONS:

  * you have to take care about attacks
  * infrastructure knowhow



  
  
"*" you can easly have the apigw hosted onprem combined with a cdn/waf with ip lockdown , however you still have to pay the traffic towards it.

[![apigw_waf](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343736363830322f6d6973632f61706967775f7761662e706e67)](<https://camo.githubusercontent.com/e66b9fb512155cc277683660dd1ab517d7e9b9e50f023d3fc53f2b7e06a9f5e7/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343736363830322f6d6973632f61706967775f7761662e706e67>)

  
  


## Scenario

I tried some cloud solution like Aws and Mashery,  
i played also with some onprem solution, anyway i found out the right compromise with [Kong](https://konghq.com/kong/)

where compromise is :

  * **Kubernetes ingress**
  * **Api managed**
  * **Documentation**
  * Admin UI (konga for the open source)
  * Basic auth
  * **Apikey**
  * **Oauth**
  * **JWT**
  * ip listing
  * **Analytics**
  * **Rate limiting**
  * **Transformation**
  * gRPC
  * Websockets
  * Service mesh (~ nice to have)
  * **Easy to use**



  
  


## Infrastructure

  * kubernetes
  * backend application
  * kong api gateway
  * konga admin ui
  * waf (free cloudflare service)



  
  


What we need for this test is a backend api application, i created a prototype just to simulate different applications behind the apigw.

Here you can find out more details

<https://github.com/lorenzogirardi/py-test-backend>

briefly recap the application answer on /api/ with the main html page with methods

HTTP Method| URI| Action  
---|---|---  
GET| http://[hostname]/api/get/context| Retrieve list of context  
GET| http://[hostname]/api/get/context/[context_id]| Retrieve a context  
POST| http://[hostname]/api/post/context| Create a new context  
PUT| http://[hostname]/api/put/context/[context_id]| Update an existing context  
DELETE| http://[hostname]/api/delete/context/[context_id]| Delete acontext  
  
  
  


## Configuration

git repo --> <https://github.com/lorenzogirardi/Kubernetes-apigw>

Let's start to work...  
I've used the public kong deployment with some changes to have the database where store the configurations.

Another trick is create it as a secondary  _ingress_ , since i've already and ingress in my infrastructure , kong is done to answer on different ports

All the yaml are subdivided per role
    
    
    kong
    |-- 01-kong-ns.yaml
    |-- 02-kong-cr.yaml
    |-- 03-kong-sa.yaml
    |-- 04-kong-rbac.yaml
    |-- 05-kong-crb.yaml
    |-- 06-kong-svc.yaml
    |-- 07-kong-dpl.yaml
    |-- 08-kong-ss.yaml
    `-- 09-kong-job.yaml
    

some differences are related to the services node ports, deployment binding host and the database is now hosted with a volumeClaimTemplates
    
    
      ports:
      - name: proxy
        nodePort: 30080
        port: 80
        protocol: TCP
        targetPort: 8000
      - name: proxy-ssl
        nodePort: 30443
        port: 443
        protocol: TCP
        targetPort: 8443
      - name: admin
        port: 8100
        protocol: TCP
        targetPort: 8100
      - name: admin-ssl
        port: 8444
        protocol: TCP
        targetPort: 8444
      selector:
        app: ingress-kong
      type: LoadBalancer
    
    
    
          containers:
          - env:
            - name: KONG_DATABASE
              value: postgres
            - name: KONG_PG_HOST
              value: postgres
            - name: KONG_PG_PASSWORD
              value: kong
            - name: KONG_PROXY_LISTEN
              value: 0.0.0.0:8000, 0.0.0.0:8443 ssl http2
            - name: KONG_PORT_MAPS
              value: 80:8000, 443:8443
            - name: KONG_ADMIN_LISTEN
              value: 0.0.0.0:8444 ssl
            - name: KONG_STATUS_LISTEN
              value: 0.0.0.0:8100
    
    
    
      volumeClaimTemplates:
      - metadata:
          name: datadir
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 250Mi
    

Here what is expected to see
    
    
    $ kubectl get svc -n kong
    NAME                      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                    AGE
    kong-proxy                LoadBalancer   10.152.183.55    <pending>     80:30080/TCP,443:30443/TCP,8100:32588/TCP,8444:31793/TCP   7d
    kong-validation-webhook   ClusterIP      10.152.183.190   <none>        443/TCP                                                    7d
    postgres                  ClusterIP      10.152.183.177   <none>        5432/TCP
    

and the pods needed to have it online
    
    
    kong           ingress-kong-686b4bc5df-nm7zk              2/2     Running     4          7d
    kong           kong-migrations-ftgdv                      0/1     Completed   0          7d
    kong           postgres-0                                 1/1     Running     2          7d
    kube-system    hostpath-provisioner-5c65fbdb4f-f5qsb      1/1     Running     6          40d
    
    
    
    NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
    datadir-postgres-0   Bound    pvc-8c5791c3-939f-4457-b5fb-74ac417ca3d7   250Mi        RWO                 k8s-hostpath   7d
    

  
  


Now i'd like to have also an Admin UI, i'm not fan of web ui ,  
however i need to know if this interface could be shared outside IT  
to delegate some business ownership and so on.

Kong Enterprise has it's own web ui instead the community one needs a third party tool... [Konga](https://github.com/pantsel/konga)

In the same way of kong, konga needs a database , and in the same way i elaborated a bit the default deployment
    
    
    konga
    |-- 01-ns-konga.yaml
    |-- 02-svc-konga.yaml
    |-- 03-ing-konga.yaml
    `-- 04-dpl-konga.yaml
    

just a couple of TIPS

postgres persistent storage
    
    
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      namespace: konga
      name: konga-pv-claim
      labels:
        app: konga-storage-claim
    spec:
      storageClassName: microk8s-hostpath
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 200Mi
    

and on deployment ...
    
    
            env:
            - name: NODE_TLS_REJECT_UNAUTHORIZED
              value: "0"
            - name: NODE_ENV
              value: "production"
    

where NODE_TLS_REJECT_UNAUTHORIZED is the option to not verify the kong api certificate and

  
NODE_ENV is way to enable production or development , if you are not using production, the other two option enables the debug model and increase the cpu usage, i suggest to use production anyway **during the first deploy you should enable development in order to bootstrap the sql schema** that production is not able to do ... or create an init container to deploy the schema

`konga konga-stable-8957b9d85-tq72z 2/2 Running 2 6d3h`

since is now up and running it will be available on the standard infrastructure ingress
    
    
      - host: konga.ing.h4x0r3d.lan
    

Konga Home 

[![konga_home](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737373233372f6d6973632f6b6f6e67615f75692e706e67)](<https://camo.githubusercontent.com/0f76d690a796607e512cffaaf5f1405516e613575ecdf562d91399e1c646b05b/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737373233372f6d6973632f6b6f6e67615f75692e706e67>)

Konga node configuration 

[![konga_node](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737373233382f6d6973632f6b6f6e67615f75695f6b6f6e676e6f64652e706e67)](<https://camo.githubusercontent.com/cda833f675290b3538fa9b4e4d49f8c91ea586bae71505e705d7646646b30fa8/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737373233382f6d6973632f6b6f6e67615f75695f6b6f6e676e6f64652e706e67>)

  
  


  * kong [INSTALLED]
  * konga [INSTALLED]
  * backend [INSTALLED]



First of all we need to expose the backend with Kong in this example i created an ingress configuration dedicated for this porpose
    
    
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      annotations:
        kubernetes.io/ingress.class: "kong"
      name: api
      namespace: pytbak
      labels:
        app: api
    spec:
      rules:
        - host: api.ing.h4x0r3d.lan
          http:
            paths:
              - path: /api/
                backend:
                  serviceName: pytbak-svc
                  servicePort: 5000
              - path: /api/get/
                backend:
                  serviceName: pytbak-svc
                  servicePort: 5000
              - path: /api/post/
                backend:
                  serviceName: pytbak-svc
                  servicePort: 5000
              - path: /api/put/
                backend:
                  serviceName: pytbak-svc
                  servicePort: 5000
              - path: /api/delete/
                backend:
                  serviceName: pytbak-svc
                  servicePort: 5000
    

`kubernetes.io/ingress.class: "kong" `to match the kong ingress on the backend application `namespace: pytbak`.  
As you can see i created mutiple paths to simulate different applications behind and create different ruls per path/application/context.

The kong configuration could be applied with

  * kubernetes yaml
  * kong api
  * konga ui (only if you have the database)



In the repo you will see some in kubernetes and some other not present , i tested the three option and the **kong api** is the best one

  
  
Create api user

`curl -k -i -X POST --url https://192.168.1.14:31793/consumers/ --data "username=slow_user"`
    
    
    HTTP/1.1 201 Created
    Date: Sat, 07 Nov 2020 19:46:05 GMT
    Content-Type: application/json; charset=utf-8
    Connection: keep-alive
    Access-Control-Allow-Origin: *
    Server: kong/2.1.4
    Content-Length: 121
    X-Kong-Admin-Latency: 16
    

`{"custom_id":null,"created_at":1604778365,"id":"3d45a73d-4024-4813-b356-43a14ecea702","tags":null,"username":"slow_user"}` [![konga_consumer](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737383930332f6d6973632f6b6f6e67615f636f6e73756d65722e706e67)](<https://camo.githubusercontent.com/34bd577bcb080782018378247fac228663f6c4508bfc1191f173a07d9f731e28/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737383930332f6d6973632f6b6f6e67615f636f6e73756d65722e706e67>)

  
  
add a key token with key-auth plugin

`curl -k -i -X POST --url https://192.168.1.14:31793/consumers/slow_user/key-auth/ --data 'key=332d05445a560ee65a76aeaa372d8904'`
    
    
    HTTP/1.1 201 Created
    Date: Sat, 07 Nov 2020 19:51:03 GMT
    Content-Type: application/json; charset=utf-8
    Connection: keep-alive
    Access-Control-Allow-Origin: *
    Server: kong/2.1.4
    Content-Length: 190
    X-Kong-Admin-Latency: 10
    

`{"created_at":1604778663,"id":"52970ca8-848d-49b5-a16f-a235fe0e9ceb","tags":null,"ttl":null,"key":"332d05445a560ee65a76aeaa372d8904","consumer":{"id":"3d45a73d-4024-4813-b356-43a14ecea702"}}` [![konga_consumer_cred](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737383930332f6d6973632f6b6f6e67615f636f6e73756d65725f63726564656e7469616c2e706e67)](<https://camo.githubusercontent.com/2a45dbea6ede1ac159c1950a65f518aa347650b73efa3539a0edc11f8dbee793/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737383930332f6d6973632f6b6f6e67615f636f6e73756d65725f63726564656e7469616c2e706e67>)

  
  


routes.id where created when we created the kong ingress configuration for pytbak `curl -k -i -X GET <https://192.168.1.14:31793/routes/>`

ROUTE NAME| PATH | ROUTE.ID  
---|---|---  
pytbak.api.00| /api/| 0a78b572-cfe0-4228-bdfe-639ef04b5117  
pytbak.api.01| /api/get/| 8d2a24aa-636d-43ee-b0aa-b0fd1d3643fd  
pytbak.api.02| /api/post/| f75ed3b6-07c3-4293-932b-4f53d0c38f74  
pytbak.api.03| /api/put | f4823063-04e4-4da8-9c11-21977a93b1ba  
pytbak.api.04| /api/delete| e396750c-9158-4a96-8d79-047197669cff  
  
[![konga_routes](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737393732302f6d6973632f6b6f6e67615f726f757465732e706e67)](<https://camo.githubusercontent.com/c17c8b10cdcbb8526fdb448c903c6dad6090e4e460de537185e6a04be663c3e4/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343737393732302f6d6973632f6b6f6e67615f726f757465732e706e67>)

  
  
now i'd like to limit the user slow_user to perform only 4 request per sec on /api/  
and 2 on /api/get/
    
    
    $ curl -k -X POST https://192.168.1.14:31793/plugins/ \
    >             --data "name=rate-limiting"  \
    >             --data "config.second=4" \
    >             --data "config.hour=1200" \
    >             --data "config.policy=local" \
    >             --data "consumer.id=3d45a73d-4024-4813-b356-43a14ecea702" \
    >             --data "route.id=0a78b572-cfe0-4228-bdfe-639ef04b5117"
    

`{"created_at":1604780031,"id":"cfd3d752-7864-493c-a3b7-3970ab6eca86","tags":null,"enabled":true,"protocols":["grpc","grpcs","http","https"],"name":"rate-limiting","consumer":{"id":"3d45a73d-4024-4813-b356-43a14ecea702"},"service":null,"route":{"id":"0a78b572-cfe0-4228-bdfe-639ef04b5117"},"config":{"hide_client_headers":false,"minute":null,"policy":"local",``"month":null,"redis_timeout":2000,"limit_by":"consumer","redis_password":null,"second":4,"day":null,"redis_database":0,"year":null,"hour":1200,"redis_host":null,"redis_port":6379,"header_name":null,"fault_tolerant":true}}`
    
    
    curl -k -X POST https://192.168.1.14:31793/plugins/ \
                    --data "name=rate-limiting"  \
                    --data "config.second=2" \
                    --data "config.hour=600" \
                    --data "config.policy=local" \
                    --data "consumer.id=3d45a73d-4024-4813-b356-43a14ecea702" \
                    --data "route.id=8d2a24aa-636d-43ee-b0aa-b0fd1d3643fd" 
    

`{"created_at":1604780101,"id":"33bfc138-776a-4116-a2a9-8c70ab9d7a2a","tags":null,"enabled":true,"protocols":["grpc","grpcs","http","https"],"name":"rate-limiting","consumer":{"id":"3d45a73d-4024-4813-b356-43a14ecea702"},"service":null,"route":{"id":"8d2a24aa-636d-43ee-b0aa-b0fd1d3643fd"},"config":{"hide_client_headers":false,"minute":null,"policy":"local","month":null,"redis_timeout":2000,"limit_by":"consumer","redis_password":null,"second":2,"day":null,"redis_database":0,"year":null,"hour":600,"redis_host":null,"redis_port":6379,"header_name":null,"fault_tolerant":true}}`

[![konga_rate_routes](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343738303230302f6d6973632f6b6f6e67615f726174655f6c696d69745f726f757465732e706e67)](<https://camo.githubusercontent.com/31f166746c62aae6fda1d4e1bdb7a85ec1683afd6bdf2d859baf6150883e302e/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f313032342f76313630343738303230302f6d6973632f6b6f6e67615f726174655f6c696d69745f726f757465732e706e67>)

  
  


## Use cases

I created 3 users , one on the example above and a new one with a double of requests per second

route| slow_user rate/s| fast_user rate/s  
---|---|---  
/api/| 4| 8  
/api/get/| 2| 4  
/api/post/| 1| 2  
/api/put/| 1| 2  
/api/delete/| 1| 2  
  
  
  
slow_user has apikey: 332d05445a560ee65a76aeaa372d8904  
fast_user has apikey: 695aa0b18c6dbd1b387ee7c32c72c513  
admin has a apikey: admin

  
  
i've also created a global rules on the service with max 9 reqquest per second for all other users with no specific rate limit rule applied (ex admin.)

`$ curl -k -X POST https://192.168.1.14:31793/plugins --data "name=rate-limiting" --data "config.second=9" --data "config.policy=local" --data "service.id=53d35e73-b285-4eb2-ad24-853d82563ca4"`
    
    
    {"created_at":1604786501,"id":"34182311-ecbf-49a3-bdcf-bdc1c9c58706",  
    "tags":null,"enabled":true,"protocols":["grpc","grpcs","http","https"],  
    "name":"rate-limiting","consumer":null,  
    "service":{"id":"53d35e73-b285-4eb2-ad24-853d82563ca4"},  
    "route":null,"config":{"hide_client_headers":false,"minute":null,"policy":"local",  
    "month":null,"redis_timeout":2000,"limit_by":"consumer",  
    "redis_password":null,"second":9,"day":null,"redis_database":0,"year":null,  
    "hour":null,"redis_host":null,"redis_port":6379,"header_name":null,  
    "fault_tolerant":true}}
    

Using siege i've benchmarked the users

[![siege_users](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343738373237302f6d6973632f73696567655f75736572732e706e67)](<https://camo.githubusercontent.com/ee42763f67ce2c6a3e6f97b945740f70bf12d80bc251a6fd1266b07cf10d68f9/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630343738373237302f6d6973632f73696567655f75736572732e706e67>)

You can check the headers with curl  
`$ while true; do curl -I -H "apikey:332d05445a560ee65a76aeaa372d8904" http://api.ing.h4x0r3d.lan:30080/api/ && sleep 0.2; done`
    
    
    HTTP/1.1 200 OK
    Content-Type: text/html; charset=utf-8
    Content-Length: 658
    Connection: keep-alive
    Server: Werkzeug/1.0.1 Python/3.9.0
    Date: Sat, 07 Nov 2020 22:21:34 GMT
    X-RateLimit-Limit-Second: 4
    X-RateLimit-Remaining-Second: 3
    X-RateLimit-Limit-Hour: 1000000
    RateLimit-Limit: 4
    X-RateLimit-Remaining-Hour: 999755
    RateLimit-Remaining: 3
    RateLimit-Reset: 1
    X-Kong-Upstream-Latency: 3
    X-Kong-Proxy-Latency: 1
    Via: kong/2.1.4
    
    HTTP/1.1 200 OK
    Content-Type: text/html; charset=utf-8
    Content-Length: 658
    Connection: keep-alive
    Server: Werkzeug/1.0.1 Python/3.9.0
    Date: Sat, 07 Nov 2020 22:21:34 GMT
    X-RateLimit-Limit-Second: 4
    X-RateLimit-Remaining-Second: 2
    X-RateLimit-Limit-Hour: 1000000
    RateLimit-Limit: 4
    X-RateLimit-Remaining-Hour: 999754
    RateLimit-Remaining: 2
    RateLimit-Reset: 1
    X-Kong-Upstream-Latency: 3
    X-Kong-Proxy-Latency: 0
    Via: kong/2.1.4
    
    HTTP/1.1 200 OK
    Content-Type: text/html; charset=utf-8
    Content-Length: 658
    Connection: keep-alive
    Server: Werkzeug/1.0.1 Python/3.9.0
    Date: Sat, 07 Nov 2020 22:21:34 GMT
    X-RateLimit-Limit-Second: 4
    X-RateLimit-Remaining-Second: 1
    X-RateLimit-Limit-Hour: 1000000
    RateLimit-Limit: 4
    X-RateLimit-Remaining-Hour: 999753
    RateLimit-Remaining: 1
    RateLimit-Reset: 1
    X-Kong-Upstream-Latency: 3
    X-Kong-Proxy-Latency: 1
    Via: kong/2.1.4
    
    HTTP/1.1 200 OK
    Content-Type: text/html; charset=utf-8
    Content-Length: 658
    Connection: keep-alive
    Server: Werkzeug/1.0.1 Python/3.9.0
    Date: Sat, 07 Nov 2020 22:21:34 GMT
    X-RateLimit-Limit-Second: 4
    X-RateLimit-Remaining-Second: 0
    X-RateLimit-Limit-Hour: 1000000
    RateLimit-Limit: 4
    X-RateLimit-Remaining-Hour: 999752
    RateLimit-Remaining: 0
    RateLimit-Reset: 1
    X-Kong-Upstream-Latency: 3
    X-Kong-Proxy-Latency: 1
    Via: kong/2.1.4
    
    HTTP/1.1 429 Too Many Requests
    Date: Sat, 07 Nov 2020 22:21:34 GMT
    Content-Type: application/json; charset=utf-8
    Connection: keep-alive
    Retry-After: 1
    Content-Length: 41
    X-RateLimit-Limit-Second: 4
    X-RateLimit-Remaining-Second: 0
    X-RateLimit-Limit-Hour: 1000000
    RateLimit-Limit: 4
    X-RateLimit-Remaining-Hour: 999752
    RateLimit-Remaining: 0
    RateLimit-Reset: 1
    X-Kong-Response-Latency: 1
    Server: kong/2.1.4
    

  
  


### Rules Precedence

A plugin will always be run once and only once per request. But the configuration with which it will run depends on the entities it has been configured for.

Plugins can be configured for various entities, combination of entities, or even globally. This is useful, for example, when you wish to configure a plugin a certain way for most requests, but make authenticated requests behave slightly differently.

Therefore, there exists an order of precedence for running a plugin when it has been applied to different entities with different configurations. The rule of thumb is: the more specific a plugin is with regards to how many entities it has been configured on, the higher its priority.

The complete order of precedence when a plugin has been configured multiple times is:

  1. Plugins configured on a combination of: a Route, a Service, and a Consumer. (Consumer means the request must be authenticated).
  2. Plugins configured on a combination of a Route and a Consumer. (Consumer means the request must be authenticated).
  3. Plugins configured on a combination of a Service and a Consumer. (Consumer means the request must be authenticated).
  4. Plugins configured on a combination of a Route and a Service.
  5. Plugins configured on a Consumer. (Consumer means the request must be authenticated).
  6. Plugins configured on a Route.
  7. Plugins configured on a Service.
  8. Plugins configured to run globally.



Example: if the rate-limiting plugin is applied twice (with different configurations): for a Service (Plugin config A), and for a Consumer (Plugin config B), then requests authenticating this Consumer will run Plugin config B and ignore A. However, requests that do not authenticate this Consumer will fallback to running Plugin config A. Note that if config B is disabled (its enabled flag is set to false), config A will apply to requests that would have otherwise matched config B.

  
  


### WAF

Last but not lease , the api is configured behind Cloudflare,

the users `slow_user fast_user` are available in case you want to test directly the service.

Endpoint: `https://services.k8s.it/api/`

  
  


`$ curl -I -H "apikey:332d05445a560ee65a76aeaa372d8904" https://services.k8s.it/api/`
    
    
    HTTP/2 200
    date: Sun, 08 Nov 2020 10:14:21 GMT
    content-type: text/html; charset=utf-8
    set-cookie: __cfduid=dab5b88be5427314b7b17a1a65d3178a41604830461;   
    expires=Tue, 08-Dec-20 10:14:21 GMT; path=/; domain=.k8s.it; HttpOnly;   
    SameSite=Lax
    vary: Accept-Encoding
    x-ratelimit-limit-second: 4
    x-ratelimit-remaining-second: 3
    x-ratelimit-limit-hour: 1000000
    ratelimit-limit: 4
    x-ratelimit-remaining-hour: 999995
    ratelimit-remaining: 3
    ratelimit-reset: 1
    x-kong-upstream-latency: 3
    x-kong-proxy-latency: 4
    via: kong/2.1.4
    cf-cache-status: DYNAMIC
    cf-request-id: 0648f2980000000f72931c2000000001
    expect-ct: max-age=604800,   
    report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
    report-to: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report?s=z4j%2FU3og%2FT2jBByhUKZtShDNLfLMWncExIy3EowJ2hOPJ17oCB0oeyY1GVaXirT%2BgWpTEEu  
    U%2F03gx%2F91uOy3mGD%2BFE4FhBGqFt7iC2bhaPc%3D"}],"group":"cf-nel","max_age":604800}
    nel: {"report_to":"cf-nel","max_age":604800}
    server: cloudflare
    cf-ray: 5eee86d319770f72-MXP
    

  
  


## Conclusion

Kong is a scalable, open source API Gateway.  
Kong runs in front of any RESTful API and is extended through Plugins, which provide extra functionality and services beyond the core platform.

The architecture is quite simple to understand and is made up of a few components…

  * Kong base-module which wraps OpenResty and Nginx and is the engine which does the actual work
  * Database layer with choice of Cassandra or Postgres to store all the configuration so it can be retrieved easily in case of failures
  * Dashboard which provides User-Interface for API administration and viewing analytics (embedded in th Enerprise, third party for the community  _Konga_)



The Powerful of Kong is also supported by plugins from authentication, security, traffic control, logging, and etc…  
  


[![uml_api_gw](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3438302f76313630373333323133302f6d6973632f554d4c5f6170695f67772e706e67)](<https://camo.githubusercontent.com/d2a4de6795889077f983473c7575a12036254ec671f5440c6e63b68889926d65/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3438302f76313630373333323133302f6d6973632f554d4c5f6170695f67772e706e67>)

Monitoring is also covered well by default kong official prometheus available here <https://github.com/Kong/kong-plugin-prometheus>

[![kong_monitor](/user/images/kubernetes-apigw/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630353732333433362f6d6973632f6b6f6e665f6d6f6e69746f722e706e67)](<https://camo.githubusercontent.com/28dedd2eae334443bd6e668838f06caedc99f3c5cbd95e5b8d12869015cad8e9/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313630353732333433362f6d6973632f6b6f6e665f6d6f6e69746f722e706e67>)

Live: [https://services.k8s.it/grafana/d/mY9p7dQmz/kong?orgId=2&refresh=1m](https://services.k8s.it/grafana/d/mY9p7dQmz/kong?orgId=2&refresh=1m)

**If your business exposes APIs, the API GATEWAY is a requirement to handle the third party.**
