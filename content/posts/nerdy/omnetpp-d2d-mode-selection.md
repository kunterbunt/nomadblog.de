+++
logo = ""
hasMath = false
draft = false
summary = ""
tags = [
  "tech", "simulation"
]
date = "2017-05-01T16:41:12+02:00"
title = "Implementing D2D mode selection in OMNeT++/SIMUlte"
showLogo = false
+++

`Device2Device (D2D) Mode Selection` is the process of deciding whether a user device should communicate directly (`D2D Mode`) or via the eNodeB base station (`Infrastructure Mode`). A number of metrics can be taken into account for this decision, such as the interference generated that will affect other devices or the expected datarate of each mode. Once a researcher has come up with an idea, they will want to evaluate it. Using the [OMNeT++ network simulator](https://omnetpp.org/) as a starting point, and adding the [SIMUlte framework](http://simulte.com/) on top enables one to simulate LTE networks.

To implement your own mode selection algorithm, follow the following steps. I have implemented a mode selection that simply forces a device to transmit via D2D if it is capable:

1. Add your implementation files to `<simulte dir>/src/stack/d2dModeSelection/d2dModeSelectionForcedD2D/D2DModeSelectionFOrcedD2D.{cc, h}`.
+ To `.../d2dModeSelection/D2DModeSelection.ned` add

    ```
    simple D2DModeSelectionForcedD2D extends D2DModeSelectionBase {
    	parameters:
    	    @class("D2DModeSelectionForcedD2D");    
    }
    ```
+ The `.ned` tells the simulator to look for a `D2DModeSelectionForcedD2D` class. In `D2DModeSelectionForcedD2D.cc` put `Define_Module(D2DModeSelectionForcedD2D);` to link them together.
+ Now all that's really left is to implement `doModeSelection()`. Consider my implementation a starting point:
    ```c++
    void D2DModeSelectionForcedD2D::doModeSelection() {
    EV << NOW << " D2DModeSelectionForcedD2D::doModeSelection for " << peeringModeMap_->size() << " nodes." << endl;

    // The switch list will contain entries of devices whose mode switch.
    // Clear it to start.
    switchList_.clear();
    // Go through all devices.
    std::map<MacNodeId, std::map<MacNodeId, LteD2DMode>>::iterator peeringMapIterator = peeringModeMap_->begin();
    for (; peeringMapIterator != peeringModeMap_->end(); peeringMapIterator++) {
        // Grab source node.
        MacNodeId srcId = peeringMapIterator->first;

        EV << NOW << "\tChecking for starting node " << srcId << std::endl;
        // Make sure node is in this cell.
        if (binder_->getNextHop(srcId) != mac_->getMacCellId()) {
            EV << NOW << "\t\tskipping. It has left the cell." << std::endl;
            continue;
        }

        // Grab handle to <dest, mode> map.
        std::map<MacNodeId, LteD2DMode>::iterator destModeMapIterator = peeringMapIterator->second.begin();
        for (; destModeMapIterator != peeringMapIterator->second.end(); destModeMapIterator++) {
            // Grab destination.
            MacNodeId dstId = destModeMapIterator->first;

            EV << NOW << "\t\t-> " << dstId << " ";

            // Make sure node is in this cell.
            if (binder_->getNextHop(dstId) != mac_->getMacCellId()) {
                EV << "has left the cell." << std::endl;
                continue;
            }

            // Skip nodes that are performing handover.
            if (binder_->hasUeHandoverTriggered(dstId) || binder_->hasUeHandoverTriggered(srcId)) {
                EV << "is performing handover." << std::endl;
                continue;
            }

            EV << std::endl;

            LteD2DMode oldMode = destModeMapIterator->second;
            // New mode is always DM.
            LteD2DMode newMode = DM; // DM = Direct Mode
            EV << NOW << " D2DModeSelectionForcedD2D::doModeSelection Node " << srcId << " will communicate with node " << dstId << " directly." << std::endl;

            if (newMode != oldMode) {
                // Mark this flow as switching modes.
                FlowId p(srcId, dstId);
                FlowModeInfo info;
                info.flow = p;
                info.oldMode = oldMode;
                info.newMode = newMode;
                switchList_.push_back(info);
                // Update peering map.
                destModeMapIterator->second = newMode;
                EV << NOW << "\tFlow: " << srcId << " --> " << dstId << " switches to " << d2dModeToA(newMode) << " mode." << endl;
            }
        }
      }
    }
    ```
