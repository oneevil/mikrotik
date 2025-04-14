# Install DNS server on MikroTik

```
/interface bridge
add name=bridge-docker
/interface veth
add address=172.55.0.3/24,172.55.0.4/24 gateway=172.55.0.1 gateway6="" name=veth2-dns
/interface list member
add interface=veth2-dns list=LAN
/interface bridge port
add bridge=bridge-docker interface=veth2-dns
/ip address
add address=172.55.0.1/24 interface=bridge-docker network=172.55.0.0
/container
add envlist=dns remote-image=oneevil/dns-server:latest interface=veth2-dns logging=yes mounts=storage root-dir=containers/dns start-on-boot=yes workdir=/
/container envs
add key=UPSTREAM name=dns value=dns.google
add key=UPSTREAM_IP name=dns value=8.8.8.8
add key=BIRD name=dns value=172.55.0.2
add key=FALLBACK_UPSTEAM name=dns value=1
add key=SECOND_TUNNEL name=dns value=1
add key=ROUTE name=dns value=192.168.50.1
add key=ROUTE_SECOND name=dns value=192.168.60.1
/container mounts
add dst=/data name=storage src=/storage

```

## Network Configuration Parameters

```text
add address=172.55.0.3/24,172.55.0.4/24 gateway=172.55.0.1 gateway6="" name=veth2-dns
```

The IP addresses assigned to the interface (comma-separated). Multiple IP addresses are required if multiple routes are configured. This allows different `next-hop` addresses to be set for each list of domains.

> **Note:** Multiple IP addresses are needed for configuring multiple `next-hop` addresses for different domain lists when using multiple routes.

## DNS Configuration Parameters

Below are the key-value pairs used to configure the DNS container:

| Key                  | Value Example       | Description                                                                 |
|----------------------|---------------------|-----------------------------------------------------------------------------|
| `UPSTREAM`           | dns.google          | The domain name of the upstream DNS provider (optional).                   |
| `UPSTREAM_IP`        | 8.8.8.8             | The IP address of the upstream DNS provider (optional).                    |
| `BIRD`               | 172.55.0.2          | IP address used for the BGP routing daemon (`bgp-server`).                 |
| `FALLBACK_UPSTEAM`   | 1                   | Enables fallback to secondary upstream DNS (`1` = enabled).                |
| `SECOND_TUNNEL`      | 1                   | Enables second tunnel interface (`1` = enabled).                           |
| `ROUTE`              | 192.168.50.1        | The IP address of the gateway used for routing.                            |
| `ROUTE_SECOND`       | 192.168.60.1        | Secondary gateway IP for routing (used if multiple routes are configured). |

> **Note:** Parameters marked as optional can be omitted if you are using a single route.

## Container Mounts

The container includes the following mount configuration:

```
add dst=/data name=storage src=/storage
```

This mount point (`/data`) contains two files:

- `domains.txt`
- `domains_second.txt`

### Example Domains File (`.txt`)

```txt
example.com
example1.com
```
