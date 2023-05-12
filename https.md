# Web Certificates

Being able to sign a website against a trusted root certificate authority is important for hosting public and
even private websites.

Certbot is a client that can generate certificates based on [LetsEncrypt](https://letsencrypt.org/).
In order to ensure that certificates are only generated for the owner of a given domain, certbot must be run
from a machine that can respond to queries sent to the domain being certified.
This means that DNS records and port (80) forwarding must already be set up and correctly routed.

```bash
pacman -S certbot                                                # Install certbot.
certbot certonly --standalone -v --email <email> -d <domains,>    # Make new certificates.
certbot certificates                                             # List certificates.
```

Test the certificates by running a simple HTTPS server.

```bash
mkdir webroot && cd webroot/ && echo '<p>Hello There</p>' > index.html
npx http-server -p 443 --log-ip -S --cert '.../fullchain.pem' --key '.../privkey.pem'
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
