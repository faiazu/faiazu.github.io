---
layout: page
title: Models
permalink: /projects/tep-models/
---

## Model Comparison At A Glance

I compared the three models below.

| Model | Input representation | Main idea | Strength | Main limitation |
| --- | --- | --- | --- | --- |
| MLP | flattened vector `(3120,)` | treat the window as one feature vector | simple and strong baseline | no explicit time or graph structure |
| CNN | matrix `(60, 52)` then `(52, 60)` for `Conv1d` | learn local temporal patterns | strong time-aware model | no explicit process graph |
| GNN | one graph per window | combine temporal links and process links | structurally meaningful | slow and currently underperforming |

> Visual placeholder: add a one-row comparison graphic showing how the same window is transformed before entering each model.

## 1. MLP Baseline

Training file:

- `src/train_baseline.py`

Model file:

- `src/models/simplemlp.py`

The MLP is intentionally simple. Each `60 x 52` window gets flattened into a vector of length `3120`, and the model predicts one of 21 classes.

Why I kept this baseline:

- it is the fastest way to verify that the faults are predictable
- it gives reference performance to compare other models against

What the MLP assumes:

- the full window of all sensor values can be treated as one feature vector
- useful structure can be learned even without explicitly preserving time of sample taken

> Visual placeholder: add a dense-layer stack diagram with `60 samples x 52 sensors ->3120 -> mlp layers -> 21 scores`.

## 2. CNN Time-Series Model

Training file:

- `src/train_cnn1d.py`

Model file:

- `src/models/cnn1d.py`

The CNN uses the same windows but keeps the time structure instead of flattening it away.

Input flow:

- processed dataset shape: `(N, 60, 52)`
- transposed for convolution: `(N, 52, 60)`

So each sensor becomes a channel and the convolution runs across time.

Why this model makes sense for TEP:

- faults often appear as patterns over time rather than single value anomalies
- local trends and transitions matter
- a convolutional model can learn short-term changes quickly

Current high-level architecture:

- `Conv1d`
- `ReLU`
- second `Conv1d`
- pooling over time
- final linear classifier

Normalization is still done with train-only statistics per sensor channel so the comparison stays consistent with the MLP setup.

> Visual placeholder: add a CNN diagram showing kernels sliding along the 60-step time axis for several sensor channels.

## 3. Graph Neural Network

Training file:

- `src/train_gnn.py`

Model file:

- `src/models/gnn_model.py`

### Current Graph Construction

For each window:

- input window shape: `(60, 52)`
- one node represents one `(time_step, sensor)` pair
- node count per graph: `60 * 52 = 3120`
- node feature shape: `(3120, 1)`

Node indexing:

- `node_id = time_step * 52 + sensor_index`

Edges come from two places.

Temporal edges:

- forward only
- same sensor
- `t -> t + 1`

Process edges:

- based on `EDGES_75`
- repeated at every time step
- treated as undirected by adding both directions

Current edge count per graph:

- `edge_index.shape = (2, 12068)`

### Work in Progress

This part of the project is still unfinished.

Right now, I tried using `SAGEConv` in PyTorch Geometric without fully understanding how a graph neural network works, and I tried applying `SAGEConv` to the TEP dataset to see if the process structure could help with fault classification.

My current approach has been to turn each window into a graph, connect points across time and process relationships, and test whether message passing can learn something useful from that structure. At this stage, it is still more of an experiment than a finished model.

### The Process Graph

The process graph is stored in `src/models/gnn_model.py` as `EDGES_75`.

The graph model's edge list is based off how:

- some variables belong to the same equipment
- some are linked by streams
- some are connected through control actions

## What I Think Each Model Is Testing

The MLP tests whether the fault patterns are separable in a simple flattened representation.

The CNN tests whether local temporal structure helps resolve ambiguity in sensor values.

The GNN tests whether combining time structure with process structure helps enough to justify graph construction and slower training.

## Next

- [Go to the results page](/projects/tep-results/)
- [Go back to the project overview](/projects/tep-fault-diagnosis/)
