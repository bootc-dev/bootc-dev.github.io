# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## 15 Aug 2025
### Attendees
- Colin Walters
- John Eckersberg (Red Hat, Inc.)
- Joseph Marrero Corchado (Red Hat, Inc.)
- Mohan Shash
- Michael Fox
- Preethi Thomas
- Robert Sturla (Tesco Bank)
- Hristo Marinov
- Laura Santamaria (she/her; Red Hat)
- Johnathan Lebon
- Sean Thrailkill
- Gursewak Mangat
- Dusty
- Michael Tunnell (Framework)
- AfterStory
- Jorge Castro (CNCF/Linux Foundation)

### Agenda
- [Jonathan] bootc plugin system
    - [Jonathan] What do we want the UX to be like? Two high-level approaches: dnf drives bootc, or bootc drives dnf. This is currently Fedora focused, but implementation details is not Fedora focused. If dnf does persistent package layering, goal should still be bootc upgrade and bootc status work and that's the user. For that to work, bootc and dnf need to work together, hence plugin system. How do we have bootc communicate with other applications (this is where it's non-Fedora centric). Logic shouldn't likely live in bootc, right? What's the plugin API look like?
    - [Colin] Wasn't there an issue about this?
    - [Jonathan] context the first time was originally rpmtree
        - {%preview https://github.com/bootc-dev/bootc/issues/337 %}
    - [Colin] Complicated. Think about progress reporting, especially as GUI. We want a progress bar, coherent representation of what's happening. Want accurate change detection. Want things visually simple to understand for user. If bootc forks off other binary, punt the issue of how representation of state works. Inclination is to punt to the tool; make it the job of the higher-level tool.
    - [Jonathan] Going to server model, you would do dnf status, for example, and dnf upgrade?
    - [Colin] Yes, can't avoid the tools and need to know about both in some cases. Could cover a lot of cases if we taught the package managers how to manage a sysext that they own locally. There's already a lot of tooling to introspect sysext.
        - {%preview https://github.com/rpm-software-management/dnf5/issues/1731 %}
    - [Dusty] if we make it generic (e.g., local layering), local packages is one plugin that's really important. Being able to add a package that's not in the thing that you're following is extremely convenient. In most cases, there's a package manager tool that you could make plugins, and tool itself could learn how to do thing in sysext, then bootc doesn't have to care (or maybe it does?). How many things do people have to touch to upgrade system? What's the better user experience? If you step away from package managers/systems, then that's where it gets iffy. Might make more sense to make bootc the one that controls that instead of having dracut try to call bootc. Might be better bootc managing as plugin way.
        - Bootc notices update, grabs it, then pipeline - output is input to next step. Output of pipeline is from plugins. Idea is more that mutiple plugins can stack. Or can combine to multistage build (yay pipelines).
    - [Colin] Wary of adding more complexity to bootc right now. Issues in place, example: We don't have unified storage (podman build/docker build is super awkward at the moment). There's prepwork we need to add. Someone else might need to own that task. Still problem of non-package content.
    - [Sean] Inclined to agree. Keep bootc simple; teaching other tech to know how to layer versus trying to edit the image that is actually booted (and security implications for that).
    - [Dusty] Use case thinking about is in a scenario where someone is autoupdating their system. e.g., normally they don't touch it. How can they keep the things that they have done to keep everything up to date? That's why like the plugin model. Then other things get updated alongside the upgrade that happens (e.g., the user who ran a command years ago and doesn't remember that command). How to keep config that was made up to date, etc. Have same problem today in ostree world where there's package layering, e.g., orphaned packages.
    - [Jonathan] Package layering is a big one as it's a common want. When you boot up a bootc system, you use bootc to manage it, from point on to add package, then you stop touching bootc and have to know other tools. Introduces split brain. Is bootc an implementation detail of dnf? Feels awkward. Would be better that bootc knows the state of the system. Make bootc the entrypoint to managing your system.
    - [Dusty] Would be nice if someone runs bootc status and copy-paste to issue filed, then shows full system. Would be helpful to have single pane of glass.
    - [Laura/Colin] Let's move to an issue for more discussion. Jonathan to make the issue.
- [Mohan] Follow up - when is the site going live: https://github.com/bootc-dev/bootc-dev.github.io? 
    - [Laura] The site is live at https://bootc-dev.github.io, but are you asking about moving it to bootc.dev?
        - [Mohan] Thanks, I wasn't aware that it was live already :-)
- [Robert] what info should we get from users who hit this? {%preview https://github.com/ostreedev/ostree/issues/2283 %}
    - [Robert] One of the most annoying bugs we face. No easy way to resolve when it happens, no reproducable steps. What can we do to help debug?
    - [Colin] Don't know offhand. Could ask folks to share journals and get more logs. AI version churning on that.
    - [Hristo, in chat] {%preview https://discussion.fedoraproject.org/t/silverblue-wont-boot-after-forced-shutdown-searching-for-ways-to-recover-it/155432 %}
    - [Jonathan] Catchall issue. Symptoms look the same, but not same root cause. When initially filed this, cause was people doing reinstalls and they had old install on disk one but new on disk 2, then partitions got the bootloader confused. Another way this can happen is if something during shutdown procedure didn't get transactionally logged. Really hard to debug. Having steps to reproduce is really key.
    - [Michael, in chat] {%preview https://github.com/ublue-os/bluefin-lts/issues/658 %}
        - "Apparently the bluefin is related to btrfs + bootc"
    - [Hristo] Spent a long amount of time trying to reproduce on baremetal machine; in all states, shutdown machine with power switch. Cannot reproduce this issue. Mentioned this in the Fedora discussion thread. But needs that reproduction details. Seems very difficult to reproduce.
    - [Laura] Can we make a big issue to collect everything?
    - [Colin] Sure?
    - [Jorge] Everyone is blaming btrfs.
    - [Dusty] Also discussed on LinuxUnplugged. Seems to only be on 16.3
    - **ACTION**: Let's make the catchall here: {%preview https://github.com/ostreedev/ostree/issues/2283 %}
- ~~[Robert] Systemd Boot - can we start with a hidden flag passed from bootc? https://github.com/coreos/bootupd/pull/978~~ Can move to discussion in the PR
    - [Laura] Thank you! Please bring it back up again next week if the discussion needs to be live!

### TODO:
- [ ] Jonathan to make an issue for the plugin discussion to continue offline.
- [ ] Laura to find out how to get the bootc.dev CNAME setup
- [ ] All to drop catchall for the persistent bug in the issue
