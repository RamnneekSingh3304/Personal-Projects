pfSense Virtual Network Lab — Project Overview

This project simulates a small but realistic enterprise network using pfSense inside a virtual lab environment. The main objective is to build a functional setup that includes routing, DHCP, DNS, VLAN segmentation, firewall rules, and a remote-access VPN—similar to what you would find in an actual business network.

By using pfSense as the core device, the lab acts as both a router and a firewall, while the virtual machines serve as client devices and separate network segments.

Overview of the Lab
1. pfSense Installation

I started by creating a pfSense virtual machine and completing the installation so it could act as the main network gateway. This provided the foundation for routing, addressing, and security policies across the virtual environment.

2. DHCP and DNS Configuration

pfSense was configured to automatically assign IP addresses to connected devices and manage name resolution. This replicated how enterprise networks centrally manage addressing and DNS for multiple users and systems.

3. VLAN Segmentation

I created separate VLANs to divide the network into isolated subnets, each with its own IP range and rules.
Since VirtualBox does not provide a real managed switch, I simulated switch behavior by creating tagged sub-interfaces (e.g., eth0.10) on the client VM. This allowed me to test VLAN isolation, routing behavior, and inter-VLAN communication in a controlled environment.

4. Firewall Rules

Custom firewall rules were added to control which networks could communicate with each other.
This demonstrated how segmentation improves security by restricting unnecessary access between different VLANs and the main LAN.

5. OpenVPN Setup

Finally, I configured an OpenVPN server on pfSense to allow remote devices to connect securely. Testing the connection from my Windows host confirmed that the VPN worked end-to-end and provided full access to the internal network.

Summary

This project demonstrates core enterprise networking concepts—including IP addressing, routing, VLANs, firewalls, and VPN access—all within a virtualized environment. It provides solid hands-on experience with how real organizations design, segment, and secure their networks using tools similar to pfSense.
