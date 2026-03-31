+++
title = "Bootc Community Meeting Notes - 29 Aug 2025"
date = 2025-08-29
slug = "2025-aug-29-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## 29 Aug 2025
### Attendees
- Laura Santamaria (Red Hat)
- Joseph Marrero Corchado (Red Hat, Inc.)
- Colin Walters
- Preethi Thomas
- John Eckersberg (Red Hat, Inc.)
- Robert Sturla (Tesco Bank / Universal Blue)
- Hristo Marinov
- Dusty Mabe
- Ben Breard
- Jonathan Lebon
- Timothée Ravier

### Agenda
- Dynamic bootloader detection in bootupd {%preview https://github.com/coreos/bootupd/pull/978 %}
    - Robert gave a review of the PR
    - Discussion? _(Sorry, coming in during the middle of the discussion... - LAS)_
        - Grub vs systemd boot
        - **Decision:** Robert will default to grub if found, but will use systemd boot if it's not there
        - composefs/bootc meeting - reach out to join. Dealing with conflict/opening branch
- Open floor time
    - [Ben] got a request to help migrate from existing installs into bootc system. Would you be able to pull package database from system? Maybe scrape etc/? Sounds like endless edge cases... Thoughts?
        - [Colin] Someone in upstream Slack is doing this and has it mostly working for their prod systems. SELinux issue. Key thing that would be really helpful is preserving existing bootloader entries and making it non-destructive. Would immensely aid testing. Totally doable.
        - [Ben] Wouldn't you need to eventually blow away the existing loaders?
        - [Colin] Yeah, garbage collection is a thing. We can blow away containers but leave everything else. Didn't want to get in the business of blowing away data. Not hard to do cleanup as you wish (systemd container in sysroot, then remove things). If data partitions, everything gets easier.
    - [Jonathan] Actually thought we were talking about something different. config starting point.
        - [Ben] Should be easy to scrape RPMs and modify base image with diff out and add into containerfile. Only handling package side.
        - [Jonathan] Could copy to containerfile then etc to preserve those?
        - [Colin] [thread](https://cloud-native.slack.com/archives/C08SKSQKG1L/p1756205380188269?thread_ts=1755600945.963309&cid=C08SKSQKG1L) for person doing install is in CNCF Slack.
- Procedural note: Laura won't be here next two weeks, will be traveling and also representing bootc at DevConf.us
- Meta: Notes about who takes notes! We'll try to assign someone to do this in each meeting on a rotating basis
- conferences - also KubeCon is coming up! Laura is talking about sharing a schedule for the booth.
- Discussion of swag: ideas? 3d prints, t-shirts, (cat ears?), *not* socks; avoid requiring sizes? RaspberryPis pre-loaded?
  - pre-flashed SD cards for RPi (cheaper)
  - Goal: something someone can try right away
- KubeCon sync up for OS folks
- #wg-sp-os holds Flatcar folks
- Looks like https://github.com/cncf/toc/blob/main/tags/tag-workloads-foundation/README.md technically would have bootc in scope
