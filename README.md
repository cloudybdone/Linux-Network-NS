# Linux Network Namespace Simulation
![Diagram of NS Simulation](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2014-37-00.png)


## **Step 1: Create Network Bridges**

Create two Linux bridges `br0` and `br1`:

```bash
sudo ip link add br0 type bridge
sudo ip link add br1 type bridge

# Bring the bridges up
sudo ip link set br0 up
sudo ip link set br1 up
```
![bridge interface](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-51-42.png)

---

## **Step 2: Create Network Namespaces**

Create the required network namespaces:

```bash
sudo ip netns add ns1
sudo ip netns add ns2
sudo ip netns add router-ns
```

Verify namespace creation:

```bash
ip netns list
```
![net ns list](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-52-15.png)
---

## **Step 3: Create Virtual Interfaces and Connections**

### **Create veth pairs**

Each veth pair acts as a virtual cable connecting the namespaces to bridges.

```bash
# veth for ns1 -> br0
sudo ip link add veth0 type veth peer name veth0-br
sudo ip link set veth0 netns ns1
sudo ip link set veth0-br master br0
sudo ip link set veth0-br up

# veth for ns2 -> br1
sudo ip link add veth1 type veth peer name veth1-br
sudo ip link set veth1 netns ns2
sudo ip link set veth1-br master br1
sudo ip link set veth1-br up

# veth for router-ns -> br0
sudo ip link add veth-r0 type veth peer name veth-br0
sudo ip link set veth-r0 netns router-ns
sudo ip link set veth-br0 master br0
sudo ip link set veth-br0 up

# veth for router-ns -> br1
sudo ip link add veth-r1 type veth peer name veth-br1
sudo ip link set veth-r1 netns router-ns
sudo ip link set veth-br1 master br1
sudo ip link set veth-br1 up
```
![virtual interfaces](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-53-20.png)
---

## **Step 4: Configure IP Addresses**

Define a subnet:

- `10.0.1.0/24` for `ns1`
- `10.0.2.0/24` for `ns2`

```bash
# Assign IPs inside ns1
sudo ip netns exec ns1 ip addr add 10.0.1.10/24 dev veth0
sudo ip netns exec ns1 ip link set veth0 up
sudo ip netns exec ns1 ip link set lo up

# Assign IPs inside ns2
sudo ip netns exec ns2 ip addr add 10.0.2.10/24 dev veth1
sudo ip netns exec ns2 ip link set veth1 up
sudo ip netns exec ns2 ip link set lo up

# Assign IPs in router-ns
sudo ip netns exec router-ns ip addr add 10.0.1.1/24 dev veth-r0
sudo ip netns exec router-ns ip addr add 10.0.2.1/24 dev veth-r1
sudo ip netns exec router-ns ip link set veth-r0 up
sudo ip netns exec router-ns ip link set veth-r1 up
sudo ip netns exec router-ns ip link set lo up
```
![virtual interfaces](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-56-24.png)

![virtual interfaces](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-56-54.png)
---

## **Step 5: Configure Routing**

Enable IP forwarding in `router-ns`:

```bash
sudo ip netns exec router-ns sysctl -w net.ipv4.ip_forward=1
```
![virtual interfaces](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-57-30.png)
Set default routes in `ns1` and `ns2` to the router:

```bash
sudo ip netns exec ns1 ip route add default via 10.0.1.1
sudo ip netns exec ns2 ip route add default via 10.0.2.1
```
![bridge interface](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-58-08.png)
![bridge interface](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-58-29.png)
---

## **Step 6: Test Connectivity**

Ping between namespaces:

```bash
# Test connectivity from ns1 to router
sudo ip netns exec ns1 ping -c 5 10.0.1.1

# Test connectivity from ns2 to router
sudo ip netns exec ns2 ping -c 3 10.0.2.1

# Test connectivity from ns1 to ns2 (should work if routing is correct)
sudo ip netns exec ns1 ping -c 5 10.0.2.10
```
![bridge interface](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-59-16.png)
![bridge interface](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2013-59-48.png)
![bridge interface](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2014-00-17.png)
![bridge interface](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2014-01-03.png)

# Enable Debugging with tcpdump
Run tcpdump inside router-ns to check if packets are being forwarded:

![bridge interface](https://github.com/cloudybdone/Linux-Network-NS/blob/main/Screenshot%20from%202025-02-10%2014-08-07.png)
---

## **Cleanup Script**

```bash
sudo ip netns del ns1
sudo ip netns del ns2
sudo ip netns del router-ns
sudo ip link del veth0-br
sudo ip link del veth1-br
sudo ip link del veth-br0
sudo ip link del veth-br1
sudo ip link del br0
sudo ip link del br1
```

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.


## Author

**Mohammed Salehuzzaman**\
 Sr.DevOps Engineer\
[GitHub](https://github.com/cloudybdone)

