# Startup configuration

This exercise demonstrates how to provide startup configuration to the lab nodes by means of the [`startup-config`](https://containerlab.dev/manual/nodes/#startup-config) node parameter in the topology file.

Startup configuration is a way to provide initial configuration to the lab nodes when they boot up. This is useful when you want to automate the configuration of the nodes and avoid manual intervention. It also brings your lab to a desired state when you need to test a specific scenario.

To enter the lab directory, run the following command from anywhere in your terminal:

```bash
[*]─[rd-13]─[~]
└──> cd ac1-workshop/15-startup/

[*]─[rd-13]─[~/ac1-workshop/15-startup]
└──>
```

We start by deploying a lab defined in the `startup.clab.yml` topology file. The lab consists of two nodes: `srl` (Nokia SR Linux) and `ceos` (Arista cEOS). Both nodes are configured with a startup configuration file that resides in the same directory as the topology file.

We will use the shortened syntax when deploying the lab; less typing and more fun!

```bash
sudo clab dep -c
```

> Note, that when calling `clab dep -c` the containerlab will try to find the `*.clab.yml` file in the current working directory. If the file is located elsewhere, you can specify the path to the file as an argument to the `clab dep` command.  
> The `-c` flag stands for `--cleanup` and it will ensure that if the lab is already running, it will be stopped and removed before deploying a new one.

The startup configuration files - [srl.cfg](srl.cfg) and [ceos.cfg](ceos.cfg) - configure the interfaces, IP addressing, loopbacks and BGP peering between SR Linux and cEOS nodes respectively.

In particular, the `srl` node is configured to announce its loopback address `10.10.10.1/32` towards the `ceos` node and the `ceos` node is configured to announce its loopback address `10.10.10.2/32` towards the `srl` node.

After the lab is deployed, we can expect that the nodes will boot up and apply the startup configuration snippets provided in the topology file. Consequently, it is fair to assume that the nodes will establish BGP peering and exchange routes.

Let's connect to the `clab-startup-srl` node and check the BGP peering status:

```bash
ssh clab-startup-srl
```

```
--{ running }--[  ]--
A:srl# show network-instance default protocols bgp neighbor 192.168.1.2
```

You should see 1 route sent/received for the aforementioned BGP neighbor.

Now, let's connect to the `clab-startup-ceos` node and make sure that it can reach the loopback address announced by the `srl` node.

```bash
ssh clab-startup-ceos
```

When in the EOS shell, issue a ping towards the `srl` node's loopback address:

```
ping 10.10.10.1
```

You should see a successful ping response.

You have successfully deployed the lab with the nodes equipped with the startup configuration. This is a powerful feature that can be used to provision the nodes with the desired configuration when they boot up.
