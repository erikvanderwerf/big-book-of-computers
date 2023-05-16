# NGINX
```bash
pacman -S nginx-mainline

systemctl enable --now nginx
ufw allow "WWW Full"
ufw delete allow "WWW Full"
```

## SSL Configuration

Reference [https](https.md) to generate an SSL certificate, then reference the
[Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) to create
an appropriate NGINX configuration file to `include`.

## Non-Terminating SSL Routing via SNI

This example will use a `stream` directive as the receiver for SSL (443) traffic, and use SNI
to inspect the TCP connection in order to transparently proxy to another host.

`/etc/nginx/nginx.conf`
```nginx.conf
user                    http;
worker_processes        auto;
worker_cpu_affinity     auto;
worker_rlimit_nofile    8196;

events {
    multi_accept        on;
    worker_connections  1024;
}

http {
    include         mime.types;
    default_type    application/octet-stream;
    
    sendfile            on;
    tcp_nopush          on;
    keepalive_timeout   65;
    gzip                on;
    types_hash_max_size 4096;
    
    include sites-enabled/*;
}

stream {
    log_format stream   '$remote_addr [$time_local] $ssl_preread_server_name $name';
    access_log          /var/log/stream.log stream;
    
    map $ssl_preread_server_name $name {
        <sni_domain>     <upstream>;
    }
    
    upstream <upstream> {
        server <address>;
    }
    
    server {
        listen      443;
        proxy_connect_timeout   1s;
        proxy_timeout           3s;
        proxy_pass              $name;
        ssl_preread             on;
}
```

Configure NGINX to serve clients that connect by IP address to the server an informative error.

`/etc/nginx/sites-available/by-ip`
```nginx.conf
server {
    listen 80 default_server;
    root /www/data;
    location / {
        try_files /index.html /index.html;
    }
}
```

Use symlinks to mark the available site as enabled.

```bash
ln -s /etc/nginx/sites-available/by-ip /etc/nginx/sites-enabled/by-ip
systemctl reload nginx
```

If NGINX will also be serving real domains (and not just proxy'ing) then create a new server block.

`/etc/nginx/sites-available/domain`
```nginx.conf
server {
    server_name <domain>;
    
    location / {
        root    <path>;
        index   <path>;
    }
    
    listen 8443 ssl;
    ssl_certificate     /etc/letsencrypt/live/.../fullchain.pem;
    ssl_vertificate_key /etc/letsencrypt/live/.../privkey.pem;
    include             /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam         /etc/letsencrypt/ssl-dhparams.pem;
}
```

Listen on a different port than 443 to allow the stream block to handle all incoming traffic
initially.
Configure this HTTP server as an `upstream` pointing to `localhost:8443` to appropriately
direct traffic.

# GoAccess
```bash
pacman -S goaccess
```

When used with the above NGINX configuration access log format, GoAccess should be configured as such.

`/etc/goaccess/goaccess.conf`
```goaccess.conf
datetime-format %d/%b/%Y:%H:%M:%S %z
```

GoAccess can then be run. Ensure the appropriate port can be reached by your browser.
```bash
goaccess /var/log/nginx/access.log                                 # Create site HTML.
goaccess --real-time-html --port 7890 /var/log/nginx/access.log    # Create site HTML that dynamically updates when opened.
```
