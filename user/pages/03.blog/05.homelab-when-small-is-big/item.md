---
title: 'Homelab , when small is big'
date: '2021-02-19 16:33'
taxonomy:
    tag:
            - backup
            - cluster
            - ghettovcb
            - homelab
            - k3s
            - kubernetes
            - microk8s
            - vmware
            - vspehere
---

When I started getting interested on the IT world , i've usually kept all my devices on 24 h to 24.

The first labs was an impressive tower in '99 with a huge amount of disk and memory banks, a good cpu cooled by a giant copper heatsink combined with a 12cm fan.  
All those components were quite similar to a helicopter noise during the night.

The evolution of the lab than , changed a lot year by year , not only reaching a good level of noise but mostly to be cost effective hardware that guarantee new a semplificate production environment.

Now some goals has changed compared with the '99 approach so let's summarize the lab expectations:

  * Cheap hardware 
  * Noiseless 
  * TDP under 20w
  * 24/24 uptime
  * Virtualization support 
  * Enough to have a Kubernetes installation
  * Backup solution
  * Automation ready 



Let's now explain the needs behind,  
i'd like to have a solution that is able to cover all my needs in terms of hobby and experiments, i don't need power , since i take care to the goodness of the solution instead the absolute performance.

### Hardware

Base: [Intel® NUC NUC7PJYH](<https://ark.intel.com/content/www/us/en/ark/products/126137/intel-nuc-kit-nuc7pjyh.html>)

![](/user/images/homelab-when-small-is-big/Screenshot-2021-02-19-at-16.39.21-2.png)

RAM: even if the website is reporting 8gb max... this is not true  
on my installation i have 16GB with G.Skill Ripjaws SO-DIMM 16GB DDR4-2400Mhz 2x8GB

![](/user/images/homelab-when-small-is-big/Screenshot-2021-02-19-at-16.39.50-2.png)

CPU: j5005 i thinks is the best tdp/performance (here compared to the old q6600 the first intel quadcore ever)

![](/user/images/homelab-when-small-is-big/j5005vsq6600.png)

DISK: 250gb with ocz vertex4 are a good starting point 

![](/user/images/homelab-when-small-is-big/Screenshot-2021-02-19-at-16.40.09.png)

### Software

Base os vmware esxi 6.7 (free/essential)

Now:

**Why** vmware?  
Because the virtualization stack is really important to have an abstraction layer ,  
see this [post](<https://www.k8s.it/kubernetes-destroyed-the-virtualization-or-not.html>)

**Why** 6.7?  
Because the vmkernel in 7.0 is changed and is not possible import external network driver (needed for this nuc model), even if there is a trick with usb network adapter and <https://flings.vmware.com/usb-network-native-driver-for-esxi> vmware labs usb drivers.

**How** build the image with Realtek network card ?  
First of all you need a windows :( , than you should need to identify your hardware driver , in my case <https://vibsdepot.v-front.de/wiki/index.php/Net55-r8168> and at the end you need to rebuild the image with [ESXi-Customizer-PS](<https://www.v-front.de/p/esxi-customizer-ps.html>)

![](/user/images/homelab-when-small-is-big/Screenshot-2021-02-19-at-17.16.41.png)

**What** we can do with this "toy"?  
Well more or less everything you are doing in your datacenter, using packer and ansible for the installation and vm configuration , you can boostrap a k3s cluster or a microk8s standalone kubernetes, few virtualmachines and so on.  
  
**But** is small compared to a server !1!1!   
different needs , different size ( but same mindset in terms of reliability , flexibility , usability)... however this is my current usage

4 VMS:

  * microk8s installation 
  * monitoring infrastructure
  * windows active directory
  * console vm (usually used as ephemeral environment)

![](/user/images/homelab-when-small-is-big/Screenshot-2021-02-19-at-17.26.10.png)

If you have some doubts about the possibility to run kubernetes in a _toy_ , is the base that i used for all my posts

![](/user/images/homelab-when-small-is-big/Screenshot-2021-02-19-at-17.30.13.png)

### Monitoring

Well again grafana and influxdb are supporting me

HOST

![](/user/images/homelab-when-small-is-big/Screenshot-2021-02-19-at-17.34.04.png)

VMS

![](/user/images/homelab-when-small-is-big/vmsgrafana.png)

### Backup

**Why** backup?  
Because even if the env is automated, some data are inside the vms and i'd like to play with the env and if needed restore the previous snapshot.

Again another free tool is helping me <https://github.com/lamw/ghettoVCB>

It's a quite simple script that i used for non production environment since the beginning.

In the main path i've created a _script_ folder with ghetto inside and other stuff like packer firewall rules

`[root@amaterasu:/vmfs/volumes/5eafe677-1260d7e8-2521-94c691a54f7d/script] ls -la  
total 2240  
drwxr-xr-x 1 root root 73728 May 6 2020 .  
drwxr-xr-t 1 root root 77824 Dec 28 10:51 ..  
-rw-r--r-- 1 root root 577 May 4 2020 email.xml  
-rwxr-xr-x 1 root root 17207 May 4 2020 ghettoVCB-restore.sh  
-rw-r--r-- 1 root root 309 May 4 2020 ghettoVCB-restore_vm_restore_configuration_template  
-rw-r--r-- 1 root root 356 May 4 2020 ghettoVCB-vm_backup_configuration_template  
-rw-r--r-- 1 root root 832 May 10 2020 ghettoVCB.conf  
-rwxr-xr-x 1 root root 72122 May 4 2020 ghettoVCB.sh  
-rwxr-xr-x 1 root root 404 May 12 2020 init.sh  
-rw-r--r-- 1 root root 372 May 4 2020 packer.xml`

The init.sh files should be added on local.sh to have it running on all machine startup   
`[root@amaterasu:/vmfs/volumes/5eafe677-1260d7e8-2521-94c691a54f7d/script] cat /etc/rc.local.d/local.sh  
#!/bin/sh  
  
# local configuration options  
  
# Note: modify at your own risk! If you do/use anything in this  
# script that is not part of a stable API (relying on files to be in  
# specific places, specific tools, specific output, etc) there is a  
# possibility you will end up with a broken system after patching or  
# upgrading. Changes are not supported unless under direction of  
# VMware support.  
  
# Note: This script will not be run when UEFI secure boot is enabled.  
  
sh -x /vmfs/volumes/amaterasu-datastore/script/init.sh  
  
exit 0`

... on the init file you need to apply all changes to have the ghetto running in crontab

`[root@amaterasu:/vmfs/volumes/5eafe677-1260d7e8-2521-94c691a54f7d/script] cat init.sh  
#!/bin/bash  
cp /vmfs/volumes/amaterasu-datastore/script/email.xml /etc/vmware/firewall/  
cp /vmfs/volumes/amaterasu-datastore/script/packer.xml /etc/vmware/firewall/  
esxcli network firewall refresh  
echo "0 22 * * 6 /vmfs/volumes/amaterasu-datastore/script/ghettoVCB.sh -a -g /vmfs/volumes/amaterasu-datastore/script/ghettoVCB.conf  
" >> /var/spool/cron/crontabs/root  
kill $(cat /var/run/crond.pid)  
crond`

it is missing only the ghettoVCB.conf configuration that depends on your needs

`[root@amaterasu:/vmfs/volumes/5eafe677-1260d7e8-2521-94c691a54f7d/script] cat ghettoVCB.conf  
DISK_BACKUP_FORMAT=thin  
VM_BACKUP_ROTATION_COUNT=3  
POWER_VM_DOWN_BEFORE_BACKUP=0  
ENABLE_HARD_POWER_OFF=0  
ITER_TO_WAIT_SHUTDOWN=3  
POWER_DOWN_TIMEOUT=5  
ENABLE_COMPRESSION=0  
VM_SNAPSHOT_MEMORY=0  
VM_SNAPSHOT_QUIESCE=0  
ALLOW_VMS_WITH_SNAPSHOTS_TO_BE_BACKEDUP=0  
ENABLE_NON_PERSISTENT_NFS=1  
UNMOUNT_NFS=1  
NFS_SERVER=192.168.1.13  
NFS_VERSION=nfs  
NFS_MOUNT=/export/backup  
NFS_LOCAL_NAME=nfs_storage_backup  
NFS_VM_BACKUP_DIR=amaterasu  
ENABLE_NFS_IO_HACK=1  
NFS_IO_HACK_LOOP_MAX=10  
NFS_IO_HACK_SLEEP_TIMER=60  
SNAPSHOT_TIMEOUT=15  
EMAIL_ALERT=0  
EMAIL_LOG=1  
EMAIL_SERVER=smtp.k8s.it  
EMAIL_SERVER_PORT=25  
EMAIL_DELAY_INTERVAL=1  
EMAIL_USER_NAME=  
EMAIL_USER_PASSWORD=  
[[email protected]](</cdn-cgi/l/email-protection>)  
[[email protected]](</cdn-cgi/l/email-protection>)  
[[email protected]](</cdn-cgi/l/email-protection>)  
WORKDIR_DEBUG=0  
VM_SHUTDOWN_ORDER=  
VM_STARTUP_ORDER=`

And ... well thats all ... you have your weekly snapshot   
`2021-02-13 22:00:00 -- info: ============================== ghettoVCB LOG START ==============================  
  
2021-02-13 22:00:01 -- info: CONFIG - USING GLOBAL GHETTOVCB CONFIGURATION FILE = /vmfs/volumes/amaterasu-datastore/script/ghettoVCB.conf  
2021-02-13 22:00:01 -- info: CONFIG - VERSION = 2019_01_06_4  
2021-02-13 22:00:01 -- info: CONFIG - GHETTOVCB_PID = 2250208  
2021-02-13 22:00:01 -- info: CONFIG - VM_BACKUP_VOLUME = /vmfs/volumes/nfs_storage_backup/amaterasu  
2021-02-13 22:00:01 -- info: CONFIG - ENABLE_NON_PERSISTENT_NFS = 1  
2021-02-13 22:00:01 -- info: CONFIG - UNMOUNT_NFS = 1  
2021-02-13 22:00:01 -- info: CONFIG - NFS_SERVER = 192.168.1.13  
2021-02-13 22:00:01 -- info: CONFIG - NFS_VERSION = nfs  
2021-02-13 22:00:01 -- info: CONFIG - NFS_MOUNT = /export/backup  
2021-02-13 22:00:01 -- info: CONFIG - VM_BACKUP_ROTATION_COUNT = 3  
2021-02-13 22:00:01 -- info: CONFIG - VM_BACKUP_DIR_NAMING_CONVENTION = 2021-02-13_22-00-00  
2021-02-13 22:00:01 -- info: CONFIG - DISK_BACKUP_FORMAT = thin  
2021-02-13 22:00:01 -- info: CONFIG - POWER_VM_DOWN_BEFORE_BACKUP = 0  
2021-02-13 22:00:01 -- info: CONFIG - ENABLE_HARD_POWER_OFF = 0  
2021-02-13 22:00:01 -- info: CONFIG - ITER_TO_WAIT_SHUTDOWN = 3  
2021-02-13 22:00:01 -- info: CONFIG - POWER_DOWN_TIMEOUT = 5  
2021-02-13 22:00:01 -- info: CONFIG - SNAPSHOT_TIMEOUT = 15  
2021-02-13 22:00:01 -- info: CONFIG - LOG_LEVEL = info  
2021-02-13 22:00:01 -- info: CONFIG - BACKUP_LOG_OUTPUT = /tmp/ghettoVCB-2021-02-13_22-00-00-2250208.log  
2021-02-13 22:00:01 -- info: CONFIG - ENABLE_COMPRESSION = 0  
2021-02-13 22:00:01 -- info: CONFIG - VM_SNAPSHOT_MEMORY = 0  
2021-02-13 22:00:01 -- info: CONFIG - VM_SNAPSHOT_QUIESCE = 0  
2021-02-13 22:00:01 -- info: CONFIG - ALLOW_VMS_WITH_SNAPSHOTS_TO_BE_BACKEDUP = 0  
2021-02-13 22:00:01 -- info: CONFIG - VMDK_FILES_TO_BACKUP = all  
2021-02-13 22:00:01 -- info: CONFIG - VM_SHUTDOWN_ORDER =  
2021-02-13 22:00:01 -- info: CONFIG - VM_STARTUP_ORDER =  
2021-02-13 22:00:01 -- info: CONFIG - RSYNC_LINK = 0  
2021-02-13 22:00:01 -- info: CONFIG - BACKUP_FILES_CHMOD =  
2021-02-13 22:00:01 -- info: CONFIG - EMAIL_LOG = 1  
2021-02-13 22:00:01 -- info: CONFIG - EMAIL_SERVER = [smtp.xxs.it](<http://smtp.k8s.it/>)  
2021-02-13 22:00:01 -- info: CONFIG - EMAIL_SERVER_PORT = 25  
2021-02-13 22:00:01 -- info: CONFIG - EMAIL_DELAY_INTERVAL = 1  
2021-02-13 22:00:01 -- info: CONFIG - EMAIL_FROM = [[email protected]](</cdn-cgi/l/email-protection#d4b3bcb1a0a0bb829796f9b5b9b5a0b1a6b5a7a194bfeca7fabda0>)  
2021-02-13 22:00:01 -- info: CONFIG - EMAIL_TO = [[email protected]](</cdn-cgi/l/email-protection#cdafacaea6b8bd8da6f5bee3a4b9>)  
2021-02-13 22:00:01 -- info: CONFIG - WORKDIR_DEBUG = 0  
2021-02-13 22:00:01 -- info: CONFIG - ENABLE NFS IO HACK = 1  
2021-02-13 22:00:01 -- info: CONFIG - NFS IO HACK LOOP MAX = 10  
2021-02-13 22:00:01 -- info: CONFIG - NFS IO HACK SLEEP TIMER = 60  
2021-02-13 22:00:01 -- info: CONFIG - NFS BACKUP DELAY = 0  
  
2021-02-13 22:00:06 -- info: Initiate backup for izanami  
2021-02-13 22:00:06 -- info: Creating Snapshot "ghettoVCB-snapshot-2021-02-13" for izanami  
2021-02-13 22:09:55 -- info: Removing snapshot from izanami ...  
2021-02-13 22:10:56 -- info: Slept 60 seconds to work around NFS I/O error  
2021-02-13 22:10:56 -- info: Backup Duration: 10.83 Minutes  
2021-02-13 22:10:56 -- info: Successfully completed backup for izanami!  
  
2021-02-13 22:11:02 -- info: Initiate backup for izanagi  
2021-02-13 22:11:02 -- info: Creating Snapshot "ghettoVCB-snapshot-2021-02-13" for izanagi  
2021-02-13 22:16:37 -- info: Removing snapshot from izanagi ...  
2021-02-13 22:17:37 -- info: Slept 60 seconds to work around NFS I/O error  
2021-02-13 22:17:37 -- info: Backup Duration: 6.58 Minutes  
2021-02-13 22:17:37 -- info: Successfully completed backup for izanagi!  
  
2021-02-13 22:17:43 -- info: Initiate backup for hiroito  
2021-02-13 22:17:43 -- info: Creating Snapshot "ghettoVCB-snapshot-2021-02-13" for hiroito  
2021-02-13 22:23:55 -- info: Removing snapshot from hiroito ...  
2021-02-13 22:24:56 -- info: Slept 60 seconds to work around NFS I/O error  
2021-02-13 22:24:56 -- info: Backup Duration: 7.22 Minutes  
2021-02-13 22:24:56 -- info: Successfully completed backup for hiroito!  
  
2021-02-13 22:25:01 -- info: Initiate backup for raijin  
2021-02-13 22:25:01 -- info: Creating Snapshot "ghettoVCB-snapshot-2021-02-13" for raijin  
2021-02-13 22:26:21 -- info: Removing snapshot from raijin ...  
2021-02-13 22:27:21 -- info: Slept 60 seconds to work around NFS I/O error  
2021-02-13 22:27:21 -- info: Backup Duration: 2.33 Minutes  
2021-02-13 22:27:21 -- info: Successfully completed backup for raijin!  
  
2021-02-13 22:27:54 -- info: ###### Final status: All VMs backed up OK! ######  
  
2021-02-13 22:27:54 -- info: ============================== ghettoVCB LOG END ================================`  
  


### Price 

nuc7PJYH ~ 160€  
G.Skill Ripjaws SO-DIMM 16GB DDR4-2400Mhz 2x8GB ~ 60€  
hard disk ssd 250db ~ 40€  
  
Is not cheap as a raspberry 4 but it's 10 time more performant and guarantee a full server experience with the possibility to play and grow your skills and challenges
