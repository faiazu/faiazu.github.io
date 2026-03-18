---
layout: page
title: Data And Windowing
permalink: /projects/tep-data-and-windowing/
---

## Why This Stage Matters

Before any model can train, the raw TEP runs have to be turned into examples with consistent shapes, labels, and splits.

## Raw Data Layout

Each raw CSV row is one time point from one simulation run.

Important columns:

- `faultNumber`
- `simulationRun`
- `sample`
- `xmeas_1..41`
- `xmv_1..11`

So each time step has:

- 52 process variables
- 1 class label
- 1 run identifier
- 1 time index

The models use the 52 process variables as input, and `faultNumber` is the label.

> Visual placeholder: add a diagram of one run as a table with rows as time and columns as process variables.

## Time Resolution And Fault Timing

Each sample is taken every 3 minutes.

Training runs:

- samples `1..500`

Testing runs:

- samples `1..960`

Fault onset differs between training and testing:

- faulty training runs are normal through sample `20`, and faulty behavior begins at sample `21`
- faulty testing runs are normal through sample `160`, and faulty behavior begins at sample `161`

This matters because a faulty run is not actually faulty from the very first row.

## Turning Runs Into Windows

The raw data is one long sequence per run, but the models need fixed-size examples, so I use sliding windows.

Current standard settings:

- `window_size = 60`
- `step_size = 10`

So one example contains:

- 60 time steps
- 52 variables per time step

Since each time step is 3 minutes, each window covers 180 minutes, or 3 hours.

Sliding the window by 10 means neighboring examples overlap, but not fully. That gives me a lot of examples without throwing away the time progression inside a run.

> Visual placeholder: add a simple sequence diagram showing a run and three overlapping windows with step size 10.

## Two Dataset Formats

I ended up needing two processed dataset formats.

Flattened dataset:

- built by `src/build_small_dataset.py`
- input shape: `(N, 3120)`
- intended for the MLP

2D dataset:

- built by `src/build_small_dataset_2d.py`
- input shape: `(N, 60, 52)`
- intended for the CNN and GNN

Both builders:

- read raw CSV files
- group rows by run
- build sliding windows
- save `.npz` files
- save metadata such as `window_size` and `step_size`

The builders also support fault and run ranges through CLI arguments, which makes it easier to work on a smaller subset before scaling up.

Example commands:

```bash
python src/build_small_dataset.py --fault-start 0 --fault-end 20 --run-start 1 --run-end 20
python src/build_small_dataset_2d.py --fault-start 0 --fault-end 20 --run-start 1 --run-end 20
```

## Labeling Windows

The label for each example is `faultNumber`, but there was one annoying detail here.

In faulty runs, the run has a fault identity from the beginning, but the actual fault only starts later in time.

That created an earlier labeling bug:

- early pre-fault windows in faulty runs were being labeled with the run's fault number even though the fault had not started yet

The fix was basically:

- decide the window label using the window end sample and the fault-start threshold
- use threshold `20` for training
- use threshold `160` for testing

With the current standard setup, `window_size = 60`, so the first training window already ends after sample `20`. That means this fix does not change much for the current training setup, but it is still the correct logic and it would matter more for smaller windows or other settings.

> Visual placeholder: add a window-labeling diagram with one faulty run, a fault onset marker, and examples of pre-fault vs post-fault windows.

## Splitting Train And Validation Correctly

I split the data by `simulationRun`, not by random rows.

I had to be careful here. If windows from the same run get mixed into both train and validation, the validation score can look better than it really is because the windows are so similar.

There was also a bug when I switched to smaller datasets:

- validation runs were hardcoded to `376..500` as originally training was on the full 500 simulation run dataset
- this caused empty validation sets when using runs `1..20`

The trainers were updated to choose validation runs adaptively:

- if runs `376..500` exist, use those
- otherwise use the last 25 percent of available runs

For runs `1..20`, that means:

- training runs `1..15`
- validation runs `16..20`

That fix matters because it keeps the setup consistent whether I am testing on a small subset or training on the full data.

## Normalization

Normalization turned out to be something I actually needed, not just a nice extra.

Different TEP variables live on very different scales. Some are small fractions and some are much bigger numbers. If I feed raw values straight into the model, the large-scale variables can dominate for the wrong reason.

The setup I use is:

- compute normalization statistics on training data only
- apply those statistics to validation data

For the MLP, normalization is applied to the flattened features.

For the CNN, normalization is applied per sensor channel.

This keeps training more stable and makes the comparison fairer.

> Visual placeholder: add a before-and-after normalization figure for a few sensors with very different scales.

## Current Small-Iteration Setup

The current default setup for faster iteration is:

- faults `0..20`
- runs `1..20`
- `window_size = 60`
- `step_size = 10`

This gives me a manageable subset to test the models on before scaling up, without having to wait forever for training to finish.

## Why This Page Matters For The Model Comparison

The processed dataset is not just a preprocessing detail. It decides what information each model actually gets to see and how fair the comparison is.

The MLP sees flattened windows.

The CNN sees time-preserving windows.

The GNN starts from the same 2D windows but turns them into graphs.


## Next

- [Go to the model details page](/projects/tep-models/)
- [Go to the results page](/projects/tep-results/)
