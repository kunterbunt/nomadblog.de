+++
title = "Simulation evaluation in OMNeT++"
draft = false
summary = ""
tags = [
  "tech", "simulation"
]
showLogo = false
logo = ""
hasMath = false
date = "2017-05-01T16:57:42+02:00"
+++

[OMNeT++ network simulator](https://omnetpp.org/) allows for extensive and modular simulation of network scenarios. When simulation concept, implementation, configuration and running are said and done, the evaluation is the next step. Some built-in functionalities of the IDE allow for an idea on how the network performed. This was cumbersone for me personally, and I prefer to evaluate the numerical scalar result values in `MATLAB` anyway.

When you run a large number of simulations, then you don't want to open each file in the IDE, select the relevant entries and export them to a `.csv` which MATLAB would understand. It's possible, sure, but also annoying. `grep` to the rescue: I just parse the scalar `.sca` files and look for the parameters of interest to me.

This `parseScalar` bash script automates the parsing:
```bash
#!/bin/bash
if (( $# < 3));
then
	echo "Usage: parseScalar <scalar file> <output file> <number of Rx devices>"
	exit
fi


file=$1
target=$2
numRx=$3

if [ -f $target ]; then
	rm $target
fi
touch $target

for (( i=0; i<$numRx; i+=1 )); do
	grep -e "^scalar LTECell.ueD2DRx\[$i\].udpApp\[0\][[:space:]]\{1,\}rcvdPk:sum(packetBytes)[[:space:]]\{1,\}" $file  | awk '{print $4}' >> $target
done

```
You can use this as a starting point. I was interested in each `ueD2DRx[$i]` device, specifically in the `sum(packetBytes)` of its `udpApp[0]`. Adjust for your needs; the biggest struggle had been being compatible with the spacing of the `.sca` file. It outputs only the numerical values into `$target`, one line for each value.

Another hint: when you're still making sure your simulations do what they're supposed to, you can save the entire event log of a simulation to a file and then go through it using grep to filter only what's interesting. To do that, put into your configuration `omnetpp.ini`:
/
```
cmdenv-express-mode = false
cmdenv-output-file = ${resultdir}/lastrun.log
```

This will save the event log to a `lastrun.log` file. It's going to be quite huge, so `grep` the contents for e.g. a point in time or all output from a specific module.
