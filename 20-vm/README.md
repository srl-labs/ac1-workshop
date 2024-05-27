# VM-based nodes in containerlab

VM nodes integration in containerlab is based on the [hellt/vrnetlab](https://github.com/hellt/vrnetlab) project which is a fork of `vrnetlab/vrnetlab` where things were added to make it work with the container networking.

Start with cloning the project:

```bash
cd ~ && git clone https://github.com/hellt/vrnetlab.git && \
cd ~/vrnetlab
```

## Building Cisco Catalyst 8000v image

Cisco c8000v qcow image is located at `~/images/c8000v-universalk9_16G_serial.17.11.01a.qcow2` and should be copied to the `~/vrnetlab/c8000v/` directory before building the container image.

```bash
cp ~/images/c8000v-universalk9_16G_serial.17.11.01a.qcow2 ~/vrnetlab/c8000v/
```

Once copied, we can enter in the `~/vrnetlab/c8000v` image and build the container image:

```bash
cd ~/vrnetlab/c8000v && make
```

Note, that c8000v image will run the VM during the container build time. This is to unpack the image once at build time, instead of doing it every time a container is started. That is why the build time is longer than for SR OS image.

While it is building the image, open another terminal window and proceed with building the SR OS image

The resulting image will be tagged as `vrnetlab/vr-c8000v:17.11.01a`.

## Building Nokia SR OS image

SR OS qcow image is located at `~/images/sros-vm-24.3.R1.qcow2` and should be copied to the `~/vrnetlab/sros/` directory before building the container image.

```bash
cp ~/images/sros-vm-24.3.R1.qcow2 ~/vrnetlab/sros/
```

Once copied, we can enter in the `~/vrnetlab/sros` image and build the container image:

```bash
cd ~/vrnetlab/sros && make
```

The image will be built and tagged as `vrnetlab/vr-sros:24.3.R1`. The tag is auto-derived from the file name.

## Deploying the VM-based nodes lab

With the images built, we can proceed with the lab deployment. First, let's switch back to the lab directory:

```bash
cd ~/ac1-workshop/20-vm
```

Now lets deploy the lab:

```bash
sudo clab dep -c
```

### Monitoring the boot process

You will notice that the lab deployment log will show the following message and the deployment process will appear stuck:

```
Waiting for clab-vm-sros to be ready. This may take a while. Monitor boot log with `sudo docker logs -f clab-vm-sros`
```

This is because containerlab waits for SR OS node to boot and become ready for containerlab to inject SSH keys to enabled passwordless SSH access. To monitor the boot process, you can open a new terminal and run the following command:

```bash
sudo docker logs -f clab-vm-sros
```

> the SR OS boot time is approx 3 minutes.

You will see the boot process of the SR OS node as it would have been seen in a terminal connected to the real hadware console.

The same `docker logs` command can be used for the `clab-vm-c8000v` node to monitor the boot log of the Cisco Catalyst 8000v node.

> The c8000v boot time is approx 7 minutes.

## Connecting to the nodes

To connect to SR OS node:

```
ssh clab-vm-sros
```

No user/password is needed, since username is injected by containerlab as well as the local public key for passwordless SSH access.

To connect to the c8000v node use `admin:admin` credentials:

```bash
ssh clab-vm-c8000v
```

## Configuring the nodes

While logged in in the SR OS CLI paste the following snippet to configure SR OS:

```
edit-config private
/configure port 1/1/c1 admin-state enable
/configure port 1/1/c1 connector breakout c1-100g
/configure port 1/1/c1/1 admin-state enable
/configure port 1/1/c1/1 description "port 1/1/c1/1"
/configure port 1/1/c1/1 ethernet mode hybrid
/configure port 1/1/c1/1 { ethernet lldp }
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge notification true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge port-id-subtype tx-if-name
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge receive true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge transmit true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs port-desc true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-name true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-desc true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-cap true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address oob admin-state enable
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address system admin-state enable

/configure router "Base" interface "toC8kv" { admin-state enable }
/configure router "Base" interface "toC8kv" { port 1/1/c1/1:0 }
/configure router "Base" interface "toC8kv" { ipv4 primary address 192.168.1.2 }
/configure router "Base" interface "toC8kv" { ipv4 primary prefix-length 24 }
commit
```

Now open the CLI session with Cisco c8000v node and paste the following configuration:

```
conf t
lldp run
interface GigabitEthernet2
ip address 192.168.1.1 255.255.255.0
lldp receive
lldp transmit
no shutdown
```

Now we configured the two systems to be able to communicate with each other and enabled LLDP on both sides. Perform a ping from SR OS towards the c8000v node and let it run:

```
[/]
A:admin@sros# ping 192.168.1.1 count 1000
PING 192.168.1.1 56 data bytes
64 bytes from 192.168.1.1: icmp_seq=1 ttl=255 time=50.4ms.
64 bytes from 192.168.1.1: icmp_seq=2 ttl=255 time=1.57ms.
```

## Exploring the VM networking setup

In a new terminal window open c8000v node's container shell:

```bash
docker exec -it clab-vm-c8000v bash
```

Explore the Linux link interfaces available in the container:

```bash
ip link
```

the `tap1` interface corresponds to the VM's interfaces and it is transparently "stitched" (with `tc mirred` rules)

You can sniff the traffic on both `eth1` and `tap1` interface, and see that they carry the same traffic, as they are merely two ends of the same virtual link.

```bash
tcpdump -nni tap1
```

The management interface doesn't use the `tapX` interface, but rather leverage the qemu user mode networking, where qemu is instructed to forward connections made to certain ports on the localhost to a special qemu-managed interface that represents the VM's management interface.

You can see those host forward rules created by vrnetlab, when you list the qemu process:

```bash
root@c8000v:/# ps auxw | grep qemu
root        18  100 12.7 4671104 4184560 pts/0 Sl+  Apr19 4252:52 qemu-system-x86_64 -enable-kvm -display none -machine pc -monitor tcp:0.0.0.0:4000,server,nowait -m 4096 -serial telnet:0.0.0.0:5000,server,nowait -drive if=ide,file=/c8000v-universalk9_16G_serial.17.11.01a-overlay-overlay-overlay.qcow2 -device pci-bridge,chassis_nr=1,id=pci.1 -device vmxnet3,netdev=p00,mac=0C:00:6b:ff:46:00 -netdev user,id=p00,net=10.0.0.0/24,tftp=/tftpboot,hostfwd=tcp::2022-10.0.0.15:22,hostfwd=udp::2161-10.0.0.15:161,hostfwd=tcp::2830-10.0.0.15:830,hostfwd=tcp::2080-10.0.0.15:80,hostfwd=tcp::2443-10.0.0.15:443 -device vmxnet3,netdev=p01,mac=0C:00:50:19:05:01,bus=pci.1,addr=0x2 -netdev tap,id=p01,ifname=tap1,script=/etc/tc-tap-ifup,downscript=no
```

Look for `hostfwd` options in the qemu command line. Those are the port forwarding rules that qemu uses to forward the connections to the VM's management interface. For example, as per the output above, the `hostfwd=tcp::2022-10.0.0.15:22` rule forwards connections made to the localhost:2022 to the VM's interface 10.0.0.15:22.

But what forwards the incoming SSH connections to this `localhost:2022` port? In case of c8000v node this is done by the `socat` processes that are launched by the vrnetlab:

```bash
root@c8000v:/# ps aux | grep socat
root        13  0.0  0.0  10304  4300 pts/0    S+   Apr19   0:00 socat TCP-LISTEN:22,fork TCP:127.0.0.1:2022
root        14  0.0  0.0  10304  2912 pts/0    S+   Apr19   0:00 socat UDP-LISTEN:161,fork UDP:127.0.0.1:2161
root        15  0.0  0.0  10304  2924 pts/0    S+   Apr19   0:00 socat TCP-LISTEN:830,fork TCP:127.0.0.1:2830
root        16  0.0  0.0  10304  2924 pts/0    S+   Apr19   0:00 socat TCP-LISTEN:80,fork TCP:127.0.0.1:2080
root        17  0.0  0.0  10304  2908 pts/0    S+   Apr19   0:00 socat TCP-LISTEN:443,fork TCP:127.0.0.1:2443
```

You can see that there are `socat` processes listening on the ports that are forwarded to special localhost ports and then finally forwarded to the VM's interface using the qemu's hostfwd rules.

Note, that some VM-based NOSes use iptables forwarding rules instead of socat processes. This is the case for Nokia SR OS.
