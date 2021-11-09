[![Docker Pulls](https://badgen.net/docker/pulls/bfg100k/zerotier-gateway)](https://hub.docker.com/r/bfg100k/zerotier-gateway)
[![Github All Releases](https://img.shields.io/github/downloads/bfg100k/zerotier-gateway/total.svg)](https://github.com/bfg100k/zerotier-gateway/pkgs/container/zerotier-gateway)
[![Multiarch build](https://github.com/bfg100k/zerotier-gateway/actions/workflows/multiarch.yml/badge.svg)](https://github.com/bfg100k/zerotier-gateway/actions/workflows/multiarch.yml)

## zerotier-gateway

#### Description

This is a container based on a lightweight Alpine Linux image and a copy of ZeroTier One. It's designed to function as a gateway between your local network and the ZT network(s). Specifically, you can configure it as 
* inbound gateway - i.e. accessing resources in the local network from nodes in the ZT network(s) only.
* outbound gateway - i.e. accessing resources in the ZT network(s) from nodes in the local network only.
* bidirectional gateway - i.e. nodes in either networks (ZT or local) can access each other.

Additionally, you can also disable the gateway function and use this container to deploy ZeroTier as a service in your host via the host network interface. 

#### Run

To run this container in the correct way requires some special options to give it special permissions and allow it to persist its files. Here's an example (tested on Ubuntu 20.04 LTS):

    docker run 
      --name zerotier-gw \
      -v /var/lib/zerotier-one:/var/lib/zerotier-one \
      -e NETWORK_IDS=[ZT_NETWORK_ID1;ZT_NETWORK_ID2] \
      -e GATEWAY_MODE=[inbound|outbound|both] \
      --cap-add=NET_ADMIN \
      --net=net_LAN_ETH0 \
      --ip=172.17.50.137 \
      bfg100k/zerotier-gateway

This runs bfg100k/zerotier-gateway in a container exposed on the physical network (i.e. it has its own IP on the local network) via a macvlan interface (preferred method although host networking works too). More details on macvlan network and docker can be found in the references section below. An example of how you can create a macvlan network in docker is shown below:

    docker network create \
      --driver macvlan \
      --gateway 172.17.50.129 \
      --subnet 172.17.50.128/28 \
      --ip-range 172.17.50.128/28 \
      -o parent=eth0 \
      net_LAN_ETH0

It also mounts /var/lib/zerotier-one to /var/lib/zerotier-one inside the container, allowing your service container to persist its state across restarts of the container itself. If you don't do this it'll generate a new identity every time. You can put the actual data somewhere other than /var/lib/zerotier-one if you want.

To join one or more zerotier networks, you can specify the network ids in the environment variable NETWORK_IDS (semi-colon delimited). 

To configure gateway mode, pass in the environment variable GATEWAY_MODE. You can choose between inbound (i.e. ZT -> Local), outbound (i.e. Local -> ZT) or both (i.e. Local <-> ZT). Note that for this to work, you may need to do a couple of additional tasks:

1) Make sure ip forwarding on the docker host is enabled before starting the container. (By default, docker will enable ip forwarding in the host when the docker engine starts so nothing to do here unless you have explicitly turned it off. In any case, the startup script will check for this and terminate the container if gateway mode is enabled but ip forwarding is disabled.) 
2) Create the necessary static route(s) in either the ZT network (inbound gateway) or the local network (outbound gateway) or both (bidirectional gateway). Settings for ZT network is done in the ZT Web admin console and in your home router/gateway admin console for your local network.

Note also that for now, gateway mode will only be configured for the first ZT network (if you have multiple ZT networks connected).


#### Source
https://github.com/bfg100k/zerotier-gateway

Forked from
https://github.com/zyclonite/zerotier-docker


#### References
1) Docker & macvlan interfaces - https://blog.scottlowe.org/2016/01/28/docker-macvlan-interfaces/
2) Turning on/off ip-forwarding in Linux - https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux
3) Static routes - https://www.ccnablog.com/static-routing/
