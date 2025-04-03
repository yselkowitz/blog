---
title: Thunderbird Fedora Flatpak rename may require manual intervention
date: 2025-04-03
layout: post
---
---
##### The following has also been posted to [Fedora Flatpak issue #50](https://gitlab.com/fedora/sigs/flatpak/fedora-flatpaks/-/issues/50).

A change in Fedora's [thunderbird RPM package](https://src.fedoraproject.org/rpms/thunderbird/c/8e08ed42a33c661d8c122e30b2d4dd2ec399187b) a few months ago led to some issues with the Fedora Flatpak thereof, not only in Software/Discover but also in desktop usage, such as windows not being associated with their desktop icon.  For instance, in KDE Plasma, a generic Wayland icon would be shown on the Task Manager bar even if the Thunderbird desktop menu entry was pinned.

I have taken the Fedora 42 update going stable tonight as an opportunity to fix that, but there are caveats.  Users should get upgraded to the new name automatically, but when starting the app afterwards, it will look like you need to start over configuring your email accounts and so on.  However, with a little manual intervention, this can be avoided.

If you have already reconfigured your accounts and are happy with the results (just not with the effort involved), then there is nothing more to do except to understand what happened and why it was ultimately necessary.  Unfortunately we have no reliable way of letting users know about changes such as this.

Otherwise, if you have started Thunderbird after the upgrade and seen this but have not done anything significant afterwards, then this should be recoverable.  Furthermore, if you have upgraded but have not yet started the app since, you can avoid this entirely, and step 2 below might not apply to you.

1. First, verify that the data from your previous usage is still intact:  
 
    ```
    find ~/.var/app/org.mozilla.Thunderbird/.thunderbird | less
    ```

2. If this shows a long list of files, and you have not yet done anything significant with the app after the "reset", then you can proceed by    removing the (very little) data from the new app name:  

    ```
    find ~/.var/app/net.thunderbird.Thunderbird -delete
    ```

3. Finally, rename the old data directory to the new name:  

   ```
   mv ~/.var/app/org.mozilla.Thunderbird ~/.var/app/net.thunderbird.Thunderbird
   ```

Launching Thunderbird from the desktop menu should now find all your accounts, emails, and so on.

My apologies for the inconvenience.
