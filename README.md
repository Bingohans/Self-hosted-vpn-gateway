Self-Hosted VPN Gateway
Project Overview
This project establishes a secure "VPN-to-home" solution that routes all internet traffic from a remote client (e.g., a laptop) through a home network via a Raspberry Pi, bypassing Carrier-Grade NAT (CGNAT). A cloud VPS acts as a relay, enabling secure, encrypted access to local network resources (e.g., Pi-hole) and making the client appear as if it's on the home network, using its public IP.

The setup demonstrates advanced DevOps skills in networking, containerization, and firewall management, with a focus on solving real-world challenges like CGNAT traversal.

Architecture
The system consists of three components:

Remote Client (macOS): Initiates the VPN connection, routing all traffic (0.0.0.0/0) through the tunnel.

Central Hub (Cloud VPS): A publicly accessible server running WireGuard in a Docker container. It forwards encrypted packets between the client and Pi.

Exit Node (Raspberry Pi): Located behind CGNAT, it receives traffic, applies NAT via iptables (MASQUERADE), and forwards it to the local network/internet.

See the network diagram for a visual representation.

Tech Stack
VPN: WireGuard

Virtualization: Docker & Docker Compose

Operating Systems: Debian/Ubuntu (VPS, Pi), macOS (client)

Networking: iptables (Firewall, NAT, Forwarding), UFW, CGNAT Traversal

Setup Instructions
Prerequisites
A Raspberry Pi (4GB RAM recommended) behind CGNAT.

A cloud VPS with a public IP (e.g., Debian/Ubuntu, 1GB RAM minimum).

A macOS client with the WireGuard app installed.

Docker and Docker Compose installed on the VPS.

Installation Steps
VPS Setup:

Clone this repository: git clone <repo-url>

Navigate to config/vps/ and update wg0.conf with your keys and IPs.

Run scripts/setup-vps.sh to install Docker, configure WireGuard, and set up firewall rules.

Start the WireGuard container: docker-compose -f config/vps/docker-compose.yml up -d

Raspberry Pi Setup:

Copy config/raspberry-pi/wg0.conf to the Pi.

Run scripts/setup-pi.sh to install WireGuard, configure wg0.conf, and set up iptables for NAT.

Enable the WireGuard service: systemctl enable wg-quick@wg0

Client Setup:

Install the WireGuard app on macOS.

Import config/client-mac/wg0.conf into the app and activate the tunnel.

Verification
Run scripts/verify-config.sh on the VPS to check WireGuard status and connectivity.

Test access to Pi-hole or other local services from the client.

Configuration Files
config/vps/docker-compose.yml: Defines the WireGuard Docker service.

config/vps/wg0.conf: VPS WireGuard config, defining peers (Pi and client).

config/raspberry-pi/wg0.conf: Pi config with PostUp/PostDown iptables rules for NAT.

config/client-mac/wg0.conf: Client config to tunnel all traffic.

Challenges & Solutions
Challenge: Encrypted packets were dropped by the VPS firewall, preventing forwarding to the WireGuard container.

Solution: Used tcpdump on eth0 and docker0 to diagnose. Added an iptables rule to the DOCKER-USER chain to allow forwarding, bypassing UFW conflicts.

Challenge: The WireGuard container ignored wg0.conf due to parsing errors.

Solution: Recreated the config using a heredoc command (cat << EOF) to eliminate invisible characters and ensure clean formatting.

Challenge: CGNAT prevented direct access to the Pi.

Solution: Configured the VPS as a relay, using WireGuard’s lightweight protocol to forward traffic efficiently.

Lessons Learned
Advanced troubleshooting with tcpdump and iptables for Docker networking.

The importance of clean configuration files to avoid silent parsing errors.

Effective strategies for bypassing CGNAT using a relay server.

Future Improvements
Automate key generation and config distribution with Ansible.

Add Prometheus/Grafana to monitor VPN performance and traffic.

Implement failover to a secondary VPS for high availability.
