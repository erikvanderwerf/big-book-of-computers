# NGINX
```bash
pacman -S nginx-mainline

systemctl enable --now nginx
ufw allow "WWW Full"
ufw delete allow "WWW Full"
```

Configure NGINX to log accesses and errors. Ensure both files exist beforehand or the NGINX service will not write to them.

`/etc/nginx/nginx.conf`
```nginx.conf
http {
    log_format vhosts '$host $remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
    
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    include sites-enabled/*;
}
```

Configure NGINX to serve multiple domains. Use HTTP initially and configure SSL later.

`/etc/nginx/sites-available/example.conf`
```nginx.conf
server {
    listen 80;
    server_name example.com
}
```

Use symlinks to mark the available site as enabled.

```bash
ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/example.conf
systemctl reload nginx
```

# GoAccess
```bash
pacman -S goaccess
```

When used with the above NGINX configuration access log format, GoAccess should be configured as such.

`/etc/goaccess/goaccess.conf`
```goaccess.conf
datetime-format %d/%b/%Y:%H:%M:%S %z
log-format %v %h %^[%x] “%r” %s %b “%R” “%u”
```

GoAccess can then be run. Ensure the appropriate port can be reached by your browser.
```bash
goaccess /var/log/nginx/access.log                                 # Create site HTML.
goaccess --real-time-html --port 7890 /var/log/nginx/access.log    # Create site HTML that dynamically updates when opened.
```
