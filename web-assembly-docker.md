# Creating WebAssembly Docker

When I have checked WebAssembly website for the first time (http://webassembly.org/), I have found typical **Getting Started** instructions (http://webassembly.org/getting-started/developers-guide/). First thing I noticed was:

    To compile to WebAssembly, we will at the moment need to compile LLVM from source. The following tools are needed as a prerequisite 

That made me to decide just to do it in Docker... I heave created a neat dockerfile, which is available here:

    https://github.com/zikalino/docker-webassembly

Well, dockerfile is just 15 lines long at the moment, but building it on my local machine takes hours... Ok, no problem, I can just run it in the background, and then push it to DockerHub when it's ready. Except that it was failing due to not sufficient memory...

I decided to comment out last two steps in the dockerfile, create smaller image, execute missing steps in the container and then just commit final container back to original image. I have somehow managed to complete the build, but... problems again.

## docker commit

It appeared that **docker commit** was just hanging forever. After quick research on the internet I found out that slowness of **docker commit** is a common problem. Considering that my commit was particularly large, maybe it would take several months to complete. So I gave up this path...

## Using Automated Build in DockerHub

This was my next idea. I have linked my GitHub repo with DockerHub, and then pressed **Create** button. **Create** has changed to **Creating** and stayed like this forever after...

## Using Docker on Ubuntu Server in Azure

Surely this must work, right?
At first I have chosen one of featured VMs. After it was created I have logged in, cloned my GitHub repo, and tried to build.

Good news was that it has taken maybe around 30 minutes to complete (almost) entire build. Unfortunately it failed due to lack of space...

I never give up, so I have created another VM with perhaps 20 cores and enormous SSD drive. The build took 10 minutes, but still failed at the end.

A few more hours of experimenting and finally I figured out the problem. Docker was not using that enormous SSD drive at all.

By running following command:

    df -h

It's easily to see that root filesystem is mapped to small 30GB drive, and large SSD drive is mounted to **/mnt**

Using following commands I have mapped enormous drive to Docker:

    cp /var/lib/docker/... /dev/sdb1
    sudo mount /dev/sdb1 /var/lib/docker

## Final Image

To get final image just type:

    docker pull dockiot/webassembly

It's still around 10GB, will take some time to pull, but at least you can avoid problems.


## Learnings

- don't use your local machine to build large docker images
- **docker commit** is extremely slow
- **Automated Docker Build** is slower than local build
