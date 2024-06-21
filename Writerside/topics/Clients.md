# Clients

The clients are, as indicated in the presentation, two:
: - Linux Workstation
: - Windows Workstation.

## Linux Workstation

The Linux Workstation is a VM running Ubuntu 22.04 with a desktop environment (GNOME) and `ssh` installed (to access to the Backup Server and Webserver as an administrator).

It has a single network interface set at `VMnet1`, to match the firewall's **LAN interface**.

There are almost no configurations to do here, just to enable the network interface and set it to `automatic`, so that the firewall gives it an IP address on the **LAN network** (10.0.0.0 /29) with the **DHCP** service.

## Windows Workstation

The Windows Workstation is a VM running Windows 11.

As the Linux Workstation, it has a single network interface set at `VMnet1` to match the firewall's **LAN interface**.

Same as before, there is only the network to configure and set to `automatic` for it to receive an IP address from the firewall.