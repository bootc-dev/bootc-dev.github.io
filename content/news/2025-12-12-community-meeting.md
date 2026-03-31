+++
title = "Bootc Community Meeting Notes - 12 Dec 2025"
date = 2025-12-12
slug = "2025-dec-12-community-meeting"
+++

# bootc community meeting

[Meeting Link/info](https://zoom-lfx.platform.linuxfoundation.org/meeting/96540875093?password=7889708d-c520-4565-90d3-ce9e253a1f65)

Feel free to add yourself to the attendee list and to add to the agenda! This is an open community meeting, and our [Code of Conduct](https://github.com/bootc-dev/bootc?tab=readme-ov-file#code-of-conduct) applies.

## Dec 12 2025
### Attendees
- Laura Santamaria (Red Hat)
- AfterStory
- Robert Sturla (Red Hat)
- Hristo Marinov
- Colin Walters
- John Eckersberg (Red Hat, Inc.)
- Preethi Thomas
- Joseph Marrero Corchado (Red Hat, Inc.)
- Dusty Mabe
- Mohan Shash
- Sean Thrailkill
- Tulip Blossom

### Agenda
- [name=Colin]: more strongly enforcing review/assignment
    - Tried the autoassign thing, but couldn't figure out how to get everything into priority queue. Hard to be consistently triaging when bot assigns PR
    - e.g., PR 1846
    - missed being assigned to reviews
    - Slackbot seems overkill
    - Maybe have a private message thing?
    - [name=Laura] what about a weekly digest post?
    - [name=Colin] yeah, there are options. But we need to find a good way to review.
    - Also, 1570, what do we do with non-critical paths?
    - Can we find a good way to efficiently prioritize
    - [name=Joseph] a lot of the PRs have someone who already commented
    - Do we ignore PRs if someone/a maintainer already looked at them, even if they were auto-assigned to someone?
    - [name=Colin] Great question. Probably make sense to change the automation to reassign if a maintainer adds a review. Only need 1 person to review, so auto-assigned should get unassigned. But note that sometimes Colin ends up reviewing a lot.
    - [name=Joseph] didn't know your workflow notified you, but it would be good to reduce load on Colin/others who review all the time. Unless deeply care about PR someone is working on, then let it go
    - [name=Laura] Rackspace group used Zoom review, PR author has to talk through code while someone else drove screenshare. Helps onboard new contributors. This helps reduce "single person knows this code" and knowledge sharing. Would help get people more comfortable with the codebase. Laura will add a link with more info!
        - {%preview https://gist.github.com/parlarjb/07aeb0efafde2a23fc43dfdd5be11c2e %}
- [name=AfterStory] generic rechunker
    - [name=Colin] The person working on it will probably be posting soon. It's a soft blocker for composefs rework. When doing images from scratch without ostree. Is a goal to make it distro agnostic, make it not specific to ostree containers. Person working on it can't be here. Will prod them to get it sooner rather than later.
- [name=Laura] KubeCon EU stuff
    - Deadlines coming up, and we've put in a request for a kiosk. If you're heading there and are interested it doesn't have to just be maintainers.
    - We can also do a project lightning 🗲 talk and a ContribFest submission.
        - ~50% attendees got All Access passes, but that means that ~50% of attendees didn't, and they're more likely to go to the lightning talks, especially in the afternoon.
        - On ContribFests, have seen talk like building UI extension for podman desktop, OpenTelemetry how to get involved. We could do exploring bootcrew. A lot of options for this! They do give passes for this one. Submissions due Sunday!
    - ContribFest format: dedidcated timeslots, 75-min workshops
    - [name=Colin] what about an option of Laura being there in person but other people dialed in?
    - RHT's OSPO can help if there's an accepted ContribFest
    - [name=Preethi] RHT doesn't know the budget yet but suggests to submit and will do what she can from her side
    - [name=Laura] post in the channel for the talk and we'll refine it
- [name=Laura] will be here next week, thanks to everyone for attending. Assume last 2 fridays will be canceleld

### TODO
- [ ] Everyone to think about triaging efficiently

