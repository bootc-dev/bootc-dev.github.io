+++
title = "Bootc Community Meeting Notes - 17 Oct 2025"
date = 2025-10-17
slug = "2025-oct-17-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## 17 Oct 2025
### Attendees
- Mohan Shash (RH)
- Robert Sturla (RH - on Monday :tada:) :heart:
- Joseph Marrero (RH)
- Jorge Castro (CNCF)
- Laura Santamaria (she/her; Red Hat)
- Colin Walters
- John Eckersberg (Red Hat, Inc.)
- Pragyan Poudyal
- Gursewak Mangat
- Jonathan Lebon
- Dusty Mabe

### Agenda
 - composefs progressing: {%preview https://github.com/bootc-dev/bootc/pull/1662 %} merged!
    - {%preview https://github.com/coreos/bootupd/pull/978#issuecomment-3411377999 %}
    - [Jorge] Is anyone doing systemd-boot on baremetal? I want a screenshot. :D
    - [Colin] Yes! Surprise :D
    - [Colin] One of the initial targets is custom sealed images. So you have your own CKI signed image with own cert, then can provision env with cert chain.
    - Lots of folks want to use machine owner keys flow (requires shim); we'll likely enhance Robert's patch to allow support, which will eventually allow secureboot to shim to secure owner keys. But if want bootc on baremetal without secureboot, you can do that now.
    - [Robert] Are you looking for testing stuff?
        - [Colin] ostree doesn't support UKIs yet. TL;DR: There is support for using GRUB and systemd backend using UKIs. In theory, things will work with things. But! not targeting the ostree backend right now for systemd-boot. Do you want that?
        - [Robert] Went off git main build, so where would testing be the most wanted? Just trying to figure out where testing is needed.
        - [Jorge] - we should tell more people about the git copr for bootc, I only discovered it exists today!
        - [Colin] bootupd is a soft requirement, so asking for people to try it out. Secondary, getting closer to composefs backend as daily driver
        - [John] Gets us close to having other distros as part of upstream CI as well. Wanting to do that so we don't break anyone else :)
        - [Jorge] Updates not working yet on other distros. Their Discord is poppin though!
        - [Jorge] SUSE did a talk on some kind of weird btrfs snapshots? May be using Podman backend.
 - [Mohan] Demo - YouTube access to CNCF is approved for all the maintainers
     - We have access!
     - Laura and Mohan to connect on Monday to establish how to do this.
     - So! If you are working on anything, make a video! Bring it to this call, and we'll review as a groups.
     - Laura can advise if you want to do it
     - [Colin] What's the quality bar?
     - Laura: offers to help mentor folks!
     - Laura: Lots of advice on setup: avoid it being a noisy environment
 - [Jorge] How's soft reboot coming along?
     - Robert: Initial soft reboot support in Bootc 1.7.0 - Doesn't work with LUKS-encrypted drives (using BTRFS/created with Anaconda?)
      - Joseph will produce a blog soonish :-)
