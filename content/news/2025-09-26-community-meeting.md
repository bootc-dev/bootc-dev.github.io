+++
title = "Bootc Community Meeting Notes - 26 Sep 2025"
date = 2025-09-26
slug = "2025-sep-26-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#user-content-code-of-conduct) applies.

## 26 Sept 2025
### Attendees
- Laura Santamaria (she/her; Red Hat)
- Dusty Mabe (he/him; Red Hat)
- Sean Thrailkill
- Robert Sturla (Tesco Bank)
- Jonathan Lebon (Red Hat)
- Tulip Blossom (Chainguard)
- Joseph Marrero (Red Hat)
- Mohan Shash (RH)
- Preethi Thomas
- Hristo Marinov
- Colin Walters

### Agenda
- [Mohan] Anything interesting from the conference - DevConf.us
    - [Laura] Bootc discussions - nothing significant for the community
- [Sean] Sean is doing a presentation on bootc - Texas Linux Fest
    - Laura to send bootc stickers for Sean
- All things Open - Conference in raleigh
    - Is bootc represented? not sure
    - Laura will look into this
- [Sean and Tulip] New KDE thing. They use mkosi + systemd-sysupdate, there is a WIP arch-bootc based KDE linux spin. Actually its from one of the KDE Linux devs. He forked arch-bootc from bootcrew
    - {%preview https://github.com/silverhadch/arch-bootc-kde %}
- [Colin] New development workflow
    - Always looking for feedback
    - {%preview https://github.com/bootc-dev/bootc/issues/1635 %}
    - {%preview https://github.com/bootc-dev/bootc/pull/1638 %}
- jlebon: Working on signing
    - Working on adding necessary stuff to container libraries to make that nicer
    - {%preview https://github.com/containers/container-libs/pull/355 %}
    - {%preview https://github.com/containers/skopeo/pull/2714 %}
    - {%preview https://github.com/containers/skopeo/issues/1829 %}
        - Skopeo ticket locked - **Blocker**
        - Needs unlocked to get unblocked for people to commment
        - Learned a lot recently when switching from ostree signing to OSCI signing.
- [Colin] Outstanding PRs?
    - Anyone else want to grab any? 1645 would be great for someone to pickup.
        - {%preview https://github.com/bootc-dev/bootc/pull/1645 %}

### TODO
- [ ] Someone needs to unlock the Skopeo ticket - Ping Lokesh

