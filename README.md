# Self-hosted-vpn-gateway
A self-hosted WireGuard VPN that uses a cloud VPS as a relay to a Raspberry Pi exit node, enabling secure remote access and bypassing Carrier-Grade NAT.

# Self-Hosted VPN Gateway via Cloud VPS

## Project Overview
This project establishes a secure "VPN-to-home" solution that routes all internet traffic from a remote client (e.g., a laptop) through a home network via a Raspberry Pi. A central cloud VPS acts as a stable relay endpoint, allowing the system to bypass the limitations of Carrier-Grade NAT (CGNAT).

The solution provides secure, encrypted access to local network resources and makes the remote client appear as if it's on the home network, using its public IP address.

## Architecture
The setup consists of three core components that work together to create the secure tunnel:

* **Remote Client (macOS):** The traveling device that initiates the VPN connection. Its configuration routes all traffic (`0.0.0.0/0`) through the tunnel.
* **Central Hub (Cloud VPS):** A publicly accessible server running WireGuard in a Docker container. It acts as a simple, efficient router, forwarding encrypted packets between the two clients.
* **Exit Node (Raspberry Pi):** Located on the home network behind CGNAT. It receives the forwarded traffic from the remote client and uses `iptables` rules to perform Network Address Translation (NAT), sending the traffic out to the local network and the internet.



## Tech Stack
* **VPN:** WireGuard
* **Virtualization:** Docker & Docker Compose
* **Operating Systems:** Linux (Debian/Ubuntu on VPS & Pi), macOS
* **Networking:** `iptables` (Firewall, NAT, Forwarding), UFW, CGNAT Traversal

## Configuration Files
The repository contains the following anonymized configuration files:
* `vps/docker-compose.yml`: Defines the WireGuard Docker container service.
* `vps/wg0.conf`: The server configuration, defining the two clients as peers.
* `raspberry-pi/wg0.conf`: The exit node configuration, including the crucial `PostUp`/`PostDown` `iptables` rules for NAT (`MASQUERADE`).
* `client-mac/wg0.conf`: The remote client configuration, set to tunnel all internet traffic.

## Challenges & Solutions
This project involved overcoming several complex technical challenges, demonstrating advanced troubleshooting skills:

* **Challenge:** Encrypted packets from clients arrived at the VPS host but were not being forwarded to the WireGuard Docker container.
* **Solution:** Using `tcpdump` on both the public (`eth0`) and Docker bridge (`docker0`) interfaces, I proved that the host's firewall (UFW) was dropping the packets. The issue was resolved by adding a specific `iptables` rule to the `DOCKER-USER` chain, which is the correct way to manage firewall exceptions for Docker, bypassing UFW conflicts.

* **Challenge:** The WireGuard Docker container failed to start or would ignore its `wg0.conf` file, despite the file being correctly mounted.
* **Solution:** After verifying the file path and permissions, I concluded the issue was a silent parsing error caused by invisible characters or incorrect text formatting in the config file. The problem was solved by deleting the file and recreating it cleanly using a `heredoc` (`cat << EOF`) shell command, which guarantees a pure text file without metadata.
