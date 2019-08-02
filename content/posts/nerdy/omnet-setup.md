+++
draft = false
title = "OMNeT++ and INET Simulator Setup"
date = "2017-10-11T10:51:10+02:00"
summary = "Installing the simulator + INET framework on Arch Linux."
tags = ["tech", "simulation"]
showLogo = false
logo = ""
hasMath = false
+++
I use the popular, modular [OMNeT++ simulator](https://omnetpp.org/) for network simulation. The [INET Framework](https://inet.omnetpp.org/) adds lots of useful wired, wireless and mobile communication protocols and entities to the simulator.   
It turns out many people have problems compiling the two. So I'd like to give a quick introduction into how *I* got them working.

I use `OMNeT++ 5.1.1` together with `INET 3.5`. Both have newer versions, but another framework I use - [simuLTE](http://simulte.com/) - requires these versions.

For 3D renders, OMNeT++ uses `osgEarth`, where osg stands for `openscenegraph`, a library that provides some rendering functionality. At the time of writing `osgEarth` from the AUR won't build due to some incompatibility with the current `geos 3.6`. For me downgrading to `geos 3.5` leads to `osgEarth` compiling. `osgEarth` installs `openscenegraph` as a dependency.

When this is installed, go to the directory you extracted `OMNeT++` into and execute

```
cd <omnet-dir>/
./configure // watch out for missing packages and install as needed
make -j4
```

Now run a sample simulation from `<omnet-dir>/samples/dyna` (or any other) and make sure the simulation runs.  
Next, install `INET` by extracting it into `<omnet-dir>/inet`. I believe it is important for some projects that `INET` resides in a directory called exactly `inet` (no versioning in the name). Now   

```
cd <omnet-dir>/samples/inet
make makefiles
make MODE=release -j4
```

And that should be it. The biggest problems arised for me from having all dependency libraries working, which sometimes don't play nice together when the versions are most recent. This is of course a problem of Arch Linux, which keeps verions most recent. Other distributions might work more easily.
