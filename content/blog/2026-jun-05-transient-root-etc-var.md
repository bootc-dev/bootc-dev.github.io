+++
title = "Video: sealed bootc with transient /etc and /var"
date = 2026-06-05
slug = "2026-jun-05-transient-root-etc-var"

[extra]
author = "cgwalters"
+++

# Video: sealed bootc with transient /etc and /var

I recorded a short demo of the new composefs mount configuration
support that landed in [bootc#2201](https://github.com/bootc-dev/bootc/pull/2201).

[![Video: sealed bootc with transient /etc and /var](https://img.youtube.com/vi/VJYLtUOCqgA/0.jpg)](https://youtu.be/VJYLtUOCqgA)

The PR adds a `/usr/lib/bootc/setup-root-conf.toml` file that image
authors can ship in their container image to control how the
composefs-backed root filesystem is mounted at boot:

- `[root] transient = true` wraps the composefs lower in a tmpfs
  overlay, so all writes to `/` are discarded on reboot.
- `[etc] mount = "transient"|"overlay"|"bind"|"none"` controls how
  `/etc` is mounted from the deployment state directory.
- `[var] mount = "none"|"bind"` controls whether `/var` is
  bind-mounted from persistent state.  When set to `none`, `/var` is left as an
  empty composefs directory, and `systemd.volatile=state` on the
  kernel command line causes bootc to automatically skip the bind-mount
  so systemd can place a fresh tmpfs there.

This builds directly on the
[sealed images series](@/blog/2026-may-04-sealed-images-security-chain.md):
with a transient root and `/etc`, each boot starts from a clean,
verified image with no persistent mutation to the OS layer.
