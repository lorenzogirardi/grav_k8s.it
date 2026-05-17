---
title: 'Kubernetes apache httpd'
date: '2020-08-11 20:48'
taxonomy:
    tag:
            - apache
            - front controller
            - httpd
            - kubernetes
            - rewrite rules
---

## The semi-unuseful apache implementation in kubernetes

Well , why we are talking about apache httpd in kubernetes ?  
We have ingress resources , we have ambassador and we are using microservices...  
True but internet was not built yesterday and for some reasons out of my knowledge ,  
people are ostinated to manage rewrite rules in apache instead to use a dedicated router application (react, zuul .. database!?!?! etc etc)  
However sometimes we have to balance between the academic vision and the reality.

## digression

Talking about apache https , nginx , haproxy ... i'm referring to the idea behind manage a complex website.  
A website could be composed my hundreds of microservices but the domain it's usually one  
[www.example.com](http://www.example.com/)

[www.example.com](http://www.example.com/) has the root path /  
/it/ managed by cms  
/it/offerte managed by cms  
/uk/ managed by cms  
/uk/offers managed by cms  
/../ whatever  
/it/clienti/ managed by customer-app  
/uk/customers/ managed by customers-app  
/../somethingelse  
/secure/ managey by payment-app  
...  
omg path clash ... so we need to exclude in apache /uk/customers/ from cms proxypass but  
meantime enable a dedicated proxy pass to a specific application endpoint.  
and we are mannually manage all language , all applications with static configurations...  
and if tomorrow we will open APAC ... what we have to do ?  
and if we need to cover a former third parti company and acquire his seo ranking we should create  
thousands of redirects ? and how we can can validate avoiding loops ?

A better design start giving the right responsability , that could be managed using a business layer  
that we can call "front controller" where we can apply all rules.

In terms of responsability all applaction behind this layer should be working by selfcontained logic  
example... if i have a seo application this should be working without this layer , removing direct dependency  
same concept for cms application , customers application and so on.  
So what this layer will do ?

A front controller should be the owner of root path of our websites "/"  
Should be also responsible to handle all the others paths after the root ones.

Main duties:

  * handle all requests and manage the backend application with business logic
  * provide dynamic path based on business rules (language + brand + something else)
  * provide the AB test logic
  * provide a validation logic
  * expose a company backoffice to trim the website layout



Examples

Having a global websites spread with multiple countris with differents domains/brands and languages we can immagine

[![frontcontroller](/user/images/kubernetes-apacherr/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313538323238393238352f6d6973632f66726f6e742d636f6e74726f6c6c65722e706e67)](<https://camo.githubusercontent.com/184bca691d84bf02cd01d06642b3aac6f582d28f/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f635f7363616c652c775f3634302f76313538323238393238352f6d6973632f66726f6e742d636f6e74726f6c6c65722e706e67>)

Where the microservice/application CMS is responsible to hangle all domains and all languages and the  
front controller take the ownership to match brand plus .tld (or paths "/fr/")  
in order to dinamically generate the right url with canonical pages.

Many and many others assumptions can be covered by this componet, however we have to stay grounded  
and check how we can manage an apache responsible to redirects proxypass an rewrite rules.

## Some concepts about this project

Even if we are working in a dynamic environment it's no rare to have the traditional layers

DMZ --> layer 1  
Application --> layer 2  
Database -->layer 3

In this picture apache httpd should usually placed on layer 1 ,  
but consider apache not for the common web server but like a "product" ,  
something that could be managed not by SRE , but Product Engeneer.  
A product that own a dedicated business, like seo, sem , vanity urls etc etc.

Having those considerations, we can "downgrade" apache httpd in layer 2 like any application  
and honor the DMZ (if needed ?!?!) on top by ingress/haproxy/bigf5 (where maybe we can terminate the TLS).

## Implementation

The code in this project is designet to manage apache configuration by configmap,  
however some websites are really complex and there are some limits implication with ectd max object size.

In this scenario it's higly raccomended to deploy the release with a standard pipeline with compiled  
container on the source.

## deploy

`kubectl apply -f apacherr/deployment/`
