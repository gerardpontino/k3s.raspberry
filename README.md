# Build a Low-Cost Kubernetes Cluster on Raspberry Pi 4 with K3s: A Step-by-Step Guide

By [Gerard Pontino](https://medium.com/@gerard.pontino)

![01](https://github.com/user-attachments/assets/b3a63f73-0203-4aa2-a9f4-7c0c5e777564)


## Introduction

Are you curious about Kubernetes and want to experiment with it? Do you have a Raspberry Pi 4's lying around and want to put it to good use? If so, you’re in luck! In this blog, I’ll show you how to build a simple and lightweight Kubernetes cluster using Raspberry Pi 4 with k3s. It’s an affordable and accessible way to explore the world of Kubernetes, without breaking the bank. Let’s get started!

## Topology of our setup

![02](https://github.com/user-attachments/assets/962c242e-df43-443b-8554-fc6ae23014ef)

## Hardware

- 3 x Raspberry Pi 4 Model B 4GB
- 2x Kingston CANVAS Select Plus SD Card 32GB
- 52pi Rack Tower Cluster Case
- 1x Kingston A400 SSD (240GB)
- TP-Link TLSG108 8-Port gigabit Desktop Switch (You may also use 4-Port Switch)

## Step-by-Step Guide

### Installing the OS

I used the Raspberry Pi Imager to install the Raspberry Pi OS (32-bit) onto an SD card. During the setup process (as outlined in Step 4 below), you’ll be prompted to configure settings such as the hostname, username, and more.”

![03](https://github.com/user-attachments/assets/9cba6084-33a9-47bd-b041-a7ea3c4c62bf)

### Booting Raspberry Pi 4 from a USB SSD

I have to admit, one step in the Raspberry Pi Kubernetes cluster setup really had me scratching my head. I was determined to get my master node up and running with an SSD, but I couldn’t seem to get it to work. After searching for hours online, I stumbled upon a [blog post](https://www.tomshardware.com/how-to/boot-raspberry-pi-4-usb) that finally shed some light on the issue. Thanks to this newfound knowledge, I was able to successfully boot my master node with an SSD.

### Networking

In order to connect your Raspberry Pi devices to the internet, you need to connect them to your switch via an Ethernet cable. Once connected, you can easily determine the IP addresses assigned to your Raspberry Pis from your router’s configuration page.

If you have set the hostname of your Raspberry Pis using the Raspberry Pi Imager, identifying their IP addresses becomes even easier. In my case, I am using the Nokia Wifi Beacon 1 router, and here are the steps I followed to verify the IP addresses of my Raspberry Pis.

When checking your node’s IP addresses, you may notice duplicate details with different IP addresses, one for the wireless connection and another for Ethernet. It’s important to identify the Ethernet IP address because it provides a more stable and reliable connection compared to wireless. To find the Ethernet IP address, check your router’s configuration page and use the one that states “Ethernet”.

![04](https://github.com/user-attachments/assets/29780a7d-3d94-4f53-ae16-5d455d92a193)

### Preparing the nodes

After identifying the IP addresses of your node, you can connect to them through an ssh terminal (I’m using the mac’s terminal) with IP.

These are the basic and important things to setup:

- Add hostnames of the nodes in /etc/hosts
- Check the necessary ports as a [requirement for k3s](https://docs.k3s.io/installation/requirements)

##### a. Add the hostnames of the nodes in /etc/hosts

To access your nodes using hostnames from your laptop’s terminal and from each node, you need to configure /etc/hosts individually. The steps are the same for all three nodes, so you will need to repeat the process for each one. Once configured, you can easily access each node using its hostname, simplifying the management process and making it easier to connect to your nodes.

```bash  
vi /etc/hosts  
## Raspberry Cluster Nodes  
192.168.18.35 k3s-master  
192.168.18.52 worker01  
192.168.18.53 worker02  
``` 

Once done, you should be able to connect to your nodes using their hostnames.

```bash
GLP/~ $ ssh iamroot@k3s-master
iamroot@k3s-master's password: 
Linux k3s-master 5.15.84-v7l+ #1613 SMP Thu Jan 5 12:01:26 GMT 2023 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Apr 23 06:44:04 2023 from 192.168.18.8
iamroot@k3s-master:~ $

```

##### b. Check the necessary ports as a requirement for k3s

Verify whether port 10250 is allowed. If not, proceed to next step.


```bash
iamroot@k3s-master:~ $ netstat -anlp | grep 10250
```

To allow port 10250 from the master node, you need to configure your system as a root user. The specific commands required to do this will vary depending on your operating system. DigitalOcean has a helpful guide that provides [detailed instructions](https://www.digitalocean.com/community/tutorials/opening-a-port-on-linux) for opening a port on Linux, which you can refer to for more information. By allowing port 10250, you will be able to establish a secure connection with your master node for the metrics server while installing K3s.

```bash
iamroot@k3s-master:~ $ sudo su -
root@k3s-master:~# iptables -A INPUT -p tcp --dport 10250 -j ACCEPT
```

To verify, run:
```bash
iptables --list | grep 10250
```

### K3s Installation

For further details about the architecture of K3s, refer to https://docs.k3s.io/architecture

#### Installation of k3s on the Master Node

First, we need to install K3s on our Master Node. Sometimes during installation, you might see an error message that says:

```bash
[INFO] Failed to find memory cgroup, you may need to add “cgroup_memory=1 cgroup_enable=memory” to your linux cmdline (/boot/cmdline.txt on a Raspberry Pi)
```

To fix this error, make sure to edit `bash /boot/cmdline.txt` with `cgroup_memory=1 cgroup_enable=memory` at the end of the line.

```bash
vi /boot/cmdline.txt
```

```bash
console=serial0,115200 console=tty1 root=PARTUUID=71c3dce0–02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles cgroup_memory=1 cgroup_enable=memory
```

Once you’ve fixed the error, you can continue with the installation process and create your Kubernetes cluster.

```bash
curl -sfL https://get.k3s.io | sh -
```

Below is the sample output of the command

```bash
iamroot@k3s-master:~ $ curl -sfL https://get.k3s.io | sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.26.3+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.26.3+k3s1/sha256sum-arm.txt
[INFO]  Skipping binary downloaded, installed k3s matches hash
[INFO]  Finding available k3s-selinux versions
sh: 407: [: unexpected operator
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, already exists
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  No change detected so skipping service start
```

To confirm that your Kubernetes installation is working properly and that the master node is available, you can perform a simple verification test by running sudo kubectl get nodes.

```bash
iamroot@k3s-master:~ $ sudo kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
k3s-master   Ready    control-plane,master   7d5h   v1.26.3+k3s1
```

### Installation of k3s agents on the Worker Nodes

Just like the master nodes, you also need to ensure that you edit the `/boot/cmdline.txt` file with `cgroup_memory=1 cgroup_enable=memory` added to the end of the line for each worker node.

Before installing the k3s agents on the other Raspberry Pis, you need to obtain the IP address and access token from the master node. To do this, simply SSH into the master node and run the following commands:

```bash
iamroot@k3s-master:~ $ hostname -I | awk '{print$1}'
192.168.18.35

## Use this IP on this command: "curl -sfL https://get.k3s.io | K3S_URL=https://<kmaster_IP_from_above>:6443"

```
```bash
iamroot@k3s-master:~ $ sudo cat /var/lib/rancher/k3s/server/node-token
K10910a9f606e89da8a95e3e37ab9faf160a3eeca46229dd82fb902c3984bec8e1b::server:e658625eecb60de3f383ca0a75df3e24
```

#### SSH into every worker node and execute/run the required command(s).

Using the IP and Node Token taken from the Master node. Run `curl -sfl https://get.k3s.io |K3S_URL=https://<Master IP>:6443 K3S_TOKEN=<Node Token> sh -`

```bash
## Use the IP's and Node Token on the command below

iamroot@worker01:~ $ curl -sfL https://get.k3s.io | K3S_URL=https://192.168.18.35:6443 K3S_TOKEN=K10910a9f606e89da8a95e3e37ab9faf160a3eeca46229dd82fb902c3984bec8e1b::server:e658625eecb60de3f383ca0a75df3e24 sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.26.3+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.26.3+k3s1/sha256sum-arm.txt
[INFO]  Skipping binary downloaded, installed k3s matches hash
[INFO]  Finding available k3s-selinux versions
sh: 407: [: unexpected operator
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, already exists
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent
```

Once you have installed the k3s agents on your worker nodes, you can verify that they have been successfully added to the cluster and are in “ready” status. To do this, log in to your master node and run the following command:

```bash
iamroot@k3s-master:~ $ sudo kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
k3s-master   Ready    control-plane,master   7d5h   v1.26.3+k3s1
worker02     Ready    <none>                 7d5h   v1.26.3+k3s1
worker01     Ready    <none>                 7d5h   v1.26.3+k3s1
```

### Managing the clusters from your laptop

#### a. Installing kubectl

Depending on your laptop’s OS, you may check the [official documention](https://kubernetes.io/docs/tasks/tools/) on how to install kubectl

#### b. Setting up kubeconfig on your laptop

To retrieve the kubeconfig file from the master node, execute the following command on the master node:

```bash
iamroot@k3s-master:~ $ sudo cat /etc/rancher/k3s/k3s.yaml 

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1..
    server: https://<Master Nodes-IP>:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: k3s
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: LS0tLS1CRUd..
    client-key-data: LS0tLS1C...
```
To use the kubeconfig file on your local machine, copy the contents of the file to a new file on your local machine, and save it as `~/.kube/config`.

Before using the configuration file, you need to update the server field in the cluster section to match the IP address of your master node. Replace the default server IP of `127.0.0.1` with the IP address of your master node. Additionally, rename the context from default to k3s.

Once you’ve updated the configuration file, you can test the connection by executing `kubectl` commands in your local terminal, using the `k3s` context you just created. For example, you can run:

```bash
GLP/~ $ kubectl config use-context k3s
Switched to context "k3s".
GLP/~ $ kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
k3s-master   Ready    control-plane,master   7d6h   v1.26.3+k3s1
worker02     Ready    <none>                 7d6h   v1.26.3+k3s1
worker01     Ready    <none>                 7d6h   v1.26.3+k3s1
```

## Wrapping things up

We successfully established a 3-node cluster — comprising of one master node and two worker nodes — by installing k3s and configuring the kubeconfig from our laptop to gain access to the cluster.

Setting up a high-availability Kubernetes cluster with three nodes is now remarkably easy and can be completed in a short span of time. With such ease of deployment, it won’t be long before you can also create a Kubernetes cluster on your Raspberry Pi within a short time frame.

This blog covers the initial setup phase of creating a cluster, but stay tuned for future projects that will demonstrate more advanced use cases for this cluster. I have several exciting projects in mind that will leverage the power and flexibility of Kubernetes.

...

>If you’re new to Kubernetes and want to explore its concepts in greater depth or are considering getting certified, I highly recommend checking out [KodeKloud’s courses](https://learn.kodekloud.com/user/courses/kubernetes-for-the-absolute-beginners-hands-on-tutorial). Their comprehensive courses can provide you with the knowledge you need to become an expert in Kubernetes.

>In addition, you might also want to check out (Killerkoda’s test simulators)[https://killercoda.com/]. These simulators are not only informative but also engaging and fun to use. They can help you gain confidence in your Kubernetes skills and prepare you for any certification exams you may take.

...


