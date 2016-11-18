# Raspberry Pi - booting from Docker with NFS

## Introduction

There seems to be a lot of information on the internet describing how to boot Raspberry Pi from the network.

There's even a long document on Raspberry Pi website.

## Prerequisites

- I was using Windows 10 laptop, so this document contains some references to Windows 10 firewall
- Docker
- Raspberry Pi with Raspian XXX

## Running Docker

Just run following command.

docker run -t -i --name nfs -p 2049:2049 --privileged dockiot/raspbian-nfs /exports

Make sure TCP port 2049 is open in your firewall.

## What needs to be done on Raspberry Pi that you want to boot from the network?

This part is pretty simple. You just need to change one line of code. I just followed instructions on Raspberry Pi website and I have modified */boot/cmdline.txt*, and added following:

*root=/dev/nfs nfsroot=xxx.xxx.xxx.xxx:/ rw ip=dhcp rootwait elevator=deadline*

After this modification and rebooting Raspberry Pi, I started receiving PORTMAP UDP packets on port 111.
I had to add port 111 to Windows firewall, and then my NFS started responding.
Unfortunately responses returned random ports, which would have to be exposed by my Docker container and also added to Windows firewall.
So this solution appeared to be not feasible.

I wanted Raspberry Pi to connect directly to port 2049 (NFS port) without going through whole PORTMAP procedure.
After some search I found some information on specifying **nfsroot** in **cmdline.txt**.
I have flashed Raspbian again, and modified **cmdline.txt** as follows:

*root=/dev/nfs nfsroot=xxx.xxx.xxx.xxx:/,v4,tcp,mountport=2049,port=2049 rw ip=dhcp rootwait elevator=deadline*

After rebooting Raspberry Pi connected directly to my laptop using port 2049.

## Running Docker

Just run following command.

docker run -t -i --name nfs -p 2049:2049 --privileged dockiot/raspbian-nfs /exports

Make sure TCP port 2049 is open in your firewall.

## Other Tools


## U-boot project


http://elinux.org/RPi_U-Boot

https://github.com/u-boot/u-boot

