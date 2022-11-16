# Certificates
```bash
pacman -S certbot certbot-nginx

certbot --nginx
```

# DDNS
DDNS continuously pushes the public IP address of the LAN upstream to nameservers, such as [ddns.net] or [domains.google.com].

`/etc/ddclient/ddclient.conf`
```ddclient.conf
use=web
daemon=900  # Long poll time for dynamic lookup

# Enable and configure the desired nameserver.
```

Test and enable the service.

```bash
ddclient -daemon=0 -debug -verbose -noquiet    # Run once to test.
systemctl enable --now ddclient                # Enable ddclient.
```
