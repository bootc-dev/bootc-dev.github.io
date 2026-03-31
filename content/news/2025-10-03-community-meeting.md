+++
title = "Bootc Community Meeting Notes - 03 Oct 2025"
date = 2025-10-03
slug = "2025-oct-03-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## 3 Oct 2025
### Attendees
- Robert Sturla (Tesco Bank)
- Laura Santamaria (Red Hat; she/her)
- AfterStory
- Tulip Blossom (Chainguard; she/her)
- Colin Walters
- Mohan Shash
- Hristo Marinov
- John Eckersberg (Red Hat)
- Gursewak Mangat
- Chris Kyrouac
- Jonathan Lebon
- Dusty Mabe (he/him; Red Hat)

### Agenda
- Laura: Intros, note use of CNCF code of conduct.
- [Mohan] Demo videos to be uploaded in CNCF channel. Goal: Re-use the demos, reference in docs etc..
    - Record and share your demos. Laura is helping us with setting up the access to post the videos on CNCF channels
    - [Laura] Ticket is in (link?), who is a maintainer and wants access?
        - Note this has to be someone who is a maintainer: {%preview https://github.com/bootc-dev/bootc/blob/main/MAINTAINERS.md %}
        - [Colin] can we all just do it? [Laura] Sure, but remember you're going to be responsible :)
        - [Colin] How about we go over videos in this meeting? (or in slack?) Basically just a mechanism to review things
        - Decision: All have access, maybe a review call after community meeting.
- [Robert] Official E2E examples/templates for different use-cases?
    - https://cloud-native.slack.com/archives/C08SKSQKG1L/p1759441407750519
    - Currently have [Fedora Bootc examples](https://gitlab.com/fedora/bootc/examples), but these are only the Containerfile definitions.
    - Jorge suggested adopting the [Universal Blue Image Template](https://github.com/ublue-os/image-template).
    - Other enterprise-ready examples could include workflows for testing with [bcvk](https://github.com/bootc-dev/bcvk) and [test.thing](https://codeberg.org/lis/test.thing) and additional tooling for security gating and compliance requirements.
    - [Colin] Base images templates make total sense!
    - [Colin] {%preview https://github.com/ublue-os/image-template/blob/main/Justfile %}
    - [Dusty] Do they build just containers or boot images?
        - [Robert] Good to have examples of both of them.
    - [Laura] What about a cookie cutter and then an awesomelist?
        - Another tool that is used to build disk images (I think in SUSE and some in Fedora) is kiwi, so if support was added there it would be a good one to add to the awesomelist.
    - [Afterstory in chat] I remember a chat with wanting to move/have a equivalent uBlue image template on bootc org repo
    - [Tulip in chat] I think it would be interesting to have integration with the ArtifactHub (https://artifacthub.io/) project for discoverability
    - {%preview https://github.com/topics/bootc %}
    - [Robert] What about a reference image for enterprise? or out of scope?
        - [Colin] Related to enterprise - disconnected environments. Would make total sense though. There's a lot in this space ([flight control project](https://github.com/flightctl/flightctl/) as one example). Some people want an opinionated system that pulls in bootc as one part; that's an interesting one to consider for a reference image/awesomelist.
        - [Robert] Yeah, flightctl does safe rollout and automated way, so would be good to show how to implement flightctl with bootc
- [Colin] composefs status!
    - Making some progress!
    - Would love to have people come work on it together
    - Multistage image for base image
    - Cleaning up the build system stuff and hoping to land the testing and things there. John is working on the build side.
    - Anyone welcome to do live chat system!

### Shoutouts
- Sean is doing a workshop at Texas Linux Fest today!

### TODO
- [ ] Laura to send replies to Riaan re: channel maintainers
- [ ] Laura to connect with Mohan re [community manager role](https://github.com/bootc-dev/bootc/blob/main/MAINTAINERS.md#community-managers)
- [ ] Open ticket for awesome list
- [ ] Make the repo for awesomelist
- [ ] Kick off templates
- [ ] If you want to work on composefs, chat in the bootc-dev channel and do some live coding

