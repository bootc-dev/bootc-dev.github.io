+++
title = "Bootc Community Meeting Notes - 23 Jan 2026"
date = 2026-01-23
slug = "2026-jan-23-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#user-content-code-of-conduct) applies.

## 23 Jan 2026
[Zoom Summary](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093-1769182200000/summaries?password=6a248ea5-6a5d-4433-a7b1-cd97d5aa4f86)

### Attendees
- Colin Walters
- Laura Santamaria (she/her; Red Hat)
- Hristo Marinov
- Chris Kyrouac
- John Eckersberg (Red Hat, Inc.)
- Mohan Shash
- Dusty
- Jonathan Lebon

### Agenda
- [name=Laura] Snow in the US! Texas is losing their minds!
- [name=Laura] Welcome!
- [name=Mohan] Zoom summary should be posted
- [name=John] Composability of package sets
    - Chunkah! Baby steps :)
    - If we have package sets like A-Z, and we want to compose them in any sort of permutation without blowing your network transfer, how could we go about doing that? Problem that has come up; something to chew on.
    - Stitching the rpm layer later?
    - [name=colin, in chat] https://grahamc.com/blog/nix-and-layered-docker-images/ ?
    - [name=Jonathan] Been thinking in this area with chunker work. Needs more context on what exactly the problem is. Is there a link with details?
    - [name=John] No link. Still working on it. If fleet of hosts with different package sets, overlap of packages. End up with distinct sets. How to proliferate with distinct container builds (which blows out storage with duplicated package data)
    - [name=Jonathan] Is this image mode or also apt container images?
    - [name=John] w/in bootc
    - [name=Jonathan] most optimal strategy is no combining at all. More possible for apt containers. e.g., Hummingbird. Some have 20 rpm; can get perfect de-dup, but gets trickier for apt containers beyond a certain size. Rare to have minimal set to fit within constraints of containers image (500 layers is arbitrary limit). One Silverblue - ~900 rpms. So need a strategy. Also layers can be too many to manage. If history of updates, can look at analysis of optimal way of grouping. Kinda like with `chunkah`. Pre-compute different versions of packaging that we could have done in the past... will get complicated, wouldn't recommend. Try to spread out as much as possible.
    - [name=John] May be just rolling bulk into one layer... could be problem. How close do you come close to overlay limit.
    - [name=Jonathan] Not close to limit for kernel; hit container runtime limit first.
    - [name=John] Container build where segmented and de-dup'd layers. Different rpm database layers at the end...
    - [name=Jonathan] RpmDatabase unstable and will always change.
    - [name=Dusty] Hardcoded limit. Can't go above that. If no history (or too complicated to look at) could we chunk smaller things together? Large grouped by srpm, then small things on kb level, group together. Will change a lot, but could avoid problems.
    - [name=Jonathan] Will link to what `chunkah` does. Used the rpm changelog to figure out measure of stability of a package. Stability and size relates to packaging. https://github.com/jlebon/chunkah/blob/main/src/packing.rs
    - [name=Dusty] Stable is how often changes, not quality of software
    - [name=Colin] Original rpmtree debated this a lot. NixOS blog is really useful to read. Keeping it simpler is good. For bootable images, it's easy 30% of size just on its own.
    - [name=Jonathan] The linked algorithm finds out those things on its own. Emerging behaviors from algorithm like this.
    - [name=Dusty] Chunking stuff we use today is built into bootc?
    - [name=Colin] The rust code is in bootc, logically exposed through rpm-ostree
    - [name=Dusty] But already in the base image, right? So when doing container build, it's a second step (chunking code). For `chunkah` specifically, not already in base image. Should we try to replace the old chunking code with new chunking code so it's just there versus conscious decision to just use it?
    - [name=Jonathan] Definitely planning to package. It's another input to the container image that they have to track. Would be nice in builder image used in pipelines like Konflux. Haven't really gotten to this part yet :smile: Chunkah - not just about bootc, works with apt containers
    - [name=John] Like having it as a separate thing. Colin did the sealed building thing. Tools needed at build time but don't want to include in image. Might be a good spot for `chunkah`. Don't want `chunkah` installed on final system.
    - [name=Dusty] That's where the current chunking code wins out. Could remain separate and pull in as library. Could be an option; avoids decision for end user.
    - [name=John] Accumulating list of container build time things. So not difficult to add to list.
    - [name=Colin] write up a productization plan for `chunkah`. Should help a lot.
    - [name=Jonathan] Maybe move to containers org.
    - [name=Colin] Does
- [name=Laura] FYI, SCaLE talk/Fedora Hatch
- one PR to bootc base images related to a packaging change in Fedora: https://gitlab.com/fedora/bootc/base-images/-/merge_requests/350

### Shoutouts
- :tada: Pragyan for lots of test fixes! :tada:
- :tada: Jonathan for `chunkah` :tada:

### TODO
- [ ] Laura to ask the projects team about org and repo

