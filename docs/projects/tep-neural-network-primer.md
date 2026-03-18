---
layout: page
title: Neural Network Primer For This Project
permalink: /projects/tep-neural-network-primer/
---

## Why I Needed A Primer

This project was one of my first real attempts to learn neural networks by actually building something.

At the start, I had the rough idea of what a neural network was, but not enough to think clearly about model choices. By that I mean I use ChatG on a daily basis. So much so to the point that I call it ChatG instead of ChatGPT.

TEP was good for forcing my learning because the same dataset can be represented in a few different ways, and each model family assumes something different about the data.

This page is not meant to be a textbook explanation. It is just me reocrding my learning along the way, from nothing to atleast a little better level of understanding.

> Visual placeholder: add a side-by-side figure showing the same window represented three ways:
>
> `flattened vector -> MLP`
>
> `sensor-by-time matrix -> CNN`
>
> `spatiotemporal graph -> GNN`

## What A Neural Network Is

I realized I did not actually know what a neural network was, so I had to go learn that first. I watched the first two videos of [this 3Blue1Brown playlist on neural networks](https://youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi&si=s2tmH6kzbQsJJRBR), and it gave me a good starting picture of what a neural network is.

A neural network is loosely inspired by the way we think about pattern recognition in the brain. It takes an input through nodes, or neurons, arranged in layers. As the data moves through those layers, the network processes it and tries to recognize useful patterns.

The connection strengths between nodes are the weights, and training is really the process of adjusting those weights so the network gets better at mapping inputs to outputs.

In this project, the input is a window of process data and the output is a set of 21 class scores, one for each `faultNumber`.

## MLP: The Simplest Baseline

The MLP is the simplest model in the project.

For a `60 x 52` window, I flatten everything into one vector:

- `60 * 52 = 3120`

So the MLP sees one long list of 3120 numbers and predicts the fault class.

Why this baseline matters:

- it proves the dataset pipeline works
- it gives me a reference point before using more structured models
- it is fast and easy to train

What it does well:

- learns global patterns from the whole window
- gives a strong baseline surprisingly quickly

What it does not do explicitly:

- it does not preserve time as a first-class dimension
- it does not use the plant graph

So the MLP can still learn useful things, but it is trying to learn time-related structure from a flattened vector instead of from an input that keeps time intact.

> Visual placeholder: add a diagram that shows a `60 x 52` block being flattened into a `3120`-length vector before entering dense layers.

## CNN: A Time-Aware Model

The CNN keeps the window as a time series instead of flattening it right away.

The input starts as:

- `(60, 52)`

For PyTorch `Conv1d`, it is rearranged to:

- `(52, 60)`

That means:

- 52 channels
- 60 time steps

The basic idea of a 1D CNN is that it learns filters that slide across time. A filter can end up responding to patterns like:

- a rapid change
- a local spike
- a sustained rise or drop
- a short repeated pattern

This matters here because a lot of faults are not just one weird sensor reading. They show up in how signals change over time.

Why I wanted this model:

- it preserves local time structure
- it is still much simpler than a graph model
- it is a fair next step after the MLP

This ended up mattering a lot because the CNN improved the hardest confusion cases much more than the MLP.

> Visual placeholder: add a small diagram of a convolution kernel sliding across time on one sensor channel, then another diagram showing multiple learned feature maps.

## GNN: A Graph-First Model

The GNN starts from a different idea. The sensors are not just random sensors in an array. All of them belong to a certain point in the chemical process flow diagram.

In the current graph design:

- one graph is built for each window
- each node represents a `(sensor, time_step)` pair
- with `60` time steps and `52` sensors, each graph has `3120` nodes
- each node currently has one scalar feature: the sensor value at that time step

The graph includes two kinds of edges:

- temporal edges (directed) that connect the same sensor from `t` to `t+1`
- process edges that connect related variables using the `EDGES_75` process graph at each time step

With this combination of edges, a spatiotemporal graph is created.

The basic intuition for the GNN is:

- each node starts with its own value
- it exchanges information with neighboring nodes
- after a few rounds, each node representation contains some local context
- a pooled graph representation is used for final classification

Why the GNN is appealing in theory:

- the process plant has structure between nodes
- some variables are related by physics and control, not just by correlation
- a graph model might capture interactions that a plain array model ignores

Why it is harder in practice:

- graph construction is more complicated with so many sensors
- training is much slower
- the extra structure does not automatically improve performance

These tradeoffs are one of the main things this project taught me.

> Visual placeholder: add a graph diagram for one short toy window, for example `3` time steps and `4` sensors, so the node and edge idea is easy to see.

## Why Compare These Three Models

The comparison makes sense because each model is really testing a different idea.

The MLP asks:

- how much can I get from a strong but simple baseline?

The CNN asks:

- does explicit time awareness help?

The GNN asks:

- does adding process structure help enough to justify the extra complexity?

It is a pretty clean progression:

1. start simple
2. keep the time dimension
3. add graph structure

This also made the project a good way for me to learn neural networks. Instead of just reading definitions, I had to decide what each model should actually be looking at and why.

## What I Learned From The Primer Stage

The biggest lesson here is that architecture should follow representation.

If I flatten the input, I should expect the model to lose some time structure.

If I keep the time axis, I should expect the model to make use of local temporal patterns.

If I build a graph, I am specified about what the nodes and edges mean physically, not just computationally.

Thinking about it that way made the later model comparisons make a lot more sense.

## Next

- [Go to data and windowing](/projects/tep-data-and-windowing/)
- [Go to the model details page](/projects/tep-models/)
