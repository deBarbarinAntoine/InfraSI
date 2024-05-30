# Firewall

The firewall is a simple computer with `pfSense` installed on it.

`pfSense` is an `FreeBSD` based software that is often used to power firewalls. This means that there are some hardware requirements involved:
> - the device must be powered by an **AMD64 CPU** (because ARM CPUs are not widely supported) and its network interfaces mustn't use **Realtek chipsets** (Intel chipset are recommended because of compatibility issues).
> - we need at least three network interfaces (**WAN**, **LAN** and **OPT1**).
> 
> `In our case, the hardware doesn't matter because it is installed on a VM.`
> 
{style="warning"}

## Description

The firewall has several uses in this architecture:
- restrict access outwards and inwards using rules applying on specific ports and protocols
- DHCP
- DNS
- act as a bridge between the two sub-LANs
- NAT rules for HTTP and HTTPS requests for the web server