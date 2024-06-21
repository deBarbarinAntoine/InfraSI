# Firewall

The firewall is a simple computer with `pfSense` installed on it.

`pfSense` is a `FreeBSD` based software that is often used to power firewalls. This means that there are some hardware requirements involved:
> - the device must be powered by an **AMD64 CPU** (because ARM CPUs are not widely supported) and its network interfaces mustn't use **Realtek chipsets** (Intel chipset are recommended because of compatibility issues).
> - we need at least three network interfaces (**WAN**, **LAN** and **OPT1**).
> 
> `In our case, the hardware doesn't matter because it is installed on a VM.`
> 
{style="warning"}

## Description

The firewall has several uses in this architecture:
- **Packets filtering**: restrict access outwards and inwards using rules applying on specific ports and protocols.
- **DHCP (Dynamic Host Configuration Protocol)**: network management protocol used on Internet Protocol (IP) networks for automatically assigning IP addresses and other communication parameters to devices connected to the network using a client–server architecture.
- **DNS (Domain Name System)**: hierarchical and distributed name service that provides a naming system for computers, services, and other resources on the Internet or other Internet Protocol (IP) networks.
- **Router**: acts as a bridge between different networks (here, between the sub-LANs and the WAN).
- **NAT (Network Address Translation)**: method of mapping an IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device (here, rules for HTTP and HTTPS requests for the web server).
- **VPN (Virtual Private Network)**: creates a secure connection on a public network.
- **Monitoring and reports**: provides tools to monitor the traffic on the network and to generate detailed reports.
- **Bandwidth management**: controls and limits the bandwidth.

## Network interfaces

### LAN (Local Area Network) & OPT1

The LAN (and OPT1) interface is connected to our internal private networks. It is responsible for the communication between devices on those private networks.

- **IP Addresses**: LAN 10.0.0.1, OPT1 10.0.0.9
- **Description**: Gives access to internet to all devices connected to that interface and applies security rules to protect the local networks.

### WAN (Wide Area Network)

The WAN interface is connected to internet. It receives a public IP address from the ISP (Internet Service Provider). **On our VM, we configured this interface in `NAT`, so that it receives an IP address from the host, without having problems with the network configurations at YNOV**.

- **IP Address**: Attributed by DHCP (VM's host)
- **Description**: Manages the inward and outward traffic, applying the security rules to protect from outside threats.

### DMZ (Demilitarized Zone)

The DMZ interface is used to host services reachable from internet, isolating it from the local network (LAN) for security purposes.

- **IP Address**: 10.0.1.1
- **Description**: Hosts our webserver which documentation is publicly reachable and applies security rules to limit access to specific services.

## Filtering Rules

### LAN & OPT1

1. **Permit all outward traffic**: Authorizes all devices of the LAN and OPT1 networks to access to internet.
    ```
    Action : Pass
    Interface : LAN
    Source : LAN subnet
    Destination : any
    ```
    ```
    Action : Pass
    Interface : OPT1
    Source : OPT1 subnet
    Destination : any
    ```

2. **Block all unauthorized inward traffic**: By default, all inward traffic is blocked if it doesn't match a positive rule.

3. **Authorize access to the webserver through SSH from a specific client (defined by IP)**: Allows access to the webserver through SSH to manage or administrate it. The administrator is situated in the LAN network and the backup server is in the OPT1 network.
    ```
    Action : Pass
    Interface : LAN
    Protocol : SSH
    Source : LAN address (client's IP)
    Destination : DMZ address (webserver's IP)
    ```
    ```
    Action : Pass
    Interface : OPT1
    Protocol : SSH
    Source : OPT1 address (Backup Server's IP)
    Destination : DMZ address (Webserver's IP)
    ```

4. **Authorize internal traffic in the LAN's and OPT1's subnets**: Allows all devices on the LAN's subnets to communicate using HTTP, HTTPS, DNS and ICMP protocols.
    ```
    Action : Pass
    Interface : LAN
    Protocol : TCP/UDP
    Source : LAN subnet
    Destination : LAN subnet
    Destination Port : 80 (HTTP), 443 (HTTPS), 53 (DNS), ICMP
    ```
    ```
    Action : Pass
    Interface : OPT1
    Protocol : TCP/UDP
    Source : OPT1 subnet
    Destination : OPT1 subnet
    Destination Port : 80 (HTTP), 443 (HTTPS), 53 (DNS), ICMP
    ```

5. **Authorize the Linux Workstation and the Backup Server to access the webserver in the DMZ through SSH**: Allows the Linux Workstation and the Backup Server to connect to the webserver through SSH.
    ```
    Action : Pass
    Interface : LAN
    Protocol : TCP/UDP
    Source : Linux client IP
    Destination : DMZ server IP
    Destination Port : 22 (SSH)
    ```
    ```
    Action : Pass
    Interface : OPT1
    Protocol : TCP/UDP
    Source : Backup Server IP
    Destination : DMZ server IP
    Destination Port : 22 (SSH)
    ```

6. **Authorize LAN's subnets to access the website hosted in the webserver (DMZ)**: Allows all devices in the LAN's subnets to access the website hosted in the webserver (DMZ) through HTTP and HTTPS protocols.
    ```
    Action : Pass
    Interface : LAN
    Protocol : TCP
    Source : LAN subnet
    Destination : DMZ webserver IP
    Destination Port : 80 (HTTP), 443 (HTTPS)
    ```

### WAN

1. **Authorize the HTTP/HTTPS traffic from the WAN to the webserver in the DMZ**: Allows the HTTP and HTTPS traffic from the internet (WAN) to be redirected to the webserver in the DMZ.
    ```
    Action : Pass
    Interface : WAN
    Protocol : TCP
    Source : any
    Destination : WAN address
    Destination Port : 80 (HTTP), 443 (HTTPS)
    NAT : Redirect to DMZ webserver IP
    ```
   
2. **Block all inward traffic**: By default, all inward traffic is blocked except the VPN connections or the ones specified before.
    ```
    Action : Block
    Interface : WAN
    Source : any
    Destination : WAN address
    ```

### DMZ

1. **Authorize HTTP/HTTPS traffic**: Allows access to the webserver in the DMZ through HTTP and HTTPS protocols.
    ```
    Action : Pass
    Interface : DMZ
    Protocol : TCP
    Source : any
    Destination : DMZ address
    Destination Port : 80 (HTTP) ou 443 (HTTPS)
    ```

2. **Block access to the LAN and OPT1 from the DMZ**: Prevents the webserver to access the LAN and OPT1 networks.
    ```
    Action : Block
    Interface : DMZ
    Source : DMZ net
    Destination : LAN net
    ```
    ```
    Action : Block
    Interface : DMZ
    Source : DMZ net
    Destination : OPT1 net
    ```

3. **Block the traffic between DMZ's subnets and LAN's/OPT1's subnets**: Prevents all communication between devices in the DMZ's subnets and those in the LAN's and OPT1's subnets.
    ```
    Action : Block
    Interface : DMZ
    Source : DMZ subnet
    Destination : LAN subnet
    ```
    ```
    Action : Block
    Interface : DMZ
    Source : DMZ subnet
    Destination : OPT1 subnet
    ```

## Conclusion

The **pfSense** firewall plays a crucial role in the security of our network infrastructure controlling all traffic between its interfaces (WAN, LAN, OPT1, DMZ) applying security rules that we chose and just presented.

---

This documentation provides a complete overview of the configurations of pfSense and its filtering rules.