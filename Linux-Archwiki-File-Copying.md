# Flashing ISO files to flash drives
I rarely got dd to work in a terminal for this purpose,
so I set out on the ArchWiki, a great source of information
for doing specific Linux tasks in a more streamlined way.

Apparently dd is not "the preferred Linux way" anymore
at least according to ArchWiki. Linux has changed so much
since I started using it in late 2016!

https://wiki.archlinux.org/title/USB_flash_installation_medium

I wanted a terminal command so I can do this faster without
needing to use BalenaEtcher or install anything, or on a
server machine like a Raspberry Pi.

## Experiment
My initial command was

`sudo bash -c "cat [iso file].iso > /dev/disk/by-id/usb-xyz"`

I initially thought `sudo bash -c "..."` would be the problem.

I thought it had to do with bash -c hanging when a process is sleeping.

I tried `sudo bash -c "echo hi; sleep 10; echo bye"`... worked fine

I went back to `sudo bash -c "cp Joplin.AppImage > /root/Joplin.AppImage"`, that works fine too

After a closer inspection, it turns out it
has something to do with disk sleep with `cp` when I looked in KSysGuard and
just for fun I looked at the output of `cat /proc/[cat pid]/task/[same pid of cat is the task id]`.

Lo and behold the (D) disk sleep indicator.

I opened a root shell using `su` and got 3 commands before 1 eventually worked.

- `cat [iso file].iso > /dev/disk/by-id/usb-xyz` did not work - the process entered "disk sleep" mode and `kill -9 [pid of cat]` did not work
    - Ctrl+C/Ctrl+D did not stop the cat process
- `cp [iso file].iso /dev/disk/by-id/usb-xyz` did not work - the process entered "disk sleep" mode
- Why is it disk sleeping and what is the fix? Lots of Googling came up with this: https://unix.stackexchange.com/questions/713699/cp-and-dd-stuck-when-writing-iso-image-to-usb-flash-drive
- `cp [iso file].iso /dev/disk/by-id/usb-xyz -v` worked -- eventually it showed the `#` prompt
- `cp [iso file].iso /dev/disk/by-id/usb-xyz && sync` -- `&& sync` coming from my previous knowledge of the exact dd command in the arch wiki in 2016 - worked

GNOME Disks worked just fine although it was a bit slower.

I thought it was just my terminal skills were a bit rusty, but the CLI can be unpredictable at times too though.

### Sidenote
These are very useful "extreme/edge" test cases for a distro by the way
(recently I learned distros like Fedora have test suites which is really cool!)
to plan for the "eventuality" of a disk sleep on a large file.

## Conclusion
It's difficult to tell "the right Linux way" from something that works.
But something that works might also not be the optimal way in terms of performance or other factors.

I wonder why the defaults of `cp` would hang the terminal? I suppose filesystem/block syncing is a secondary priority so you say exactly what to do
on Linux (and nothing else) - this can be critical on embedded systems where a block sync isn't wanted at all maybe for efficiency purposes.

I wish the Linux CLI was more friendly to desktop users sometimes but nothing is perfect.

I also probably should not trust `cp` alone when copying a large folder between 2 drives in case
it gets stuck; although I think the flags can be hard to remember, `rsync -azhvP` is more reliable in my experience
(but not for flash drives I think).
