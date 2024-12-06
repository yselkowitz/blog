---
title: Potential failures in the upcoming mass rebuild
date: 2024-12-05
layout: post
---
---
There are less than six weeks until the [Fedora 42 mass rebuild](https://fedorapeople.org/groups/schedule/f-42/f-42-key-tasks.html).
While mass rebuilds inevitably result in some breakage, the following issues appear likely to be common causes of 
[FTBFS](https://docs.fedoraproject.org/en-US/fesco/Fails_to_build_from_source_Fails_to_install/#_glossary):

### GCC 15

While not yet announced as a change for Fedora 42, each winter mass rebuild is immediately preceded by a new major (prerelease) version of GCC. 
(I *might* still be recovering from GCC 14 which coincided with the branching of RHEL 10.)  Each major GCC version ends up introducing some 
strictness which results in compile errors in existing (particularly older) code.  Hopefully we'll be hearing more from the Tools team about 
what to expect this time, but in the interim there is a WIP [list of major changes](https://gcc.gnu.org/gcc-15/changes.html).

### Automake 1.17

Newly introduced in rawhide just yesterday, this is the first major release in more than six years.  The 
[upstream release announcement](https://lists.gnu.org/archive/html/info-gnu/2024-07/msg00006.html) does not indicate any major incompatibilities, 
but I have already seen [one failure in a package](https://src.fedoraproject.org/rpms/lasso/pull-request/16) which tried to make now-outdated 
assumption automake version numbers, and there are likely to be more such changes needed.

### Setuptools 74

So far, the major incompatible change here is that the long-deprecated `setup.py test` is no longer supported, which will break dozens of packages. 
The Python SIG has already [filed bugs](https://bugzilla.redhat.com/show_bug.cgi?id=2319387) for at least some of these.  If you haven't already, 
now is the time to fix your Python packages by [calling the tests directly](https://docs.fedoraproject.org/en-US/packaging-guidelines/Python/#_other_test_runners).

### CMake 3.31

Unfortunately, the latest cmake has caused some issues in Qt/KDE land by [breaking builds involving QML](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1087385#12). 
Also, there was a regression with the `%ctest` macro and the double-hyphen separator before short arguments (e.g. `%ctest -- -E foo`) but that should now be fixed.

### glibc 2.41

This may not affect that many packages, but the introduction of `struct sched_attr` and userspace functions for the `sched_getattr` and `sched_setattr` syscalls 
([upstream commit](https://sourceware.org/git/?p=glibc.git;a=commit;h=21571ca0d70302909cf72707b2a7736cf12190a0)) has caused conflicts with code which had been 
providing their own such wrappers.  The "proper" way to handle this is to guard these based on configuration tests, but for those that do not do so, 
`!__GLIBC_PREREQ(2,41)` will only help once 2.41 is released.  In the meantime, workarounds may be necessary 
(e.g. [here](https://src.fedoraproject.org/rpms/realtime-tests/pull-request/1) and [here](https://src.fedoraproject.org/rpms/stalld/pull-request/1)).

There will likely be more FTBFS themes to emerge in the coming weeks, but it's not too early to fix your packages for any of the above, 
and to check for any other issues rebuilding your package in rawhide, particularly if you haven't rebuilt in a while.
