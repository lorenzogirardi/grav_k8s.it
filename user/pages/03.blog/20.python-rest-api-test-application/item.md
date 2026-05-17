---
title: 'Python rest api test application'
date: '2020-10-30 10:39'
taxonomy:
    tag:
            - api
            - backend
            - DELETE
            - docker
            - GET
            - kubernetes
            - POST
            - PUT
            - python
            - rest
            - rest-api
            - restapi
            - test
---

Working most of the times (always) in Platform i'm usually play with the infrastructure, however sometimes to create prototype i need backends that are done for the specific purpose.

## GOALS:

  * The application must be a REST api
  * Run in kubernetes  
  
  
  




### Docker and kubernetes

The application is working with GET, POST, PUT, DELETE  
enouth to cover most of the usages based on rest api.

Inside the docker path you can run easly as local docker

`docker build --tag pytbak:0.1 .`  
`docker run -t -p 5000:5000 pytbak:0.1`  
and access on it with `locahost:5000/api/`

instead if you want to run it in kubernetes ,  
starting from the main folder you can apply the kubernetes folder on your environment  
`kubectl apply -f kubernetes/`
    
    
    $ kubectl  get pods -n pytbak
    NAME                             READY   STATUS    RESTARTS   AGE
    pytbak-stable-5dfb4fbfd4-n64kx   1/1     Running   0          40m
    

Remember to change the host in the ingress file configuration
    
    
    $ cat 03-ing-pytbak.yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: pytbak-ingress
      namespace: pytbak
      annotations:
        ingress.kubernetes.io/proxy-connect-timeout: "10"
        ingress.kubernetes.io/proxy-read-timeout: "30"
        ingress.kubernetes.io/proxy-send-timeout: "30"
    spec:
      rules:
      - host: pytbak.ing.h4x0r3d.lan
        http:
          paths:
          - path: /
            backend:
              serviceName: pytbak-svc
              servicePort: 5000
    

_host: pytbak.ing.h4x0r3d.lan_ this is my ingress dns resolution

TIPS: for ingress endpoints you can manage multiple namespace creating an A record in your DNS (as a subdomain) with a wildcard dedicated only for ingress matching the namespace and the host created

in my case i have the home dns as `h4x0r3d.lan`  
and `*.ing.h4x0r3d.lan` as a record A of my kubernetes ingress  
in this way if i create a namespace  _pippo_ the dns that i have to call to reach it out will be  _pippo.ing.4x0r3d.lan_ with no change on DNS.  
  
  
  


### Usage

The application answer on /api/ with the main html page with methods

HTTP Method| URI| Action  
---|---|---  
GET| http://[hostname]/api/get/context| Retrieve list of context  
GET| http://[hostname]/api/get/context/[context_id]| Retrieve a context  
POST| http://[hostname]/api/post/context| Create a new context  
PUT| http://[hostname]/api/put/context/[context_id]| Update an existing context  
DELETE| http://[hostname]/api/delete/context/[context_id]| Delete acontext  
  
Following the methods example
    
    
    $ curl -i http://pytbak.ing.h4x0r3d.lan/api/get/context
    HTTP/1.1 200 OK
    Server: openresty/1.15.8.1
    Date: Wed, 28 Oct 2020 20:02:02 GMT
    Content-Type: application/json
    Content-Length: 696
    Connection: keep-alive
    Vary: Accept-Encoding
    
    {
      "context": [
        {
          "description": "RHEL 6 based",
          "done": false,
          "title": "Cento 6",
          "uri": "http://pytbak.ing.h4x0r3d.lan/api/get/context/1"
        },
        {
          "description": "RHEL 7 based",
          "done": false,
          "title": "Centos 7",
          "uri": "http://pytbak.ing.h4x0r3d.lan/api/get/context/2"
        },
        {
          "description": "RHEL 8 based",
          "done": false,
          "title": "Centos 8",
          "uri": "http://pytbak.ing.h4x0r3d.lan/api/get/context/3"
        },
        {
          "description": "Fedora + RHEL based",
          "done": false,
          "title": "Centos stream",
          "uri": "http://pytbak.ing.h4x0r3d.lan/api/get/context/4"
        }
      ]
    

  
  

    
    
    $ curl -i -H "Content-Type: application/json" -X POST -d '{"title":"Ubuntu 20.04 LTS", "description":"focal"}' http://pytbak.ing.h4x0r3d.lan/api/post/context
    HTTP/1.1 201 CREATED
    Server: openresty/1.15.8.1
    Date: Wed, 28 Oct 2020 19:49:31 GMT
    Content-Type: application/json
    Content-Length: 165
    Connection: keep-alive
    
    {
      "task": {
        "description": "focal",
        "done": false,
        "title": "Ubuntu 20.04 LTS",
        "uri": "http://pytbak.ing.h4x0r3d.lan/api/get/context/5"
      }
    }
    

  
  

    
    
    $ curl -i -H "Content-Type: application/json" -X PUT -d '{"description":"Focal Fossa"}' http://pytbak.ing.h4x0r3d.lan/api/put/context/5
    HTTP/1.1 200 OK
    Server: openresty/1.15.8.1
    Date: Wed, 28 Oct 2020 20:02:43 GMT
    Content-Type: application/json
    Content-Length: 171
    Connection: keep-alive
    
    {
      "task": {
        "description": "Focal Fossa",
        "done": false,
        "title": "Ubuntu 20.04 LTS",
        "uri": "http://pytbak.ing.h4x0r3d.lan/api/get/context/5"
      }
    }
    

  
  

    
    
    $ curl -i -H "Content-Type: application/json" -X DELETE http://pytbak.ing.h4x0r3d.lan/api/delete/context/5
    HTTP/1.1 200 OK
    Server: openresty/1.15.8.1
    Date: Wed, 28 Oct 2020 20:04:47 GMT
    Content-Type: application/json
    Content-Length: 21
    Connection: keep-alive
    
    {
      "result": true
    }
    $ curl -i http://pytbak.ing.h4x0r3d.lan/api/get/context/5
    HTTP/1.1 404 NOT FOUND
    Server: openresty/1.15.8.1
    Date: Wed, 28 Oct 2020 20:04:54 GMT
    Content-Type: application/json
    Content-Length: 27
    Connection: keep-alive
    
    {
      "error": "Not found"
    }
