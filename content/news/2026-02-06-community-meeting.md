+++
title = "Bootc Community Meeting Notes - 06 Feb 2026"
date = 2026-02-06
slug = "2026-feb-06-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#user-content-code-of-conduct) applies.

## 06 Feb 2026
[Zoom summary](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093-1770391800000/summaries?password=cc5e77d7-e34a-4b74-9033-c4f1f38bcd99)

### Attendees
- Laura Santamaria (Red Hat)
- Sunny Fugate
- Hristo Marinov
- Preethi Thomas (Red Hat LLC)
- Joseph Marrero Corchado (Red Hat)
- Jonathan Lebon
- Anish Bhatt (Salesforce.com, Inc.)
- Colin Walters
- John Eckersberg
- Dusty Mabe

### Agenda
- [name=Joseph] Talk at CentOS Connect - FOSDEM Fringe [YouTube video](https://www.youtube.com/live/CNOGKUTYhng?t=11008s)
- [name=Colin] {%preview https://github.com/containers/composefs-rs/pull/224 %} will do a deeper dive on Monday
- [name=Laura] Calendar updates
    - Dev/Engineering Sync on Mondays and Wednesdays: [bootc project calendar](https://zoom-lfx.platform.linuxfoundation.org/meetings/bootc)
- [name=Colin] CI fixed up
- [name=JL] chunkah update!
    - {%preview https://github.com/jlebon/chunkah/pull/34%}
    - [name=Joseph] Is this added to the CoreOS CI yet?
        - [name=JL] Not yet. Convincing it's working and analyzing it. Need to make CI better first. Big gap: One principle of chunkah is not touching the rootfs, and there's no real CI for that (no CI that does a rootfs diff).
        - [name=Colin] Why harder to mount? (Answer: Not in CI yet) This could be trivial in the bootc CI.
        - [name=JL] Where would we do this? Upstream bootc CI setup?
        - [name=Colin] yep! Used to just build a binary, but got burned by that because previous container image had state leakage. Had to switch to always building an RPM... Then we do a from-scratch build as an override. So single layer. Should be able to toss chunkah in there!
        - Justfile as entrypoint
- [name=Anish] Is there a good place to read up on chunking?
        - [name=John] This is a good place to read up on the current state with rpm-ostree: {%preview https://docs.fedoraproject.org/en-US/bootc/building-from-scratch/#_optimizing_container_images %}
        - [name=JL] For the newer way that goes past rpm-ostree, read the chunkah readme: {%preview https://github.com/jlebon/chunkah %}
- [name=Laura] Shoutout time :)
    - [name=Dusty] regarding the review request and packaging reviews, could use some AI rubbed on it :D
- [name=Laura] Welcome to new folks!

### Shoutouts
- @jeckersb for reviewing chunkah Review Request!
- @joseph for the FOSDEM talk!
