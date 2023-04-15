# Kubernetes
[Kubernetes](https://wiki.archlinux.org/title/kubernetes) is a system for automating the deployment, scaling, and management of containerized applications.

## K3s
K3s is a lightweight version of Kubernetes, developed by Rancher.
Install the latest or a specific version of K3s by running the install script.
Collect the desired version from the releases page on [their GitHub](https://github.com/k3s-io/k3s/releases/).

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.24.7+k3s1" sh -s –

# Uninstall K3s.
/usr/local/bin/k3s-uninstall.sh
```

TODO There is a problem with K3s when the `ufw` firewall is enabled. Investigate which firewall rules are required.
