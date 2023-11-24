# Bootloader

TODO: 2023-11-24: Verify below claim...

Be aware that some virtualization tools (like QEMU on TrueNAS Scale...) will only look for a bootloader in the boot partition at `/ETI/BOOT/bootx64.efi`.
Currently the best option for the FAT32 boot partition is to simply make a copy of the GRUB bootloader to that location.
Remember to keep it up-to-date when GRUB is updated.

# Networking

Be aware that on [TrueNAS Scale](https://www.truenas.com/docs/scale/scaletutorials/virtualization/accessingnasfromvm/),
additional configuration is required to enable communicate between the host and guest.
