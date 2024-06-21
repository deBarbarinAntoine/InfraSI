# Web Server

## Overview

The web server is a Linux server that hosts this documentation and is accessible from outside or inside the enterprise's private local network.

This website is static and generated with [Jetbrains Writerside](https://lp.jetbrains.com/writerside/), a plugin used to make documentation.

## Configuration

The static HTML, CSS and JS files are served using `nginx` as a server and reverse-proxy.

- as a server, `nginx` listens to a specific port and answers requests sending the corresponding ressources (web pages in our case);
- as a reverse-proxy, it inspects the requests to see if it matches the domain or subdomain name specified in its configuration and if true, redirects it to a specific destination.

## Server

The server listens to port `8100` and serves files present in `/var/www/infraSI/`, where the static files for this documentation are located.

: `/etc/nginx/nginx.conf`:
    ```Bash
    ...
    server {
      listen localhost:8100;
      
      location / {
        root /var/www/infraSI/;
        index index.html;
        autoindex off;
      }
    }
    ...
    ```

## Reverse-proxy

The reverse-proxy listens to port `80` (HTTP port) and checks if there are requests to the `infrasi.adebarbarin.com` subdomain. If true, it redirects it to port `8100`, where the server is listening, allowing the client requesting this online documentation to access it.

: `/etc/nginx/sites-enabled/infrasi.adebarbarin.com.conf`:
    ```Bash
    server {
            listen 80;
    
            server_name infrasi.adebarbarin.com;
    
            location / {
                    proxy_pass http://localhost:8100/;
            }
    }
    ```