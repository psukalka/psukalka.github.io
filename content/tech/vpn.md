+++
date = '2025-07-08T18:04:41+05:30'
draft = false
title = 'VPN'
+++
We generally use VPN (Virtual Private Network) to test whether content piece is accessible from another country. Using VPN our servers are tricked as if the request is coming from another country. How does this happen ? 

On a simple level, when VPN app is running on my device it will send all my requests (app or browser requests) to the VPN server that is located in some another country. VPN server will then forward these requests to the destination server. Since the request has actually come from another country, destination server will respond. This response will then be forwarded back to me. Thus VPN has acted as a middle-man or proxy to send my request. 

So, is VPN server just a proxy that redirects my requests ? If I install Nginx then will it be a VPN server ? Not really. Even though on a high level what they are doing is similar the way they do it is entirely different. NGINX operates at layer 7 (i.e. application layer) protocols like HTTP, HTTPS, ... while VPN acts at layer 2 (Data Link Layer) or layer 3 (IP layer / network layer). In simple terms, NGINX can route packets based on request paths but VPN operates at IP address layer. It modifies IP address of packet and encrypts the content. It doesn't matter to VPN whether it is TCP (layer 4 packet), HTTP or websocket or any packet. It will operate at even lower layer and has control to change its IP address. 

So, if I start VPN on my linux machine and make some request from browser, then VPN client captures the packet, modifies its IP address, encrypts the contents (so that ISP or anyone can't sniff into it) and sends it to VPN server. VPN server then decrypts the packet, reads the original destination, attaches that destination IP on the packet and forwards it to the destination server. The destination server sees that the packet is valid and it responds. The response is received by VPN server. It again encrypts and forwards the response to my machine where it is finally received and decrypted. Interesting. 

But how is the VPN client able to take control of packets ? If I want to make my VPN client how can I make it ? 

First, the VPN client creates a virtual network interface. Then directs all the traffic to this interface. Once the VPN program receies packet it then attaches VPN server IP address to it and forwards it over regular network. 

On a Linux machine, the kernel provides `TUN/TAP` interfaces. `TUN` (Tunnel) operates at layer 3 (Netowrk layer) to handle IP packets and `TAP` (Terminal Access Point) operates at layer 2 (Data link layer) to handle ethernet frames. You can create a virtual network interface as follows:

```
# Create TUN Interface
sudo ip tuntap add dev myvpn mode tun

# Assign it an IP address 
sudo ip addr add 10.0.0.1/24 dev myvpn

# Bring it up
sudo ip link set myvpn up 

# Redirect all traffic through this interface
sudo ip route add default dev myvpn 
```

You can check available network interfaces using: 
```
ip addr show 
```

Route table can be checked with:
```
ip route show
```

When we create a `myvpn` network interface it creates a file descriptor at `/dev/net/tun` . You can now write C code to read this descriptor and tamper IP addresses on these packets and encryt their contents before sending it ahead. Similarly need to write decryption logic for received packets and eventually forwarding them to appropriate local port so that correct application receieves the packet. It should not happen that browser tab that was watching youtube now receieves hotstar packet. 

---

I ran the interface commands on an EC2 instance. There I saw `ens5` (main EC2 network interface), `lo` (for loopback interface that is used for inter-process communication) and `docker0` (docker bridge interface). So, when we install docker it creates a virtual network interface for inter-docker communication. I have been using it, but never thought about it until today.