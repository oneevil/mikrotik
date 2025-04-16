# Install BGP & DNS server on MikroTik

```
/interface bridge
add name=bridge-docker
/interface veth
add address=172.20.0.2/24,172.20.0.3/24,172.20.0.4/24 gateway=172.20.0.1 gateway6="" name=veth2-bgpdns
/interface list member
add interface=veth2-bgpdns list=LAN
/interface bridge port
add bridge=bridge-docker interface=veth2-bgpdns
/ip address
add address=172.20.0.1/24 interface=bridge-docker network=172.20.0.0
/container envs
add key=DNSTAP name=bgpdns value=172.20.0.3
add key=DNSTAP_SECONDIP name=bgpdns value=172.20.0.4
add key=UPSTREAM name=bgpdns value=dns.google
add key=UPSTREAM_IP name=bgpdns value=8.8.8.8
add key=BIRD name=bgpdns value=172.20.0.2
add key=FALLBACK_UPSTEAM name=bgpdns value=1
add key=SECOND_TUNNEL name=bgpdns value=1
add key=ROUTE name=bgpdns value=192.168.50.1
add key=ROUTE_SECOND name=bgpdns value=192.168.60.1
add key=TTL name=dns value=720h
/container mounts
add dst=/data name=storage src=/storage
/container
add envlist=bgpdns remote-image=oneevil/dns-server:latest interface=veth2-bgpdns logging=yes \
    mounts=storage root-dir=containers/dns start-on-boot=yes workdir=/
/routing bgp connection
add as=65010 disabled=no local.address=172.20.0.1 .role=ebgp multihop=yes \
    name=bgp remote.address=172.20.0.2/32 .as=64515 .port=179 router-id=\
    172.20.0.1 routing-table=main
/ip dns
set servers=172.20.0.2

```

## Network Configuration Parameters

```text
add address=172.20.0.2/24,172.20.0.3/24,172.20.0.4/24 gateway=172.20.0.1 gateway6="" name=veth2-bgpdns
```

The IP addresses assigned to the interface (comma-separated). Three IP addresses are required if multiple routes are configured, else if you use one routes needed two IP addresses. This allows different `next-hop` addresses to be set for each list of domains.

> **Note:** Multiple IP addresses are needed for configuring multiple `next-hop` addresses for different domain lists when using multiple routes.

Below are the environment variables used to configure the network settings:

| Variable           | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| `DNSTAP`           | The first IP address of the local DNS server configured with `dns-server`. |
| `DNSTAP_SECONDIP`  | The second IP address of the local DNS server with `dns-server` (optional). |

> **Note:** Parameters marked as optional can be omitted if you are using a single route.

## DNS Configuration Parameters

Below are the key-value pairs used to configure the DNS container:

| Key                  | Value Example       | Description                                                                 |
|----------------------|---------------------|-----------------------------------------------------------------------------|
| `UPSTREAM`           | dns.google          | The domain name of the upstream DNS provider (optional).                   |
| `UPSTREAM_IP`        | 8.8.8.8             | The IP address of the upstream DNS provider (optional).                    |
| `BIRD`               | 172.20.0.2          | IP address used for the BGP routing daemon (`bgp-server`).                 |
| `FALLBACK_UPSTEAM`   | 1                   | Enables fallback to secondary upstream DNS (`1` = enabled).                |
| `SECOND_TUNNEL`      | 1                   | Enables second tunnel interface (`1` = enabled).                           |
| `ROUTE`              | 192.168.50.1        | The IP address of the gateway used for routing.                            |
| `ROUTE_SECOND`       | 192.168.60.1        | Secondary gateway IP for routing (used if multiple routes are configured). |
| `TTL`                | 720h                | Storage time of the route received through the domain name (optional).     |

> **Note:** Parameters marked as optional can be omitted if you are using a single route.

## Container Mounts

The container includes the following mount configuration:

```
add dst=/data name=storage src=/storage
```

This mount point (`/data`) contains two folders:

- `static`
- `static_second`

Each of these folders contains `.txt` files with static routing rules.

### Example Route File (`.txt`)

```txt
route 64.233.165.94/32 reject;
route 35.207.188.57/32 reject;
route 35.207.81.249/32 reject;
route 35.207.171.222/32 reject;
route 195.62.89.0/24 reject;
route 66.22.192.0/18 reject;
route 64.71.8.96/29 reject;
route 34.0.240.0/22 reject;
route 12.129.184.160/29 reject;
```

> **Note:** You can include multiple `.txt` files in each folder. All valid route entries will be processed automatically.

This mount point (`/data`) contains two files:

- `domains.txt`
- `domains_second.txt`

### Example Domains File (`.txt`)

```txt
example.com
example1.com
```

## Applying Changes to Static Routes

After modifying the files in the `static` and/or `static_second` folders, it is necessary to apply the changes for them to take effect.

### Options for Applying Changes

1. **Restart the container:**
   - You can restart the container to apply the changes.
   - This method will reload all configurations, but it might be slower and interrupt active connections.

2. **Using the container's shell (recommended):**
   - A more efficient approach is to directly apply the changes using the container's shell.
   - Access the container via the terminal and execute the following command:

   ```bash
   /container shell 0
   # birdc configure

## Applying Changes to DNS Routes

1. **Restart the container:**
   - You can restart the container to apply the changes.