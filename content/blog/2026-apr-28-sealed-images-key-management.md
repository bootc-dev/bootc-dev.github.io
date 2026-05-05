+++
title = "Sealed images: key management for Secure Boot"
date = 2026-04-28
slug = "2026-apr-28-sealed-images-key-management"

[extra]
author = "jeckersb"
+++

# Sealed images: key management for Secure Boot

In the [previous post](@/blog/2026-apr-27-sealed-images-security-chain.md),
we walked through the security chain that sealed images provide, from
firmware through runtime file verification.  That chain depends on
cryptographic keys at every step.  In this post, we'll cover the keys
involved, how to generate them, and how they get enrolled into your
system's firmware.

## The UEFI Secure Boot key hierarchy

UEFI Secure Boot uses a hierarchy of three key types, each serving a
distinct role:

**Platform Key (PK)** -- The root of the Secure Boot trust hierarchy.
There is exactly one PK.  The PK controls who can modify the Secure
Boot key databases.  It is used to sign updates to the KEK database.
When the PK is enrolled, the firmware transitions from Setup Mode to
User Mode, enabling Secure Boot enforcement.

**Key Exchange Key (KEK)** -- One or more KEKs can be enrolled.  A KEK
is authorized to sign updates to the signature database (db).  The
KEK exists to allow delegation: the PK owner can authorize other
parties to manage the set of trusted signing keys without giving them
full control over the Secure Boot configuration.

**Signature Database (db)** -- The db contains the keys or hashes
that the firmware uses to verify boot binaries.  When the firmware
loads a bootloader or UKI, it checks the binary's signature against
the keys in the db.  If the signature matches a db entry, the binary
is allowed to execute.

In practice, many organizations will only need to generate a single
set of these keys.  The db key is the one that does the day-to-day
work: it signs your UKIs and bootloader.  The PK and KEK exist
primarily to control who can modify the db.

If you already have Secure Boot keys provisioned in your
infrastructure, you don't need to generate new ones.  You only need
access to a db key (or the ability to add your signing key to the
existing db) in order to sign your UKIs.  The rest of this post
assumes you're starting from scratch, but the signing steps apply
regardless of where your keys came from.

## Generating keys

The following commands generate a complete set of Secure Boot keys.
These examples use `openssl` directly, but you should use whatever
key management practices are appropriate for your organization.
Hardware security modules (HSMs), vault systems, or other key
management infrastructure may be more appropriate for production
use.

First, generate a GUID to identify the key owner.  This is embedded
in the key enrollment data and can be any unique identifier:

```
$ uuidgen --random > GUID.txt
```

Generate the Platform Key:

```
$ openssl req -newkey rsa:2048 -nodes -keyout PK.key \
    -new -x509 -sha256 -days 3650 \
    -subj '/CN=My Platform Key/' -out PK.crt
$ openssl x509 -outform DER -in PK.crt -out PK.cer
```

Generate the Key Exchange Key:

```
$ openssl req -newkey rsa:2048 -nodes -keyout KEK.key \
    -new -x509 -sha256 -days 3650 \
    -subj '/CN=My Key Exchange Key/' -out KEK.crt
$ openssl x509 -outform DER -in KEK.crt -out KEK.cer
```

Generate the Signature Database key:

```
$ openssl req -newkey rsa:2048 -nodes -keyout db.key \
    -new -x509 -sha256 -days 3650 \
    -subj '/CN=My Signature Database Key/' -out db.crt
$ openssl x509 -outform DER -in db.crt -out db.cer
```

Each command produces three files: a private key (`.key`), a PEM
certificate (`.crt`), and a DER-encoded certificate (`.cer`).  The
private keys are what you need to protect.  The certificates are
public and are what gets enrolled into the firmware.

## Signing boot artifacts

With a db key in hand, you can sign the two boot binaries that need
Secure Boot verification: the bootloader (systemd-boot) and the
Unified Kernel Image.

Signing systemd-boot:

```
$ sbsign --key db.key --cert db.crt \
    --output systemd-bootx64.efi.signed \
    /usr/lib/systemd/boot/efi/systemd-bootx64.efi
```

The UKI signing process is more involved because the UKI embeds the
composefs digest, which must be computed from the filesystem first.
We'll cover building sealed images (including UKI generation and
signing) in detail in the next post.  For now, the important thing
to know is that `ukify` (the tool that assembles UKIs) accepts the
same `--secureboot-private-key` and `--secureboot-certificate`
options and uses `sbsign` under the hood to sign the final EFI
binary.

## Enrolling keys into firmware

The keys need to be enrolled into the UEFI firmware's Secure Boot
database before the firmware will use them for verification.  There
are several ways to do this depending on your environment.

### Manual enrollment via firmware setup

Most UEFI firmware implementations provide a setup menu (accessed
during early boot, typically via a key like F2 or Del) where Secure
Boot keys can be managed.  The DER-encoded certificates (`.cer`
files) can be enrolled from a USB drive or the EFI System Partition.
This is straightforward for individual systems but does not scale
well.

### Programmatic enrollment for virtual machines

For virtual machines using OVMF (the open source UEFI firmware for
QEMU/KVM), keys can be enrolled directly into the firmware variable
store using the `virt-fw-vars` tool from the
[virt-firmware](https://gitlab.com/kraxel/virt-firmware) project:

```
$ GUID=$(cat GUID.txt)
$ virt-fw-vars \
    --input /usr/share/edk2/ovmf/OVMF_VARS.fd \
    --secure-boot \
    --set-pk  $GUID PK.crt \
    --add-kek $GUID KEK.crt \
    --add-db  $GUID db.crt \
    -o OVMF_VARS_ENROLLED.fd
```

This produces a firmware variable store with your keys pre-enrolled.
When QEMU starts with this variable store, Secure Boot is active and
will only accept binaries signed with your db key.  This is
particularly useful for testing and for VM-based deployments where
you control the firmware image.

### Auto-enrollment via systemd-boot

systemd-boot supports automatic key enrollment when the firmware is
in Setup Mode (no Platform Key enrolled).  This works by placing
signed UEFI authenticated variable files (`.auth` files) on the EFI
System Partition.

Creating `.auth` files requires converting your certificates into
UEFI signature lists and signing them with the appropriate parent
key:

```
$ GUID=$(cat GUID.txt)
$ attr=NON_VOLATILE,RUNTIME_ACCESS,BOOTSERVICE_ACCESS,TIME_BASED_AUTHENTICATED_WRITE_ACCESS

# Create EFI signature lists
$ sbsiglist --owner $GUID --type x509 --output PK.esl PK.cer
$ sbsiglist --owner $GUID --type x509 --output KEK.esl KEK.cer
$ sbsiglist --owner $GUID --type x509 --output db.esl db.cer

# Sign the variables (PK and KEK signed by PK, db signed by KEK)
$ sbvarsign --attr $attr --key PK.key --cert PK.crt --output PK.auth PK PK.esl
$ sbvarsign --attr $attr --key PK.key --cert PK.crt --output KEK.auth KEK KEK.esl
$ sbvarsign --attr $attr --key KEK.key --cert KEK.crt --output db.auth db db.esl
```

These `.auth` files can then be placed on the ESP where systemd-boot
expects them.  bootc can automate this: if you place the `.auth`
files in your container image at
`/usr/lib/bootc/install/secureboot-keys/<name>/`, bootc will copy
them to the ESP during installation.  On the next boot, systemd-boot
can enroll them into the firmware.

The enrollment behavior is controlled by the `secure-boot-enroll`
setting in `loader.conf`.  See the
[systemd-boot documentation](https://www.freedesktop.org/software/systemd/man/latest/loader.conf.html)
for details on the available modes.

## Key security considerations

A few things worth keeping in mind:

**Protect your private keys.**  The `.key` files are what allow
signing boot artifacts.  Anyone with access to your db private key
can sign binaries that your firmware will trust.  Store private keys
with appropriate access controls, and consider using an HSM or key
management system for production deployments.

**Key rotation.**  The examples above use 10-year expiry (`-days
3650`), which is generous.  Plan for key rotation before your
certificates expire.  The UEFI key hierarchy makes this manageable:
you can use your KEK to authorize a new db key without needing to
re-enroll the PK.

**Separation of concerns.**  In a CI/CD pipeline, the signing step
should ideally be isolated from the build step.  The build process
produces an unsigned UKI, and a separate signing service (with access
to the private key) signs it.  This limits the blast radius if the
build environment is compromised.

## What's next

With keys generated and an understanding of how they fit into the
trust chain, the next post will walk through building a sealed image
end-to-end: writing a Containerfile, computing the composefs digest,
generating and signing the UKI, and producing a deployable container
image.
