---
title: 'Kubernetes has destroyed the virtualization... or NOT'
date: '2020-12-07 14:40'
taxonomy:
    tag:
            - cloud
            - datacenter
            - kubernetes
            - pod
            - scalability
            - virtual machines
            - virtualization
---

Well ... **NO!**

Ok maybe are missing some context

I started this topic looking around the data center world compared with the cloud providers,  
the advantage of manage services , the mindset infrastructure as a code and so on ...

In the last 4y we saw a kubernetes rush , a lot of company are sharing the advantages and the perfect picture create into kubernetes, how they save money or scale fast ... but is always true what we see?  
Most of the times we are saw only builded success case.

Today we will take in consideration those topics :

  * Virtual machines
  * Data center
  * Kubernetes



I know that the Data center topic should be covered more in deep, maybe talking about the entire ecosystem (db, vertical services , corporate usage , LEGACY!!!!) however today i'd like to explore only the kubernetes and virtualization rather than lose the discussion in thousand of concepts.

### Long story short

A long time ago Borg was born in Google .. now formally named kubernetes,  
in the first period the main evaluation was:  
"_why the virtualization overhead if we will use kubernetes_ ".  
  
This concept was justified in some way for  _overlapping_ of duties,  
if i migrate a vm as a kubernetes pod this means i can remove the vm   
  


![](/user/images/kubernetes-destroyed-the-virtualization-or-not/vm_vs_pod.png)

We can imagine a picture like this looking back of some years ago

![](/user/images/kubernetes-destroyed-the-virtualization-or-not/vm_1.png)The introduction was good , almost all declared this could be a new way.  
Moving faster, all companies changed a bit this picture with an expansion of kubernetes ![](/user/images/kubernetes-destroyed-the-virtualization-or-not/vm_2.png)![](Kubernetes destroyed the virtualization... or NOT - LAB_files/vm_2.png)

Like an infection (good this time, not the 20covid ones) kubernetes come as a main platform for most of the online company...  
In a datacenter , you know, there no something eternal:

  * hardware in EOL
  * generation refresh
  * brand change
  * good is ok but cheaper is better (usually from finance)



Is also true , that sometimes this encouraged a forced/incorrect usage of kubernetes node labeling in order to bend the infrastructure to better fit the products migrated or the existing hardware availability

![](/user/images/kubernetes-destroyed-the-virtualization-or-not/vm_3.png)![](Kubernetes destroyed the virtualization... or NOT - LAB_files/vm_3.png)

### Houston we have a problem...

Working in this way, the environment could be fragmented ,  
even if the technology is the same there are some logical segmentations that create some problems in the day by day activities.

  * isolate some workloads with labels can create an hw lock-in (better approach a multi-cluster with unique controlplane)
  * updates require to be managed in a different way looking for the label/role
  * please do not touch the storage nodes now (fiber channel with external storage)
  * please do not touch the storage nodes tomorrow (fiber channel with external storage)
  * omg we lost a storage nodes (fiber channel with external storage)
  * and so on ...



Moreover an hardware fault could create problems ....  
YES i know in a infrastructure as a code , with puppet foreman chef salt ansible terraform etc, this will be recovered fast ... but this is not always true ... you know... some machines/roles in a mid/ent company are still fragile.

  * legacy with no ownership
  * legacy with wrong ownership
  * single point of failure ... (yes also in 2020)



Other problems are related with new hardware...

  * is the provisioning working as expected with new generations ?
  * and if we change the hardware vendor ?
  * etc etc when you have to handle new preseed/ks because the layout is changed ☠️



Last but not least ... what about the bare metal density ?

`Allocated resources:`  
` (Total limits may be over 100 percent, i.e., overcommitted.`  
` CPU Requests CPU Limits Memory Requests Memory Limits`  
` ------------ ---------- --------------- -------------`  
` 55 (98%) 101300m (180%) 86376Mi (44%) 121672Mi (62%)`

This HW has 56 core with HT , we reached the 98% but this number is high  
because some pods needs a default cpu just to fit the nodes and start, while the running cost is really low. 

### So why not kubernetes hosted in virtual machines ?

  * Having an abstraction layer, is quite simple tune the provisioning based on this , rather than the bare metal, it's just a metter of roles not to the entire provisioning.
  * Hardware fault could be easily covered by live migration
  * Maintenance and update can be manage creating a blue green approach and rolling updates
  * Density could be increased with mode nodes in the same hw
  * Flexibility to scale up and to upset a new datacenter in case of migration will be higher
  * Team at the and is also able to scale , working from the vmware and not from bare metal

![](/user/images/kubernetes-destroyed-the-virtualization-or-not/team_scale.png)

### 

### Conclusions

Virtualization and kubernetes are not antagonist, must be in addition , the first one has just change the scope to better cooperate with kubernetes

Instead a layer for microservices , virtualization is now the layer that guarantee an homogeneous platform were a team can work on top to create the infrastructure having the advantage of a fully consolidated technology that is able to promote the right abstraction 

Last but not least , we don't need to reinvent something , we can just copy from the major competitors for the kubernetes topic  
How is working GKE , AKS , EKS ? .... virtual machines :)
