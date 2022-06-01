# XRDP
XRDP is a program that allows a user to create a graphical connection to a remote Linux computer via RDP. Very useful for connecting from Windows.

| Debian | SUSE | systemctl |
| ------ | ---- | --------- |
| xrdp   | xrdp | xrdp      |

## Manually Running
`(suse) /usr/lib/qsystemd/system/xrdp.service`

XRDP runs as two programs which must be run simultaneously. Use two terminals.
```bash
(suse) $ /usr/sbin/xrdp-sesman
(suse) $ /usr/sbin/xrdp
```

Running manually gives insight into the default config file locations.

## Change the Default Desktop
### SUSE
Open `/etc/xrdp/startwm.sh` and change the `SESSION` variable to the desired environment.
