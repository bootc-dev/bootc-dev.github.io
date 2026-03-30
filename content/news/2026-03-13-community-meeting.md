# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## 13 March 2026
### Attendees
- Colin Walters
- John Eckersberg
- Laura Santamaria
- Sean Thrailkill
- Chris Kyrouac
- Joeseph Marrero Corchado
- Mohan Shash
- Prasanth Baskar (8gears AG)
- Anish Bhatt (Salesforce.com, Inc.)
- Preethi Thomas

### Agenda
- {%preview https://github.com/bootc-dev/infra/pull/137 %}
    - [name=Colin] Trying to describe the desired state. CI is really hard to deal with. The aspirational goal <-- described here. Please read!
    - [name=John] Motivation +1, really need to get everything set up and standardized and DOCUMENTED is really nice
    - [name=Colin] some things were missing renovate, for example
- [name=Colin] Adding bootcrew to linked infra
    - Would love to include in orbit, so in composefs we use the debian bootcrew image in integration testing. So debian bootcrew needs to be maintained for the project. Did a PR to sync renovate config in composefs org.
    - Would like to file an issue.
    - Bring gating on bootcrew images into the main bootc dev CI. Will expand matrix.
- [name=Laura] Summary of SCALE talk, demoed switching between Atomic Desktops (w/rpm-ostree). Well received.
- [name=Laura] Justin Garrison gave state of immutable talk (compared ostree vs other toolchains) - more neutral vs positive. Disagreed with some technical decisions but he is running a bootc system. (TODO find links to talk here)
- [name=Laura] A lot of potential new users asking questions, Laura also brought a ublue Aurora
- [name=Laura] Josh Berkus also did a talk on custom images and installing on bare metal. Anaconda workflow could be improved
- [name=Prasanth] Arch bootcrew expanding on memory and in storage on memory. Thoughts?
    - [name=Prasanth] Using gitlab private workflows with a bootc image (base k3s distro for test pipeline). Takes too much storage. Anything they can do?
    - Will be at KubeCon and can talk about it.
    - Registry server or per device?
        - Image gets bigger because adding more packages over time. Eating the registry's s3 bucket storage
        - There's no one solution to this. But, there's Chunkah! https://github.com/jlebon/chunkah. Best practice is to try to split out large layers into their own isolated things to avoid cascading changes. Avoid bundling configs with packages. Chunkah will resplit everything up and canonicalize it.
        - [name=John] Also, you're likely using the composefs backend. Until very recently, it didn't have garbage collection. But as of the release this week, it has garbage collection! So try updating bootc and composefs and see if it cleans everything up

### TODO
- [ ] Reach out to Tulip on Slack <-- Colin
