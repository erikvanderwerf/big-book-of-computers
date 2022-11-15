The [Arch Wiki](https://wiki.archlinux.org/) is the best resource for learning about Arch Linux.

# Installation
Follow the [Installation Guide](https://wiki.archlinux.org/title/Installation_guide) to install Arch.

```bash
    pacstrap /mnt base linux grub efibootmgr networkmanager vim openssh
```

Be aware that some virtualization tools (like QEMU on TrueNAS Scale...) will only look for a bootloader in the boot partition at `/ETI/BOOT/bootx64.efi`.
Currently the best option for the FAT32 boot partition is to simply make a copy of the GRUB bootloader to that location.
Remember to keep it up-to-date when GRUB is updated.

* TODO document how to configure GRUB to place the bootloader in the right place.

# Pacman
[Pacman](https://wiki.archlinux.org/title/pacman) is the package manager for Arch.

```bash
    pacman -Sy <package>        # Install a package
    pacman -Syu                 # Upgrade the host
    pacman -R <package>         # Remove a package
    pacman -Rs <packge>         # Remove a package and its orphaned dependencies.
    pacman -Qdt                 # Query for unnecessary packages.
    pacman -Rns $(pacman -Qdt)  # Remove (including config) unnecessary packages.
```

# Devices and File Systems
```bash
    mount     # Basic list of mounts.
    lsblk     # List block devices.
```

* [Btrfs - Arch Wiki](https://wiki.archlinux.org/title/btrfs)
* [NFS - Arch Wiki](https://wiki.archlinux.org/title/NFS)

# Systemd and Systemctl
Arch uses [systemctl](https://wiki.archlinux.org/title/Systemd#Basic_systemctl_usage) to command `systemd`.

```bash
    systemctl enable --now <unit>   # Enable and Start a unit.
    systemctl status <unit>         # Get the status of a unit.
    systemctl disable --now <unit>  # Disable and Stop a unit.
    systemctl restart <unit>        # Restart the unit process.
    systemctl reload <unit>         # Instruct unit to reload configuration.
```

* TODO try out `systemdgenie` in Plasma.

## Restart a Service on File Change
Create a path unit that triggers a systemd unit to accomplish the desired task.

`/etc/systemd/system/thing-restart.service`
```unit
[Service]
Type=OneShot
ExecStart=/usr/bin/systemctl restart thing.service
```

`/etc/systemd/system/thing-restart.path`
```unit
[Path]
PathChanged=/path/to/file

[Install]
WantedBy=multi-user.target
```

Enable the path unit.
```bash
systemctl enable --now thing-restart.path
```

# Networking
## Hostname
Set the hostname with NetworkManager, or by editing `/etc/hostname`.

## NetworkManager
[NetworkManager](https://wiki.archlinux.org/title/NetworkManager) is the preferred program for automatically configuring networks.
NetworkManager casing is not consistent.

```bash
    pacman -S networkmanager
    systemctl enable --now NetworkManger.service
    
    nmcli    # Command Line Interface
    nmtui    # Text-Based Interface (Terminal Graphics)
```

Remember to enable the interface to "automatically" start on reboot.
Otherwise the interface will not be enabled next time.

Be aware that on [TrueNAS Scale](https://www.truenas.com/docs/scale/scaletutorials/virtualization/accessingnasfromvm/), additional configuration is required to communicate between the host and guest.

## ufw Firewall
The [Uncomplicated Firewall](https://wiki.archlinux.org/title/Uncomplicated_Firewall) provides a command line interface to control the system firewall.

```bash
    ufw allow ssh     # Allow SSH preconfigured network rules.
    ufw limit ssh     # Allow SSH with rate limits.
    ufw delete ssh    # Delete rules for SSH.
    ufw enable        # Turn on the firewall.
    
    # Allow TCP from a subnet to port 7890 on any destination.
    ufw allow proto tcp from 10.4.0.0/24 to any port 79890
    
    ufw status numbered    # List enabled rules with an index.
    ufw delete 6           # Delete the 6th active rule.
```

Remember to allow (or limit) SSH access before enabling the firewall.

## Local Hostname DNS Resolution
There are two technologies available which allow devices on a local network to automatically resolve local hostnames and domains.
By default, Unix prefers mDNS while Windows is configured to use NetBIOS.

### nDNS
idk

### NetBIOS
Install the [Samba](https://wiki.archlinux.org/title/samba) package.

To run the server, configure the `smb.conf` and enable the `nmb.service`.

`/etc/samba/smb.conf`
```smb.conf
netbios aliases = name1, name2, ...
```

```bash
    testparm                              # Validate Samba config.
    systemctl enable --now nmb.service    # Enable NetBIOS
    ufw allow CIFS                        # Allow incoming UDP/137 requests.
```

To run the client on Linux, enable the `winbind.service` and enable name resolution through it.

`/etc/nsswitch.conf`
```nsswitch.conf
hosts: ... wins
```
