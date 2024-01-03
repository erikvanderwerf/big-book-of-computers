Everything in Linux is a file.
Files are logically abstracted from physical bytes on a device through a
[file system](https://wiki.archlinux.org/title/file_systems).

# Mount

The `mount` command is used to mound a file system to a directory.
Mount must be called to make the mount on an existing directory.
Linux overlays the mounted file system on top of any existing data,
hiding it until the filesystem is `umount`'d.

```bash
findmnt    # Lists known mounts. Run as root to see everything.
mount      # Mount a file system.
umount     # Unmount a file system (source or target).
```

## NFS

[NFS](https://wiki.archlinux.org/title/NFS) is a distributed file
system protocol.

* NFS is not encrypted.
* NFS does not have any user authentication.
* NFS does not translate user/group ids from local to remote.

NFS v4 has more features...

```bash
mount -t nfs -o rw,nconnect=16 host:/export/root /mount/on/client    # TrueNAS, NFSv3
```

Beware that a Docker Compose issue that presents as a permissions issue may
actually be solved by adding a file to an empty NFS mount.

```
Error response from daemon: failed to copy file info for .../_data: failed to chown .../_data: lchown .../_data: operation not permitted
```

https://stackoverflow.com/questions/56126490/docker-run-on-nfs-mount-causes-fail-to-copy-and-fail-to-chown/71621978#71621978
