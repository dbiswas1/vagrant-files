# Setup Production like Kubernetes Cluster in Mac and Windows

In this blog we will setup a microK8 so that developers can experience the production like K8 cluster from there laptop.

## what is microk8 ?
The smallest, fastest, fully-conformant Kubernetes that tracks upstream releases and makes clustering trivial. MicroK8s 
is great for offline development, prototyping, and testing. Use it on a VM as a small, cheap, reliable k8s for CI/CD. 
The best Kubernetes for appliances. Develop IoT apps for k8s and deploy them to MicroK8s on your Linux boxes. More on [MicroK8](https://microk8s.io/docs/)

## Tools used
- we are using (vagrant)[https://www.vagrantup.com/] to spin up the microK8 in the laptop both Mac and Windows
- (Virtualbox)[https://www.virtualbox.org/] to spinup the vm to host microK8
**NOTE: Please make sure you install vagrant and virtualbox following the official document**

## Setup MicroK8 - macOS

Create the following vagrant file

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.network "forwarded_port", guest: 8443, host: 9090, guest_ip: "10.1.1.5"
  config.vm.network "public_network"
  config.vm.provision :docker
  config.vm.provision "shell", inline: <<-SHELL
    usermod -aG docker vagrant
    snap install microk8s --classic
    sudo usermod -a -G microk8s vagrant
    microk8s.start
    microk8s.enable dns dashboard
  SHELL

end
```

then type following command, when you type vagrant up it will ask you to select the network, you can predefine the same
in vagrant file in this blog i have not defined and hence selecting it run time

```
vagrant init
vagrant up 

#### Curtialed output ########
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/bionic64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/bionic64' version '20190708.0.0' is up to date...
==> default: Setting the name of the VM: microk8_default_1582012340271_14492
==> default: Clearing any previously set network interfaces...
==> default: Available bridged network interfaces:
1) en7: *********
2) en5: USB Ethernet(?)
3) en0: Wi-Fi (Wireless)
4) ****
5) ***
6) en1: Thunderbolt 1
7) en2: Thunderbolt 2
8) en3: Thunderbolt 3
9) en4: Thunderbolt 4
10) bridge0
==> default: When choosing an interface, it is usually the one that is
==> default: being used to connect to the internet.
    default: Which interface should the network bridge to? 1)
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: bridged
==> default: Forwarding ports...
    default: 8443 (guest) => 9090 (host) (adapter 1)
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection reset. Retrying...
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!

```
### verify the macOS installation
log in to vagrant host you just created, and then query the K8 cluster to see output see below
```
vagrant ssh
vagrant@ubuntu-bionic: microk8s.kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   ********   <none>        443/TCP   3m18s

microk8s.kubectl get services --all-namespaces
NAMESPACE     NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes                  ClusterIP   *************    <none>        443/TCP                  4m11s
kube-system   dashboard-metrics-scraper   ClusterIP   *************    <none>        8000/TCP                 3m54s
kube-system   heapster                    ClusterIP   *************    <none>        80/TCP                   3m53s
kube-system   kube-dns                    ClusterIP   *************    <none>        53/UDP,53/TCP,9153/TCP   4m5s
kube-system   kubernetes-dashboard        ClusterIP   *************    <none>        443/TCP                  3m55s
kube-system   monitoring-grafana          ClusterIP   *************    <none>        80/TCP                   3m53s
kube-system   monitoring-influxdb         ClusterIP   *************    <none>        8083/TCP,8086/TCP        3m53s

```

## Setup MicroK8 - Windows

### verify the Windows installation