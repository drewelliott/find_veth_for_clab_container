# find_veth_for_clab_container

There are occasions when you may need to identify the veth attached to your docker container which is part of a containerlab topology.

There are two types of veth to consider.
1) Management veth (management connection from host to container)
2) Point-to-point veth (data path between containers)

In both cases, you will need to know the container id - you can get this either using docker or clab:

`docker ps`

`clab inspect -t <topology file>`

Next, we need to find the link number:

`docker exec -it <container id> ip link`

```
╭─drelliot@snoopy ~ 
╰─$ docker exec -it 959d3c1ec4f6 ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: dummy-mgmt0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 22:35:fe:d3:8e:24 brd ff:ff:ff:ff:ff:ff
3: gway-2800@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 4e:47:56:2f:bc:1a brd ff:ff:ff:ff:ff:ff link-netns srbase-mgmt
5: monit_in@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9234 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 7e:32:70:43:30:e8 brd ff:ff:ff:ff:ff:ff link-netns monit
6: mgmt0-0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 9a:8b:7b:fd:f1:ef brd ff:ff:ff:ff:ff:ff link-netns srbase-mgmt
    alias mgmt0.0
7: gway-2801@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 8a:64:a1:dd:48:29 brd ff:ff:ff:ff:ff:ff link-netns srbase-default
8: e1-1-0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ce:55:d0:c1:80:06 brd ff:ff:ff:ff:ff:ff link-netns srbase-default
    alias ethernet-1/1.0
9: e1-2-0@if5: <BROADCAST,MULTICAST> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether ea:f2:7e:85:ba:82 brd ff:ff:ff:ff:ff:ff link-netns srbase-default
    alias ethernet-1/2.0
570: mgmt0@if571: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1514 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:0a:0a:01:65 brd ff:ff:ff:ff:ff:ff link-netnsid 0
576: e1-1@if577: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9232 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 1a:22:01:ff:00:01 brd ff:ff:ff:ff:ff:ff link-netnsid 1
580: e1-2@if581: <BROADCAST,MULTICAST> mtu 9232 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 1a:22:01:ff:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 2
```
In the example, I have an SR Linux container with two point-to-point links and the management link. 

The management link is tied to the management bridge, so that is very straightforward to identify.

`awk '{print FILENAME,$0}' /sys/class/net/veth*/ifindex`

If you need to shut down one of the veth interfaces, you can follow these steps:

Find the container pid:
`docker inspect --format '{{.State.PID}}' <container id>`

Identify the interface:
`nsenter -t <container pid> -n ip link`

Modify the interface:
`nsenter -t <container pid> -n ip link set e1-1 down`
