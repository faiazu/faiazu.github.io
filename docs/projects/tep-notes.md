---
layout: page
title: What The Tennessee Eastman Process Is
permalink: /projects/tep-notes/
---

## Why This Page Exists

Before getting into the models, I should explain what the Tennessee Eastman Process actually is.

The short version is that TEP is a chemical process simulation dataset. It models a plant with connected units, control actions, measured variables, manipulated variables, and different fault scenarios.

What I like about it is that it does not just feel like random sensor data. It's a flow from one thing to another. That is a big part of why the GNN idea made sense to try in the first place.

![Tennessee Eastman Process flow diagram](/assets/images/projects/tep/TEP_flow_diagram.jpg)

This is just like the process flow diagrams I learned about in CHE113 (Concepts in Chemical Engineering).

This flow diagram makes the dataset a lot easier to think about. Instead of seeing `xmeas_*` and `xmv_*` as disconnected variable names, you can actually place them onto parts of the process like the reactor, separator, stripper, recycle loop, purge, and product stream. I have yet to label where these sensors actually are, but I will do that.

## What TEP Represents

The Tennessee Eastman Process is a simulated chemical process involving several connected units:

- reactor
- condenser
- separator
- stripper
- compressor

Material and energy move through these units, and the control system is also doing its own thing to keep the plant running.

So the dataset has a few things going on at the same time:

- process dynamics over time
- coupling between units
- control actions
- normal operating behavior
- abnormal behavior after faults are introduced

That is why TEP is such a good learning project for me. It gives me a reason to care about time-series models and graph models on the same problem.

## What The Raw Files Contain

I got the CSV version of the dataset from [this Kaggle dataset](https://www.kaggle.com/datasets/afrniomelo/tep-csv/).

That version is derived from the original source here: [Harvard Dataverse](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/6C3JR1).

The raw CSV download contains four files:

- `TEP_FaultFree_Training.csv`
- `TEP_FaultFree_Testing.csv`
- `TEP_Faulty_Training.csv`
- `TEP_Faulty_Testing.csv`

Each row is one time point from one simulation run.

Important columns:

- `faultNumber`
- `simulationRun`
- `sample`
- `xmeas_1..41`
- `xmv_1..11`

What those mean:

- `faultNumber`: the class label I want to predict
- `simulationRun`: which run of the process that row belongs to
- `sample`: time index within that run (every 3 mins a sample is taken)
- `xmeas_*`: measured variables from the plant
- `xmv_*`: manipulated variables that the control system can adjust

The 52 process variables (xmeas_* & xmv_*) are the actual inputs to the models.

> Visual placeholder: add a simple annotated table screenshot of one CSV row with arrows pointing to `faultNumber`, `simulationRun`, `sample`, `xmeas_*`, and `xmv_*`.

## Measured Vs Manipulated Variables

One thing about the TEP dataset that I didn't realize at first is the difference between what the plant measures and what the control system changes.

`xmeas_*` are measured variables. These are sensor readings such as flows, temperatures, pressures, levels, and compositions.

`xmv_*` are manipulated variables. These are values the controller can change on purpose, such as valve positions or control actions.

Some variables are telling you what the plant is doing. Others are telling you what the controller is changing. It also helps explain why a process graph connecting the sensor nodes makes sense here. Some variables are related because they are in the same process unit, some because material moves between them, and some because a manipulated variable directly affects a measured one.

## What The Sensor Values Actually Are

One thing that confused me early on was that `xmeas_1`, `xmeas_2`, and so on do not tell you much by themselves. They are not random names though. They each correspond to an actual process variable.

I got this mapping from [this blog post](https://keepfloyding.github.io/posts/data-explor-TEP-1/), which points to [this source](https://www.sciencedirect.com/science/article/abs/pii/S0098135414000969?via%3Dihub).

For the measured variables:

- `xmeas_1`: A feed stream
- `xmeas_2`: D feed stream
- `xmeas_3`: E feed stream
- `xmeas_4`: Total fresh feed to stripper
- `xmeas_5`: Recycle flow into reactor
- `xmeas_6`: Reactor feed rate
- `xmeas_7`: Reactor pressure
- `xmeas_8`: Reactor level
- `xmeas_9`: Reactor temperature
- `xmeas_10`: Purge rate
- `xmeas_11`: Separator temperature
- `xmeas_12`: Separator level
- `xmeas_13`: Separator pressure
- `xmeas_14`: Separator underflow
- `xmeas_15`: Stripper level
- `xmeas_16`: Stripper pressure
- `xmeas_17`: Stripper underflow
- `xmeas_18`: Stripper temperature
- `xmeas_19`: Stripper steam flow
- `xmeas_20`: Compressor work
- `xmeas_21`: Reactor cooling water outlet temperature
- `xmeas_22`: Condenser cooling water outlet temperature
- `xmeas_23`: Composition of A in reactor feed
- `xmeas_24`: Composition of B in reactor feed
- `xmeas_25`: Composition of C in reactor feed
- `xmeas_26`: Composition of D in reactor feed
- `xmeas_27`: Composition of E in reactor feed
- `xmeas_28`: Composition of F in reactor feed
- `xmeas_29`: Composition of A in purge
- `xmeas_30`: Composition of B in purge
- `xmeas_31`: Composition of C in purge
- `xmeas_32`: Composition of D in purge
- `xmeas_33`: Composition of E in purge
- `xmeas_34`: Composition of F in purge
- `xmeas_35`: Composition of G in purge
- `xmeas_36`: Composition of H in purge
- `xmeas_37`: Composition of D in product
- `xmeas_38`: Composition of E in product
- `xmeas_39`: Composition of F in product
- `xmeas_40`: Composition of G in product
- `xmeas_41`: Composition of H in product

For the manipulated variables:

- `xmv_1`: D feed flow valve
- `xmv_2`: E feed flow valve
- `xmv_3`: A feed flow valve
- `xmv_4`: Total feed flow stripper valve
- `xmv_5`: Compressor recycle valve
- `xmv_6`: Purge valve
- `xmv_7`: Separator pot liquid flow valve
- `xmv_8`: Stripper liquid product flow valve
- `xmv_9`: Stripper steam valve
- `xmv_10`: Reactor cooling water flow valve
- `xmv_11`: Condenser cooling water flow valve

One small detail: some references also list `xmv_12` as agitator speed, but the CSV version I am using only has `xmv_1..11`, so the working dataset for this project still has 52 features total.

> Visual placeholder: add a labelled plant diagram that maps a few `xmeas_*` and `xmv_*` variables back onto the reactor, separator, stripper, purge, and recycle loop.

## Time Structure

Time is built right into the dataset.

- one sample is recorded every 3 minutes
- training runs have samples `1..500`
- testing runs have samples `1..960`

So each training run covers about 25 hours, and each testing run covers about 48 hours.

Faults are introduced partway through faulty runs:

- faulty training runs look normal up to sample `20`, and the fault begins at sample `21`
- faulty testing runs look normal up to sample `160`, and the fault begins at sample `161`

So a faulty run has both normal-looking and faulty behavior inside it. That ends up mattering later when building labels for windows.

> Visual placeholder: add a training vs testing timeline with clear fault-onset markers.

## What The Fault Numbers Actually Mean

The `faultNumber` labels are not just arbitrary class IDs. Each one corresponds to a specific deviation from normal operation.

`0` means normal operation.

For the fault cases, the mapping is:

| Fault ID | Description | Type |
| --- | --- | --- |
| 1 | A/C Feed ratio, B Composition constant | Step |
| 2 | B Composition, A/C Ratio constant | Step |
| 3 | D Feed temperature | Step |
| 4 | Reactor cooling water inlet temperature | Step |
| 5 | Condenser cooling water inlet temperature | Step |
| 6 | A Feed loss | Step |
| 7 | C Header pressure loss - reduced availability | Step |
| 8 | A, B, C Feed composition | Random variation |
| 9 | D Feed temperature | Random variation |
| 10 | C Feed temperature | Random variation |
| 11 | Reactor cooling water inlet temperature | Random variation |
| 12 | Condenser cooling water inlet temperature | Random variation |
| 13 | Reaction kinetics | Slow drift |
| 14 | Reactor cooling water valve | Sticking |
| 15 | Condenser cooling water valve | Sticking |
| 16 | Unknown | Random variation |
| 17 | Unknown | Random variation |
| 18 | Unknown | Step |
| 19 | Unknown | Random variation |
| 20 | Unknown | Random variation |

What those fault types mean:

- `Step`: the process variable changes suddenly and then stays shifted
- `Random variation`: the fault shows up as noisy or fluctuating variation rather than one clean jump
- `Slow drift`: the effect builds gradually over time instead of appearing all at once
- `Sticking`: usually means a valve or actuator is not moving smoothly, so it can get stuck or respond in a jerky way

This helps a lot when looking at results later, because some confusions make more sense once you know what the faults actually are. A confusion between two similar cooling-related faults means something very different from a completely unrelated mix-up.

The models do not get these fault descriptions directly, and I do not manually tell them which sensors each fault should affect. They only see windows of sensor values and the final fault label.

I want the models to learn from the process data itself, not from extra hand-given fault information. Google's data checklist says data leakage happens when a model uses "predictive information that won't be available at serving time" ([Google](https://services.google.com/fh/files/blogs/data-prep-checklist-ml-bd-wp-v2.pdf)), and IBM defines leakage as using information that "wouldn't be available at the time of prediction" ([IBM](https://www.ibm.com/think/topics/data-leakage-machine-learning)). In this project, if the process was running real time, we wouldn't have the exact faults and how they occur. The only data would be from the sensors, so that is what I want the models to rely on.

> Visual placeholder: add a grouped fault chart showing step faults, random-variation faults, slow-drift faults, and sticking faults.

## Why TEP Is A Good Benchmark For This Project

TEP works well for the kind of comparison I want to make.

For the MLP, it asks whether a simple classifier can learn enough from flattened windows (unaware of sample time).

For the CNN, it asks whether local temporal patterns carry important information.

For the GNN, it asks whether explicitly using the process structure helps beyond just treating all the sensors as an array.

So the comparison feels fair. All three model families make sense on this dataset for different reasons.

## Next

- [Go to the neural network primer](/projects/tep-neural-network-primer/)
- [Go to data and windowing](/projects/tep-data-and-windowing/)
