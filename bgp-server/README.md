# Install BGP server on MikroTik

```
/interface bridge
add name=bridge-docker
/interface veth
add address=172.55.0.2/24 gateway=172.55.0.1 gateway6="" name=veth1-bgp
/interface list member
add interface=veth1-bgp list=LAN
/interface bridge port
add bridge=bridge-docker interface=veth1-bgp
/ip address
add address=172.55.0.1/24 interface=bridge-docker network=172.55.0.0
/container
add envlist=bgp remote-image=oneevil/bgp-server:latest interface=veth1-bgp logging=yes mounts=unblock root-dir=containers/bgp start-on-boot=yes workdir=/
/container envs
add key=DNSTAP name=bgp value=172.55.0.3
add key=DNSTAP_SECONDIP name=bgp value=172.55.0.4
add key=ROUTE name=bgp value=192.168.50.1
add key=ROUTE_SECOND name=bgp value=192.168.60.1
/container mounts
add dst=/data name=unblock src=/unblock
/routing bgp connection
add as=65010 disabled=no local.address=172.55.0.1 .role=ebgp multihop=yes \
    name=bgp remote.address=172.55.0.2/32 .as=64515 .port=179 router-id=\
    172.55.0.1 routing-table=main
```

## Configuration Parameters

Below are the environment variables used to configure the network settings:

| Variable           | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| `DNSTAP`           | The first IP address of the local DNS server configured with `dns-server`. |
| `DNSTAP_SECONDIP`  | The second IP address of the local DNS server with `dns-server` (optional). |
| `ROUTE`            | The IP address of the gateway used for routing.                             |
| `ROUTE_SECOND`     | The second IP address of the gateway used for routing (optional).           |

> **Note:** Parameters marked as optional can be omitted if not needed.