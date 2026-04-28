+++
title = "Bootc Community Meeting Notes - 16 Jan 2026"
date = 2026-01-16
slug = "2026-jan-16-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#user-content-code-of-conduct) applies.

## 16 Jan 2026
[Zoom summary](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093-1768577400000/summaries?password=763143f2-7c8e-4e2c-81ca-5966f0c6ba0c)

### Attendees
- Colin Walters
- Laura Santamaria (Red Hat; she/her)
- Joseph Marrero Corchado (Red Hat)
- John Eckersberg (Red Hat)
- Chris Kyrouac
- Jonathan Dieter | CIQ
- Sean Thrailkill
- Mohan Shash
- Dusty
- Preethi Thomas

### Agenda
- [name=Colin] Review pending release notes in https://github.com/bootc-dev/bootc/releases/edit/untagged-cd5086ebeadb8bb2bcfb and release ceremony
    - Colin showing the draft
    - General format: Taking autogen release notes, tweaking it and adjusting with an AI agent. Features, bugfixes, and other changes.
    - Now everyone reads!
    - [name=Joseph] Making sure ostree fix is in
    - [name=Colin] A lot has happened in the composefs side. A lot pending, so roll into next release.
    - [name=John] Since patch release, should we have branched from 1.12 tag with just bug fixes?
        - [name=Colin] Could do that, but no procedure for that right now b/c tag is branch is release.
        - [name=John] Something to consdier/asipre for
            - [name=Colin] Could consider that with sufficient automation. Discuss as wider group. People want the newer features, too, though; everyone wants the features without the bugs :sweat_smile: 
    - Let's do the release! Aaaaand done
- [name=Laura] kubevirt+bootc (https://groups.google.com/g/kubevirt-dev/c/K-jNJL_Y9bA/m/ZTH78OqFBAAJ ) also https://github.com/bootc-dev/bcvk/issues/22
    - [name=Colin] The thing to connect with is asking what is the podman and rhel 9.6 do in this situation... bcvk ephemeral is so nice. Nice to spin up as if vm but not actually vm. Connection with kubevirt side is... Meme about cattle vs pets, but argues that not true in actuality. Kubevirt side; would make total sense to expose model of ephemeral - mount container's root but use it running kernel in the container but ... kata containers... Laura got lost :p
    - [name=Laura] Interesting idea, also relates to kairos.io which is in CNCF. (Like KubeVirt which is CNCF (incubating)). We should try to build more cross-collaboration with others in the org.
    - [name=Laura] Congrats for accepted ContribFest! If you're going to KubeCon EU let's make sure it works! This is a prime spot right before lunch when folks are excited (and available?) We should be ready to help people at that time.
    - [name=Laura] Kiosk is approved (prime spot!)
    - [name=Joseph] Lighting talk is accepted too!
    - [name=Laura] Now we need to write the talks! Let's collab in Slack calling out Sean!
- [name=Laura] Fedora (Atomic) "midstream" initiative is accepted!
- [name=Colin] Lots of groups working in various spots. e.g., Image Builder team midstream/downstream, and e.g. Fedora CoreOS. Consider consolidating :) There's a lot of things that are moving parts.
- [name=Laura] Agree lots of interesting things happening thing there! Should be enabling drive-by contributions. Also need to think about bootkrew and Bazzite e.g. and help we're enabling them but also ensuring their work is generalized.
- [name=Laura] As we chat with Kairos etc. should think about how this applies to moving to Incubuation which should be a strong possibility. Should keep this in mind!
- [name=Colin] Vaguely thinking about inviting other folks to work on repos and ensuring that works well
- [name=Laura] Should have a contributor ladder, clear guidelines on maintainer/reviewer promotions. And everyone should fall under the same ladder. Consider an OWNERS/CODEOWNERS style model that helps add people to review rotation. Any folks interested can join that conversation around how the ladder looks! Or we can move slowly. Beginning of the year so we're still gaining speed.
- [name=Colin] likes the Rust bors @delegate approach (edit; https://github.com/bootc-dev/infra/issues/83)
- [name=Laura] can add people lightweight temporarily - we want to enable drive-by contribution, but also convert drive-by to a more regular contributor.
- [name=Laura] Zoom code reviews and group invite - helps everyone learn and how to contribute. Can do this for ContribFest too!
- [name=Laura] Time to end!

### Shoutouts
- All the people who worked on KubeCon EU proposals

### TODO
- [ ] Laura to drop docs
