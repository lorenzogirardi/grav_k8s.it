---
title: 'Maximum yield with minimum expense for a static website'
date: '2020-12-24 20:53'
taxonomy:
    tag:
            - cloudflare
            - cms
            - headers
            - kubernetes
            - minio
            - publii
            - security
            - serverless
            - static
            - static cms
            - website
            - workers
---

Great marketing quote, however i think we can better share a value that is always true...  
_keep it simple, keep it safe._

I would structure this article in few relevant topics

  * Static website
  * Tools
  * Security
  * Monitor



Each of the above could be a specific topic that will fit others use cases

 _This is not a post about how to create a blog... this post will inspect into some technology that can simplify your life or your work with small but important concepts_

### Static website

Among the famous open cms i tried probably all of those, during my career.  
Basically are a web application with sql database in most of the cases.

Now we can discuss about the meaning of content generation , that should be dynamic and be part of the user experience rather than a page generated, however today i will talk just about the usage of a cms for a static(tbd) reality.

If your goal is promote your company, your hobbies, your work ... you have already heard about wordpress, joomla, drupal etc etc ... that are good (if not directly exposed) but had a huge problem... since are open and used by a relevant number of people, are subjected of vulnerability, moreover , if you install plugins the situation could be dramatic.

As much you install **plugins** (and really some cms are not able to do anything with a plugin),  
as much your website is bloat.  
With the same ratio the performances will decrease.

On top of this, a huge percentage of the plugins in the cms repository are old and out of date yet still available to install on your system.

So if you don't need something special (most of the blogs are wordpress only because it's easy to use) and you want to create a static website for your company, your business and so on , you can probably evaluate something different looking for , the first pillar mentioned, keep it simple.  
Moreover the trend in a enterprise is moving from dynamic content to static as per the **speed** is the major KPI for a website

On a secondary binary in the last years the static cms are growth

  * Jekyll
  * Gatsby
  * Siteleaf
  * Netlify
  * Hugo
  * $whatever-js (every day we have a new js framework)
  * etc etc



However if you are not so nerd ... or your are tremendously lazy (like me)  
a grate alternative is [Publii](https://getpublii.com/)

Publii includes a WYSIWYG (What You See Is What You Get) editor for content creation,  
really easy to use , and the client is available for multiple platform (win,linux,mac)

### Tools

I mentioned Publii before, this tool is just a simple way for lazy people to create a static website, i'll not go in deep with this software since is well explained in its website however offer multiple possibility where store the files , the main ones are s3 bucket and github pages.  
Publii is not a pretty collaborative tool but is not supposed to have this kind of interaction on a static website, is more close to a human interaction only.  
On the opposite side Jekyll and the other families are more close to be collaborative based on a git project and part of a pipeline flow (runner/actions)

Based on the backend for you html files, lets explore the [github pages](https://pages.github.com/)  
This service is free with few limits  
 _Published GitHub Pages sites may be no larger than 1 GB._  
_GitHub Pages sites have a soft bandwidth limit of 100GB per month._  
_GitHub Pages sites have a soft limit of 10 builds per hour._

It's really easy to use and demand the security on the github account (always 2fa)  
They offer a dns based on your github account and you can manage paths,  
moreover you have the TLS/SSL for free

With Publii configuration is quite easy and you can follow the instruction [here](https://getpublii.com/docs/host-static-website-github-pages.html)

Another solution is use s3 as a repository, this require a bit more stuff sometimes, however even if it's a cheeper option, why not use an s3 on-prem if it's available   
(this is just an excuse to use kubernetes, i still suggest to use the github pages, however i like to have this kind of exercise)

So welcome [minio](https://min.io/) , a kubernetes object storage  
The configuration is quite simple, as usual you can start with the namespace creation  
`kind: "Namespace"  
apiVersion: "v1"  
metadata:  
name: "minio"  
labels:  
name: "minio"`

The service (this is just a single instance so no StatefulSets set is needed)  
`apiVersion: v1  
kind: Service  
metadata:  
name: minio-svc  
namespace: minio  
labels:  
app: minio  
spec:  
ports:  
- port: 9000  
protocol: TCP  
selector:  
app: minio`

PersistentVolumeClaim (from microk8s host provisioner)  
`apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
namespace: minio  
name: minio-pv-claim  
labels:  
app: minio-storage-claim  
spec:  
storageClassName: microk8s-hostpath  
accessModes:  
- ReadWriteOnce  
resources:  
requests:  
storage: 1Gi`

Deployment (remember to use secrets instead write directly into deployment)  
`kind: Deployment  
metadata:  
name: minio-deployment  
namespace: minio  
spec:  
selector:  
matchLabels:  
app: minio  
strategy:  
type: Recreate  
template:  
metadata:  
labels:  
app: minio  
spec:  
volumes:  
- name: storage  
persistentVolumeClaim:  
claimName: minio-pv-claim  
containers:  
- name: minio  
image: minio/minio:latest  
args:  
- server  
- /storage  
env:  
- name: MINIO_ACCESS_KEY  
value: "$something"  
- name: MINIO_SECRET_KEY  
value: "$somethingsecret"  
- name: MINIO_DOMAIN  
value: minio.h4x0r3d.lan  
ports:  
- containerPort: 9000  
hostPort: 9000  
volumeMounts:  
- name: storage  
mountPath: "/storage"`

Ingress resource  
`kind: Ingress  
metadata:  
name: minio-ingress  
namespace: minio  
annotations:  
kubernetes.io/ingress.class: nginx  
nginx.org/client-max-body-size: 1000m  
ingress.kubernetes.io/proxy-body-size: 1000m  
spec:  
rules:  
- host: minio.h4x0r3d.lan  
http:  
paths:  
- path: /  
backend:  
serviceName: minio-svc  
servicePort: 9000`

And at the end you will have something like this

![](/user/images/maximum-yield-with-minimum-expense/Screenshot-2020-12-26-at-13.55.38.png)

you can configure Publii in this way

where bucket is the third level name of you dns and prefix "the bucket"

![](/user/images/maximum-yield-with-minimum-expense/Screenshot-2020-12-26-at-13.56.43.png)

Btw an object storage is **NOT** a web server so you need an apache or nginx configuration  
to have the site ready to be used  
`<VirtualHost *:80>  
ServerAdmin [[email protected]](</cdn-cgi/l/email-protection>)  
DocumentRoot /usr/local/apache2/htdocs/  
ServerName www.k8s.it  
ErrorLog logs/www.k8s-error_log  
CustomLog logs/www.k8s-access_log combined  
LoadModule rewrite_module modules/mod_rewrite.so  
ProxyRequests Off  
ProxyPass / http://minio-svc.minio.svc.cluster.local:9000/blog/  
RewriteEngine on  
RewriteRule ^(.*)/$ /$1/index.html [PT,L]  
</VirtualHost>`

Again ... **keep it simple**... there no right solutions overall , there is the right way for you needs, in my case i'm still using github pages because are free, enough to cover my needs and _serverless_

### 

### Security

Even if the security is demanded to github, why not extend a sort of protection on top of it

Second pillar , **keep it safe**

In my case i really appreciate Cloudflare and the free plan that give me the possibility to test and explore the functionality offered

I've already shared how to configure a website with Cloudflare with terraform <https://www.k8s.it/terraform-your-free-claudflare-account.html>

Even if we are covered by attacks since it's static webpage ... and we also added the Cloudflare protection we can have some _cosmetics_ feature that will share a better compliance with the security in 2020

As i said github pages and s3 are not webserver so ...  
how we can introduce the security headers ?

![](/user/images/maximum-yield-with-minimum-expense/screencapture-securityheaders-2020-12-24-20_33_23.png)

Again Cloudflare help us with the [Workers](https://blog.cloudflare.com/cloudflare-workers-unleashed/) feature

![](/user/images/maximum-yield-with-minimum-expense/screencapture-dash-cloudflare-9fb3653b948e447567a30f6b83e51eaf-workers-edit-security-headers-2020-12-24-20_38_25-2.png)

The code ([rif](https://scotthelme.co.uk/security-headers-cloudflare-worker/).) is adding those headers with serverless function for each request

`const securityHeaders = {  
"Content-Security-Policy": "upgrade-insecure-requests",  
"Strict-Transport-Security": "max-age=3600",  
"X-Xss-Protection": "1; mode=block",  
"X-Frame-Options": "DENY",  
"X-Content-Type-Options": "nosniff",  
"Permissions-Policy": "geolocation=()",  
"Referrer-Policy": "strict-origin-when-cross-origin"  
};  
  
  
async function addHeaders(req) {  
const response = await fetch(req),  
newHeaders = new Headers(response.headers),  
setHeaders = Object.assign({}, securityHeaders);  
  
if (newHeaders.has("Content-Type") && !newHeaders.get("Content-Type").includes("text/html")) {  
return new Response(response.body, {  
status: response.status,  
statusText: response.statusText,  
headers: newHeaders  
});  
}  
  
Object.keys(setHeaders).forEach(name => newHeaders.set(name, setHeaders[name]));  
  
  
return new Response(response.body, {  
status: response.status,  
statusText: response.statusText,  
headers: newHeaders  
});  
}  
  
addEventListener("fetch", event => event.respondWith(addHeaders(event.request)));`

Now the website is better from this point of view, adding the worker to the main route

![](/user/images/maximum-yield-with-minimum-expense/Screenshot-2020-12-27-at-13.41.47.png)

Results

![](/user/images/maximum-yield-with-minimum-expense/screencapture-securityheaders-2020-12-24-20_34_27.png)

Just a warning because Strict-Transport-Security has a low value , but it's something acceptable to me

Cloudflare details the usage of workers that is limited in the free account for 100k req/day and 1k/min

![](/user/images/maximum-yield-with-minimum-expense/screencapture-dash-cloudflare-9fb3653b948e447567a30f6b83e51eaf-workers-view-security-headers-2020-12-27-13_42_52.png)

OKR 

  * website is managed with no database
  * we have serverless way to host it and with free options
  * overall protection from Cloudflare
  * security headers from Cloudflare workers, again another serveless option



``````

````

### Monitor

What is missing now ?

A bit of **awareness**

Are we safe ? maybe but better an external opinion , around the online scanner there are multiple tools , however [Probely](https://probely.com/) has a good free plan

![](/user/images/maximum-yield-with-minimum-expense/Screenshot-2020-12-26-at-14.58.41.png)

Here some hints about sensitive data shared with their crowler

![](/user/images/maximum-yield-with-minimum-expense/Screenshot-2020-12-26-at-15.00.00.png)

Nothing relevant in my case... but useful to know

Is now time to know , how is performing the website to do this i already had a deepdive about website usability

<https://www.k8s.it/kubernetes-sitespeedio.html>

but again we have some other feature serverless style like [upptime](https://github.com/upptime/upptime) ,with github actions you can create a monitoring system like this <https://lorenzogirardi.github.io/status/>

![](/user/images/maximum-yield-with-minimum-expense/Screenshot-2020-12-26-at-15.04.18.png)![](/user/images/maximum-yield-with-minimum-expense/Screenshot-2020-12-27-at-13.36.35.png)

Another alternative is [hetrixtools](https://hetrixtools.com/) , free up top 15 probes that give you the possibility to monitor your websites and smtp, and have an alerting system out of the box with a telegram bot email and sms (up to 50)

![](/user/images/maximum-yield-with-minimum-expense/screencapture-hetrixtools-report-uptime-c7596615176a822f81d67cfddc4aa179-2020-12-26-15_14_22.png)![](/user/images/maximum-yield-with-minimum-expense/Screenshot-2020-12-26-at-15.07.32.png)

Hope this post cover how the technology could be used as much as possible in a small and cheap scenario to reflect some hints to an enterprise scenario.
