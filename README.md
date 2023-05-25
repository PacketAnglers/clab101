# Container Lab Topology Operations 

## Pre Requisites

### Directory Layout

```bash
|---images
    |--- ( cEOS image files live here )
|---osuwmcdc
    |---topologies
        |---fulldc
            |---configs
                |---ack
                    |--- ( All device startup configs for ACK location live here )
                |---SOC
                    |--- ( All device startup configs for SOC location live here )
                |---cvx
                    |--- ( All device startup configs for CVX nodes live here )
            |--- dc_data_only.yml ( topology file for leaf/spine nodes not including OOB network )
            |--- dc_full_topo.yml ( topology file for full infrastructure )
            |---clab-OSUWMC-DC
                |--- ( dynamically created folder with device container data / running configs / etc )
```

### Topology File

The topology file specifies the nodes in the virtual environment that are going to run, their docker management IP, the cEOS image they will run, their startup configuration location, and all the connections between the devices.

This is an example of a node configuration section from the topology file:

```bash
<node name>:
      kind: ceos 
      mgmt_ipv4: 10.38.4.130
      startup-config: configs/ack/SPI-ACK-01A.cfg ( maps to location in directory structure )
      ports: ( define ports you would use to remotely connect to the device )
        - '22001:22'
        - '8001:80'
        - '44301:443'
      labels:
        graph-level: 2 ( level on graphite image )
        graph-icon: switch ( icon for graphite image )
```

This is an example of a connection definition from the topology file, which specifies two connecting endpoints:

```bash
 - endpoints: ["node1:et1", "node2:et1"]
 - endpoints: ["node1:et2", "node2:et2"]
```

## Starting the Container Lab topology

To start the topology, you should be in the directory that the `.yml` topology file lives in.  Once you are there, you can start the topology wih the following command:

```bash
sudo clab deploy -t <name of topology .yml file> --reconfigure
```

Once the topology is running, you can check the status of the nodes with the following commands:

```bash
sudo clab inspect -a
```

You can check the resources currently being used by the topology in two ways:

To see the overall consumption of the server, use the top command:

```bash
top
```

To see the consumption of each individual container, use the following command:

```bash
sudo docker stats
```

You can exit this command with `ctl+c`.

## Accessing each Container Lab Node

There are several ways to access each container lab node, remotely via ssh, locally from the command line of the linux server, and from the graphite diagram.

To access the node remotely via ssh, find the port definitions in the topology file, and ssh to the container lab server IP:<port defined>.

To access the node from the command line of the linux server, you can issue the following command:

```bash
sudo docker exec -it <node name> Cli
```

To access the node from the graphite diagram, find the node you want to connect to, right click, and click SSH.


## Shutting Down the Topology

In order to make any changes to the underlying topology or the topology file, you will need to shut down the virtual environment.  

In order to shut down the virtual environment, issue following command:

```bash
sudo clab destroy -t <topology name.yml> --cleanup
```

Due to the size of the topology, it may not shut down cleanly and give errors indicating it could not exit the containers as they did not respond to the shutdown command.  When this happens you will want to monitor the docker containers until they all show exited, and then remove them.

To monitor the docker containers and see if they are exited, issue the following command:

```bash
sudo docker ps -a
```

All the containers should show `exited x min ago1`.

Once they are exited, you can manually remove and clean up the containers with the following command:

```bash
sudo docker container prune
```