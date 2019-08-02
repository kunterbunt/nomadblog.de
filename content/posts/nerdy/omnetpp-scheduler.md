+++
showLogo = false
logo = ""
hasMath = false
draft = false
summary = ""
tags = [
  "tech", "simulation"
]
date = "2017-05-01T15:59:19+02:00"
title = "Implementing an LTE resource scheduler in OMNeT++/SIMUlte"

+++

[SIMUlte](http://simulte.com/) is an extension framework for the network simulator [OMNeT++](https://omnetpp.org/). It enables simulations of LTE telecommunications.

In this context, **resource scheduling** is the problem of allocating transmission resources residing in the dimensions of time and of frequency band to user devices. A resource block can be depicted like this:

![An LTE resource block](/imgs/omnetpp-scheduler/resource-block.png)

There are a number of optimization targets we would like to meet:

- system throughput: we want to push out as much data as possible
- fairness: we want all devices to be able to push out data
- power efficiency: we want to use as little power as possible

So in the end, resource scheduling boils down to a complex optimization problem. What's the best way of deciding who gets to transmit using which resource, so that our targets are met optimally? And since the problem can be shown to be NP-hard, we can relax it a bit, and instead of an optimal solution look for a *rather good* one:

![A scheduler](/imgs/omnetpp-scheduler/scheduler.png)

There are numerous ideas and concepts out there for scheduler implementations, each with their strong and weak suits. At some point a researcher will want to evaluate their proposed method. This is what I did within my research project at the Technical University Hamburg.

So this is a general how-to on implementing a scheduling algorithm for the OMNeT++ simulator and the SIMUlte framework:

1. Create your implementation files at `<simulte dir>/src/stack/mac/scheduling_modules/<your scheduler.{cc, h}`
+ Have your class inherit from `LteScheduler`, and include `omnetpp.h, LteScheduler.h, LteCommon.h`.
+ Open `LteCommon.h`, look for `enum SchedDiscipline { ... }` and add your scheduler class to the `enum`. Also look for and add to `const SchedDisciplineTable disciplines[] = { ELEM(DRR), ..., ELEM(YOUR_SCHEDULER)};`. Don't forget to `#include <YOUR_SCHEDULER>` in this file.
+ Now look for `LteSchedulerEnb::getScheduler(SchedDiscipline discipline)` and modify it to include your scheduler - just follow the already present examples. Of course, `#include <YOUR_SCHEDULER>` once again.
+ While in `LteSchedulerEnb.h`, also add your class as a `friend class`.
+ Now you can get to the implementation of your scheduler. The `enum` from before tells you the keyword you can put into your `omnetpp.ini` so that your scheduler is used:

```
**.schedulingDisciplineDl = "YOUR_SCHEDULER"
**.schedulingDisciplineUl = "YOUR_SCHEDULER"
```

You can use the following code as a starting point. The comments should tell you a little about what each function should be doing. This simple scheduler simply assings `band 0` to all devices - I had used it to determine if the simulator supports `band reassignment`: it doesn't at the time of writing, so don't expect good performance from this naive scheduler:

```c++
#include <omnetpp.h>
#include <LteScheduler.h>
#include "LteCommon.h"

class LteReassignment : public virtual LteScheduler {
public:
  LteReassignment() {}
  virtual ~LteReassignment() {}

  /**
   * Apply the algorithm on temporal object 'activeConnectionTempSet_'.
   */
  virtual void prepareSchedule() override {
    // Copy currently active connections to a working copy.
    activeConnectionTempSet_ = activeConnectionSet_;

    // Go through all active connections.
    for (ActiveSet::iterator iterator = activeConnectionTempSet_.begin(); iterator != activeConnectionTempSet_.end(); iterator++)
        MacCid currentConnection = *iterator;
        MacNodeId nodeId = MacCidToNodeId(currentConnection);
        EV << NOW << " LteReassignment::prepareSchedule Considerung node " << nodeId << "." << std::endl;
        if (getBinder()->getOmnetId(nodeId) == 0) {
            EV << NOW << "LteReassignment::prepareSchedule removing node " << nodeId << " because its ID is unknown" << std::endl;
            activeConnectionTempSet_.erase(activeConnectionTempSet_.begin() + i);
            i--;
            continue;
        }
        // Assign band 0.        
        SchedulingResult result = schedule(currentConnection, Band(0));        
        EV << NOW << " LteReassignment::prepareSchedule Scheduled node " << nodeId << " on band 0: " << schedulingResultToString(result) << std::endl;

        if (result == SchedulingResult::INACTIVE) {
            EV << NOW << " LteReassignment::prepareSchedule removing node " << nodeId << " because it is now INACTIVE" << std::endl;            
            activeConnectionTempSet_.erase(activeConnectionTempSet_.begin() + i);
            i--;            
        }
    }
  }

  /**
   * Put the results from prepareSchedule() into the actually-used object 'activeConnectionSet_'.
   */
  virtual void commitSchedule() override {
    EV_STATICCONTEXT;
    EV << NOW << " LteReassignment::commitSchedule" << std::endl;
    activeConnectionSet_ = activeConnectionTempSet_;
  }

  /**
   * When the LteSchedulerEnb learns of an active connection it notifies the LteScheduler.
   * It is essential to save this information. (I think this should be the default behaviour and be done in the LteScheduler class)
   */
  void notifyActiveConnection(MacCid cid) override {
      EV_STATICCONTEXT;      
      EV << NOW << " LteReassignment::notifyActiveConnection(" << cid << ")" << std::endl;
      activeConnectionSet_.insert(cid);
  }

  /**
   * When the LteSchedulerEnb learns of a connection going inactive it notifies the LteScheduler.
   */
  void removeActiveConnection(MacCid cid) override {
      EV_STATICCONTEXT;      
      EV << NOW << " LteReassignment::removeActiveConnection(" << cid << ")" << std::endl;
      activeConnectionSet_.erase(cid);
  }

protected:
  enum SchedulingResult {
        OK = 0, TERMINATE, INACTIVE, INELIGIBLE
    };

  std::string schedulingResultToString(LteReassignment::SchedulingResult result) {
        return (result == SchedulingResult::TERMINATE ? "TERMINATE" :
                result == SchedulingResult::INACTIVE ? "INACTIVE" :
                result == SchedulingResult::INELIGIBLE ? "INELIGIBLE" :
                "OK");
    }

  LteReassignment::SchedulingResult schedule(MacCid connectionId, Band band) {
    bool terminate = false;
    bool active = true;
    bool eligible = true;

    std::vector<BandLimit> bandLimitVec;
    BandLimit bandLimit(band);
    bandLimitVec.push_back(bandLimit);

    // requestGrant(...) might alter the three bool values, so we can check them afterwards.
    unsigned long max = 4294967295U; // 2^32    
    unsigned int granted = requestGrant(connectionId, max, terminate, active, eligible, &bandLimitVec);
    EV << " " << granted << " bytes granted." << std::endl;
    if (terminate)
        return LteReassignment::SchedulingResult::TERMINATE;
    else if (!active)
        return LteReassignment::SchedulingResult::INACTIVE;
    else if (!eligible)
        return LteReassignment::SchedulingResult::INELIGIBLE;
    else
        return LteReassignment::SchedulingResult::OK;
  }
};
```
