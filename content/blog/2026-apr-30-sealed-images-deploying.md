+++
title = "Sealed images: deploying and verifying a sealed image"
date = 2026-04-30
slug = "2026-apr-30-sealed-images-deploying"

[extra]
author = "jeckersb"
+++

# Sealed images: deploying and verifying a sealed image

In the [previous post](@/blog/2026-apr-29-sealed-images-building.md)
we built a sealed, signed container image.  Now it's time to deploy
it to a virtual machine and verify that the full security chain is
active.

## bcvk: a tool for running bootc VMs

[bcvk](https://github.com/bootc-dev/bcvk) (bootc virtualization kit)
is a tool that makes it easy to run bootc container images as virtual
machines.  Under the hood, it calls `bootc install to-disk` to create
a bootable disk image, then manages the VM lifecycle via libvirt.
For sealed images, bcvk also handles Secure Boot key enrollment into
the VM's UEFI firmware automatically using
[virt-firmware](https://gitlab.com/kraxel/virt-firmware).

bcvk is available as a package on Fedora and EPEL:

```
$ dnf install bcvk
```

## Deploying the image

With the sealed image built from the previous post and keys generated
from the key management post, deploying is a single command:

```
$ bcvk libvirt run \
    --ssh-wait \
    --name sealed-demo \
    --filesystem=ext4 \
    --secure-boot-keys target/keys \
    localhost/sealed-host:latest
```

Let's break down what's happening here:

- `--filesystem=ext4` tells `bootc install` to use ext4 for the root
  filesystem.  The default root filesystem may be XFS, which does not
  support fs-verity, so we explicitly select ext4 here.  (fs-verity
  is supported on ext4 and btrfs.)
- `--secure-boot-keys target/keys` points bcvk to the directory
  containing our PK, KEK, and db certificates.  bcvk uses
  `virt-fw-vars` to enroll these keys into a copy of the OVMF
  firmware variable store, so the VM boots with Secure Boot enabled
  and configured to trust our signing key.
- `--ssh-wait` tells bcvk to wait until SSH is available before
  returning, so we know the system has fully booted.

If you're using the
[examples repository](https://github.com/redhat-cop/rhel-bootc-examples/tree/main/sealing),
this is wrapped up in a convenient `just` recipe:

```
$ just bcvk-ssh
```

This builds the image, boots the VM, waits for it to reach
`multi-user.target`, and opens an interactive SSH session.

## Verifying the seal

Once the system is booted and we have an SSH session, we can verify
that the full security chain is active.

### Check the kernel command line

```
$ cat /proc/cmdline
composefs=3a7f... (128-character SHA-512 hex digest) rw
```

The presence of `composefs=<digest>` in the kernel command line
confirms that the initramfs received the composefs digest from the
signed UKI.  This is the digest that was computed during the build
and embedded in the UKI's command line section.

### Check the root mount

```
$ findmnt -t overlay /
TARGET SOURCE                    FSTYPE  OPTIONS
/      overlay[/sysroot/state..] overlay ro,relatime,...,verity=require,...
```

The key thing to look for is `verity=require` in the mount options.
This confirms that the kernel is enforcing fs-verity verification
on every file access through the composefs mount.  Any file in the
operating system whose content doesn't match its expected fs-verity
digest will produce an I/O error when read.

### Check Secure Boot status

```
$ mokutil --sb-state
SecureBoot enabled
```

This confirms that the firmware verified the bootloader and UKI
signatures before allowing them to execute.

### What this tells us

These three checks together confirm the full chain from the
[first post](@/blog/2026-apr-27-sealed-images-security-chain.md):

1. Secure Boot verified the bootloader and UKI (`mokutil --sb-state`)
2. The UKI delivered the composefs digest to the initramfs
   (`/proc/cmdline`)
3. The kernel is enforcing per-file verification against that digest
   (`verity=require`)

## What happens if something is tampered with?

It's worth understanding why tampering with a sealed system is
difficult in practice.

The operating system files are stored in a content-addressed object
store on disk.  Each object has fs-verity enabled, which means the
kernel has recorded a cryptographic hash of its contents.  The files
are immutable: you cannot write to an fs-verity protected file.  Any
attempt to open a verity-protected file for writing will fail.

Even if an attacker were to bypass the filesystem and corrupt data
directly on the underlying block device, the kernel would detect
the corruption.  When a process attempts to read the tampered file,
the kernel verifies the data against the stored hash before
returning it.  If the data doesn't match, the read fails with
`EIO`.  The corrupted data is never served to any process.

This is a meaningful distinction from a system where tampered files
are silently served.  On a sealed system, corruption doesn't go
unnoticed -- it causes an immediate, visible failure.

## Conclusion

Over the course of this series, we've walked through the complete
lifecycle of a sealed image:

1. [The security chain](@/blog/2026-apr-27-sealed-images-security-chain.md)
   from firmware to filesystem that makes sealed images possible
2. [Key management](@/blog/2026-apr-28-sealed-images-key-management.md)
   for generating and enrolling Secure Boot keys
3. [Building a sealed image](@/blog/2026-apr-29-sealed-images-building.md)
   with a multi-stage container build
4. Deploying and verifying the seal (this post)

The complete working example is available in the
[rhel-bootc-examples](https://github.com/redhat-cop/rhel-bootc-examples)
repository under the
[sealing](https://github.com/redhat-cop/rhel-bootc-examples/tree/main/sealing)
directory.
