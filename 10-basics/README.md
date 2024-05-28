# Containerlab Basics

This workshop section introduces you to containerlab basics - topology file, image management workflows and lab lifecycle. It is loosely based on the official [Containerlab quickstart](https://containerlab.dev/quickstart/).

## Repository

Clone the repository to your workshop VM:

```bash
cd ~ && git clone https://github.com/srl-labs/ac1-workshop.git \
&& cd ac1-workshop/10-basics
```

The repo should be cloned and you should be in the `ac1-workshop` directory as per the output below:

```
[*]─[rd-13]─[~/ac1-workshop/10-basics]
└──>
```

## Topology

The topology file `basic.clab.yml` defines the lab we are going to use in this basics exercise. It consists of the two nodes:

* Nokia SR Linux
* Arista cEOS

The nodes are interconnected with a single link over their respective first Ethernet interfaces.

```yaml
name: basic
topology:
  nodes:
    srl:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux
    ceos:
      kind: arista_ceos
      image: ceos:4.32.0F

  links:
    - endpoints: [srl:e1-1, ceos:eth1]
```

## Deployment attempt #1

Try to deploy the lab:

```bash
sudo containerlab deploy -t basic.clab.yml
```

Deployment fails, why?

## Image management

Check what images are available on the system:

```bash
docker images
```

Pull SR Linux container image:

```bash
docker pull ghcr.io/nokia/srlinux
```

Import cEOS image and pay attention to the 2nd argument for the `docker import` command where you have to specify the image:

```bash
docker import ~/images/cEOS64-lab-4.32.0F.tar.xz ceos:4.32.0F
```

Check the local image store again:

```bash
docker images
```

## Deployment attempt #2

Now that the images are available, try to deploy the lab again:

```bash
sudo containerlab deploy -t basic.clab.yml
```

Note, you can use a shortcut version of the same command - `sudo clab dep -t basic.clab.yml`.

The deployment should succeed.

## Connecting to the nodes

Connect to the Nokia SR Linux node using the container name:

```bash
ssh clab-basic-srl
```

Connect to cEOS node using its IP address:

```bash
ssh admin@172.20.20.3
```

## Containerlab hosts automation

Containerlab creates `/etc/hosts` entries for each deployed lab so that you can access the nodes using their names. Check the entries:

```bash
cat /etc/hosts
```

## Containerlab ssh config automation

Containerlab creates ssh config entries in `/etc/ssh/ssh_config.d/clab-<lab-name>.conf` file to provide easy access to the nodes. Check the entries:

```bash
cat /etc/ssh/ssh_config.d/clab-basic.conf
```

## Checking network connectivity

SR Linux and cEOS are started with their first Ethernet interfaces connected. Check the connectivity between the nodes:

The nodes also come up with LLDP enabled, our goal is to verify that the basic network connectivity is working by inspecting

```bash
ssh clab-basic-srl
```

and checking the LLDP neighbors on ethernet-1/1 interface

```
show /system lldp neighbor interface ethernet-1/1
```

The expected output should be:

```
--{ running }--[  ]--
A:srl# show /system lldp neighbor interface ethernet-1/1
  +----------+----------+---------+---------+---------+---------+---------+
  |   Name   | Neighbor | Neighbo | Neighbo | Neighbo | Neighbo | Neighbo |
  |          |          |    r    |    r    | r First | r Last  | r Port  |
  |          |          | System  | Chassis | Message | Update  |         |
  |          |          |  Name   |   ID    |         |         |         |
  +==========+==========+=========+=========+=========+=========+=========+
  | ethernet | 00:1C:73 | ceos    | 00:1C:7 | 20      | 16      | Etherne |
  | -1/1     | :46:95:5 |         | 3:46:95 | hours   | seconds | t1      |
  |          | C        |         | :5C     | ago     | ago     |         |
  +----------+----------+---------+---------+---------+---------+---------+
```

## Listing running labs

When you are in the directory that contains the lab file, you can list the nodes of that lab simply by running:

```bash
[*]─[rd-13]─[~/ac1-workshop/10-basics]
└──> sudo containerlab inspect
INFO[0000] Parsing & checking topology file: basic.clab.yml
+---+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
| # |      Name       | Container ID |         Image         |     Kind      |  State  |  IPv4 Address  |     IPv6 Address     |
+---+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
| 1 | clab-basic-ceos | c279d892ea22 | ceos:4.32.0F          | arista_ceos   | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
| 2 | clab-basic-srl  | 7c46eb454f51 | ghcr.io/nokia/srlinux | nokia_srlinux | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
+---+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
```

If the topology file is located in a different directory, you can specify the path to the topology file:

```bash
[*]─[rd-13]─[/tmp]
└──> sudo containerlab inspect -t ~/ac1-workshop/10-basics/
INFO[0000] Parsing & checking topology file: basic.clab.yml
+---+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
| # |      Name       | Container ID |         Image         |     Kind      |  State  |  IPv4 Address  |     IPv6 Address     |
+---+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
| 1 | clab-basic-ceos | c279d892ea22 | ceos:4.32.0F          | arista_ceos   | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
| 2 | clab-basic-srl  | 7c46eb454f51 | ghcr.io/nokia/srlinux | nokia_srlinux | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
+---+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
```

You can also list all running labs regardless of where their topology files are located:

```bash
[*]─[rd-13]─[~/ac1-workshop/10-basics]
└──> sudo containerlab inspect --all
+---+----------------+----------+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
| # |   Topo Path    | Lab Name |      Name       | Container ID |         Image         |     Kind      |  State  |  IPv4 Address  |     IPv6 Address     |
+---+----------------+----------+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
| 1 | basic.clab.yml | basic    | clab-basic-ceos | c279d892ea22 | ceos:4.32.0F          | arista_ceos   | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
| 2 |                |          | clab-basic-srl  | 7c46eb454f51 | ghcr.io/nokia/srlinux | nokia_srlinux | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
+---+----------------+----------+-----------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
```

The output will contain all labs and their nodes.

Shortcuts:

* `sudo clab ins` == `sudo containerlab inspect`
* `sudo clab ins -a` == `sudo containerlab inspect --all`

## Lab directory

Lab directory stores the artifacts generated by containerlab that are related to the lab:

* tls certificates
* startup configurations
* inventory files
* topology export json file
* bind bounted directories

To list the contents of the lab directory, run:

```
[*]─[rd-13]─[~/ac1-workshop/10-basics]
└──> tree -L 3 clab-basic/
```

## Destroying the lab

When you are done with the lab, you can destroy it:

```bash
sudo containerlab destroy -t basic.clab.yml
```

You finished the basics lab exercise!
