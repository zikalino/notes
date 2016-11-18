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

Modify **/boot/cmdline.txt**, and replace everything after **root=** with following:

**root=/dev/nfs nfsroot=xxx.xxx.xxx.xxx:/,v4,tcp,mountport=2049,port=2049 rw ip=dhcp rootwait elevator=deadline**

where **xxx.xxx.xxx.xxx** is the address of the machine where Docker container with Raspbian is running.

Specyfying **v4** and **port** is important, so Raspberry Pi will just directly use port **2049** to connect to NFS.
When these parameters are not specified, Raspberry Pi would try to use version 3 of NFS protocol, and will try to obtain NFS port and mount port via **PORTMAP** protocol (port 111).
Portmap would return some random port numbers allocated on the fly.

