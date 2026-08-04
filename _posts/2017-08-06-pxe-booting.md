---
layout: post
title: "PXE booting"
date: 2017-08-06 00:00:00 +0000
tags: ["bios", "preseed", "preseeding", "proxydhcp", "pxe", "tftp", "ubuntu", "uefi", "unattended installation", "vagrant"]
---

TL;DR The code used here is available on [Github](https://github.com/dlresende/pxe-server).

This article describes the steps I walked through in order to install an Ubuntu server on my Dell PowerEdge 110T server using Preboot eXecution Environment (PXE).

One may ask: why bother with PXE for ocasional installations? Wouldn't be simpler to just use a bootable USB memory stick?

Well, from a practical point-of-view the answer is yes. Although, I decided to go through this process mainly for 2 reasons:

1. I always wanted to do this, either professionally, or as a learning exercise (it looks like Iron man, and that's cool)
2. as a lazy developer, I challenge myself to automate everything whenever possible

That said, before we get started, it is important to understand some concepts and technologies, and how the whole thing works together.

# PXE mechanics

Preboot eXecution Environment (PXE, pronounced as pixie) specification describes a standardized client-server environment that boots a software assembly, retrieved from the network, on PXE-enabled clients. – [Wikipedia](https://en.wikipedia.org/wiki/Preboot_Execution_Environment)

On the client side it requires only a PXE-capable [network interface controller](https://en.wikipedia.org/wiki/Network_interface_controller) (NIC), and uses a small set of industry-standard network protocols such as [Dynamic Host Configuration Protocol (DHCP)](https://en.wikipedia.org/wiki/DHCP) and [Trivial File Transfer Protocol (TFTP)](https://en.wikipedia.org/wiki/Trivial_File_Transfer_Protocol). In many computers, the PXE capability can be enabled in the [BIOS](#bios-x-uefi).

When a machine is booted in PXE mode, it tries to acquire an IP address during the boot phase through DHCP, and it sends some PXE-specific options in its DHCPDISCOVER packet. A PXE-enabled DHCP server will then respond to the client's request with a DHCPOFFER packet containing not only default network information (i.e. IP address), but also the TFTP server IP address and the name of the Network Bootstrap Program (NBP), in this case pxelinux.0. The client can then download the NBP from the TFTP server and start the boot process. More information about DHCP sessions [here](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol#Operation).

Once NBP is booted, it will load the configuration from pxelinux.cfg/default in order to print the installation menu or just proceed with the installation, following what has been configured. The installation menu configuration also specifies the [initrd](https://en.wikipedia.org/wiki/Initial_ramdisk) image that will handle the installation process. Check the [Syslinux Project wiki](http://www.syslinux.org/wiki/index.php?title=PXELINUX) for more details about the PXE boot process.

Although, I'd like to keep using my router as my default DHCP server (setup a second DHCP server just for PXE sounds like a bit of waste in my case). The problem is that my router does not have any PXE capability.

In order to address this issue, there is something called proxyDHCP, which is a second DHCP server which only provides PXE-specific information to the PXE client through DHCP.

## BIOS x UEFI

The Basic Input/Output System (BIOS) is a firmware that comes pre-installed on computers, and sits between hardware and software.

Once the machine is turned on, the BIOS is responsible for initialising and configuring all the components, like CPU, memory, disks, etc. If the BIOS was not there, we'd probably be using jumpers to handle all the configuration. After the initialisation, the BIOS will perform the [power-on self test (POST)](https://en.wikipedia.org/wiki/Power-on_self-test), to check whether all the components are working properly. Finally, the BIOS will look for bootable devices in order to hand off control to the operating system (OS).

The BIOS has been there for a long time and has some serious limitations. The BIOS can only handle [Master Boot Records (MBR)](https://en.wikipedia.org/wiki/Master_boot_record) until roughly 2 terabytes, for example. Slow boot time, lack of mouse support and poor graphics also figure in the list of BIOS limitations.

The [Unified Extensible Firmware Interface (UEFI)](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) is the successor of BIOS. The UEFI aims to fix BIOS limitations and add more features, like looking for malware during the boot process, adding some capabilities for remote troubleshooting and configuration, and more.

That said, it is important to know which boot mechanism is being used. A PXE server configured to work only with BIOS won't work with UEFI, and the other way round. A netboot image is commonly prepared having one or another of these two mechanisms in mind. This article describes a PXE server configuration to be used with BIOS.

# Setup a PXE server

In order to setup a PXE server, the following tools and techniques were used:

- [Preseeding](https://help.ubuntu.com/lts/installation-guide/armhf/apbs02.html): in order to spare myself from manually installing the OS
- [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html): in order to setup a proxyDHCP and a TFTP server
- [Vagrant](https://www.vagrantup.com/)/ [VirtualBox](https://www.virtualbox.org/)/ [Ubuntu server](https://www.ubuntu.com/server): in order to avoid installing tools I won't be using everyday on my machine, and to be able to re-use the same mechanism in another environment

## Setup a Vagrant box

The first step is to configure a Vagrant box running Ubuntu server over VirtualBox. At the network level, the box will be configured in [bridge mode](https://www.virtualbox.org/manual/ch06.html#network_bridged). In practice, this will leave the Virtual Machine (VM) running at the same level as the machine hosting it, allowing it to communicate directly with the DHCP server, and sparing some [Network Address Translation (NAT)](https://en.wikipedia.org/wiki/Network_address_translation) configuration.

Here's the [Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/) that configures a box in bridge mode:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "public_network"
end
```

Based on the Vagrantfile, the command bellow will download an Ubuntu box from a [HashiCorp's catalogue](https://app.vagrantup.com/boxes/search), apply some configuration (if any) and start the VM.

```bash
$ vagrant up
```

Since this configuration is using bridge networking through Vagrant [public_network](https://www.vagrantup.com/docs/networking/public_network.html), it is necessary to inform Vagrant which network it should use in the bridge. This will be the network interface the host machine uses to connect to the DHCP server and, consequently, the internet (in this case en0).

```text
default: Available bridged network interfaces:
1) en0: Wi-Fi (AirPort)
2) en1: Thunderbolt 1
...
default: Which interface should the network bridge to?
```

## Download the Ubuntu Netboot Image

A netboot image is a bootable image optimised to perform an installation via network (as opposed to an ISO image). It generally contains:

1. a small network boot manager program like pxelinux.0 (or NBP), which is used to deploy the installation menu and load a full installation image;
2. a second-stage bootable full image (typically initrd), used to perform the installation

Download an Ubuntu netboot image that will handle the installation process over the internet:

```bash
$ wget http://archive.ubuntu.com/ubuntu/dists/xenial/main/installer-amd64/current/images/netboot/netboot.tar.gz \
  -O netboot.tar.gz \
  --quiet
$ mkdir -p /tmp/tftp
$ tar xf netboot.tar.gz -C /tmp/tftp
```

## Install and Configure Dnsmasq

The dnsmasq is a convenient tool that brings together many network services, like DNS, DHCP, TFTP, and more. Dnsmasq is used here to setup a proxyDHCP and a TFTP server.

Ensure dnsmask is installed in the Vagrant box:

```bash
$ sudo apt-get install dnsmasq
```

Finally, create the file /etc/dnsmasq.d/proxydhcp.conf, configure the TFTP server and enable the proxyDHCP:

```conf
# Set the boot filename for netboot/PXE.
dhcp-boot=pxelinux.0
# Loads pxelinux.0 from dnsmasq TFTP server.
pxe-service=x86PC, "Install Linux", pxelinux
# Set the root directory for files available via FTP.
tftp-root=/tmp/netboot
# Log lots of extra information about DHCP transactions.
log-dhcp
# This range(s) is for the public interface, where dnsmasq functions
# as a proxy DHCP server providing boot information but no IP leases.
dhcp-range=REPLACE_THIS_WITH_THE_IP_ADDRESS_OF_INTERFACE_USED_IN_THE_BRIDGE,proxy
```

## Preseed the Installation

Preseeding is a method for automating the installation of the Debian operating system and its derivatives. Answers to installation questions, which would normally be answered interactively by an operator, are predetermined and supplied via a configuration file (and sometimes boot parameters). – [Wikipedia](https://en.wikipedia.org/wiki/Preseed)

This process is sometimes called unattended installation.

In order to activate the unattended installation, append the preseed configuration to the boot command line in the installation menu (preseed/url=tftp://…/preseed.cfg) and activate the automation of the installation process (auto=true):

```bash
$ cat > /tmp/tftp/pxelinux.cfg/default << EOF
default install
label install
kernel ubuntu-installer/amd64/linux
append vga=788 initrd=ubuntu-installer/amd64/initrd.gz auto=true priority=critical preseed/url=tftp://REPLACE_THIS_WITH_THE_IP_ADDRESS_OF_INTERFACE_USED_IN_THE_BRIDGE/preseed.cfg
EOF
```

Finally, create a preseed.cfg file with the desired parameters for the installation, and place it under the TFTP root directory (in this case /tmp/tftp). [Here](https://help.ubuntu.com/lts/installation-guide/example-preseed.txt)'s an example of preseed configuration.

## Troubleshooting

### How to check if the TFTP service is running?

```bash
$ netstat -tul
```

A TFTP service listening for connections should appear as follows:

```text
udp 0 0 *:tftp *:*
udp6 0 0 [::]:tftp [::]:*
```

### Check that I can get files from the TFTP server

Install the TFTP client and try to get files from the server.

```bash
$ sudo apt-get install tftp
$ tftp 192.168.1.106
tftp> get pxelinux.0
Received 43114 bytes in 0.0 seconds
tftp> quit
```

### How to check if the proxyDHCP server is receiving the DHCPDISCOVERY packet with PXE-specific options?

```bash
$ sudo tcpdump -i enp0s8 udp -s0 -vvv
```

When the PXE client is started, UDP packets with the NBP pxelinux.0 in the response should be intercepted by tcpdump.

Alternatively, dnsmasq writes its logs in /var/log/syslog. Since we are using the log-dhcp option in the dnsmasq configuration, we should also have more information in that file.

### My computer seems not to be booting in PXE mode

- Do you need to press some key during boot time to boot in PXE mode?
- Are you using UEFI, and not BIOS?
- Does your NIC has to be configured through BIOS to have PXE enabled?

To answer all these questions you might need to check your computer's manual.
