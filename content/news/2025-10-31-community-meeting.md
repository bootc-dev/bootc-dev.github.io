# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## 31 Oct 2025 (Happy Halloween! :jack_o_lantern: :ghost: :skull: )
### Attendees
- Colin Walters
- John Eckersberg
- Mohan Shash (RH)
- Laura Santamaria (I'm a teapot)
- Robert Sturla (RH)
- Jorge Castro (CNCF)
- Tulip Blossom (Chainguard)
- Joseph Marrero Corchado (RH)
- AfterStory
- Sean Thrailkill
- Hristo Marinov
- Gursewak Mangat
- Preethi Thomas (RH)

### Agenda
- [name=Jorge] Who's going to kubecon? [Colin, Joseph, Laura, Jorge, ...]
- [name=Jorge] Reference Architecture for bootc
    - CNCF projects have a reference arch, usually published by end users. e.g., how Adobe uses Argo. {%preview https://architecture.cncf.io/ %}
    - Approaching teams to post their architecture
    - Working on a way to have talks that are reference architectures highlihgted more during keynote and during track (more likelihood of talks accepted)
    - Proposed: KubeCon NA 2026 - submit two ref archtiectures. First, for Red Hatters, find bank or someone who is doing it with RHEL Image Mode and write it up. (Prefer pairup with already end user company).
    - Bazzite: BK and Kyle - kyle works on rpm packaging at MS, and is a great presenter!
    - Same for podman and buildah - WinBoat for Podman (https://www.winboat.app/; https://github.com/TibixDev/winboat)
    - [name=Mohan] How does this write up work?
        - [name=Jorge] CNCF will work with an end user - definition is partners on the landscape
        - [name=Jorge] Also think about other users like bootcrew
- [name=Colin] KubeCon agenda now? Or a breakout in chat actually probably
    - Let's discuss in chat :) Avoid FOMO
- [name=Colin] Helping mentor contributions more reliably/regularly
    - Really feel like the 1:1 and smaller meetings are effective at getting everyone space to talk.
    - Rotation in the channel for office hours
    - [name=Laura] Yes, this is common :) Other projects do developer office hours.
    - [name=Jorge] or can tack to end of this meeting
    - [name=Colin] Maybe move this one earlier?
- [name=Jorge] demo (have me go last)
    - finpilot
    - {%preview https://github.com/castrojo/finpilot %}

### Shoutouts
- [name=Jorge] Bluefin was featured in a [GitHub Universe Keynote](https://youtu.be/q1IxyisKcZI?si=zHNrlBxFkQnXLQos&t=1322) - the two people who did that last 20% are first time contributors and are now part of the greater bootc community. THANK YOU SO MUCH.
- [name=Tulip] for https://github.com/bootc-dev/homebrew-bcvk
- [name=Sean] Colin got podman working with codespaces!
- [name=Jorge] bootc-image-builder new ISO support https://github.com/osbuild/bootc-image-builder/pull/1094 (still in flight)

### TODO
- [ ] Can someone add a timedate.com widget/link to the community meeting in the readme?
