# Web Server

## Overview

The web server is a Linux server that hosts this documentation and is accessible from outside or inside the enterprise's private local network.

This website is static and generated with [Jetbrains Writerside](https://lp.jetbrains.com/writerside/), a plugin used to make documentation.

## Configuration

The static HTML, CSS and JS files are served using `caddy` as a server.

- as a server, `caddy` listens to a specific port and answers requests sending the corresponding ressources (web pages in our case);
- it also generates certificates (auto-signed or using Let's Encrypt, depending on the domain name specified in the configuration).

## Server

The server listens to port `80` and `443` and serves files present in `/var/www/infraSI/`, where the static files for this documentation are located.
All `HTTP` requests are redirected to `HTTPS` for security purposes.

![Caddyfile.png](Caddyfile.png)