---
title: Potential failures in the F43 mass rebuild
date: 2025-06-26
layout: post
---
---
There are less than four weeks until the [Fedora 43 mass rebuild](https://fedorapeople.org/groups/schedule/f-42/f-43-key-tasks.html).
While mass rebuilds inevitably result in some breakage, the following issues appear likely to be common causes of 
[FTBFS or FTI](https://docs.fedoraproject.org/en-US/fesco/Fails_to_build_from_source_Fails_to_install/#_glossary):

### Missing sysusers.d configs

RPM now generates virtual dependencies on `user(USERNAME)` and/or `group(GROUPNAME)` under various conditions (e.g. a file in `%files` is listed with such a user/group ownership).
In order to resolve these dependencies, packages must now provide sysusers.d config files which are automatically parsed to generate similar virtual provides.
Without these configs, a newly rebuilt package will FTI, and anything which buildrequires it will fail to build for being unable to install its dependencies.
See the [Change documentation](https://fedoraproject.org/wiki/Changes/RPMSuportForSystemdSysusers) for more details.

### Python 3.14

While the Python team has already rebuilt many Python-dependent packages for 3.14, some packages still need to be fixed for changes in the new version 
(see [What's New in Python 3.14](https://docs.python.org/3.14/whatsnew/3.14.html)).
Furthermore, packages which use Python only at build-time (e.g. for build scripts, or running tests, etc.) have yet to be rebuilt, and may need to be fixed as well.

### Aclocal macros moved in gettext 0.25

In previous versions of gettext-devel, its various aclocal macros were installed in the default macro search path, and therefore were generally found by `autoreconf` without any effort, or even when they shouldn't have been.
However, that actually caused conflict with `autopoint` when trying to pin an older macros version with `AM_GNU_GETTEXT_VERSION`.
With gettext 0.25, these macros are now located in a private path, and while `autopoint` still works as designed, other (technically unsupported) use cases have broken as a result, such as the use of `AM_ICONV` by itself.
The workaround for such cases is to `export ACLOCAL_PATH=/usr/share/gettext/m4/` before `autoreconf`.

### X11 disablement in mutter breaks xwfb-run

As part of the ongoing deprecation of the X11 servers, many packages previously switched from `xvfb-run` to `xwfb-run` (part of `xwayland-run`), the latter requiring a Wayland compositor to be specified.
The next step of that for F43 was disabling the X11 session support in the GNOME desktop, which included disabling the X11 window management support in mutter.
Many GNOME packages, and all such packages in the RHEL/ELN set, are using `mutter` as the compositor. 
(Packages outside of the GNOME and RHEL sets may use `weston` instead and are not impacted.)

Unfortunately, due to a bug in mutter before 49.alpha, disabling X11 also mistakenly disabled the `--no-x11` option which (counterintuitively) is *not* the opposite of the now-disabled `--x11` argument (to run as an X11 window manager) but disabled the launching of an XWayland server.
With the option disabled but XWayland support enabled, that means mutter is always trying to launch XWayland even where it shouldn't (such as in a moch chroot, where it doesn't work).
`xwayland-run` relied on this option to avoid that, but with that not working, any use of `xwfb-run -c mutter` (or the like) are currently failing.

This should be fixed as part of the 49~alpha.0 update which is [currently in testing](https://bodhi.fedoraproject.org/updates/FEDORA-2025-b79b32194c) but blocked by additional packages needing to be updated in tandem.
Hopefully the Workstation WG can get this fixed in time for the F43 mass rebuild.

### glibc 2.42 changes

The development version of glibc now in rawhide/F43 includes a few potentially breaking changes:

* The ancient `<termio.h>` and `struct termio` interfaces [have been removed](https://sourceware.org/git/?p=glibc.git;a=commitdiff;h=e04afb71771710cdc6025fe95908f5f17de7b72d). Using `<sys/ioctl.h>` and/or `<termios.h>`, and `struct termios`, instead should cover the common use cases.
* `lockf` now has a `warn_unused_result` attribute ([commit](https://sourceware.org/git/?p=glibc.git;a=commitdiff;h=f3c82fc1b41261f582f5f9fa12f74af9bcbc88f9)), which will generate warnings (or promoted errors) in code where the result is ignored.

### GCC 15

While many of the [changes in GCC 15](https://gcc.gnu.org/gcc-15/changes.html) were handled in the aftermath of [the previous F42 mass rebuild](https://yselkowitz.github.io/blog/2024/12/05/f42-mass-rebuild.html) , there are still some packages which have yet to be rebuilt.
Particularly, mingw-gcc 15 did not land until after the mass rebuild, so some mingw packages may see failures. (For those with corresponding native packages, hopefully the fix is already available.)
Also, some later changes in GCC 15 (after the mass rebuild) may also cause failures ([one example](https://github.com/openucx/ucx/issues/10663)).

### Flaky tests

All too often, builds fail in `%check` due to flaky or architecture-dependent tests.  Rerunning an entire build (sometimes multiple times!) just in order to get tests to pass is a waste of time and resources.
Instead, please consider moving tests out of the spec file and [into CI instead](https://docs.fedoraproject.org/en-US/ci/).

## Note:

This list was ~~generated by AI~~ -- just kidding!
Actually, this is the result of an [early mass rebuild in ELN](https://github.com/fedora-eln/eln/issues/258), which was done with the express purpose of finding such issues in advance, and get a head start on fixing them.
Hopefully this information, and the fixes provided along the way (linked in the tracker) will help maintainers to have a smoother mass rebuild experience.
