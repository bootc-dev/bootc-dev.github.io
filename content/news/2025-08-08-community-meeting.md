# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## 8 Aug 2025
### Attendees
- Robert Sturla (Tesco Bank / Universal Blue)
- Hristo Marinov
- Laura Santamaria (she/her; Red Hat)
- Colin Walters
- Joseph Marrero Corchado (Red Hat)
- John Eckersberg (Red Hat, Inc.)
- Chris Kyrouac
- Dusty Mabe (he/him; Red Hat)
- Gursewak Mangat
- Jonathan Lebon
- Sean Thrailkill
- afterstory

### Agenda
- [Colin] More on composefs {%preview https://github.com/bootc-dev/bootc/issues/1498 %}
    -  [Colin] Clientside reworking on how we store files. Then there's the build side to this whole thing. What's the user experience look like for creating an image
    -  [Colin] Recap from previous meeting
    -  [Colin] Fatal flaw of binary copying into host
    -  [Colin] Issue is about build side
    -  [Colin] Logistically simplest - containers build however you want, bootc cleans up, then user provide secure boot key
    -  [Jonathan] Not following super closely, but feels like we're starting with the hard case. Any work around non-sealed composefs builds today? Does it work?
        -  [Colin] Yes, kinda. "It Depends":tm: Gets into whole problem around how we transition installs. Doesn't need to be sealed, but only target UKIs to start. We get into mechanical issue of default Fedora base images aren't set up for UKI. ostree backend does not really support UKIs (at least not with UEFI). Yes, it definitely will support nonsealed systems. Assuming we want to run this tool on your container images anyway.
        -  [Jonathan] Need to make sure that, when designing, want to consider how migrate existing systems, and how will work with unsealed systems. Sealed case may be a specialized thing in beginning. May affect design; overconstrain.
        - [Colin] We will definitely support transitioning existing systems. In end, can always boot without secure boot, too.
        - [Colin] not committing to anything; this is still experimental. Nothing stops us from allowing flow within Dockerfiles, too.
    - [John] Haven't really explored it a whole lot. May change, and likely will hit many walls. Just wanted to start discussion. Please feel free to comment in issue! Help find flaws in plan :)
    - [Colin] Want people to feel free to ask questions in this meeting! Don't be afraid to ask for clarity, ask to help contribute.
- [Sean] Recently gave talk at Flock about state of bootc! Big fan, want to get involved. We've shifted away from how installation process used to work. Used to be build your artifacts, now transitioning to replace the existing install, especially in more cloud-centric areas. What was the impetus for the change in direction?
    - [Colin] Very interested. Wouldn't say it's backing away. We know we have to support a flow where you have a container image, then allow to turn into ISO or raw disk file. Too many use cases for that to give it up (e.g., edge, IOT devices, want to preconfigure before ship). We have to support that. Tension is in some clouds, managing disk images kinda stinks. Esp. AWS, as an example. There's two different worlds, and we do need to do both.
    - [Dusty] bootc image builder supports uploading AMI for AWS, but usually you have to figure that part out on your own. Now also options to boot and then replace. That's more of a "let's remove the extra step from the user's responsibility, allow for existing thing that can be paved over." Might make it easier.
    - [Sean] Makes sense. Sounds like this is to make it simpler for the user; pave over idea makes sense for the user.
    - [Dusty] idea is they already have podman, which can run the container than can then pave your system. Bootc is on engineer's device. Positives and negatives. Positive: You start with whatever image for OS is on your cloud, and then you have to rebase that instance to your container. Negative: Feels like there's a new cloud that pops up everyday. Feels like we're constantly chasing the ball. Better experience is wherever you are is a starting point. So this allows to start from *something* you can start from. Doesn't even have to be EL-based. Can be Ubuntu. Can get to success without having to create your own disk image.
    - [Colin] Not an installer in the partitioning sense. We want to make it easy/happy path for someone making own OS or distro - closer you can get to OS is container, then the installer area can be smaller. Bootc doesn't know how big you want your partition to be, for example, but if you can use whatever installer to install your containerized OS, then you're in a better space. Freya Labs also investigating bootc, called readymade.
    - [Laura] I'll see if I can find them. (chat noted that they're in the universal blue discord)
- [Jonathan] Choice of filesystem in container image and possibility of putting the choice of the filesystem in the image itself
    - [Colin] Would like to support systemd-repart as part of MVP first, possibly able to support something like this later.
- Lots of talk about new friends, inviting new friends, and timezone issues (we know this isn't great for Europeans and others.)
- Discussion of recording meeting, general agreement we should probably record and post
 - One option: Split 50%/50% recorded vs not, the advantage of this would be that the second half can be for "stupid questions" that people may not be comfortable having on the Internet Forever
