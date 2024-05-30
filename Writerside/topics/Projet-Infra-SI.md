# Projet Infra SI

Authors
: <img alt="person-icon" src="person-icon.svg" title="person-icon"/> Antoine de Barbarin
: <img alt="person-icon" src="person-icon.svg" title="person-icon"/> Nicolas Moyon
: <img alt="person-icon" src="person-icon.svg" title="person-icon"/> Sabrina Eloundou

In this project, we will describe and explain an IT solution for a little enterprise setting up a private local network connected to internet, in which there are 5 devices:

> - a firewall,
> - a web server,
> - a backup server,
> - a linux client,
> - a windows client.
>
{style="note"}

The **web server** hosts a web documentation of the enterprise's network solution and it should be reachable from inside or outside of the private local network.

The **backup server** needs to periodically save the web server's files so that it can be rolled back anytime to a previous pristine/stable/working version.

## Overview

![network-map](network-map.png "network-map")

There are few things in the graphic just shown:
: - the firewall is the only device directly connected to the exterior, to internet
: - the servers and client are on two differents subnets inside the enterprise's LAN
: - the two different sub-LANs are differentiated through the firewall's DHCP and the two physical LAN interfaces (LAN and OPT1 on pfSense)
: - each sub-LAN has a switch of its own and it can be extended if necessary
