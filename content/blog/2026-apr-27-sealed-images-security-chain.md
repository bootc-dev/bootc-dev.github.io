+++
title = "Sealed images: the security chain from firmware to filesystem"
date = 2026-04-27
slug = "2026-apr-27-sealed-images-security-chain"

[extra]
author = "jeckersb"
+++

# Sealed images: the security chain from firmware to filesystem

This is the first in a series of posts about bootc sealed images.  In
this post we'll take a tour through the full security chain that makes
sealed images work, from the moment your system powers on through
every file read at runtime.  Future posts will cover key management,
building sealed images, and deploying them to various environments.

## The gap

Typically a default Linux install comes "unlocked", allowing you to
be root and make persistent changes.  While many distributions
support Secure Boot, it's typically only the kernel that's signed.
The cryptographic chain of trust from firmware doesn't cover the
root filesystem or applications.  Every library loaded, every binary
executed, every configuration file read after the kernel boots runs
on trust.

This is the gap.  The boot chain may be verified.  The operating
system is not.

Sealed images close this gap by extending cryptographic verification
from the firmware all the way through every file in the operating
system.  Not as a one-time check at boot, but continuously, at every
file read, for the entire lifetime of the system.

Let's walk through how each piece fits together.

## Secure Boot

Secure Boot is the foundation of the chain.  It's a UEFI firmware
feature that ensures only signed code runs during the boot process.

The firmware maintains a database of trusted keys.  When the system
powers on, the firmware loads the bootloader and checks its signature
against those keys.  If the signature is valid, the bootloader runs.
If not, the system refuses to boot.

This is well-established technology and not new to sealed images.
What matters for our purposes is that Secure Boot gives us a root of
trust: a guarantee that the first piece of software to execute after
the firmware is something we explicitly chose to trust.

## Unified Kernel Images

Traditionally, the boot chain after Secure Boot looks something like
this: the signed bootloader loads a kernel, an initramfs, and a kernel
command line, often from separate files on a boot partition.  The
bootloader may verify the kernel's signature, but the initramfs and
command line are typically not covered by any signature.

A [Unified Kernel Image (UKI)](https://uapi-group.org/specifications/specs/unified_kernel_image/)
changes this by bundling the kernel, the initramfs, and the kernel
command line into a single EFI binary.  The entire bundle is signed
as one unit.  This means the firmware (or a signed bootloader like
systemd-boot) can verify the whole thing in one step.

This is important for sealed images because the kernel command line
is no longer an unverified input.  Anything embedded in the command
line is covered by the same signature that covers the kernel itself.
As we'll see next, this is where the composefs digest lives.

## composefs: EROFS + overlayfs + fs-verity

Here's where things get interesting.  composefs brings together
three kernel technologies to provide cryptographic verification of
the entire filesystem.

When a sealed image is built, the operating system filesystem is
processed into an
[EROFS](https://docs.kernel.org/filesystems/erofs.html) metadata
image.  This image describes every file, directory, symlink, and
permission in the tree, but it does not contain the actual file
contents.  Instead, each file entry in the EROFS image records the
[fs-verity](https://docs.kernel.org/filesystems/fsverity.html)
digest of that file's contents.  The actual file data is stored
separately in a content-addressed object store, where each file has
fs-verity enabled by the kernel.

At mount time, the kernel stitches these pieces together using
[overlayfs](https://docs.kernel.org/filesystems/overlayfs.html).
The EROFS image serves as a metadata layer, and the object store
provides the file data.  The overlayfs `verity=require` mount option
tells the kernel to enforce that every file served through the mount
has an fs-verity digest matching what the EROFS metadata expects.

fs-verity itself is a kernel feature that provides per-file integrity
verification.  When fs-verity is enabled on a file, the kernel
computes a cryptographic hash of the file's contents and stores a
hash tree alongside the file on disk.  From that point on, every
read is verified at the block level -- the kernel checks only the
blocks being read, not the entire file, and caches the results so
repeated reads don't pay the cost again.

A key behavior to understand is what happens when verification
fails: the kernel returns an I/O error.  The corrupted or tampered
data is never served to any process.  The `open()` call on the file
succeeds (the file exists, after all), but `read()` on the corrupted
portion returns `EIO`.  The data is simply unreadable.

The EROFS metadata image itself also has an fs-verity digest: a
single cryptographic hash that covers the complete filesystem
description.  This is the composefs digest.  It is embedded in the
UKI's kernel command line:

```
composefs=abc123def456...  (128-character SHA-512 hex digest)
```

Because the UKI is signed, this digest is now part of the verified
boot chain.  The firmware verifies the UKI signature, and the UKI
carries within it a cryptographic commitment to the exact filesystem
that should be mounted as the operating system.

## Putting it all together

The full chain looks like this.  Each step must succeed before the
next can proceed:

1. The firmware verifies the signed bootloader.
2. The bootloader verifies the signed Unified Kernel Image (UKI).
3. The initramfs (inside the verified UKI) reads the composefs
   digest from the kernel command line (also inside the verified UKI).
4. The initramfs mounts the composefs image and verifies the EROFS
   image's fs-verity digest matches the digest from the command line.
5. Every subsequent file read from the operating system is
   individually verified by the kernel against the fs-verity digest
   recorded in the EROFS metadata.

There is no point after firmware where unverified code executes or
unverified data is served.  The chain is continuous.

## What sealed images cover

It's worth being precise about scope.  The seal covers every file in
the immutable operating system image: the base OS, any packages you
added, your monitoring agents, security tools, application binaries.
Everything that was part of the image when it was built is sealed.

By default, `/etc` and `/var` are mutable and persistent.  It is
possible to make `/etc` transient today, and this is a good idea if
it matches your use case.  More suggestions on this will be in the
documentation.

## What's next

This post covered the architecture.  In the next posts, we'll get
hands-on:

- **Key management**: generating Secure Boot keys, enrolling them,
  and understanding the signing chain
- **Building sealed images**: writing a Containerfile, producing a
  signed UKI, and verifying the result
- **Deploying sealed images**: installing to bare metal, virtual
  machines, and cloud environments

The signing chain is fully under your control.  You generate the keys,
you sign the images, you decide what your systems trust.  The next
post will show you how.
