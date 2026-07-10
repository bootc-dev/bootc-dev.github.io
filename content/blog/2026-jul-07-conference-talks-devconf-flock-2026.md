+++
title = "bootc ecosystem at DevConf.CZ and Flock to Fedora 2026"
date = 2026-07-07
slug = "2026-jul-07-conference-talks-devconf-flock-2026"

[extra]
author = "cgwalters"
+++

# bootc ecosystem at DevConf.CZ and Flock to Fedora 2026

June was a busy month for the bootc community, with back-to-back
conferences in the Czech Republic: [Flock to
Fedora](https://fedoraproject.org/flock/2026/) (June 14-16, Prague)
and [DevConf.CZ](https://www.devconf.info/cz/) (June 18-19, Brno).
Both featured a strong presence of talks covering bootc, image-mode
systems, and the broader ecosystem.  Here's a roundup of the
recordings and resources.

## DevConf.CZ 2026

The [full DevConf.CZ 2026 schedule](https://pretalx.devconf.info/devconf-cz-2026/schedule/)
(including slides) is available on pretalx.  Individual talk recordings
are on the [DevConf YouTube channel](https://www.youtube.com/channel/UCmYAQDZIQGm_kPvemBc_qwg).

[**Hardening OS Distribution: Verifiable and sealed OS with bootc and composefs**](https://www.youtube.com/watch?v=voB5oUDQ3FE)
(Colin Walters, 35 min) — Covers the shift toward OCI-native OS
delivery using bootc and composefs, including a demo of deploying a
sealed UKI composefs-backed system on RHEL 10.2.  Related to the
[sealed images blog series](@/blog/2026-may-04-sealed-images-security-chain.md)
here on bootc.dev.

[**These bootc are made for mailin'**](https://www.youtube.com/watch?v=zfXQViQpS3s)
(Michael Scherer, 36 min) — A practical walkthrough of combining bootc
and Fedora Image Mode with the Stalwart mail server for a low-maintenance,
self-hosted email stack running on the cheapest possible VM.

[**Local package layering on bootc systems with DNF5**](https://www.youtube.com/watch?v=0eNZOVVTkEI)
(Evan Goode, 26 min) — Explores the challenge of making `dnf install`
work on immutable bootc systems, introducing `dnf rebuild`, a new DNF5
plugin that reconciles imperative package management with declarative
container workflows.

[**Foreman, Katello and bootable containers**](https://www.youtube.com/watch?v=2MIor88lO8M)
(Adam Růžička, 35 min) — How Foreman and Katello enable fleet
management across both traditional package-mode and the new image-mode
approach, from image building through operational management.

[**Applying CI/CD Patterns to Image Mode for RHEL**](https://www.youtube.com/watch?v=htpPxxaHeXY)
(Alessandro Rossi, 27 min) — Practical examples of applying CI/CD
practices to Image Mode for RHEL using Tekton, GitHub Actions, GitLab
CI, Jenkins, and Ansible Automation Platform.

## Flock to Fedora 2026

Flock sessions were recorded as room-level livestreams on the
[Fedora Project YouTube channel](https://www.youtube.com/@fedora).
Individual talk pages on the
[Flock schedule](https://cfp.fedoraproject.org/flock-to-fedora-2026/schedule/)
link to slides and other resources.

[**The Future of Fedora Atomic: Unifying on Bootc and Shared Infrastructure**](https://cfp.fedoraproject.org/flock-to-fedora-2026/talk/M87M8M/)
(Hristo Marinov, BoF, 1h40) — A collaborative session discussing the
roadmap for migrating Fedora Atomic deliverables (Silverblue, CoreOS,
etc.) to a bootc-based workflow, covering architectural shifts and
build infrastructure changes.

[**Fedora bootc: Building and Operating Image-Based Systems**](https://cfp.fedoraproject.org/flock-to-fedora-2026/talk/3PHYFQ/)
(Gursewak Mangat, 25 min;
[slides](https://fedoraproject.org/w/uploads/f/f6/BootcInFedoraSlides.pdf))
— A practical introduction to bootc for Fedora contributors,
demonstrating how bootc's container-native approach makes it easy to
test OS images in CI using standard container tooling (BCVK) before
deploying to production.

[**From Compliance to Containers: Managing Fedora Fleets in the Atomic Era**](https://cfp.fedoraproject.org/flock-to-fedora-2026/talk/GMFYQH/)
(Adam Baali and Jonathan Billings, 55 min) — Bridging compliance
requirements with immutable image-based systems like Silverblue,
Kinoite, and bootc, demonstrating transparent tooling for NIST
controls.

[**What's new in Fedora CoreOS in 2026?**](https://cfp.fedoraproject.org/flock-to-fedora-2026/talk/JLKXBS/)
(Jean-Baptiste Trystram and Joel Capitao, 25 min) — Highlights from
the past year of Fedora CoreOS development, including OCI distribution
migration, bootc integration progress, and the move to
bootc-image-builder.
