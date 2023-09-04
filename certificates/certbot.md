# Certbot Basics

Being able to sign a website against a trusted root certificate authority is important for hosting public and
even private websites.

Certbot is a client that can generate certificates based on [LetsEncrypt](https://letsencrypt.org/).
In order to ensure that certificates are only generated for the owner of a given domain, certbot must be run
from a machine that can respond to queries sent to the domain being certified.
This means that DNS records and port (80) forwarding must already be set up and correctly routed.

```bash
pacman -S certbot                                                 # Install certbot.
certbot certonly --standalone -v --email <email> -d <domains,>    # Make new certificates.
certbot certonly                                                  # Renew expired certificates.
certbot certificates                                              # List certificates.
```

Test the certificates by running a simple HTTPS server.

```bash
mkdir webroot && cd webroot/ && echo '<p>Hello There</p>' > index.html
npx http-server -p 443 --log-ip -S --cert '.../fullchain.pem' --key '.../privkey.pem'
```

# Wildcard Certificates on Google Domains

Wildcard certificate generation and renewal must be performed by creating a custom DNS subdomain name configured
to point to the correct place.
The `--manual` generation option is available, but the process can also be automated using a 
[community plugin](https://aur.archlinux.org/packages/certbot-dns-google-domains)
for certbot.

The latest instructions are available on Github: https://github.com/aaomidi/certbot-dns-google-domains.

```bash
certbot certonly \
  --authenticator 'dns-google-domains' \
  --dns-google-domains-credentials '<Google Domains Config>' \
  -d <root> -d <*.root> -d <etc>
```

By default, this certificate does not work for the root domain unless also included separately.
Multi-level subdomain wilcard certificates do not exist, by design.

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
