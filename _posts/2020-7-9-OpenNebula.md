---
layout: post
title: Open Nebula
---

## I. System Information
On this example of Open Nebula (ONE) installation guides, a total of 7 servers are used; 1 server for ONE Front-end, 2 servers for KVM-Host hipervisor, 3 servers for CEPH Storage cluster, and 1 server for SAN Storage.

| Server Role | Hostname | OS | Management Network | Storage Network | Cluster Network |
| ----------- | -------- | --- | ----------------: | --------------: | --------------: |
| ONE Front-end | nebula-fe.nuansa.com | Centos 7 | 192.168.0.35 | 192.168.100.35 | - |
| KVM Host1 | one-kvm1.nuansa.com | Centos 7 | 192.168.0.32 | 192.168.100.32 | -	|
| KVM-Host2 | one-kvm2.nuansa.com | Centos 7 | 192.168.0.33 | 192.168.100.33 | - |
| CEPH node-1 | one-ceph1.nuansa.com | Proxmox 5.4 | 192.168.0.43 | 192.168.100.43 | 192.168.200.43 |
| CEPH Node-2 | one-ceph2.nuansa.com | Proxmox 5.4 | 192.168.0.44 | 192.168.100.44 | 192.168.200.44 |
| CEPH Node-3 | one-ceph3.nuansa.com | Proxmox 5.4 | 192.168.0.45 | 192.168.100.45 | 192.168.200.45 |
| SAN Storage | san.nuansa.com | Centos 7	 | 192.168.0.30	| 192.168.100.30 | - |

## II. Open Nebula Front-End Installation
Disable selinux. Open the /etc/selinux/config file and set the SELINUX to disabled:

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#       enforcing - SELinux security policy is enforced.
#       permissive - SELinux prints warnings instead of enforcing.
#       disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#       targeted - Targeted processes are protected,
#       mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

Add OpenNebula repository.

```
$ cat << EOT > /etc/yum.repos.d/opennebula.repo
[opennebula]
name=opennebula
baseurl=https://downloads.opennebula.org/repo/5.8/CentOS/7/x86_64
enabled=1
gpgkey=https://downloads.opennebula.org/repo/repo.key
gpgcheck=1
#repo_gpgcheck=1
EOT
```

Activate EPEL repository.

``` $ yum install epel-release```

Install Open Nebula.

```
 $ yum install opennebula-server opennebula-sunstone opennebula-ruby \
opennebula-gate opennebula-flow
```

Run ruby runtime installation script.

```
 $ /usr/share/one/install_gems
```

Enable MySQL/MariaDB:

```
$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor. [...]

mysql> GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin' IDENTIFIED BY '<thepassword>';
Query OK, 0 rows affected (0.00 sec)
mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

Edit Open Nebula DB configuration in /etc/one/oned.conf

```
...
# Sample configuration for MySQL
DB = [ backend = "mysql",
       server  = "localhost",
       port    = 0,
       user    = "oneadmin",
       passwd  = "<thepassword>",
       db_name = "opennebula" ]
...
```

Login as oneadmin. Change the default random generated password to your desired password. (e.g: mypassword)

```
 $ su - oneadmin
 $ echo "oneadmin:mypassword" > ~/.one/one_auth  
```

Start Open Nebula services

```
 $ systemctl start opennebula
 $ systemctl start opennebula-sunstone
```

Allow firewall for Open Nebula services.

```
 $ firewall-cmd --permanent --add-port=9869/tcp ; 
 firewall-cmd --permanent --add-port=29876/tcp ; 
 firewall-cmd --permanent --add-port=2633/tcp ; 
 firewall-cmd --permanent --add-port=2474/tcp ; 
 firewall-cmd --permanent --add-port=5030/tcp ; 
 firewall-cmd --reload
```

Check with command 'oneuser show'. If command run successfully, then try login via opennebula-sunstone(web GUI) on http://nebula-fe.nuansa.com:9869/

```
$ oneuser show
USER 0 INFORMATION                                                              
ID              : 0                   
NAME            : oneadmin            
GROUP           : oneadmin            
PASSWORD        : 7bc8559a8fe509e680562b85c337f170956fcb06
AUTH_DRIVER     : core                
ENABLED         : Yes                 

TOKENS                                                                          

USER TEMPLATE                                                                   
SUNSTONE=[ DEFAULT_VIEW="admin" ]
TOKEN_PASSWORD="63b65545410a15c14e6571e86b5bff4dc4f5d01f"

VMS USAGE & QUOTAS                                                              

VMS USAGE & QUOTAS - RUNNING                                                    

DATASTORE USAGE & QUOTAS                                                        

NETWORK USAGE & QUOTAS                                                          

IMAGE USAGE & QUOTAS                                                            
```

Before adding any KVM-Host Node or Storage Node, make sure to add the server's IP addresses on the /etc/hosts.
e.g:

```
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# management network
192.168.0.35  nebula-fe.nuansa.com nebula-fe
192.168.0.32  one-kvm1.nuansa.com one-kvm1
192.168.0.33  one-kvm2.nuansa.com one-kvm2

# storage network
192.168.100.30 san-node.nuansa.com san-node
192.168.100.35 frontend.nuansa.com front-end
192.168.100.32 node1.nuansa.com node1
192.168.100.33 node2.nuansa.com node2
192.168.100.43 ceph-node1
192.168.100.44 ceph-node2
192.168.100.45 ceph-node3
```

## III. Open Nebula KVM-Host Node Installation
Add Open Nebula repository.

```
$ cat << EOT > /etc/yum.repos.d/opennebula.repo
[opennebula]
name=opennebula
baseurl=https://downloads.opennebula.org/repo/5.8/CentOS/7/x86_64
enabled=1
gpgkey=https://downloads.opennebula.org/repo/repo.key
gpgcheck=1
#repo_gpgcheck=1
EOT
```

Install the software, and restart libvirt.

```
 $ sudo yum install opennebula-node-kvm
 $ sudo systemctl restart libvirtd
```

Disable Selinux. Open the /etc/selinux/config file and set the SELINUX to disabled:

```
 ...
 SELINUX=disabled
 ...
```

Configure passwordless SSH. Open Nebula FE needs to connect to hypervisor host using SSH.

* On KVM-Host node, set temporary password for oneadmin user.

```
[root@one-kvm1 ~]$ passwd oneadmin
```

* Login to nebula-fe node as oneadmin user and generate ssh-key and distribute it to KVM-Host node.

```
[oneadmin@nebula-fe ~]$ ssh-keygen
[oneadmin@nebula-fe ~]$ ssh-copy-id one-kvm1
```

* Test passwordless login.

```
[oneadmin@nebula-fe ~]$ ssh one-kvm1
```

Link qemu-kvm binary. (For Centos 7 KVM, qemu binary got difference on the naming)

```
 [root@one-kvm1 ~]$ ln -s /usr/libexec/qemu-kvm /usr/bin/qemu-system-x86_64
```

Network configuration.<br>
The simplest network model in opennebula is bridged model drivers. For this driver we need to setup a linux bridge to the physical driver of the hosts. Create new bridge network. (e.g: vmbr0)

```
[root@one-kvm1 ~]$ vi /etc/sysconfig/network-scripts/ifcfg-vmbr0
```

And configure as following example. (change example IP addr & DNS as per needed)

```
DEVICE="vmbr0"
BOOTPROTO="static"
IPADDR="192.168.0.32"
NETMASK="255.255.255.0"
GATEWAY="192.168.0.254"
DNS1="8.8.8.8"
DNS2="8.8.4.4"
ONBOOT="yes"
TYPE="Bridge"
NM_CONTROLLED="no"
```

Modify the network configuration of the existing interface in such a way that it points to a bridge interface. In this example, will be using ens18 interface. (Change ens18 as per needed)

```
[root@one-kvm1 ~]$ vi /etc/sysconfig/network-scripts/ifcfg-ens18
```

Configure as follow.

```
TYPE="Ethernet"
BOOTPROTO="none"
DEVICE="ens18"
ONBOOT="yes"
NM_CONTROLLED=no
BRIDGE=vmbr0
```

Adding a new KVM-Host into Open Nebula system can be achieved by two ways.

1. Open Nebula Sunstone (Web GUI):
	* Login to nebula-fe web interface.
	* Click on Infrastructure Tab --> Click Hosts --> Click the plus icon ("+") button.
	* Fill in the FQDN of the KVM host in the Hostname field. Then click "Create"
2. CLI:
	* Login to nebula-fe as oneadmin user.
	* run the followings command:
 	```[oneadmin@nebula-fe ~]$ onehost create one-kvm1 -i kvm -v kvm```
	* check host status.
```
[oneadmin@nebula-fe ~]$ onehost list
ID NAME            CLUSTER   TVM      ALLOCATED_CPU      ALLOCATED_MEM   STAT  
0 one-kvm1        default     2      0 / 800 (0%)      2G / 7.6G (26%)   on
```
    
## IV. Configure CEPH Datastore Back-End
!! On this section we assume you already have a functional PVE Ceph cluster in place. !!<br>
!! This post only cover for opennebula configuration. ~~CEPH cluster installation will be available on future post~~ !!

### CEPH Cluster Setup
Login to one of the ceph node and create a pool for opennebula datastore. In this example: pool name is 'one' with pg number=128.

```
 $ ceph osd pool create one 128
 $ ceph osd lspools
  0 data,1 metadata,2 rbd,6 one,
```

Define a Ceph user to access the datastore pool. For this example, we are using 'libvirt' as a user ID.

``` $ ceph auth get-or-create client.libvirt mon 'profile rbd' osd 'profile rbd pool=one'```

Get a copy of the key of this user to distribute it later to the OpenNebula nodes.

```
 $ ceph auth get-key client.libvirt | tee client.libvirt.key
 $ ceph auth get client.libvirt -o ceph.client.libvirt.keyring
```

Use rbd format=2. Check in /etc/ceph/ceph.conf and make sure it includes. If not, add on the [global] section.

```
 [global]
 auth client required = cephx
 auth cluster required = cephx
 auth service required = cephx
 cluster network = 192.168.200.0/24
 fsid = 10789b9b-af71-41aa-94a1-d842396ee962
 keyring = /etc/pve/priv/$cluster.$name.keyring
 mon allow pool delete = true
 osd journal size = 5120
 osd pool default min size = 2
 osd pool default size = 3
 public network = 192.168.200.0/24
 rbd_default_format = 2
 ...
```

### Front-end and Host Node Setup for CEPH Client
!! Ceph client tools is required for the node to access ceph pool.<br>
!! at least 1 node required to be configured as ceph client.<br>

Configure CEPH repository.

```
 cat << EOM > /etc/yum.repos.d/ceph.repo
 [ceph-noarch]
 name=Ceph noarch packages
 https://download.ceph.com/rpm-luminous/el7/noarch
 enabled=1
 gpgcheck=1
 type=rpm-md
 gpgkey=https://download.ceph.com/keys/release.asc
 EOM
```

Install CEPH client tools.

``` $ sudo yum install ceph-common```

Create and define monitor daemon in **/etc/ceph/ceph.conf** for all nodes (FE and hosts). (Or you can copy the ceph.conf file from pve ceph node).

``` $ sudo scp root@one-ceph1:/etc/ceph/ceph.conf /etc/ceph/ceph.conf```

Copy CEPH user keyring (ceph.client.libvirt.keyring) to all nodes(FE & host) under /etc/ceph directory, and the user key (client.libvirt.key) to oneadmin home.

```
 $ sudo scp root@one-ceph1:ceph.client.libvirt.keyring /etc/ceph/.
 $ scp root@one-ceph1:client.libvirt.key ~oneadmin/.
```

### Host Node Additional Setup
Nodes need extra steps to setup credentials in libvirt:

1. Generate a secret for the CEPH user and copy it to the nodes under oneadmin home. e.g:

```
 [oneadmin@one-kvm1 ~]$ UUID=`uuidgen` ; echo $UUID
 f3c1b0c5-96cf-4aa3-8e51-6abcbd00029b
 
 [oneadmin@one-kvm1 ~]$ cat > secret.xml << EOF
 <secret ephemeral='no' private='no'>
   <uuid>$UUID</uuid>
   <usage type='ceph'>
       <name>client.libvirt secret</name>
   </usage>
 </secret>
 EOF
```

Copy secret.xml file into the other KVM-Host node.

``` [oneadmin@one-kvm1 ~]$ scp secret.xml oneadmin@one-kvm2:.```

**NOTE**: Remeber to save the UUID value since this will be used again later on libvirt configuration.

2. Define the a libvirt secret and remove key files in the nodes. Login as oneadmin, then define the secret and set the value of user key (using client.libvirt.key file).<br>
**NOTE**:
	- Do this step on all KVM-Host node.
	- make sure to use the same **UUID** that generated on the previous step.

```
 $ virsh -c qemu:///system secret-define secret.xml
 $ virsh -c qemu:///system secret-set-value --secret $UUID –base64 $(cat client.libvirt.key)
```

3. The oneadmin account needs to access the CEPH Cluster using the libvirt CEPH user defined above. To test that CEPH client is properly configured in the host node, as oneadmin user, run this command on all KVM-Host node:
NOTE: **"one"** is the name of the pool on CEPH storage, and **"libvirt"** is the user we used to authenticate into CEPH.

``` [oneadmin@one-kvm ~]$ rbd ls -p one --id libvirt```

if no permission error shown on the shell, then your host configuration is ready to use. If still got permission error, check again the client keyring file under /etc/ceph or the secret defined on the libvirt.

4. Allow firewall for noVNC console to guest VMs.

```
 $ firewall-cmd --permanent –add-port=5900-65535/tcp
 $ firewall-cmd --reload
```

### Open Nebula Datastore Configuration
On Front-End node, create Image and System datastore that will be using CEPH back-end.

1. Login to nebula-fe as oneadmin user. Then create template file for system datastore.

``` $ vi ceph_systemds.txt```

configure the file as following example.

```
 NAME = ceph_system
 TM_MAD = ceph
 TYPE = SYSTEM_DS
 DISK_TYPE = RBD
 
 POOL_NAME = one
 CEPH_HOST = "one-ceph1 one-ceph2 one-ceph3"
 CEPH_USER = libvirt
 CEPH_SECRET = "f3c1b0c5-96cf-4aa3-8e51-6abcbd00029b"
 
 BRIDGE_LIST = “one-kvm1 one-kvm2”
```

create system datastore by using following command.

```
 $ onedatastore create ceph_systemds.txt
 ID: 100
```

2. Create template file for image datastore.

``` $ vi ceph_imageds.txt```

configure the file as following example.

```
 NAME = "cephds"
 DS_MAD = ceph
 TM_MAD = ceph
 
 DISK_TYPE = RBD
 
 POOL_NAME   = one
 CEPH_HOST   = "one-ceph1 one-ceph2 one-ceph3"
 CEPH_USER   = libvirt
 CEPH_SECRET = "f3c1b0c5-96cf-4aa3-8e51-6abcbd00029b"
 
 BRIDGE_LIST = “one-kvm1 one-kvm2"
```

create image datastore by using following command:

```
 $ onedatastore create ceph_imageds.txt
 ID: 101
```

## V. Configure LVM Datastore Back-End
To use LVM Datastore back-end for Open Nebula system, we need a SAN storage server that will shares LUN iSCSI to all KVM-Host. In this example we configure SAN server (san.nuansa.com), with the following details:

* targetcli will be used for iSCSI service.
* 1 disk (sdb) is configured to be 1 LUN target.
* iscsi target name: iqn.2019-08.local.storage.server:lun1

### Storage Node setup
Login to SAN server and install targetcli.

``` [root@san ~]$ yum install targetcli```

Create LVM from disk that we want to share (In this example **/dev/sdb**). Create new partition and set type to Linux LVM.

```
[root@san ~]$ fdisk /dev/sdb

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-41943039, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-41943039, default 41943039): 
Using default value 41943039
Partition 1 of type Linux and of size 20 GiB is set

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

Configure LVM: create volume group and logical volume. In this example we configure the LVM size to 20G. (Change this size based on your requirements).

```
 [root@san ~]$ pvcreate /dev/sdb1
 [root@san ~]$ vgcreate vg0 /dev/sdb1
 [root@san ~]$ lvcreate -L20G -n storage_lun1 vg0
```

Configure targetcli by running following command to get iSCSI CLI for an interactive prompt.

```
 [root@san ~]$ targetcli
 Warning: Could not load preferences file /root/.targetcli/prefs.bin.
 targetcli shell version 2.1.fb41
 Copyright 2011-2013 by Datera, Inc and others.
 For help on commands, type 'help'.
 />
```

Now use an existing logical volume (/dev/vg0/storage_lun1) as a block-type backing store and create storage object. In this example we create object storage named iscsi_storage_lun1.

```
 /> cd backstores/block
 /backstores/block> create iscsi_storage_lun1 /dev/vg0/storage_lun1
```

Create a target.

```
 /backstores/block> cd /iscsi
 /iscsi> create iqn.2019-08.local.storage.server:lun1
```

Create ACL for client initiator machine (each of the KVM Host). This the IQN which clients use to connect.

```
 /iscsi> cd /iscsi/iqn.2019-08.local.storage.server:lun1/tpg1/acls
 /iscsi/iqn.20..un1/tpg1/acls> create iqn.2019-08.local.nebula.server:node1
 Created Node ACL for iqn.2019-08.local.nebula.server:node1
 /iscsi/iqn.20..un1/tpg1/acls> create iqn.2019-08.local.nebula.server:node2
 Created Node ACL for iqn.2019-08.local.nebula.server:node2
```

Create LUN under target. The LUN should use the previously mentioned backing storage object named "iscsi_storage_lun1"

```
 /iscsi/iqn.20..un1/tpg1/acls> cd /iscsi/iqn.2019-08.local.storage.server:lun1/tpg1/luns
 /iscsi/iqn.20..un1/tpg1/luns> create /backstores/block/iscsi_storage_lun1
 Created LUN 0.
 Created LUN 0->0 mapping in node ACL for iqn.2019-08.local.nebula.server:node1
 Created LUN 0->0 mapping in node ACL for iqn.2019-08.local.nebula.server:node2
```

Verify and save configuration.

```
 /iscsi/iqn.20..un1/tpg1/luns> cd /
 /> ls
   o- / ............................................................................................. [...]
     o- backstores .................................................................................. [...]
     | o- block ...................................................................... [Storage Objects: 1]
     | | o- iscsi_storage_lun1 ..................... [/dev/vg0/storage_lun1 (20.0GiB) write-thru activated]
     | |   o- alua ....................................................................... [ALUA Groups: 1]
     | |     o- default_tg_pt_gp ........................................... [ALUA state: Active/optimized]
     | o- fileio ..................................................................... [Storage Objects: 0]
     | o- pscsi ...................................................................... [Storage Objects: 0]
     | o- ramdisk .................................................................... [Storage Objects: 0]
     o- iscsi ................................................................................ [Targets: 1]
     | o- iqn.2019-08.local.storage.server:lun1 ................................................. [TPGs: 1]
     |   o- tpg1 ................................................................... [no-gen-acls, no-auth]
     |     o- acls .............................................................................. [ACLs: 2]
     |     | o- iqn.2019-08.local.nebula.server:node1 .................................... [Mapped LUNs: 1]
     |     | | o- mapped_lun0 ........................................ [lun0 block/iscsi_storage_lun1 (rw)]
     |     | o- iqn.2019-08.local.nebula.server:node2 .................................... [Mapped LUNs: 1]
     |     |   o- mapped_lun0 ........................................ [lun0 block/iscsi_storage_lun1 (rw)]
     |     o- luns .............................................................................. [LUNs: 1]
     |     | o- lun0 ................ [block/iscsi_storage_lun1 (/dev/vg0/storage_lun1) (default_tg_pt_gp)]
     |     o- portals ........................................................................ [Portals: 1]
     |       o- 0.0.0.0:3260 ......................................................................... [OK]
   o- loopback ............................................................................. [Targets: 0]
 /> saveconfig
 Last 10 configs saved in /etc/target/backup/.
 Configuration saved to /etc/target/saveconfig.json
 /> exit
 Global pref auto_save_on_exit=true
 Last 10 configs saved in /etc/target/backup/.
 Configuration saved to /etc/target/saveconfig.json
```

Enable and restart target service

```
 [root@san ~]$ systemctl enable target
 [root@san ~]$ systemctl restart target
```

Allow firewall.

```
 [root@san ~]$ firewall-cmd --persistent --add-port=3260/tcp
 [root@san ~]$ firewall-cmd --reload
```

### KVM Host Setup
Do this step on all KVM-Host node.<br>
Login to KVM host node and install iscsi initiator.

``` $ yum install iscsi-initiator-utils```

Edit the iscsi initiator name in /etc/iscsi/initiatorname.iscsi.

``` $ vi /etc/iscsi/Initiatorname.iscsi```

Change the Initiatorname to match the initiator name that we allow and configured on SAN server.<br>
For KVM host1:

``` InitiatorName=iqn.2019-08.local.nebula.server:node1```

For KVM host2:

``` InitiatorName=iqn.2019-08.local.nebula.server:node2```

Disable lvmetad.<br>
Edit file /etc/lvm/lvm.conf and set parameter use_lvmetad to **0**.

```
 ...
 use_lvmetad = 0
 ...
```

Stop and disable the lvmetad services.

```
 $ systemctl stop lvm2-lvmetad.service
 $ systemctl stop lvm2-lvmetad.socket
 $ systemctl disable lvm2-lvmetad.service
 $ systemctl disable lvm2-lvmetad.socket
```

Add oneadmin user to group disk.

``` $ usermod -aG disk oneadmin```

Next is to configure shared LUN device. **Only run the following steps on 1 KVM-Host only**. **In this example, the steps are executed in one-kvm1**.<br>
Connect and login to iSCSI LUN.

```
 [root@one-kvm1 ~]$ iscsiadm -m discovery -t st -p san.nuansa.com
 san.nuansa.com:3260,1  iqn.2019-08.local.storage.server:lun1
 [root@one-kvm1 ~]$ iscsiadm -m node -T "iqn.2019-08.local.storage.server:lun1" --login
```

Check iSCSI disk, and make sure it detected as a new disk. (In this example it's /dev/sdb)

``` [root@one-kvm1 ~]$ fdisk -l | grep sd```

Prepare and configure the disk to LVM partition.

```
 [root@one-kvm1 ~]$ parted /dev/sdb mklabel gpt
 [root@one-kvm1 ~]$ parted /dev/sdb mkpart primary 1 100%
 [root@one-kvm1 ~]$ parted /dev/sdb set 1 lvm on
```

Create physical volume.

```[root@one-kvm1 ~]$ pvcreate /dev/sdb1```

Create volume group with a name format like this: **"vg-one-<datastore_id>"**<br>
In this example, the LVM Datastore will be using ID 103

``` [root@one-kvm1 ~]$ vgcreate vg-one-103 /dev/sdb```

After done setting up VG for a shared block device, connect and login all the other KVM Host to iSCSI LUN.

```
 [root@one-kvm2 ~]$ iscsiadm -m discovery -t st -p san.nuansa.com
 san.nuansa.com:3260,1  iqn.2019-08.local.storage.server:lun1
 [root@one-kvm2 ~]$ iscsiadm -m node -T "iqn.2019-08.local.storage.server:lun1" --login
```

Check that VG vg-one-103 is shared and detected on all KVM host.

```
 [root@one-kvm2 ~]$ vgs
   VG         #PV #LV #SN Attr   VSize  VFree
   VolGroup00   1   0   0 wz--n- 20,00g  20,00
```

Login as oneadmin and create directory for LVM system datastore under **/var/lib/one/datastore/<datastore_id>**.<br>
Do this step in all the KVM host.

``` $ mkdir /var/lib/one/datastore/103```

Take note: if want to enable VM live migration between host node, this directory need to be **shared across KVM host**.
For example, we can use NFS and mount the shared filesystem into the /var/lib/one/datastore/103

### Open Nebula Datastore Configuration
Login to Open Nebula front-end as oneadmin, and create LVM system and image datastore.<br>
Create the image datastore template file.

``` [oneadmin@nebula-fe ~]$ vi lvm_imgds.txt```

Create as following example.

```
 NAME = image_ds-lvm
 DS_MAD = fs
 TM_MAD = fs_lvm
 DISK_TYPE = "BLOCK"
 TYPE = IMAGE_DS
```

Create LVM image datastore.

```
 [oneadmin@nebula-fe ~]$ onedatastore create lvm_imgds.txt
 ID: 102
```

Create the system datastore template file.

``` [oneadmin@nebula-fe ~]$ vi lvm_systemds.txt```

Create as following example.

```
 NAME   = lvm_system
 TM_MAD = fs_lvm
 TYPE   = SYSTEM_DS
```

Create LVM system datastore.

```
 [oneadmin@nebula-fe ~]$ onedatastore create lvm_systemds.txt
 ID: 103
```

Edit LVM system datastore.

``` [oneadmin@nebula-fe ~]$ onedatastore update 103```

and add parameter "BRIDGE_LIST" with values containing all KVM host nodes.

```
 ALLOW_ORPHANS="NO"
 DISK_TYPE="FILE"
 DS_MIGRATE="YES"
 RESTRICTED_DIRS="/"
 SAFE_DIRS="/var/tmp"
 SHARED="YES"
 TM_MAD="fs_lvm"
 TYPE="SYSTEM_DS"
 BRIDGE_LIST="one-kvm1 one-kvm2"         ######### Add this line
```

Check the datastore status.

```
 [oneadmin@nebula-fe ~]$ onedatastore list
 ID 	NAME	SIZE 	AVAIL 	CLUSTERS 	IMAGES 	TYPE 	DS 	TM	STAT
 103 	lvm_system	20G 	72%   	 0	0 	sys  	- 	fs_lvm	on  
 102 	image_ds-lvm	8G	6%	 0	4	img	fs	fs_lvm	on
```


## VII. Troubleshoot
Troubleshooting steps for common problems that are found on our example so far: [OpenNebula Troubleshoot](https://rdnuansa.github.io/opennebula_troubleshoot.md)

## VIII. Reference
[1] [OpenNebula](http://docs.opennebula.org/5.8/deployment/cloud_design/open_cloud_architecture.html#open-cloud-architecture) <br>
[2] [OpenNebula CEPH Datastore](http://docs.opennebula.org/5.8/deployment/open_cloud_storage_setup/ceph_ds.html#ceph-datastore)<br>
[3] [OpenNebula LVM Datastore](http://docs.opennebula.org/5.8/deployment/open_cloud_storage_setup/lvm_drivers.html?#lvm-datastore)

[1]: http://docs.opennebula.org/5.8/deployment/cloud_design/open_cloud_architecture.html#open-cloud-architecture
[2]: http://docs.opennebula.org/5.8/deployment/open_cloud_storage_setup/ceph_ds.html#ceph-datastore
[3]: http://docs.opennebula.org/5.8/deployment/open_cloud_storage_setup/lvm_drivers.html?#lvm-datastore
