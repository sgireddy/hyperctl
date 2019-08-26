# Kubernetes and Docker on Mac and Windows

## Supported scenarios
- Actual, multi-node Kuberenetes on CentOS/Ubuntu in Hyper-V/Hyperkit
- Docker on Desktop without Docker for Desktop
- Lightweight Kubernetes in Docker with Talos

## Limitations
- TODO

## Advantages
- TODO

## Changelog
- Current state: pre-release; TODO: k8s helm setup

# Mac / Hyperkit
```bash

# note: `sudo` is necessary for access to macOS Hypervisor and vmnet frameworks, and /etc/hosts config

# download the script
cd workdir
curl https://raw.githubusercontent.com/youurayy/k8s-hyperkit/master/hyperkit.sh -O
chmod +x hyperkit.sh

# display short synopsis for the available commands
./hyperkit.sh help
'
  Usage: ./hyperkit.sh command+

  Commands:

     install - install basic homebrew packages
      config - show script config vars
       print - print contents of relevant config files
         net - create or reset the vmnet config
        dhcp - append to the dhcp registry
       hosts - append node names to etc/hosts
       image - download the VM image
      master - create and launch master node
       nodeN - create and launch worker node (node1, node2, ...)
        info - display info about nodes
        init - initialize k8s and setup host kubectl
      reboot - soft-reboot the nodes
    shutdown - soft-shutdown the nodes
        stop - stop the VMs
       start - start the VMs
        kill - force-stop the VMs
      delete - delete the VM files
         iso - write cloud config data into a local yaml
    timesync - setup sleepwatcher time sync
      docker - setup local docker with the master node
       share - setup local fs sharing with docker on master
'

# performs `brew install hyperkit qemu kubernetes-cli kubernetes-helm`.
# (qemu is necessary for `qemu-img`)
# you may perform these manually / selectively instead.
./hyperkit.sh install

# display configured variables (edit the script to change them)
./hyperkit.sh config
'
    CONFIG: bionic
    DISTRO: ubuntu
    WORKDIR: ./tmp
 GUESTUSER: name
   SSHPATH: /Users/name/.ssh/id_rsa.pub
  IMAGEURL: https://cloud-images.ubuntu.com/releases/server/19.04/release/ubuntu-19.04-server-cloudimg-amd64.vmdk
  DISKFILE: ubuntu-19.04-server-cloudimg-amd64.raw
      CIDR: 10.10.0.0/24
      CPUS: 4
       RAM: 4GB
       HDD: 40GB
       CNI: flannel
    CNINET: 10.244.0.0/16
   CNIYAML: https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
'

# print external configs that this script can change
./hyperkit.sh print

# cleans or creates /Library/Preferences/SystemConfiguration/com.apple.vmnet.plist
# and sets the CIDR configured in the script.
# if other apps already use the vmnet framework, then you don't want to change it, in
# which case don't run this command, but instead set the CIDR inside this script
# to the value from the vmnet.plist (as shown by the 'print' command).
./hyperkit.sh net

# appends IPs and MACs from the NODES config to the /var/db/dhcpd_leases.
# this is necessary to have predictable IPs.
# (MACs are generated from UUIDs by the vmnet framework.)
./hyperkit.sh dhcp

# appends IP/hostname pairs from the NODES config to the /etc/hosts.
# (the same hosts entries will also be installed into every node)
./hyperkit.sh hosts

# download, prepare and cache the VM image templates
./hyperkit.sh image

# create/launch the nodes
./hyperkit.sh master
./hyperkit.sh node1
./hyperkit.sh nodeN...
# ---- or -----
./hyperkit.sh master node1 node2 nodeN...

# ssh to the nodes if necessary (e.g. for manual k8s init)
# by default, your `.ssh/id_rsa.pub` key was copied into the VMs' ~/.ssh/authorized_keys
# uses your host username (which is the default), e.g.:
ssh master
ssh node1
ssh node2
...

# performs automated k8s init (will wait for VMs to finish init first)
./hyperkit.sh init

# after init, you can do e.g.:
hyperctl get pods --all-namespaces
'
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-5c98db65d4-b92p9         1/1     Running   1          5m31s
kube-system   coredns-5c98db65d4-dvxvr         1/1     Running   1          5m31s
kube-system   etcd-master                      1/1     Running   1          4m36s
kube-system   kube-apiserver-master            1/1     Running   1          4m47s
kube-system   kube-controller-manager-master   1/1     Running   1          4m46s
kube-system   kube-flannel-ds-amd64-6kj9p      1/1     Running   1          5m32s
kube-system   kube-flannel-ds-amd64-r87qw      1/1     Running   1          5m7s
kube-system   kube-flannel-ds-amd64-wdmxs      1/1     Running   1          4m43s
kube-system   kube-proxy-2p2db                 1/1     Running   1          5m32s
kube-system   kube-proxy-fg8k2                 1/1     Running   1          5m7s
kube-system   kube-proxy-rtjqv                 1/1     Running   1          4m43s
kube-system   kube-scheduler-master            1/1     Running   1          4m38s
'

# reboot the nodes
./hyperkit.sh reboot

# show info about existing VMs (size, run state)
./hyperkit.sh info
'
NAME    PID    %CPU  %MEM  RSS   STARTED  TIME     DISK  SPARSE  STATUS
master  36399  0.4   2.1   341M  3:51AM   0:26.30  40G   3.1G    RUNNING
node1   36418  0.3   2.1   341M  3:51AM   0:25.59  40G   3.1G    RUNNING
node2   37799  0.4   2.0   333M  3:56AM   0:16.78  40G   3.1G    RUNNING
'

# shutdown all nodes thru ssh
.\hyperv.ps1 shutdown

# start all nodes
.\hyperv.ps1 start

# stop all nodes
./hyperkit.sh stop

# force-stop all nodes
./hyperkit.sh kill

# delete all nodes' data (will not delete image templates)
./hyperkit.sh delete

# kill only a particular node
sudo kill -TERM 36418

# delete only a particular node
rm -rf ./tmp/node1/

# remove everything
sudo killall -9 hyperkit
rm -rf ./tmp

```

# Windows / Hyper-V
```powershell

# note: admin access is necessary for access to Windows Hyper-V framework, and etc/hosts config

# open PowerShell (Admin) prompt
cd $HOME\your-workdir

# download the script
curl https://raw.githubusercontent.com/youurayy/k8s-hyperv/master/hyperv.ps1 -outfile hyperv.ps1
# enable script run permission
set-executionpolicy remotesigned

# display short synopsis for the available commands
.\hyperv.ps1 help
'
  Usage: .\hyperv.ps1 command+

  Commands:

     install - install basic chocolatey packages
      config - show script config vars
       print - print etc/hosts, network interfaces and mac addresses
         net - install private or public host network
       hosts - append private network node names to etc/hosts
       image - download the VM image
      master - create and launch master node
       nodeN - create and launch worker node (node1, node2, ...)
        info - display info about nodes
        init - initialize k8s and setup host kubectl
      reboot - soft-reboot the nodes
    shutdown - soft-shutdown the nodes
        save - snapshot the VMs
     restore - restore VMs from latest snapshots
        stop - stop the VMs
       start - start the VMs
      delete - stop VMs and delete the VM files
      delnet - delete the network
'

# performs `choco install 7zip.commandline qemu-img kubernetes-cli kubernetes-helm`.
# you may instead perform these manually / selectively instead.
# note: 7zip is needed to extract .xz archives
# note: qemu-img is needed convert images to vhdx
.\hyperv.ps1 install

# display configured variables (edit the script to change them)
.\hyperv.ps1 config
'
    config: bionic
    distro: ubuntu
   workdir: .\tmp
 guestuser: name
   sshpath: C:\Users\name\.ssh\id_rsa.pub
  imageurl: https://cloud-images.ubuntu.com/releases/server/18.04/release/ubuntu-18.04-server-cloudimg-amd64.img
  vhdxtmpl: .\tmp\ubuntu-18.04-server-cloudimg-amd64.vhdx
      cidr: 10.10.0.0/24
    switch: switch
   nettype: private
    natnet: natnet
      cpus: 4
       ram: 4GB
       hdd: 40GB
       cni: flannel
    cninet: 10.244.0.0/16
   cniyaml: https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
'

# print relevant configuration - etc/hosts, mac addresses, network interfaces
.\hyperv.ps1 print

# create a private network for the VMs, as set by the `cidr` variable
.\hyperv.ps1 net

# appends IP/hostname pairs to the /etc/hosts.
# (the same hosts entries will also be installed into every node)
.\hyperv.ps1 hosts

# download, prepare and cache the VM image templates
.\hyperv.ps1 image

# create/launch the nodes
.\hyperv.ps1 master
.\hyperv.ps1 node1
.\hyperv.ps1 nodeN...
# ---- or -----
.\hyperv.ps1 master node1 node2 nodeN...

# ssh to the nodes if necessary (e.g. for manual k8s init)
# by default, your `.ssh/id_rsa.pub` key was copied into the VMs' ~/.ssh/authorized_keys
# uses your host username (which is the default), e.g.:
ssh master
ssh node1
ssh node2
...

# perform automated k8s init (will wait for VMs to finish init first)
# (this will checkpoint the nodes just before `kubeadm init`)
.\hyperv.ps1 init

# after init, you can do e.g.:
hyperctl get pods --all-namespaces
'
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-5c98db65d4-b92p9         1/1     Running   1          5m31s
kube-system   coredns-5c98db65d4-dvxvr         1/1     Running   1          5m31s
kube-system   etcd-master                      1/1     Running   1          4m36s
kube-system   kube-apiserver-master            1/1     Running   1          4m47s
kube-system   kube-controller-manager-master   1/1     Running   1          4m46s
kube-system   kube-flannel-ds-amd64-6kj9p      1/1     Running   1          5m32s
kube-system   kube-flannel-ds-amd64-r87qw      1/1     Running   1          5m7s
kube-system   kube-flannel-ds-amd64-wdmxs      1/1     Running   1          4m43s
kube-system   kube-proxy-2p2db                 1/1     Running   1          5m32s
kube-system   kube-proxy-fg8k2                 1/1     Running   1          5m7s
kube-system   kube-proxy-rtjqv                 1/1     Running   1          4m43s
kube-system   kube-scheduler-master            1/1     Running   1          4m38s
'

# reboot the nodes
.\hyperv.ps1 reboot

# show info about existing VMs (size, run state)
.\hyperv.ps1 info
'
Name   State   CPUUsage(%) MemoryAssigned(M) Uptime           Status             Version
----   -----   ----------- ----------------- ------           ------             -------
master Running 3           5908              00:02:25.5770000 Operating normally 9.0
node1  Running 8           4096              00:02:22.7680000 Operating normally 9.0
node2  Running 2           4096              00:02:20.1000000 Operating normally 9.0
'

# checkpoint the VMs
.\hyperv.ps1 save

# restore the VMs from the lastest snapshot
.\hyperv.ps1 restore

# shutdown all nodes thru ssh
.\hyperv.ps1 shutdown

# start all nodes
.\hyperv.ps1 start

# stop all nodes thru hyper-v
.\hyperv.ps1 stop

# delete all nodes' data (will not delete image templates)
.\hyperv.ps1 delete

# delete the network
.\hyperv.ps1 delnet

# NOTE if Hyper-V stops working after a Windows update, do:
# Windows Security -> App & Browser control -> Exploit protection settings -> Program settings ->
# C:\WINDOWS\System32\vmcompute.exe -> Edit-> Code flow guard (CFG) -> uncheck Override system settings -> # net stop vmcompute -> net start vmcompute

```

#### License: https://www.apache.org/licenses/LICENSE-2.0