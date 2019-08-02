+++
date = "2017-03-29T16:43:13+02:00"
title = "Reading module and simulation parameters in OMNeT++"
draft = false
summary = "How to access general simulation parameters at runtime."
tags = [
  "tech", "simulation"
]
showLogo = false
logo = ""
hasMath = false
+++

I had a hard time figuring out how to access general simulation parameters such as the maximum simulation time `max-sim-time` at runtime from whatever module needs it.

For `modules` it is rather straightforward. From any `cModule` object, there's a `par(const char* parname)` function. Say you're writing a module named `OmniscientEntity` (as I did), and it is defined in **`OmniscientEntity.ned`** as this:

```
simple OmniscientEntity {            	    
    parameters:
        // Used for CQI computation. Copy over from your channel.xml.
        double targetBler = default(0.001);
        double lambdaMinTh = default(0.02);
        double lambdaMaxTh = default(0.2);
        double lambdaRatioTh = default(20);
        // The simulation time at which final configuration takes place.
        double configTimepoint = default(0.005);
        // The entity will take a snapshot of the some network statistics every x seconds.
        double updateInterval = default(0.05);
        // 180kHz is LTE's bandwidth per resource block as per its specification.
        double resourceBlockBandwidth @unit(kHz) = default(180kHz);
    @display("i=block/table");
    @class(OmniscientEntity);    
}
```

then to access its parameter `targetBler`, just call `par("targetBler")` on its object and that's it. You can traverse the module hierarchy using `getParentModule()`, or get to the top via `getSystemModule()`.

This won't work with general parameters set in your `omnetpp.ini`, though, because they're not part of your module. I had thought that you would be able to access them via
```
getSystemModule()->par(...) // doesn't work
```
but apparently that *also* doesn't work. Go figure.   
After some significant tinkering, I finally found in the documentation that you can get access to the configuration via the globally available `getEnvir()->getConfig()`. From there you have functions that get you what you want, such as
```
getAsDouble(cConfigOption *option, double fallbackValue=0)
```   
Cool, but what's a `cConfigOption`? It has a big, nasty constructor:
```
cConfigOption(const char* name, bool isGlobal, Type type, const char* unit, const char* defaultValue, const char* description)
```

Learning by example is easiest, so to finally find the `sim-time-limit` you're looking for, do this:
```
cConfigOption simTimeConfig("sim-time-limit", true, cConfigOption::Type::CFG_DOUBLE, "s", "300", "");
double maxSimTime = getEnvir()->getConfig()->getAsDouble(&simTimeConfig);
```

and adjust as you need for other parameters. Talk about easy.
