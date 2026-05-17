---
title: 'Terraform your free Claudflare account'
date: '2018-07-06 22:10'
taxonomy:
    tag:
            - automation
            - cdn
            - cloudflare
            - gitlab
            - terraform
            - waf
---

So...

after some experience with akamai 
    
    
     _www.ethzero.it is an alias for aku-cs-akamaiflowers.com.edgesuite.net._  
    _aku-cs-akamaiflowers.com.edgesuite.net is an alias for a169.dscksd.akamai.net._  
    _a169.dscksd.akamai.net has address 193.45.15.98_  
     _a169.dscksd.akamai.net has address 193.45.15.122_  
     _a169.dscksd.akamai.net has IPv6 address 2001:2030:0:27::c12d:f62_  
     _a169.dscksd.akamai.net has IPv6 address 2001:2030:0:27::c12d:f7a_

and some others with Incapsula
    
    
     _bunker.ethzero.it is an alias for ve7cu.x.incapdns.net._  
    _ve7cu.x.incapdns.net has address 107.154.167.38_

It's now the moment to talk about Cloudflare

[Claudflare](<https://www.cloudflare.com/>) provide a good free account that allow you to hide your source ip or you want to discover how it's work the free waf protection or have a https certificate having your origin in http and so on ... but what is really cool and quickwin than the competitors is the API support and the native integration with [Terraform](<https://www.terraform.io/docs/providers/cloudflare/index.html>)

![](/user/images/terraform-your-free-claudflare-account/git-terraform-cloudflare.png)

Terraform is the most integrated tool for cloud (not only) automation and infrastructure as a code 

Here some steps that can show how it's esay deploy a configuration

First of all , retrive the api key from Cloudflare portal

![](/user/images/terraform-your-free-claudflare-account/cloudflare-api-key.png)

![](/user/images/terraform-your-free-claudflare-account/cloudflare-global-apikey.png)

then start the main terraform file in a dedicated folder (better in a [gitlab](<https://gitlab.com/>) repo)
    
    
    $ cat cloudflare-auth.tf  
    provider "cloudflare" {  
     email = "cloudflare_registration_email"  
     token = "cloudflare_token_api_key"  
    }  
    

make a $`terraform init` and now you can start to configure your account

manage your domain(s)
    
    
    $ cat cloudflare_domains.tf  
    variable "domain" {  
     default = "k8s.it"  
    } 

create dns configuration
    
    
    $ cat cloudflare_dns.tf  
    resource "cloudflare_record" "www" {  
     domain = "${var.domain}"  
     name = "www"  
     value = "something.github.io"  
     type = "CNAME"  
     proxied = true  
    }  
      
    resource "cloudflare_record" "smtp" {  
     domain = "${var.domain}"  
     name = "smtp"  
     value = "IP.OF.PRIVATE.VPS"  
     type = "A"  
     proxied = false  
    }  
      
    resource "cloudflare_record" "services" {  
     domain = "${var.domain}"  
     name = "services"  
     value = "something.related.to.my.home"  
     type = "CNAME"  
     proxied = true  
    }  
      
    resource "cloudflare_record" "mx" {  
     domain = "${var.domain}"  
     name = "${var.domain}"  
     value = "smtp.k8s.it"  
     type = "MX"  
     priority = "1"  
    }  
      
    resource "cloudflare_record" "google-verification" {  
     domain = "${var.domain}"  
     name = "${var.domain}"  
     value = "google-site-verification=fdsfdssgfdg4teurtxh"  
     type = "TXT"  
    }  
      
    resource "cloudflare_record" "spf" {  
     domain = "${var.domain}"  
     name = "${var.domain}"  
     value = "v=spf1 ip4:IP.OF.PRIVATE.VPS mx ~all"  
     type = "TXT"  
    }

Force the HTTPS with page rules
    
    
    $ cat cloudflare_rules.tf  
    resource "cloudflare_page_rule" "always_use_https_www" {  
     zone = "${var.domain}"  
     target = "http://www.${var.domain}/*"  
     priority = 1  
      
     actions = {  
     always_use_https = "true",  
     }  
    }  
      
    resource "cloudflare_page_rule" "always_use_https_services" {  
     zone = "${var.domain}"  
     target = "http://services.${var.domain}/*"  
     priority = 2  
      
     actions = {  
     always_use_https = "true",  
     }  
    }

And finally create the common zone settings 
    
    
    $ cat cloudflare_zone.tf  
    resource "cloudflare_zone_settings_override" "k8s-settings" {  
     name = "${var.domain}"  
      
     settings {  
     tls_1_3 = "on"  
     ssl = "flexible"  
     opportunistic_encryption = "on"  
     brotli = "on"  
     automatic_https_rewrites = "on"  
     security_level = "medium"  
     minify {  
     css = "on"  
     js = "on"  
     html = "on"  
     }  
     browser_cache_ttl = "14400"  
     }  
    }

use the terraform plan to check if everything it's of and terrafom plan to apply the changes
    
    
    $ terraform apply  
    cloudflare_record.mx: Refreshing state... (ID: c3624cc8fd163199ec082649a55f337a)  
    cloudflare_record.services: Refreshing state... (ID: 09041dfd16cbbf3b6915e79b14dc5466)  
    cloudflare_page_rule.always_use_https_services: Refreshing state... (ID: 92e196e6c7c8cba1f0759a60d713b0b9)  
    cloudflare_record.spf: Refreshing state... (ID: 3743b9f0be0a2c7a69245e169cf06ea5)  
    cloudflare_zone_settings_override.k8s-settings: Refreshing state... (ID: d4acc5e5713a0dede4f238b829f98947)  
    cloudflare_record.smtp: Refreshing state... (ID: 2d4628fc07c491d97e34b07da9ec60db)  
    cloudflare_page_rule.always_use_https_www: Refreshing state... (ID: 59c25cdd1096177a1019f834178de62f)  
    cloudflare_record.google-verification: Refreshing state... (ID: 131f35a9f8d2304a786a780db38b37e0)  
    cloudflare_record.www: Refreshing state... (ID: 76e3906f5ea172b627bed4922ebc1da7)  
      
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
    
    
    $ host www.k8s.it  
    www.k8s.it has address 104.27.164.146  
    www.k8s.it has address 104.27.165.146  
    www.k8s.it has IPv6 address 2400:cb00:2048:1::681b:a492  
    www.k8s.it has IPv6 address 2400:cb00:2048:1::681b:a592
    
    
    $ echo | openssl s_client -servername www.k8s.it -connect www.k8s.it:443  
    CONNECTED(00000003)  
    depth=2 C = GB, ST = Greater Manchester, L = Salford, O = COMODO CA Limited, CN = COMODO ECC Certification Authority  
    ---  
    Certificate chain  
     0 s:/OU=Domain Control Validated/OU=PositiveSSL Multi-Domain/CN=sni154797.cloudflaressl.com  
     i:/C=GB/ST=Greater Manchester/L=Salford/O=COMODO CA Limited/CN=COMODO ECC Domain Validation Secure Server CA 2  
     1 s:/C=GB/ST=Greater Manchester/L=Salford/O=COMODO CA Limited/CN=COMODO ECC Domain Validation Secure Server CA 2  
     i:/C=GB/ST=Greater Manchester/L=Salford/O=COMODO CA Limited/CN=COMODO ECC Certification Authority  
     2 s:/C=GB/ST=Greater Manchester/L=Salford/O=COMODO CA Limited/CN=COMODO ECC Certification Authority  
     i:/C=SE/O=AddTrust AB/OU=AddTrust External TTP Network/CN=AddTrust External CA Root
