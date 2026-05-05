+++
title = "Sealed images: building a sealed image"
date = 2026-04-29
slug = "2026-apr-29-sealed-images-building"

[extra]
author = "jeckersb"
+++

# Sealed images: building a sealed image

In the [first post](@/blog/2026-apr-27-sealed-images-security-chain.md)
we covered the security chain behind sealed images, and in the
[second post](@/blog/2026-apr-28-sealed-images-key-management.md) we
generated the Secure Boot keys needed to sign our boot artifacts.
Now it's time to put it all together and build a sealed image.

A complete working example is available in the
[rhel-bootc-examples](https://github.com/redhat-cop/rhel-bootc-examples)
repository under the
[sealing](https://github.com/redhat-cop/rhel-bootc-examples/tree/main/sealing)
directory.  This post walks through the key concepts behind that
example and explains why the build is structured the way it is.

## The chicken-and-egg problem

Building a sealed image has a complication that isn't immediately
obvious.  Recall from the first post that the composefs digest is
embedded in the UKI's kernel command line, and the UKI lives in the
image under `/boot`.  So we have a dependency cycle: we need the
filesystem to compute the composefs digest, but we need the digest
to produce the UKI, and we need the UKI to finalize the filesystem.

The solution is a multi-stage container build.  We build the base
filesystem first, without any UKI.  Then in a separate stage, we
compute the composefs digest from that filesystem, generate and
sign the UKI, and finally layer the UKI back on top of the base.
The UKI lives under `/boot`, which is not part of the composefs-
managed root filesystem, so adding it doesn't change the digest.

## The Containerfile

The
[Containerfile](https://github.com/redhat-cop/rhel-bootc-examples/blob/main/sealing/Containerfile)
in the examples repository uses four stages.  Let's walk through
each one.

### Stage 1: rootfs-builder

```dockerfile
FROM quay.io/centos-bootc/centos-bootc:stream10 AS rootfs-builder

RUN dnf install -y \
    epel-release \
    systemd-boot-unsigned \
    systemd-ukify \
    sbsigntools

RUN dnf remove -y bootupd
```

This starts from a CentOS Stream 10 bootc base image and installs
the tooling we need: `systemd-ukify` to build the UKI,
`sbsigntools` to sign it, and `systemd-boot-unsigned` to provide
the bootloader binary.  We remove `bootupd` because this example
uses systemd-boot rather than bootupd + GRUB.

This stage also signs systemd-boot with our db key:

```dockerfile
RUN --mount=type=secret,id=secureboot_key \
    --mount=type=secret,id=secureboot_cert <<EOF
sbsign --key /run/secrets/secureboot_key \
    --cert /run/secrets/secureboot_cert \
    --output /tmp/systemd-bootx64.efi \
    /usr/lib/systemd/boot/efi/systemd-bootx64.efi
install -m 0644 /tmp/systemd-bootx64.efi \
    /usr/lib/systemd/boot/efi/systemd-bootx64.efi
EOF
```

Note the use of `--mount=type=secret`.  The private key is mounted
into the build step but is never copied into any image layer.  This
is how we keep key material out of the final image.

### Stage 2: flatten to a single layer

```dockerfile
FROM scratch AS base
COPY --from=rootfs-builder / /
LABEL containers.bootc 1
LABEL ostree.bootable 1
```

This stage copies the entire filesystem from stage 1 into a `FROM
scratch` image, which flattens everything into a single layer.
This is important because the composefs digest is computed over
the complete filesystem tree.  If the image had multiple layers,
the digest would depend on how those layers happened to be stacked,
which could vary depending on the container runtime and storage
driver.  (For more on why multi-layer images can produce
non-deterministic results, see the earlier post on
[pitfalls of incomplete tar archives](@/blog/2025-dec-15-blog-containers-pitfalls-of-incomplete-tar-archives.md).)
Flattening eliminates that variable.

### Stage 3: generate and sign the UKI

```dockerfile
FROM base AS kernel
RUN --mount=type=bind,from=base,target=/target \
    --mount=type=secret,id=secureboot_key \
    --mount=type=secret,id=secureboot_cert <<EOF
bootc container ukify --rootfs /target \
    -- \
    --signtool sbsign \
    --secureboot-private-key /run/secrets/secureboot_key \
    --secureboot-certificate /run/secrets/secureboot_cert \
    --output /out/uki.efi
EOF
RUN kver=$(bootc container inspect --json | jq -r '.kernel.version') && \
    mkdir -p /boot/EFI/Linux && \
    mv /out/uki.efi "/boot/EFI/Linux/${kver}.efi"
```

This is where the magic happens.  `bootc container ukify` does
several things in one step:

1. Reads the filesystem at `/target` (the flattened base from
   stage 2, mounted via `--mount=type=bind`).
2. Computes the composefs digest (a SHA-512 hash of the EROFS
   image that describes the complete filesystem).
3. Discovers the kernel and initramfs from the filesystem.
4. Assembles a UKI containing the kernel, initramfs, and a
   command line that includes `composefs=<digest>`.
5. Signs the UKI with the db key via `sbsign`.

The result is a single `.efi` file that embeds a cryptographic
commitment to the exact filesystem it was built from, signed by
a key we control.

A second `RUN` step then discovers the kernel version and places
the UKI at the expected path under `/boot/EFI/Linux/`.

### Stage 4: the final image

```dockerfile
FROM base
COPY --from=kernel /boot /boot
```

The final image takes the flattened base and overlays the `/boot`
directory from the kernel stage.  This gives us a complete image:
the sealed root filesystem plus the signed UKI that references it.

## Building the image

With the Containerfile and keys in place, building the image is a
single `podman build` command:

```
$ podman build \
    --secret id=secureboot_key,src=target/keys/sb-db.key \
    --secret id=secureboot_cert,src=target/keys/sb-db.crt \
    -t localhost/sealed-host:latest \
    .
```

The two `--secret` flags make the db private key and certificate
available to the build stages that need them, without ever
persisting them in the image.

If you're using the examples repository, the
[Justfile](https://github.com/redhat-cop/rhel-bootc-examples/blob/main/sealing/Justfile)
wraps this for convenience:

```
$ just keygen       # generate keys (one-time)
$ just build-host   # build the sealed image
```

## Secret handling in CI

The examples repository includes a
[GitHub Actions workflow](https://github.com/redhat-cop/rhel-bootc-examples/blob/main/sealing/.github/workflows/build-sealed.yml)
that demonstrates how to handle key material in CI.  The db private
key is stored as a GitHub Actions secret and written to a temporary
file during the build.

For pull request builds, where the secret is not available, the
workflow generates an ephemeral key pair on the fly.  This allows
PRs to validate that the build works without requiring access to
production key material.

## What's next

At this point we have a sealed, signed container image.  The UKI
inside is signed with our Secure Boot key and embeds a composefs
digest that covers every file in the operating system.  In the
next post, we'll deploy this image to a system and verify the
seal is active.
