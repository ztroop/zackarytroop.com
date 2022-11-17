+++
title = "Troubleshooting VirtualBox"
slug = "troubleshooting-virtualbox"
date = 2019-08-24
+++

# Summary

There's a strong reliance on VirtualBox in the classes I teach because the college has a BYOD (Bring Your Own Device) policy.

The first thing you will need to do is to navigate to the [Downloads](https://www.virtualbox.org/wiki/Downloads) page and download the most recent version.

## Extension Pack

This is an optional package that you can install. If you download it, make sure it matches the version of VirtualBox you have installed. It provides the following functionality:

- Support for USB 2.0 and USB 3.0 Devices
- VirtualBox RDP
- Disk Encryption
- NVMe and PXE Boot

## Common Issues

I’ve observed several issues that students have encountered over time.

There are many confusing errors that VirtualBox can throw. I’ll try to cover some examples. When in doubt, "_Google it_".

## Enabling Virtualization (BIOS/UEFI)

The most common issue is that virtualization is not enabled in their PC.

You will need to enter the BIOS or UEFI in your computer during the boot up process (before the operating system starts).

Check your laptop model online to see what _key combination_ is needed during the boot process to access the computer settings.

When you get into the BIOS/UEFI settings, confirm that _virtualization_ is enabled. Sometimes this can be labeled as `VT-X` or `AMD-V` (depending on you CPU manufacturer).

## I/O or Disk Error

- Check the virtual disk exists in the path that is configured.
- Check your virtual machine’s virtual disk is out of space.
- Check your system is out of disk space, especially if you’re using a dynamically growing virtual disk.
- Check your system’s disk is healthy. If you’re using a SSD, you might be out of reads/writes.

## Register Error

Check to see if you already have a VM profile that’s already using that virtual disk.

## Extension Pack

- I can't install the Extension Pack. Confirm that it is the same virtual as the VirtualBox you have installed.
- I'm seeing `VERR ALREADY EXISTS`. You probably already have the Extension Pack installed.
