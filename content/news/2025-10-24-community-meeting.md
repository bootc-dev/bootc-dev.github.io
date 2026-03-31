+++
title = "Bootc Community Meeting Notes - 24 Oct 2025"
date = 2025-10-24
slug = "2025-oct-24-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#user-content-code-of-conduct) applies.

## 24 Oct 2025
### Attendees
- Colin Walters
- Robert Sturla (Red Hat)
- Sean Thrailkill
- Hristo Marinov
- Laura Santamaria (Red Hat; she/her)
- Preethi Thomas
- Mohan Shash (RH)

### Agenda
- {%preview https://github.com/bootc-dev/bcvk/pull/86 %}
    - Looking for feedback
    - Improving iteration speed for testing for humans and AI (autonomous changes). Also dealing with technical debt.
    - Demo!
        - This is inspired a bit by Vagrant. There's a new project verb that has a lot of syntactic sugar for bcvk. (Note that this is mostly generated via an AI agent, but Colin reviews all things before committing.)
        - The idea is in other libvirt verbs, you have to give it to specific VMs
        - This one scopes to specific repo
        - Refers to locally built bootc image
        - Initializes project, allows you to find local VM; caches disc images (cached base image corresponding to base image built; or could just build)
        - The sugar is autobind mount the host container storage
        - Now, when ssh in, you can -a to autoupdate
        - If rebuild on host the container image (build new bootc image from code from repo), then bcvk project ssh -a, then inner VM fetch just new layers and will allow you to ssh in and try out new code
    - [Sean] Awesome on how easy this is. Allows all kinds of users to do all kinds of cool stuff.
    - [Colin] not just for devs, you can do this for things like tweaking firewall rules - iterating locally and then pushing to cloud or metal. CDK replaces podman-bootc. Trying also to wrap bib as well.
    - [Colin] Several ways to do installs, but day2 stuff should all be the same. So if wrap Anaconda, then can do the local runs and libvirt VM with all the sugar added.
    - [Sean] Why start developing this tool versus updating podman-bootc?
        - [Colin] Podman-bootc had some technical debt. Fundamental redesign was because of requiring podman machine on Linux, which causes cascading problem.
        - [Colin] There's some vendoring that hops us on the CVE train
        - [Colin] So instead, this just forks Podman and Rust things, so no bundled co-libraries to deal with layered CVEs
- [Mohan] Working on first video edits. Hopefully will have a cool new video on this wink wink
- [Robert] Wants to enforce bootc switch globally config option for universal blue.
    - [Colin] {%preview https://github.com/containers/container-libs/pull/355 %}
    - [Colin] Jonathan was working on this for CoreOS. Was this the same thing?
    - [Robert] Haven't seen this yet, but looks similar on first quick look
    - [Colin] This links to Skopeo issue 309, which is this thing. Plumb this through into Skopeo proxy then plumb into bootc. Want to align how bootc feels and works to other container runtimes feel and work, so should enforce sigs on logically bound apps and stuff, too. Maybe once library side of this finally merges, then we should talk about whether this is a CLI flag or global, right?
    - [Robert] Would be good to have it be global. Secure by default design. Currently no method for global config save the thing from ostree.
    - [Colin] We should be posting the same policy json. Just drop out insecureAcceptAnything default should do it.
    - [Robert] This should affect the host image and the podman image right?
    - [Colin] Yup, the whole family of tools
    - [Robert] Not everyone signs...
    - [Colin] This came up in a different context (may need docs). Short answer no. If run bootc as systemd unit, could easily use bind paths. Not the most beautiful thing and would like first class support, but that's the status quo on it right now. Might be an env var to change where policy.json is searched for. Agree on usecase, though. Want to get that in :)

### Shoutouts
- Shoutout to Robert for joining Red Hat!
- Thank you to Jonathan Dieter and Eli from CIQ for meeting with Sean and getting bootc images out for Rocky! [Link](https://git.resf.org/sig_containers/rocky-bootc)
    - Shoutout to Sean for having those meetings!
