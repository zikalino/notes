# Integrating IoT Hub Explorer with Visual Studio Code using Docker

This document describes how to encapsulate command line tool called **IoT Hub Explorer** with Visual Studio Code using Docker.

## What will be used?

* Docker Runner extension

## Step 1 - Create iothub-explorer image

I have created very simple image, based on **node** and just installed **iothub-explorer** on the top:

    FROM node
    MAINTAINER zikalino

    RUN npm install -g iothub-explorer

    ENTRYPOINT [ "bash" ]
 
## Step 2 - Integrate with terminal

I have used **Docker Runner** to find and pull the image:

**Alt+Ctrl+D** -> **Search Images** -> **iothub-explorer**

Select **dockiot/iothub-explorer** -> **Pull & Pin to the Menu**




## Step 3 - Add commands to command pallette

## Step 4 - Create menu

## Step 5 - Capture & Display output

## What if user doesn't have Docker?
