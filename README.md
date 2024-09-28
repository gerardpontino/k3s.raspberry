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

I have to admit, one step in the Raspberry Pi Kubernetes cluster setup really had me scratching my head. I was determined to get my master node up and running with an SSD, but I couldn’t seem to get it to work. After searching for hours online, I stumbled upon a blog post (https://www.tomshardware.com/how-to/boot-raspberry-pi-4-usb) that finally shed some light on the issue. Thanks to this newfound knowledge, I was able to successfully boot my master node with an SSD.

### Networking

In order to connect your Raspberry Pi devices to the internet, you need to connect them to your switch via an Ethernet cable. Once connected, you can easily determine the IP addresses assigned to your Raspberry Pis from your router’s configuration page.

If you have set the hostname of your Raspberry Pis using the Raspberry Pi Imager, identifying their IP addresses becomes even easier. In my case, I am using the Nokia Wifi Beacon 1 router, and here are the steps I followed to verify the IP addresses of my Raspberry Pis.

When checking your node’s IP addresses, you may notice duplicate details with different IP addresses, one for the wireless connection and another for Ethernet. It’s important to identify the Ethernet IP address because it provides a more stable and reliable connection compared to wireless. To find the Ethernet IP address, check your router’s configuration page and use the one that states “Ethernet”.

![04](https://github.com/user-attachments/assets/29780a7d-3d94-4f53-ae16-5d455d92a193)

### Preparing the nodes

After identifying the IP addresses of your node, you can connect to them through an ssh terminal (I’m using the mac’s terminal) with IP.

These are the basic and important things to setup:

- Add hostnames of the nodes in /etc/hosts
- Check the necessary ports as a requirement for k3s (https://docs.k3s.io/installation/requirements)

a. Add the hostnames of the nodes in /etc/hosts

To access your nodes using hostnames from your laptop’s terminal and from each node, you need to configure /etc/hosts individually. The steps are the same for all three nodes, so you will need to repeat the process for each one. Once configured, you can easily access each node using its hostname, simplifying the management process and making it easier to connect to your nodes.

```bash  
vi /etc/hosts  
## Raspberry Cluster Nodes  
192.168.18.35 k3s-master  
192.168.18.52 worker01  
192.168.18.53 worker02  
``` 


...

## Conclusion

_Conclusion text from the blog_
