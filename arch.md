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
    pacman -Sy <package>        # Install a package.
    pacman -Syu                 # Upgrade the host.
    pacman -R <package>         # Remove a package.
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
    
    nmcli    # Command Line Interface.
    nmtui    # Text-Based Interface (Terminal Graphics).
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

### mDNS
idk

### NetBIOS
Install the [Samba](https://wiki.archlinux.org/title/samba) package.

To run the server, configure the `smb.conf`.

`/etc/samba/smb.conf`
```smb.conf
netbios aliases = name1, name2, ...
```

Then enable the `nmb.service`.
```bash
    testparm                              # Validate Samba config.
    systemctl enable --now nmb.service    # Enable NetBIOS.
    ufw allow CIFS                        # Allow incoming UDP/137 requests.
```

To run the client on Linux, enable the `winbind.service` and enable name resolution through it.

`/etc/nsswitch.conf`
```nsswitch.conf
hosts: ... wins
```

## Other Useful Programs
```bash
    ip addr                    # Interface info.
    ip neighbor                # ARP table
    drill google.com           # DNS Checker
    nmap 10.4.0.0/24           # Check local network
    traceroute google.com      # Track network path.
    curl checkip.dyndns.org    # Get public IP.
```

# Users
On Arch, [sudo](https://wiki.archlinux.org/title/sudo) is not installed automatically.

```bash
    # Uncomment the wheel group to grant sudo access.
    EDITOR=vim visudo    # Edit the sudoers file.
    
    useradd -m <user>           # Create a new user with a home directory.
    usermod -aG wheel <user>    # Add this user to the wheel group.
    passwd <user>               # Change password for user.
```

# SSH
Configure [SSH](https://wiki.archlinux.org/title/Secure_Shell) to allow remote connections to a machine.

```bash
    systemctl enable --now sshd
```

## Create and Copy a Public Key
Authenticating SSH connections with public keys is more secure, and also easier, compared to connecting with passwords.
Create an identity if needed on the client, and then copy the key to the host.
Prefer ED25519 keys.

```bash
    mkdir "$HOME/.ssh"
    chmod 600 ~/.ssh
    
    # Set a passphrase.
    ssh-keygen -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')" -f "$HOME/.ssh/$(hostname)-$(date +'%d-%m-%Y')"
    
    ssh-copy-id -i ~/.ssh/identity.pub <user>@<host>    # Use -f to disable check for private key.
    
    # Test the connection.
    ssh <user>@<host> -i ~/.ssh/identity -o PasswordAuthentication=no -vv
```
## Configure the Client SSH Agent
The SSH Agent allows the OS to cache decrypted keys and pass them to SSH programs automatically.
This reduces the number of password prompts the user sees.
Be sure to start only a single agent.

On systems without `systemd` (like Windows), configure the agent to start inside the `.bashrc`.
This will start a new `ssh-agent` process if one is not currently running, and source the environment file if the current environment is not using the agent socket file.

`~/.bashrc`
```bash
AGENT_CONFIG= ="${XDG_RUNTIME_DIR:=$HOME}"
# Check for running agent.
if ! ps -ef | grep "$USER" | awk '{print $6}' | grep "ssh-agent" > /dev/null; then
    ssh-agent -t 1h > "$AGENT_CONFIG/ssh-agent.env"
fi
# Check for environment variables.
if [[ ! -f "$SSH_AUTH_SOCK" ]]; then
        source "$AGENT_CONFIG/ssh-agent.env" >/dev/null
fi
echo "SSH Agent Started: ${SSH_AGENT_PID}"
```

Be aware that since this is inside the `.bashrc` it will only run once the user opens a Bash prompt.
Add to a different file if needed under other circumstances.
If `systemd` is installed, [just use that](https://wiki.archlinux.org/title/SSH_keys#Start_ssh-agent_with_systemd_user).

## Configure the Client SSH Config
Configure SSH to use an identity file and host shorthands.

`~/.ssh/config`
```.ssh/config
Match *
    AddKeysToAgent yes

Host <host1> <host2>
    User <user>
    IdentitiesOnly yes
    IdentityFile ~/.ssh/<identity>
    UpdateHostKeys ask
```

## Lock Down the Host
Run [ssh-audit](https://archlinux.org/packages/community/any/ssh-audit/) to find open SSH vulnerabilities.

```bash
    ssh-audit localhost
```

`/etc/ssh/sshd_config`
```sshd_config
Protocol 2
StrictMode yes

KexAlgorithms -x,y
MACs -x,y
HostKeyAlgorithms -x,y

PermitRootLogin no
PermitEmptyPasswords no

AuthenticationMethods publickey
PasswordAuthentication no
```

# Shells
## Starship
[Starship](https://starship.rs/) is a customizable prompt that can be run on top of Bash.

* TODO how to select a font???

## Bash
Remember to configure a `.bashrc` for the root account too!

`~/.bashrc`
```bash
alias ls='ls --color=auto'
alias ll='ls -la'

eval "$(starship init bash)"
```
