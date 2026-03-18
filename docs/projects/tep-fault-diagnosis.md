---
layout: page
title: Tennessee Eastman Process Fault Diagnosis
permalink: /projects/tep-fault-diagnosis/
---

## Overview

This project is basically my introduction to neural networks through a chemical engineering problem that I saw while doomscrolling LinkedIn.

I am working on fault diagnosis with the Tennessee Eastman Process (TEP) dataset. If you do not know what that is, I wrote a page on [what the Tennessee Eastman Process is](/projects/tep-notes/).

The task is to predict `faultNumber` for 21 classes:

- `0` for normal operation
- `1` to `20` for different injected fault scenarios

The main comparison is between three model families:

- a simple MLP baseline
- a 1D CNN for multivariate time-series windows
- a graph neural network built from a process graph

I am not just trying to train a model and move on. I am also using this project to learn how these different model types actually behave on the same problem, what parts of the pipeline matter, and what goes wrong when the setup is bad.

> Visual placeholder: insert visual diagram showing the project pipeline.
>
> Diagram: `Raw TEP CSVs -> Sliding training windows -> MLP / CNN / GNN -> Model evaluation results`.

## Why This Project

I did not want to learn neural networks on some toy example where none of the model choices really matter.

I wanted something real enough that:

- the data had structure
- the model choice actually mattered
- I would have to think about preprocessing, labels, and evaluation properly
- at the same time, I wanted to learn via application in chemical processes

TEP ended up being a a perfect example of that.

Why:

- it is a realistic process simulation rather than a toy dataset
- it contains many interacting variables instead of a single signal
- faults appear over time, so time structure matters
- the plant itself has a natural graph structure, which makes the GNN idea worth testing
- it's literally data from a running chemical process 

This is also why I enjoy working on this project so much. The MLP, CNN, and GNN are not just three random models I picked. They are three different ways of representing the exact same problem.

## What The Data Looks Like

The raw TEP data is organized by simulation runs. Each row is one time point from one run.

Important columns:

- `faultNumber`
- `simulationRun`
- `sample`
- `xmeas_1..41`
- `xmv_1..11`

So at each time step, there are 52 actual process variables.

Each sample is taken every 3 minutes:

- training runs have samples `1..500`
- testing runs have samples `1..960`

Fault timing matters:

- in faulty training data, the fault begins at sample `21`
- in faulty testing data, the fault begins at sample `161`

That part mattered more than I expected because it affects how windows should be labeled. A faulty run is not actually faulty from the very first sample.

> Visual placeholder: add a small timeline figure showing normal period and fault-onset period for training vs testing runs.

## Windowing And Inputs

The models do not just look at an entire simulation run at once. I turn each simulation run into multiple sliding windows.

Current standard settings:

- `window_size = 60`
- `step_size = 10`

Since each sample is 3 minutes, that means each window covers 3 hours of process running.

I use two dataset formats:

- flattened windows for the MLP: `(N, 3120)`
- 2D windows for CNN and GNN: `(N, 60, 52)`
(N is the number of windows)

This part sounds small, but it really drives most of the project. Once the window shape is set, it affects the model input, the labels, the training speed, and how fair the model comparison is.

## What I Built

There are three training pipelines:

- `MLP`: a simple feedforward classifier on flattened windows
- `CNN`: a 1D convolutional model that keeps the time dimension intact
- `GNN`: a graph model that uses both temporal edges and process edges

All models are trained on a split of simulationRuns. For example, if the generated dataset is of runs 1-20, then runs 1..15 will be for training and runs 16..20 for validation (where it doesn't learn).

I also built an evaluation script which is produces the same metrics for all three models, so the comparison between the models is consistent.

## Main Lessons So Far

The biggest thing I learned early is that a simple baseline can be a lot stronger than you expect.

The MLP works well enough that the more complicated models actually have to justify themselves.

The second thing I learned is that time awareness really does matter here. The CNN improved the hardest confusion cases (based on per-fault accuracy and confusion matrix) much more than the MLP.

The third thing is that the graph idea is interesting, but it is not automatically better just because the plant has structure. The GNN is slower, more complicated, and so far has been disappointing (in my implementation) compared to the CNN.

However, I do believe I did not implement the GNN properly. In my theory, it should have the highest accuracy, as a Chemical Process runs with lots of correlatons (edges) between the sensors (nodes).

## How To Read The Rest

I split this project into a few pages so I do not have to dump everything into one giant write-up.

- [What the Tennessee Eastman Process is](/projects/tep-notes/)
- [Neural network primer](/projects/tep-neural-network-primer/)
- [Data and windowing](/projects/tep-data-and-windowing/)
- [Models](/projects/tep-models/)
- [Results and interpretation](/projects/tep-results/)

If you just want the short version, this page is enough. If you want the details, the rest of the pages go through the whole process properly.

## Next

- [Start with what the Tennessee Eastman Process is](/projects/tep-notes/)
- [Or jump to me attempting to explain neural networks](/projects/tep-neural-network-primer/)
