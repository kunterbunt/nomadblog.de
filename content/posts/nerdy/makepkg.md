+++
draft = false
title = "Building an AUR package with invalid MD5Checksum"
summary = ""
tags = ["tech"]
showLogo = false
logo = ""
hasMath = false
date = "2017-10-11T10:28:28+02:00"
+++
I had an annoying problem: for the simulator I use for research I needed a specific library, as the simulator uses it.   
This specific library `osgearth` is available in the Arch Linux's AUR, but the package maintainer put a wrong `MD5 Checksum` into the `PKGBUILD`. Let's say we trust the maintainer and don't believe someone tinkered with the files. What do you do?

Go to an arbitrary directory

```
cd ~/Download
mkdir osgearth
cd osgearth
```

Download the `Snapshot` from the AUR which should contain the `PKGBUILD`; extract it from the archive file.   

Execute `makepkg -g` which downloads the files and computes the `MD5 Checksums`. Copy the checksum, edit the `PKGBUILD` and put it in there.   
Now `makepkg -sc`, where `-s` installs missing dependencies, and `-c` cleans up afterwards.   
Install the compiled package with `pacman -U <package>`.
