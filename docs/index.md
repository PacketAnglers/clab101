# Getting Started with Container Lab (cLab 101)

## **Introduction**

[ContainerLab](https://containerlab.dev) is an open source project that provides a quick and easy method to emulate network toplogies. The examples on this site will make use of Arista's cEOS-lab, but there is [wide ranging vendor support](https://containerlab.dev/manual/kinds/) for both containerized (and non-containerized) Network Operating Systems.

This guide will focus on how to quickly get started with ContainerLab.

## **Requirements**

Getting up and running with ContainerLab requires the following:

<div class="annotate" markdown>

- Linux host (this guide uses Ubuntu 20.04)
- Sufficient memory (1) and compute (2) to run desired topologies
- Docker 
- ContainerLab
- Containerized Network Operating System of choice (3)

</div>

1. :pencil2: Containerized Network Operating Systems are much more resource efficient than their tradtionally virtualized conterparts. For example, as of 4.30.0F, each **cEOS-lab** node consumes ~850MB of memory when up and running. **vEOS-lab** requires 4GB.
    
2. :pencil2: Topologies are most CPU intensive at boot. There is a `startup-delay` toggle that can be used to help manage this when a topology is too large to simultaneously boot all nodes.

3. :pencil2: We will be using cEOS-Lab for this guide. However, we can use other Network Operating Systems to create large multi-vendor topologies as needed.


## **Installation**

### **Install Docker**

The first step is to install docker on the Linux host. [These instructions](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04) cover, step by step, how to perform this isntallation on Ubuntu 20.04

### **Install ContainerLab**

This one-line command (taken from [this guide](https://containerlab.dev/install/#install-script)) will install ContainerLab on the Linux host:

```bash
# download and install the latest release (may require sudo)
bash -c "$(curl -sL https://get.containerlab.dev)"
```

We can validate that ContainerLab was installed by using the `sudo clab version` command:

```bash hl_lines="10"
mitch@mitchlab2:~$ sudo clab version

                           _                   _       _     
                 _        (_)                 | |     | |    
 ____ ___  ____ | |_  ____ _ ____   ____  ____| | ____| | _  
/ ___) _ \|  _ \|  _)/ _  | |  _ \ / _  )/ ___) |/ _  | || \ 
( (__| |_|| | | | |_( ( | | | | | ( (/ /| |   | ( ( | | |_) )
\____)___/|_| |_|\___)_||_|_|_| |_|\____)_|   |_|\_||_|____/ 

    version: 0.41.2
     commit: fe51f41f
       date: 2023-05-18T14:57:44Z
     source: https://github.com/srl-labs/containerlab
 rel. notes: https://containerlab.dev/rn/0.41/#0412
```

### **Download Containerized NOS**

We will be using Arista's cEOS-lab in this guide. This is a free download that can be found under the **cEOS-lab** directory [here](https://www.arista.com/en/support/software-download) (1)
{ .annotate }

1. :pencil2: When downloading cEOS-lab, choose one of the images ending in `.tar.xz`

### **Import Containerized NOS**

Before defining, and booting, a topology; The containerized NOS will need to imported into Docker. This can be done via the following command:

```bash
sudo docker import [path to cEOS-lab .tar.xz file] ceos:[release ID]
```

As a more complete example, here is a command that imports cEOS64-lab-4.30.0F.tar.xz into docker as ceos:4.30.0F

```bash
sudo docker import cEOS64-lab-4.30.0F.tar.xz ceos:4.30.0F #(1)!
```

1. :pencil2: This command was run from the directory where the cEOS64-lab-4.30.0F.tar.xz image was located. This is not a requirement, but makes the example command easier to read.

We can validate that the image was imported by using the `sudo docker images` command:

```bash hl_lines="4"
mitch@mitchlab2:~$ sudo docker images 
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
ceos                  trunk     1d73d2ef5663   7 days ago      2.37GB
ceos                  4.30.0F   0ca9c82f8c3b   2 weeks ago     2.47GB
mitchv85/devhost      latest    973bcbccca1d   3 weeks ago     1.83GB
netreplica/graphite   latest    96300cf0339b   11 months ago   206MB
```

### **Creating a Topology**

At this point, we have Docker host running on Ubuntu 20.04 with ContainerLab installed. We have also downloaded, and imported, cEOS-lab into Docker. :champagne:

Now, let's define a topology file that ContainerLab can use to create, and deploy, a topology

For reference, here is the topology we will be creating:

![Topology](assets/images/topo.svg){: style="width:800px"}

ContainerLab makes use of YAML files to create, and deploy, topologies. For this topology, we'll create a file named `lab.yml` (1). This file, and it's components, are shown below for reference:
{ .annotate }

1. :pencil2: The topology file can be named whatever we'd like. We are just calling it `lab.yml`for simplicity.

```yaml
---
# -------------------------------------------------------------
# L3LS EVPN Demo Topology
# 2 Spines & 2 Leaf Pairs with MLAG
# -------------------------------------------------------------

name: evpn-demo #(1)!
prefix: "" #(2)!

mgmt: #(3)!
  network: mgmt
  ipv4-subnet: 172.100.100.0/24

topology:

  defaults:
    env:
      INTFTYPE: et #(4)!

  kinds: #(5)!
    ceos:
      image: ceos:4.30.0F #(6)!
    linux:
      image: mitchv85/devhost #(7)!

  nodes: #(8)!

#########################
# SPINES                #
#########################

    SPINE1: #(9)!
      kind: ceos #(10)!
      mgmt-ipv4: 172.100.100.101 #(11)!
      startup-config: startup-configs/SPINE1.cfg #(12)!
      ports: #(13)!
        - '22001:22' #(14)!
        - '8001:80' #(15)!
        - '44301:443' #(16)!

    SPINE2:
      kind: ceos
      mgmt-ipv4: 172.100.100.102
      startup-config: startup-configs/SPINE2.cfg
      ports:
        - '22002:22'
        - '8002:80'
        - '44302:443'

#########################
# LEAF                  #
#########################

    LEAF1:
      kind: ceos
      mgmt-ipv4: 172.100.100.103
      startup-config: startup-configs/LEAF1.cfg
      ports:
        - '22003:22'
        - '8003:80'
        - '44303:443'

    LEAF2:
      kind: ceos
      mgmt-ipv4: 172.100.100.104
      startup-config: startup-configs/LEAF2.cfg
      ports:
        - '22004:22'
        - '8004:80'
        - '44304:443'

    LEAF3:
      kind: ceos
      mgmt-ipv4: 172.100.100.105
      startup-config: startup-configs/LEAF3.cfg
      ports:
        - '22005:22'
        - '8005:80'
        - '44305:443'

    LEAF4:
      kind: ceos
      mgmt-ipv4: 172.100.100.106
      startup-config: startup-configs/LEAF4.cfg
      ports:
        - '22006:22'
        - '8006:80'
        - '44306:443'

###########################################
# HOSTS                                   #
###########################################

    HostA:
      kind: linux
      mgmt-ipv4: 172.100.100.201
      ports:
        - '22201:22'
      exec:
        - bash /usr/local/bin/hostnetconfig.sh -b -i 10.10.10.101/24 -g 10.10.10.1 #(17)!

    HostB:
      kind: linux
      mgmt-ipv4: 172.100.100.202
      ports:
        - '22202:22'
      exec:
        - bash /usr/local/bin/hostnetconfig.sh -b -i 10.30.30.101/24 -g 10.30.30.1

    HostC:
      kind: linux
      mgmt-ipv4: 172.100.100.203
      ports:
        - '22203:22'
      exec:
        - bash /usr/local/bin/hostnetconfig.sh -b -i 10.10.10.102/24 -g 10.10.10.1

    HostD:
      kind: linux
      mgmt-ipv4: 172.100.100.204
      ports:
        - '22204:22'
      exec:
        - bash /usr/local/bin/hostnetconfig.sh -b -i 10.20.20.101/24 -g 10.20.20.1

####################
# SPINE1 to LEAF   #
####################
    - endpoints: ["SPINE1:et1", "LEAF1:et1"] #(18)!
    - endpoints: ["SPINE1:et2", "LEAF2:et1"]
    - endpoints: ["SPINE1:et3", "LEAF3:et1"]
    - endpoints: ["SPINE1:et4", "LEAF4:et1"]

####################
# SPINE2 to LEAF   #
####################
    - endpoints: ["SPINE2:et1", "LEAF1:et2"]
    - endpoints: ["SPINE2:et2", "LEAF2:et2"]
    - endpoints: ["SPINE2:et3", "LEAF3:et2"]
    - endpoints: ["SPINE2:et4", "LEAF4:et2"]

##################
# LEAF1 to LEAF2 #
##################
    - endpoints: ["LEAF1:et5", "LEAF2:et5"]
    - endpoints: ["LEAF1:et6", "LEAF2:et6"]

####################
# LEAF3 to LEAF4   #
####################
    - endpoints: ["LEAF3:et5", "LEAF4:et5"]
    - endpoints: ["LEAF3:et6", "LEAF4:et6"]

####################
# HOSTA            #
####################
    - endpoints: ["HostA:eth1", "LEAF1:et3"]
    - endpoints: ["HostA:eth2", "LEAF2:et3"]

####################
# HOSTB            #
####################
    - endpoints: ["HostB:eth1", "LEAF1:et4"]
    - endpoints: ["HostB:eth2", "LEAF2:et4"]

####################
# HOSTC            #
####################
    - endpoints: ["HostC:eth1", "LEAF3:et3"]
    - endpoints: ["HostC:eth2", "LEAF4:et3"]

####################
# HOSTD            #
####################
    - endpoints: ["HostD:eth1", "LEAF3:et4"]
    - endpoints: ["HostD:eth2", "LEAF4:et4"]
```

1. :pencil2: The name of the topology
2. :pencil2: (Optional) Setting the "prefix" of each lab node name to null. By default, the name of the lab is prepended to every node (container) that is created in the topology.
3. :pencil2: (Optional) Each node will have an IP assigned to it from this block. This is the equivalent of an "Out of Band Management" network in the cLab world; This network only exists within the cLab host (by default), and is used by the nodes for communication to/from the world outside of the Docker host.
4. :pencil2: (Optional) This setting is unique to cEOS-lab. It is required in order for the OSPF and IS-IS routing protocols to function properly on cEOS-lab nodes.
5. :pencil2: This is where we define the different **kinds** of containerized nodes we will be using in our topology.
6. :pencil2: Anywhere we define a node as `ceos`, it will run our imported 4.30.0F cEOS-lab image
7. :pencil2: Anywhere we define a node as `linux`, it will run the publicly available [mitchv85/devhost](https://hub.docker.com/repository/docker/mitchv85/devhost/general) image.
8. :pencil2: Here we start defining the nodes that will exist in our topology.
9. :pencil2: Defines the `SPINE1` node
10. :pencil2: Sets `SPINE1` node as running `ceos` ***kind*** of container. Which we set to ceos:4.30.0F
11. :pencil2: (Optional) Statically define the management address that we'd like assigned to the `SPINE1` node.
12. :pencil2: (Optional) Path to a defined startup-config that the `SPINE1` node should use when the topology is deployed
13. :pencil2: (Optional) Define a list of ports on the Docker Host that we'd like forwarded to the `SPINE1` node
14. :pencil2: (Optional) If an inbound connection to TCP port **22001** is received on the Docker host, it will forward this to TCP port **22** (SSH) on the `SPINE1` node
15. :pencil2: (Optional) If an inbound connection to TCP port **8001** is received on the Docker host, it will forward this to TCP port **80** (HTTP) on the `SPINE1` node
16. :pencil2: (Optional) If an inbound connection to TCP port **44301** is received on the Docker host, it will forward this to TCP port **443** (HTTP) on the `SPINE1` node
17. :pencil2: (Optional) Unique to the mitchv85/devhost image. Instructs cLab to run the `hostnetconfig.sh` shell script to configure IP addressing and/or LACP after deploying the node. More informaiton [here](https://hub.docker.com/repository/docker/mitchv85/devhost/general)
18. :pencil2: Define a link between `SPINE1, Ethernet1` and `LEAF1, Ethernet1`. This can be customized for modular interfaces as well. For example, `et1_1_1` would evaluate to `Ethernet1/1/1` in cEOS-lab.

--8<-- "includes/abbreviations.md"

### **Deploying a Topology**

Next, let's tell ContainerLab to deploy our topology