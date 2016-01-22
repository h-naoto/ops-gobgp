# Ops-GoBGP

Ops-GoBGP relaying [OpenSwitch](http://www.openswitch.net) and [GoBGP](https://github.com/osrg/gobgp). Specifically, you can relay the bgp settings to gobgp that you set in the console of OpenSwitch,
and can relay the route that has GoBGP to OpenSwitch.

This page explains how to run Ops-GoBGP.
This example have to relay settings and routes between OpenSwitch and Gobgp using Ops-GoBGP in docker container.

## Prerequisites

Install the docker environment referring to the [here](https://docs.docker.com/engine/installation/ubuntulinux/).

## Getting and Running OpenSwitch container
- Getting image

 Thera are two ways, to get OpenSwitch container image.
 First way, get image from DockerHub using the **docker pull** command.
 Second way, build the iamge from Dockerfile in Ops-GoBGP repository using the **docker build** command.

 - In the case of **docker pull**
  ```bash
  $ sudo docker pull nhanaue/openswitch
  $ sudo docker images
  REPOSITORY           TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
  nhanaue/openswitch   latest              6509d58db7f5        2 hours ago         1.278 GB
  ```

 - In the case of **docker build**
  ```bash
  $ git clone https://github.com/h-naoto/ops-gobgp.git
  $ cd ops-gobgp/docker
  $ sudo docker build -t nhanaue/openswitch .
  $ sudo docker images
  REPOSITORY           TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
  nhanaue/openswitch   latest              bcff835707aa        1 hours ago         1.278 GB
  ```


- Running container
 ```bash
 sudo docker run --privileged -v /tmp:/tmp -v /dev/log:/dev/log -v /sys/fs/cgroup:/sys/fs/cgroup -h ops --name ops nhanaue/openswitch /sbin/init &
 $ sudo docker ps -a
 CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS               NAMES
 452f31c31be6        nhanaue/openswitch   "/sbin/init"        4 seconds ago       Up 2 seconds                            ops
 $ sudo docker exec -it ops bash
 ```

## Installing GoBGP

- Preparation
 ```bash
 bash-4.3# echo "nameserver 8.8.8.8" >> /etc/resolv.conf
 ```

- Go get  GoBGP
 ```bash
 bash-4.3# go get -v github.com/osrg/gobgp/gobgpd
 bash-4.3# go get -v github.com/osrg/gobgp/gobgp
 ```

## Installing Ops-GoBGP
- Clone source
 ```bash
 bash-4.3# git clone https://github.com/h-naoto/ops-gobgp.git
 ```

- Install dependent libraries of python
 ```bash
 bash-4.3# cd ops-gobgp
 bash-4.3# pip install -r pip-requires.txt
 ```

## Starting relay using the Ops-GoBGP

- Setting OpenSwitch
 ```bash
 bash-4.3# vtysh
 switch# show running-config
 Current configuration:
 !
 !
 !
 switch# configure terminal
 switch(config)# router bgp 65001
 switch(config-router)# bgp router-id 10.0.255.1
 switch(config-router)# neighbor 10.0.255.2 remote-as 65002
 switch(config-router)# do show running-config
 Current configuration:
 !
 !
 !
 router bgp 65001
     bgp router-id 10.0.255.1
     neighbor 10.0.255.2 remote-as 65002
 !
 ```

- Starting GoBGP
 ```bash
 bash-4.3# gobgpd -p
 INFO[0000] gobgpd started
 INFO[0000] Read Configuration from OpenSwitch
 ```

- Starting Ops-GoBGP
 ```bash
 bash-4.3# cd ~/ops-gobgp
 bash-4.3# python openswitch.py
 2016-01-06 20:49:36,412 - connection - INFO - Connecting to OpenSwitch...
 2016-01-06 20:49:36,618 - connection - INFO - Connected
 2016-01-06 20:49:36,639 - connection - INFO - Connecting to Gobgp...
 2016-01-06 20:49:36,640 - connection - INFO - Connected
 2016-01-06 20:49:36,641 - connection - INFO - Run run_ops_to_gogbp thread...
 2016-01-06 20:49:36,642 - connection - INFO - Run run_gogbp_to_ops thread...
 2016-01-06 20:49:36,644 - connection - INFO - Wait for a change the bestpath from gobgp...
 ```

 If GoBGP has routes in Global Rib, routes relayed to the OpenSwitch as follows:
 ```bash
 bash-4.3# vtysh
 switch# show ip bgp
 Status codes: s suppressed, d damped, h history, * valid, > best, = multipath,
               i internal, S Stale, R Removed
 Origin codes: i - IGP, e - EGP, ? - incomplete

 Local router-id 10.0.255.1
    Network          Next Hop            Metric LocPrf Weight Path
 *  10.10.10.0/24    10.0.255.2            0      0  32768 65002 i
 *  10.10.11.0/24    10.0.255.2            0      0  32768 65002 i
 *  10.10.12.0/24    10.0.255.2            0      0  32768 65002 i
 Total number of entries 3
 ```

## Licensing

Ops-GoBGP is licensed under the Apache License, Version 2.0. See
[LICENSE](https://github.com/osrg/ops-gobgp/blob/master/LICENSE) for the full
license text.
