---
title: "Quickstart Docker Swarm"
date:
  created: 2017-01-24

linktitle: "Quickstart Docker Swarm"
slug: "quickstart-docker-swarm"

description: "Quick introduction into the docker swarm mode with some examples"

tags:
- RaspberryPi
- Docker
- Quickstart
- "#HowTo"

authors:
- harry
---
![Image Description](../images/20170124-docker_swarm_logo.png)

## What is Docker Swarm Mode?

> A swarm is a cluster of Docker engines, or nodes, where you deploy services.

### What is a node?

> ***A node is an instance of the Docker engine participating in the swarm. You can also think of this as a Docker node***.

>You can run one or more nodes on a single physical computer or cloud server, but production swarm deployments typically include Docker nodes distributed across multiple physical and cloud machines.

That means I can create a Docker Swarm with my '6-Node Cluster of Raspberries'.

<!-- more -->

### More terms and definitions

> When you deploy the service to the swarm, the swarm manager accepts your service definition as the desired state for the service.

>Then it schedules the service on nodes in the swarm as one or more replica tasks.

>The tasks run independently of each other on nodes in the swarm.

See this picture to understand this better

![Docker Services Diagram](https://docs.docker.com/engine/swarm/images/services-diagram.png)

## Lets get started

### Create a swarm

Be sure this node is not already part of a swarm

```sh
$ docker info |grep Swarm
Swarm: inactive
```

On the first node (the manager node)

```sh
$ docker swarm init --advertise-addr 192.168.1.161
Swarm initialized: current node (lqj66j634438rg5u9w342erjd) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-4vnoa0plx8rqwp6g8o2jj82k1tfezgjvbiq44pnow4kzq9ss83-dyho8y63twklyy81dkwg1ix2f \
    192.168.1.161:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Verify

```sh
$ docker info
...
Swarm: active
 NodeID: lqj66j634438rg5u9w342erjd
 Is Manager: true
 ClusterID: u4rwd7q50timcvqwin7vfkgq3
 Managers: 1
 Nodes: 1
...
```

```sh
docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
lqj66j634438rg5u9w342erjd *  pi1       Ready   Active        Leader
```

### Add nodes to the swarm

You can imagine which command you enter now on all the other nodes ;-)

Example:

```sh
$ docker swarm join \
>     --token SWMTKN-1-4vnoa0plx8rqwp6g8o2jj82k1tfezgjvbiq44pnow4kzq9ss83-dyho8y63twklyy81dkwg1ix2f \
>     192.168.1.161:2377
This node joined a swarm as a worker.
```

```sh
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
gk000yb0w1ui192jiq58p0mwq    pi2       Ready   Active
l7b32egoj0nxo673gqulmt9kl    pi3       Ready   Active
lqj66j634438rg5u9w342erjd *  pi1       Ready   Active        Leader
vvg9f81rjtqm0umbl7dckdwl5    pi6       Ready   Active
yurf21emabz6jxn6fsdrrixcl    pi4       Ready   Active
zthgrxmw5u447p0p929k5ozx4    pi5       Ready   Active
```

### Promote more nodes

If the swarm looses the manager node (Leader), your services will continue to run, but you will need to create a new cluster to recover.

> To take advantage of swarm mode’s fault-tolerance features, Docker recommends you implement an **odd** number of nodes.

Note to myself: I need an additional Pi ;-)

> Docker recommends a maximum of seven manager nodes for a swarm.

> **Important Note**: Adding more managers does NOT mean increased scalability or higher performance. In general, the opposite is true.

> An N manager cluster will tolerate the loss of at most (N-1)/2 managers.

I'm adding here two additional managers

```sh
$ docker node promote pi3
Node pi3 promoted to a manager in the swarm.
$ docker node promote pi5
Node pi5 promoted to a manager in the swarm.
```

```
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
gk000yb0w1ui192jiq58p0mwq    pi2       Ready   Active
l7b32egoj0nxo673gqulmt9kl    pi3       Ready   Active        Reachable
lqj66j634438rg5u9w342erjd *  pi1       Ready   Active        Leader
vvg9f81rjtqm0umbl7dckdwl5    pi6       Ready   Active
yurf21emabz6jxn6fsdrrixcl    pi4       Ready   Active
zthgrxmw5u447p0p929k5ozx4    pi5       Ready   Active        Reachable
```

### Remove a node from the swarm

#### Example: Node6 should leave the swarm

On node6:
```sh
$ docker swarm leave
Node left the swarm.
```

On node1:

```sh
$  docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
gk000yb0w1ui192jiq58p0mwq    pi2       Ready   Active
l7b32egoj0nxo673gqulmt9kl    pi3       Ready   Active        Reachable
lqj66j634438rg5u9w342erjd *  pi1       Ready   Active        Leader
vvg9f81rjtqm0umbl7dckdwl5    pi6       Down    Active
yurf21emabz6jxn6fsdrrixcl    pi4       Ready   Active
zthgrxmw5u447p0p929k5ozx4    pi5       Ready   Active        Reachable
```

```sh
$ docker node rm pi6
pi6
```

```sh
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
gk000yb0w1ui192jiq58p0mwq    pi2       Ready   Active
l7b32egoj0nxo673gqulmt9kl    pi3       Ready   Active        Reachable
lqj66j634438rg5u9w342erjd *  pi1       Ready   Active        Leader
yurf21emabz6jxn6fsdrrixcl    pi4       Ready   Active
zthgrxmw5u447p0p929k5ozx4    pi5       Down    Active        Reachable
```

#### Example: A manager node (node5) should leave the swarm

On node5

```sh
$ docker swarm leave
Error response from daemon: You are attempting to leave the swarm on a node that is participating as a manager. The only way to restore a swarm that has lost consensus is to reinitialize it with `--force-new-cluster`. Use `--force` to suppress this message.
```

You have two options here

##### 1. Use the `--force` option

```sh
$ docker swarm leave --force
Node left the swarm.
```

```sh
docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
gk000yb0w1ui192jiq58p0mwq    pi2       Ready   Active
l7b32egoj0nxo673gqulmt9kl    pi3       Ready   Active        Reachable
lqj66j634438rg5u9w342erjd *  pi1       Ready   Active        Leader
yurf21emabz6jxn6fsdrrixcl    pi4       Ready   Active
zthgrxmw5u447p0p929k5ozx4    pi5       Down    Active
```

Now you can remove node5 with `docker node rm pi5`

##### 2. Downgrade or ***demote*** the node first

On node1

```sh
$ docker node demote pi5
Manager pi5 demoted in the swarm.
```

```sh
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
gk000yb0w1ui192jiq58p0mwq    pi2       Ready   Active
l7b32egoj0nxo673gqulmt9kl    pi3       Ready   Active        Reachable
lqj66j634438rg5u9w342erjd *  pi1       Ready   Active        Leader
vvg9f81rjtqm0umbl7dckdwl5    pi6       Down    Active
yurf21emabz6jxn6fsdrrixcl    pi4       Ready   Active
zthgrxmw5u447p0p929k5ozx4    pi5       Ready   Active
```

```sh
$ docker node rm pi5
pi5
```

```sh
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
gk000yb0w1ui192jiq58p0mwq    pi2       Ready   Active
l7b32egoj0nxo673gqulmt9kl    pi3       Ready   Active        Reachable
lqj66j634438rg5u9w342erjd *  pi1       Ready   Active        Leader
yurf21emabz6jxn6fsdrrixcl    pi4       Ready   Active
```

### Destroy a swarm

If the last manager node leaves the swarm, it will destroy the swarm.

```sh
$ docker swarm leave --force
```

## Links

- https://docs.docker.com/swarm/
-https://docs.docker.com/swarm/overview/
-https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/
-https://docs.docker.com/engine/swarm/admin_guide/
