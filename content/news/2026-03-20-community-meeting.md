+++
title = "Bootc Community Meeting Notes - 20 Mar 2026"
date = 2026-03-20
slug = "2026-mar-20-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## 20 March 2026
### Attendees
- John Eckersberg
- Sean Thrailkill
- Joeseph Marrero Corchado
- Hristo Marinov
- Anish Bhatt
- Colin Walters

### Agenda

- https://github.com/bootc-dev/bootc/pull/2071
    - [name=Colin] Do we hold on merging this until we determine the correct scope for testing transient root?
    - [name=John] Not for a bug fix, which is the purpose of this MR
    - MR was merged by Colin!
- Discussion of testing: Expanding the matrix: unified storage
- Agreement to extend more testing in postsubmit and also gate for releases
- Debugging https://bodhi.fedoraproject.org/updates/FEDORA-2026-cfa95147df
- https://github.com/containers/skopeo/pull/2714
    - We have `--enforce-container-sigpolicy` for `bootc switch` that would have to be updated to support this work
    - Do we create a Bootc config file where users could specify that? Don't want to recreate in file form what we already do for `podman` command
    - OCI Sealing enforces signature and everything else we want
    - There are some edge cases that need to be considered but this seems feasible

### Shoutouts!
- Giuseppe is working on unified storage related things!
- Ublue and secureblue!
