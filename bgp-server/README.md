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
add envlist=bgp remote-image=oneevil/bgp-server:latest interface=veth1-bgp logging=yes mounts=storage root-dir=containers/bgp start-on-boot=yes workdir=/
/container envs
add key=DNSTAP name=bgp value=172.55.0.3
add key=DNSTAP_SECONDIP name=bgp value=172.55.0.4
add key=ROUTE name=bgp value=192.168.50.1
add key=ROUTE_SECOND name=bgp value=192.168.60.1
/container mounts
add dst=/data name=storage src=/storage
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