---
title: How to Use OpenH264 with Fedora Flatpaks
date: 2025-08-12
layout: post
---
---
OpenH264 provides an open source and fully licenced codec for the widely-used H.264 format. However, [due to legal constraints](https://docs.fedoraproject.org/en-US/quick-docs/openh264/), the binaries thereof must be distributed directly by Cisco, who provides a repository of RPMs particularly built for Fedora.  The question then arises, how can we use this with Fedora Flatpaks? 

Flatpak (the program) includes an `extra-data` functionality which allows for downloading and installing artifacts (the metadata for which is embedded in the distributed flatpak) on the users' machine during installation, rather than including the content in the flatpak itself.  This is perfect for a case where we want to install something but cannot distribute the content ourselves.

Unfortunately, flatpak (the program) [does not handle this properly](https://github.com/flatpak/flatpak/issues/3790) when the flatpak (content) is downloaded from an OCI registry, such as we do in Fedora. (If we could just implement that, this article may not be necessary.)  However, there are ways around this, in increasing order of difficulty.

All of these options require network access (at some point), the `flatpak-module-tools` package, and aside from Option 1, the `fedpkg` package.  You only need one of these options, as they all just different ways to install the same extension (named `org.fedoraproject.Platform.Codecs.openh264`).

*NOTE 1:* because of the nature of these workarounds, each user on a system will need to run these steps, and will need to redo them for each Fedora Flatpak version (42, 43, etc.) and occasionally in between when there are updates to openh264 itself.

*NOTE 2:* In all of the following examples, `RELEASE` or `@RELEASE@` must be replaced with the Fedora Flatpak runtime release for which the extension should be installed (currently, `f42`).

### Option 1 -- Download the Extension

This is the easiest option, provided that you are also able to install the `openh264` RPM.

1. Browse [the openh264-flatpak builds in Koji](https://koji.fedoraproject.org/koji/packageinfo?packageID=41585) and select the most recent build corresponding to the version of your Fedora Flatpak runtime(s) (currently, `f42`).
2. Download the `.oci.tar` file corresponding to your architecture.
3. Install the extension locally:  
```  
flatpak-module install /path/to/openh264-flatpak-RELEASE-NN.ARCH.oci.tar
```

### Option 2 -- Build the Extension

If Option 1 does not work, or there is no corresponding extension build available for download, then you can build and install the very same extension yourself.

1. Clone the extension spec:
```
fedpkg clone -ab RELEASE flatpaks/openh264 && cd openh264
```
2. Build and install the extension:  
```
flatpak-module build-container-local --install  
```

### Option 3 -- Build the Package and Extension

If you are somehow *unable* to install the `openh264` RPM on your system, then the previous methods will not work.  Instead, you will need to build openh264 itself and an extension which includes it (instead of downloading it, as above).

*NOTE: this option requires `flatpak-module-tools` 1.2.0 or later.*

1. Create the `container.yaml` flatpak spec:  
```yaml
flatpak:
    id: org.fedoraproject.Platform.Codecs.openh264
    build-extension: true
    runtime: org.fedoraproject.Platform
    runtime-version: @RELEASE@
    sdk: org.fedoraproject.Sdk
    name: openh264
    component: openh264-flatpak
    branch: @RELEASE@
    packages:
        - openh264
        - openh264-flatpak-config
        # for cleanup-commands (part of runtime)
        - coreutils-single
    cleanup-commands: |
        mkdir -p /app/lib
        mv /usr/lib64/libopenh264.so* /app/lib/
```
2. Clone the RPM spec:
```
fedpkg clone -ab RELEASE rpms/openh264
```
3. Build the RPM to be included in the extension:  
```
flatpak-module build-rpms-local ./openh264
```
4. Build and install the extension:  
```
flatpak-module build-container-local --install  
```

### Option 4 -- The Fallback

If none of these are present but you have any Flathub content installed, the Fedora Flatpak runtimes uses the Flathub openh264 extension instead.  However, this will no longer be an option as of the F43 release of the Fedora Flatpak runtime.

---

Hopefully, one day we will no longer have to deal with the artificial encumbrances of software patents, and will be able to distribute all open source software freely.  Until that time, the `Codecs` extension space provides a flexible framework for extending the multimedia support of all Fedora flatpaks.
