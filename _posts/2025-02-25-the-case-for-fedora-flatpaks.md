---
title: The Case for Fedora Flatpaks
date: 2025-02-25
layout: post
---
---
*(Please note that this reflects my personal opinions on the subject at hand, and does not necessarily represent that of the Fedora project as a whole or other parts thereof, or of my employer.)*

### Background

The Fedora Flatpak project first shipped content in Fedora 29 (2018), starting with two dozen apps, and growing into the 80-some range in the next two releases, where it pretty much held steady until Fedora 36, where it dipped down under 50.  My first involvement came during Fedora 37 (2022), where I helped not only get the existing content current but also expanded our selection to 200 applications.  I later created a KDE runtime for Fedora, which allowed for Qt and KDE applications to be flatpaked for Fedora as well, and continued to expand and improve the ecosystem over subsequent releases.

Unfortunately, around the time of Fedora 39, all others who had previously been involved stopped having time to work on flatpaks due to their other responsibilities, and that pretty much left just me to keep things going.  Nevertheless, I continued to maintain everything with the spare time I had, and wrote some tools to help manage the work.  I also started to run into various issues with particular apps which required fixes or new features to the runtime and tooling, and while I started accumulating changes to address them, there was nobody left to review those changes.

Finally, just a month ago, it was agreed that I should offically take over the Fedora Flatpaks, and was given the permissions needed to do so.  As a result, users are just starting to benefit from months of work that had been blocked, and which makes possible many more improvements which I haven't had a chance yet to roll out.

### The Recent Debacle

As the saying goes, nature abhors a vacuum, and of course it was in the midst of this transition that someone from OBS Studio dropped in to our matrix channel, and then filed a ticket in our issue tracker.  Unfortunately, due to some misunderstandings and downright poor communication from others, the impression made was that we didn't care about their issues, and the situation escalated from there.  Thanks to those who thoughtfully intervened and lowered the temperature, we were finally able to meet and productively discuss their issues.  It was a good and mutually beneficial meeting, and it turned out that the major issues were already resolved or were in the process, and the remaining issues were all manageable.  As a result, all demands have been rescinded, and new relationships forged.

However, some others have taken this an opportunity to question the purpose, fitness, and even the very right to existence of Fedora Flatpaks.  To that end, I hope this serves as my perspective on the subject, address the questions raised, and why I continue to invest in and advocate on their behalf.

### Flatpak vs Flathub

From all the recent discussion, it seems many people think these are synonymous, and conflate the goals of each.  However, if you look at them separately, then many of the subsequent questions become clearer.

**Flatpak** is a packaging format for (primarily) desktop applications which allows for a separation between host, runtime, and application, which means:
* each can be installed and updated independently of each other
* updating runtimes and applications does not require restarting the system
* works regardless of distribution ("build once, run anywhere", within architecture limits)
* live installation on immutable ("Atomic") systems without the need for rebooting, overlays, or the like
* a permissions model similar to those found in modern mobile environments
* control over the runtime version and non-runtime dependencies used

**Flathub** is a site and repository which provides authors and communities to upload their own runtimes and applications in the Flatpak format.  Flatpaks are what make Flathub possible, but it does not mean that flatpaks are limited in any way to Flathub.  Like any packaging format, there can be more than one source of packages, which flatpak supports natively, and other communities themselves do take advantage of that ability.

Furthermore, while Flathub enables upstreams to build and distribute their software in a way that all may easily consume it, as far as Free and Open Source Software is concerned, that does not preclude others from doing the same.  In other words, just because a FOSS upstream has their preferred packaging does not negate or counteract the rights of downstreams to distribute their own packages of the same software in line with the license which the upstreams have themselves freely chosen.

Unfortunately, this has been forgotten or ignored by certain upstreams, which, now being empowered to build and ship their projects themselves, suddenly believe that nobody else should distribute their software, and have even become outright hostile to downstreams in general.  (To be clear, I am not accusing OBS Studio of falling in this category.)  It is an unfortunate and possibly unintended side effect of this new dynamic that has eroded the relationship between (certain) upstreams and downstreams.  When this attitude is taken to its ultimate conclusion, then it is not just Fedora Flatpaks which are questioned, but any form of downstream packaging of their applications.

Some have even misconstrued the Fedora principle of "Upstream First" to mean that we should not "interfere" with upstream's own packaging by creating our own.  That is not the intent of this principle, and others in the community are currently working to clarify this misconception through our documentation.

However, I am firmly of the opinion that downstream distributions still have their place.  While I'm glad that upstreams now have the ability to deliver themselves through Flathub, and I do use some Flathub Flatpaks myself, I still prefer Fedora-produced content whenever possible, and specifically in the form of Fedora Flatpaks.

### Why Not Flathub?

So why should we continue to package desktop applications in any form when Flathub exists?  Because, as projects and communities, Fedora and Flathub have different and sometimes conflicting goals and standards.  That doesn't mean that any given application on Flathub is worse than our own (in whichever form), but that as a potential source of software on the whole, we cannot guarantee that it meets the standards that Fedora users expect from Fedora itself.

Some of these differences include:

* Lack of source and build system provenance.  Many projects which simply include .deb packages (which are of course not originally built on Flathub infrastructure, and are not always from Debian either) as "sources" and simply repack them.  Some projects also include upstream-built individual binaries as "sources" and include them as well, even for dependencies which are not their own.  So not everything shipped in Flathub is actually built on Flathub infrastructure (or other infrastructure trusted thereby), and not all sources are provided for the binaries being distributed.
* Lack of separation between FOSS, legally encumbered, and proprietary software.  While proprietary software is (supposed to) be so tagged in flatpak definitions, and that tag is shown (with varying degrees of visibility) in the various interfaces, they are still in a single repository with no filtering.  There is also no separation of legally encumbered software, which may affect users in various jurisdictions.  Users who only want legally unencumbered FOSS need to carefully examine each application individually.
* Lack of systemic upgrading of applications to the latest runtime.  Generally, the latest two versions of any given Flathub runtime are supported.  However, there is no attempt to move all applications to the latest runtime once released, meaning that users easily end up needing multiple versions of the same runtimes for an extended period.  Furthermore, applications may continue to be built and shipped even against runtimes which are out of support.
* Lack of coordination of changes to non-runtime dependencies.  When an upstream includes another package (which is usually not included the runtime) as a dependency, there are no limitations as to the versions, configuration, or modifications made to that dependency.  That means users can end up with multiple different copies of the same code in their various applications.  It also means that upstreams can essentially fork its dependencies rather than working with the dependencies' upstreams to make the changes necessary to meet their needs.
* Lack of systemic community engagement.  Flathub is expressly not a distribution.  Since each application is a product of its own community, there is no particular way that any given user can contribute to any given application. (My own experience in attempting to contribute fixes to, or raise issues with, various Flathub Flatpaks has mostly been met with silence.)

None of this is to say that Flathub does not have its own standards, or that any of the above should not be allowed.  However, they do not conform with Fedora's goals and standards, and our users expect our content to meet those.  As such, the Fedora Engineering Steering Committee (FESCo) already decided that Flathub needs to be opt-in and lower in priority to Fedora's own content.  

All that being said, Fedora still respects the users' choice, and users have a choice of software source for every application they install, and each source can be enabled or disabled in both GNOME Software and KDE Plasma Discover.  (Discover also allows for changing the priorities of each software source, thereby affecting which is chosen by default when multiple choices exist.  Fedora Workstation and Silverblue users would benefit by Software gaining the same functionality.)

### Why Fedora Flatpaks?
Fedora Flatpaks bring all the advantages which users expect from our distribution in general, and adds the unique qualities of the Flatpak packaging format.

* All packages are built from source code on Fedora infrastructure, by the same spec files used to build our RPM packages.
* All applications that we ship (regardless of packaging method) are FOSS and legally unencumbered.
* We actively work to move all applications to the latest runtime version as quickly as possible, and work with any individual package maintainers to address any issues that prevent doing so.
* All packages use the same version of non-runtime dependencies.  All dependents benefit equally from any given fix or update to a dependency.  Specifically in the context of Fedora Flatpaks, not only does this simplify and speed up builds, but these will actually deduplicate on-disk because they are the very same builds packed with each dependent application.
* All Fedora packaging is part of a single community.  Even though SIGs or individual maintainers may focus on certain parts of the distribution, there is a single means for submitting contributions (Pull Requests on src.fedoraproject.org).

Fedora Flatpaks simply serve as an additional packaging format for our builds of desktop applications.  Because Fedora features several editions and spins which are ostree-based ("Atomic" or "immutable" systems), Fedora Flatpak content is particularly needed to allow those users to install Fedora content in a way that is most compatible with that format (where adding packages to the host requires various additional steps.)  Even users of traditional systems (such as myself) can still benefit from Flatpaks over RPMs, and if this is indeed the future of desktop applications, then all users are better served by a robust selection of software in this format.

Furthermore, only by packaging a broad selection of applications as Fedora Flatpaks have issues and limitations of our packaging, runtimes, and tooling been discovered, which then makes it possible to fix and improve them, benefiting the entire ecosystem.  Not only will users of Fedora Flatpaks benefit from these changes, but sometimes Fedora RPM users as well.  These improvements should then eventually filter down to the RHEL Flatpak runtime and SDK.  (Some changes are already on track for RHEL 10.0, with more hopefully to follow.)

Even Flathub itself, and the upstreams who ship their own software thereon, could end up benefiting from robust alternatives.  There are plenty of individual or systemic issues which I have discovered in Flathub Flatpaks only by virtue of working on Fedora Flatpaks, and they too could benefit if there were reliable means to communicate them.

In summary, users still want distributions of software, choose a distribution by varying factors, and expect their distribution to meet its stated goals and standards.  Fedora Flatpaks simply serve the next step in the Fedora distribution with respect to desktop applications in this era of new desktop and system technologies.

### How To Participate

If you are genuinely interested in the future of Fedora Flatpaks, then there are many ways in which you can become involved:
* Automation
* Bug wrangling
* Creation of new content
* Documentation
* Maintenance of existing content
* Marketing
* Testing
* Working with upstreams to use portals to avoid the need for sandbox holes

Our current means of communication are:
* Chat: [\#flatpaks:fedoraproject.org](https://chat.fedoraproject.org/#/room/#flatpaks:fedoraproject.org) on Matrix
* Issue Tracker: [fedora-flatpaks on Gitlab](https://gitlab.com/fedora/sigs/flatpak/fedora-flatpaks/-/issues)
