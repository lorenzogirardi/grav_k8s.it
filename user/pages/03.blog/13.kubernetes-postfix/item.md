---
title: 'Kubernetes postfix'
date: '2020-08-25 18:53'
taxonomy:
    tag:
            - dkim
            - dmarc
            - email
            - kubernetes
            - mail
            - postfix
            - spf
            - tls
---

# 

Long story short my vps provider changed tha ammount for the small instance from 1$ to 3$  
so i took the advantage to switch my postfix service from cloud to onprem.

Why, since the world is moving to cloud ?  
Because my own domain is used just for alerting and small other stuff and 3$ month is creazy :)  
Anyway i'll lost some good stuff like static ip and possibility ti have a ptr dns.  
Shit happens it's just for my stuff.

However we have 3 different topics to describe in this project

  * what is an email
  * postfix configuration
  * postfix in kubernetes



## what is an email

send email sometimes is a dedicated work in an enterprise company.  
There are many aspect to consider , not only infrastructure , like che segmentation of domains strategy,  
email answer engagement and each isp is using a different threashold to mark your email as a spam,  
however now we will focus only on the generic configuration with some minimum requirements.

Here some topic that you probably know:

  * TLS (add an encrypted connection in delivery)
  * Sender-Id (mostly used for microsoft ecosystem)
  * PTR record (used to check the corrispondence of a real smtp)
  * SPF record (used on TXT level to define an whitelist of "smtp allowed to ...")
  * Dkim (a private and public certificate to match the smtp and emails)
  * Dmarc (a mix with SPF and Dkim to protect the domain from unauthorized use )



## postfix configuration

The second one is more related to middleware behaviour  
i have a domain "example.com" and i'd like to

  * receive email from and to this domain
  * have catchall system to do not configure any email account but using everything [[email protected]](</cdn-cgi/l/email-protection#badbfadfc2dbd7cad6df94d9d5d7>), [[email protected]](</cdn-cgi/l/email-protection#680a280d10090518040d460b0705>), [[email protected]](</cdn-cgi/l/email-protection#24534c45504152415664415c45495448410a474b49>)
  * relay all the emails to my gmail account
  * reject the email that are tried to sent outside my chosed domains



For all those topics the main configuration is in virtual and transport map in  _postfix_  
There are some articles in internet, i'll avoid to reinvent hot water with the explanation.

## postfix in kubernetes

just as recap this was the configuration in the cloud vps

[![vps](/user/images/kubernetes-postfix/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313539383336363436332f6d6973632f7670732e706e67)](<https://camo.githubusercontent.com/66d40f312601abf8ba03699a66366d6455c0805f/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313539383336363436332f6d6973632f7670732e706e67>)

now i just whitched the MX to my own microk8s lab

[![microk8s](/user/images/kubernetes-postfix/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313539383336363436352f6d6973632f6d6963726f6b38732e706e67)](<https://camo.githubusercontent.com/fd6de4c35300c6596ca2eafcfde2d501bd4c9576/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313539383336363436352f6d6973632f6d6963726f6b38732e706e67>)

However , inside the box a lot of implementation way can be taken

### Scenario 1:

you can build an image for each dedicate purpose

  * postfix
  * rsyslog
  * opendkim
  * tls



### Scenario 2:

looking the configuration adopted in  _Scenario 1_ , you can add an ingress tcp forward where  
you can also add a tls layer with cert-manager (the new kube-lego).

### Scenario 3:

_Scenario 2_ plus external storage for logs  
etc etc

### Scenario Lazy:

Embedded docker images with all you need with a node port (used only because i have a standalone kubernetes node).

Well having everything in one single container the docker image reflect the pachage used in a virtual machine
    
    
    FROM debian:buster
    MAINTAINER lgirardi <[[email protected]](</cdn-cgi/l/email-protection>)>
    
    EXPOSE 25/tcp
    
    RUN apt-get -y update && apt-get -yq install \
    	postfix \
    	bsd-mailx \
    	opendkim \
    	opendkim-tools \
    	sasl2-bin \
    	rsyslog \
            supervisor
    
    
    # Add files
    ADD run.sh /opt/run.sh
    
    # Run
    CMD /opt/run.sh;/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
    

where the `run.sh` is the supervisord daemon managing the process.

  * postfix
  * rsyslog
  * opendkim



The structure used is done to leave the configuration as a configmap and secrets  
this will help in case you have to tune your system whiout regenerate the images that honestly make no sense to be build each deploy.

As you see the deployment files has a huge number of volumes
    
    
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: postfix
      namespace: postfix
    spec:
      replicas: 1
      strategy:
        type: RollingUpdate
      revisionHistoryLimit: 10
      selector:
        matchLabels:
          app: postfix
      template:
        metadata:
          labels:
            app: postfix
        spec:
          containers:
          - name: postfix
            image: lgirardi/kubernetes-postfix:v0.2
            lifecycle:
              postStart:
                exec:
                  command: [ "bin/bash", "-c", "postmap /etc/postfix/virtual && postmap /etc/postfix/transport && supervisorctl restart postfix" ]
            securityContext:
              capabilities:
                add:
                - NET_ADMIN
            ports:
            - name: smtp
              containerPort: 25
            volumeMounts:
            - name: opendkim-key
              mountPath: /etc/mail/dkim-k8s/keys/YOUR_DOMAIN/YOUR_DOMAIN.private
              subPath: opendkim-key
            - name: ca-crt
              mountPath: /etc/postfix/tls/ca.crt
              subPath: ca-crt
            - name: ca-key
              mountPath: /etc/postfix/tls/ca.key
              subPath: ca-key
            - name: postfix-transport
              mountPath: /etc/postfix/transport
              subPath: transport
            - name: postfix-virtual
              mountPath: /etc/postfix/virtual
              subPath: virtual
            - name: postfix-headerchecks
              mountPath: /etc/postfix/header_checks
              subPath: header_checks
            - name: postfix-maincf
              mountPath: /etc/postfix/main.cf
              subPath: main.cf
            - name: postfix-opendkimconf
              mountPath: /etc/opendkim.conf
              subPath: opendkim.conf
            - name: postfix-keytable
              mountPath: /etc/opendkim/KeyTable
              subPath: KeyTable
            - name: postfix-signingtable
              mountPath: /etc/opendkim/SigningTable
              subPath: SigningTable
            - name: postfix-trustedhosts
              mountPath: /etc/opendkim/TrustedHosts
              subPath: TrustedHosts
          volumes:
            - name: postfix-transport
              configMap:
                name: postfix-conf
                items:
                - key: transport
                  path: transport
            - name: postfix-virtual
              configMap:
                name: postfix-conf
                items:
                - key: virtual
                  path: virtual
            - name: postfix-headerchecks
              configMap:
                name: postfix-conf
                items:
                - key: header_checks
                  path: header_checks
            - name: postfix-maincf
              configMap:
                name: postfix-conf
                items:
                - key: main.cf
                  path: main.cf
            - name: postfix-opendkimconf
              configMap:
                name: postfix-conf
                items:
                - key: opendkim.conf
                  path: opendkim.conf
            - name: postfix-keytable
              configMap:
                name: postfix-conf
                items:
                - key: KeyTable
                  path: KeyTable
            - name: postfix-signingtable
              configMap:
                name: postfix-conf
                items:
                - key: SigningTable
                  path: SigningTable
            - name: postfix-trustedhosts
              configMap:
                name: postfix-conf
                items:
                - key: TrustedHosts
                  path: TrustedHosts
            - name: opendkim-key
              secret:
                secretName: postfix-secret
            - name: ca-crt
              secret:
                secretName: postfix-secret
            - name: ca-key
              secret:
                secretName: postfix-secret
    

Here we have the configuration for:

  * main.cf
  * transport
  * virtual
  * header_checks
  * opendkim.conf
  * KeyTable
  * SigningTable
  * TrustedHosts



You can change this aspect , is mostly related to the infrastructure where you fit the pod  
and the company CI/CD

**Remember to ADD your right values inside secrets and configmap where you have CAPS like`YOUR_DOMAIN`**

running the  _apply_ on kubernetes folder you will have the pod running  
`postfix postfix-7d664f786c-rmf54 1/1 Running 0 29m`

and loogking the logs i can see

[![postfixlog](/user/images/kubernetes-postfix/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313539383336393033322f6d6973632f706f73746669786c6f672e706e67)](<https://camo.githubusercontent.com/694d0f364b26c0474b15921ff1ff8a4135190933/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313539383336393033322f6d6973632f706f73746669786c6f672e706e67>)

The `key data is not secure` just a working because i've enabled this option in the configmap `RequireSafeKeys false`  
but the certificate is applied `opendkim[22]: 91E522A000C: DKIM-Signature field added`

Last one is the check on the client side with the righ option enabled

[![clientemail](/user/images/kubernetes-postfix/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313539383336383833352f6d6973632f656d61696c6f7074696f6e2e706e67)](<https://camo.githubusercontent.com/66fffe878e76dbeff1853c4abaf19f846af46529/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f6574687a65726f2f696d6167652f75706c6f61642f76313539383336383833352f6d6973632f656d61696c6f7074696f6e2e706e67>)

## Conclusion

You can add postfix infrastructure to kubernetes with no problems at all ,  
this project is a  _prototype_ , in a production infrastructure you can work around this to let it rock solid and scalable
