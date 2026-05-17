---
title: 'Kubernetes guacamole'
date: '2020-08-11 20:46'
taxonomy:
    tag:
            - bastion
            - bastion host
            - guacamole
            - guacd
            - kubernetes
            - rdp
            - ssh
---

## Here we are , another apache guacamole implementation in kubernetes

This service is designed to avoid the usage of mysql and create a standalone project

The main idea is to use the **user-mapping.xml** as a config map

For production environment i suggest to add the ldap auth (ad.openldap,freeipa),  
mysql database should be managed with a dedicated instances and mantained in case of "exit"

## what is a bastion host

On the Internet, a bastion host is the only host computer that a company allows to be addressed  
directly from the public network and that is designed to screen the rest of its network from security exposure.

## how this tool can be used

The tool is designed to be used when you have some dedicated service in production and you have to keep  
the control of access and account used , guacamole has the ability to manage the most used platforms (windows and linux)  
as host in backend to be reached from developers ... contractors ...

## why in kubernetes

Since the auth method could scale by configmap or ldap or mysql , is designed to scale  
we have also the benefits to have a low footprint compared to a traditional vm.

## config to change

Before deploy you need to specify the following parameters in guacamole folder

  * YOUR_DOMAIN to reflect your domain url in 03-guacamole-ing.yaml
  * user YOUR_USERNAME / YOUR_MD5_PWD and hosts xml configuration in 04-guacamole-cfm.yaml following <https://guacamole.apache.org/doc/gug/configuring-guacamole.html#user-mapping>



screenshots

[![windows](/user/images/kubernetes-guacamole/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313538303835303535322f6d6973632f67756163616d6f6c652d77696e2e706e67)](<https://camo.githubusercontent.com/905e74d0ce81c8b239f67d405567100a3f181292/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313538303835303535322f6d6973632f67756163616d6f6c652d77696e2e706e67>)  
[![linux](/user/images/kubernetes-guacamole/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313538303835303535322f6d6973632f67756163616d6f6c652d6c696e75782e706e67)](<https://camo.githubusercontent.com/fd53b1991cd9693679aae9aadf72bc75d025c2ad/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313538303835303535322f6d6973632f67756163616d6f6c652d6c696e75782e706e67>)

## deploy

`kubectl apply -f guacd`

`kubectl apply -f guacamole`

You can secure the connection with kube-lego and use cillium to add network rules 
