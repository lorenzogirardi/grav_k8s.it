---
title: 'Kubernetes for mere mortals'
date: '2018-04-10 23:29'
taxonomy:
    tag:
            - arm
            - armbian
            - k8s
            - kubernetes
            - lab
            - orangepi
            - traefik
---

Well, what do you need to build your homemade cluster?

![](/user/images/kubernetes-for-mere-mortals/k8s.arm-lg.jpeg)

![](/user/images/kubernetes-for-mere-mortals/IMG_20170605_150237.jpg)

  * 1x usb hub Anker 60W PowerPort 6
  * 4x orangepi plus 2e



armbian 4.10.1 

kubeadm and kubelet 1.6.4

traefik as a ingress controller

heapster, collectd, influxdb and grafana as a monitoring stack

![](https://media.licdn.com/dms/image/C5612AQFJsWW1J0HYig/article-inline_image-shrink_1500_2232/0?e=2122596000&v=beta&t=O4agFAs9-5g0xGt6frAU-koLZROOeVmv2p7yhmZyEAY)

![](/user/images/kubernetes-for-mere-mortals/0)
    
    
    NAMESPACE     NAME                                        READY     STATUS    RESTARTS   AGE
    home-prd      k8s-helloworld-3468567185-2fwhb             1/1       Running   0          10d
    home-prd      k8s-helloworld-3468567185-lbgpk             1/1       Running   2          73d
    home-prd      k8s-helloworld-3468567185-vz9n5             1/1       Running   2          73d
    kube-system   etcd-k8s-node001                            1/1       Running   19         81d
    kube-system   heapster-3703175019-7fz27                   1/1       Running   2          76d
    kube-system   kube-apiserver-k8s-node001                  1/1       Running   8          81d
    kube-system   kube-controller-manager-k8s-node001         1/1       Running   22         81d
    kube-system   kube-dns-279829092-r8595                    3/3       Running   39         81d
    kube-system   kube-flannel-ds-08z1k                       2/2       Running   44         81d
    kube-system   kube-flannel-ds-3bqmf                       2/2       Running   0          10d
    kube-system   kube-flannel-ds-f51rq                       2/2       Running   10         81d
    kube-system   kube-flannel-ds-l5wjs                       2/2       Running   8          81d
    kube-system   kube-proxy-3gn3x                            1/1       Running   12         81d
    kube-system   kube-proxy-88762                            1/1       Running   2          81d
    kube-system   kube-proxy-dghmh                            1/1       Running   0          10d
    kube-system   kube-proxy-dtf9c                            1/1       Running   3          81d
    kube-system   kube-scheduler-k8s-node001                  1/1       Running   29         81d
    kube-system   kubernetes-dashboard-1707270776-jgxbp       1/1       Running   2          81d
    kube-system   traefik-ingress-controller-49053153-kgb3p   1/1       Running   1          30d
    

backend docker example (nginx on arm) k8s-helloworld-arm

[live demo](<https://services.k8s.it/hello/>)

[monitoring](<https://services.k8s.it/grafana/dashboard/db/all-k8s-nodes?refresh=1m&orgId=2&from=now-24h&to=now>)
