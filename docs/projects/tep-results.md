---
layout: page
title: Results And Interpretation
permalink: /projects/tep-results/
---

## Evaluation Setup

These are the results from the validation split used by `src/evaluate_model.py`.

- validation runs: `16` to `20`
- evaluated examples: `4725`
- number of classes: `21`
- windows per fault on validation: `225`
- all three models here were trained for only `3` epochs

So these are not supposed to be the absolute best numbers the models could ever get. This is just what they got with short training runs. If I train longer, I would expect at least some of these accuracies to go up.

Dataset files used:

<div align="center">

| Model | Dataset file | Input shape |
| --- | --- | --- |
| MLP | `data/processed/small_fault0_20_runs1_20_W60_step10.npz` | `(N, 3120)` |
| CNN | `data/processed/small_fault0_20_runs1_20_W60_step10_2d.npz` | `(N, 60, 52)` |
| GNN | `data/processed/small_fault0_20_runs1_20_W60_step10_2d.npz` | `(N, 60, 52)` |

</div>

Dataset build commands:

```bash
python src/build_small_dataset.py --fault-start 0 --fault-end 20 --run-start 1 --run-end 20
python src/build_small_dataset_2d.py --fault-start 0 --fault-end 20 --run-start 1 --run-end 20
```

Training commands:

```bash
python src/train_baseline.py
python src/train_cnn1d.py
python src/train_gnn.py
```

Evaluation commands:

```bash
python src/evaluate_model.py --model mlp
python src/evaluate_model.py --model cnn
python src/evaluate_model.py --model gnn
```

## Overall Comparison

<div align="center">

| Model | Top-1 | Top-2 | Top-3 | Short read |
| --- | --- | --- | --- | --- |
| MLP | `69.71%` | `77.02%` | `80.99%` | stronger than I expected, but it still mixes up some faults badly |
| CNN | `84.13%` | `88.93%` | `93.84%` | best result so far and the most reliable overall |
| GNN | `32.11%` | `44.36%` | `52.49%` | basically my rough `SAGEConv` attempt, so not something I would call fully figured out yet |

</div>

The CNN is clearly the best model so far on this split. The MLP is still a good baseline. The GNN result needs more work done because it's pretty much just me trying `SAGEConv` / GraphSAGE on the TEP data via *vibe coding* before I fully understand how a graph neural network should really be set up.

Also, all of these numbers are from models trained for only `3` epochs. So I think the comparison is still useful, but I would not act like these are the final best accuracies each model could reach.

> Chart placeholder: add a grouped bar chart for top-1, top-2, and top-3 accuracy across MLP, CNN, and GNN.

## Per-Fault Accuracy Comparison

<div align="center">

| Fault | MLP | CNN | GNN |
| --- | --- | --- | --- |
| 0 | `0.00%` | `0.00%` | `0.00%` |
| 1 | `100.00%` | `100.00%` | `88.89%` |
| 2 | `100.00%` | `100.00%` | `91.11%` |
| 3 | `0.00%` | `100.00%` | `0.00%` |
| 4 | `99.11%` | `100.00%` | `0.00%` |
| 5 | `100.00%` | `100.00%` | `4.44%` |
| 6 | `100.00%` | `100.00%` | `85.33%` |
| 7 | `100.00%` | `100.00%` | `0.00%` |
| 8 | `83.56%` | `96.44%` | `5.78%` |
| 9 | `1.78%` | `8.44%` | `78.22%` |
| 10 | `64.00%` | `94.22%` | `35.56%` |
| 11 | `58.22%` | `99.56%` | `0.00%` |
| 12 | `91.56%` | `99.11%` | `72.00%` |
| 13 | `93.33%` | `94.22%` | `0.00%` |
| 14 | `100.00%` | `100.00%` | `0.00%` |
| 15 | `0.00%` | `0.00%` | `0.00%` |
| 16 | `13.33%` | `92.00%` | `2.67%` |
| 17 | `97.33%` | `96.89%` | `92.44%` |
| 18 | `92.44%` | `92.44%` | `29.33%` |
| 19 | `75.56%` | `100.00%` | `88.44%` |
| 20 | `93.78%` | `93.33%` | `0.00%` |

</div>

This is probably the most useful table on the page because it shows where each model actually works and where it falls apart. The CNN is the best overall, but it still completely misses faults `0` and `15`, and it only gets `8.44%` on fault `9`. The MLP is a lot less consistent, and the current GNN is still rough enough that I would not read too much into it yet.

> Chart placeholder: add a grouped per-fault accuracy bar chart with one bar each for MLP, CNN, and GNN across faults `0` to `20`.

## Confusion Matrices

For all three matrices below:

- rows = true label
- columns = predicted label
- class order = `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20`

### MLP Confusion Matrix

The MLP does better than I expected for something this simple, but when it misses, it misses in a pretty concentrated way. Faults `0`, `3`, `9`, `15`, and `16` are where it struggles the most, and a lot of those windows end up getting pushed into class `19` or class `11`.

```text
    0    1    2   3    4    5    6    7    8   9    10   11   12   13   14  15  16   17   18   19   20
0    0    0    0   0    0    0    0    0    0   4    7   26    0    0    0   0   5    0    0  183    0
1    0  225    0   0    0    0    0    0    0   0    0    0    0    0    0   0   0    0    0    0    0
2    0    0  225   0    0    0    0    0    0   0    0    0    0    0    0   0   0    0    0    0    0
3    0    0    0   0    0    0    0    0    0   7    7   41    0    0    0   0   5    0    0  165    0
4    0    0    0   0  223    0    0    0    0   0    0    2    0    0    0   0   0    0    0    0    0
5    0    0    0   0    0  225    0    0    0   0    0    0    0    0    0   0   0    0    0    0    0
6    0    0    0   0    0    0  225    0    0   0    0    0    0    0    0   0   0    0    0    0    0
7    0    0    0   0    0    0    0  225    0   0    0    0    0    0    0   0   0    0    0    0    0
8    0   15    0   0    0    0    0    0  188   0    1    8    9    2    0   0   1    0    0    1    0
9    0    0    0   0    0    0    0    0    0   4    8   25    0    0    0   0   6    0    0  182    0
10   0    0    0   0    0    0    0    0    0   0  144   11    5    1    0   0  13    0    0   51    0
11   0    0    0   0   19    0    0    0    0   1    2  131    0    0    0   0   4    0    0   68    0
12   0    0    0   0    0    5    0    0    0   0    7    4  206    0    0   0   0    0    0    3    0
13   0    0    0   0    0    0    0    0    0   0    0    8    0  210    0   0   0    0    0    7    0
14   0    0    0   0    0    0    0    0    0   0    0    0    0    0  225   0   0    0    0    0    0
15   0    0    0   0    0    0    0    0    0   4    7   26    0    0    0   0   5    0    0  183    0
16   0    0    0   0    0    0    0    0    0   1   54    8    0    0    0   0  30    0    0  132    0
17   0    0    0   0    0    0    0    0    0   0    0    3    0    0    0   0   0  219    0    3    0
18   0    0    0   0    0    0    0    0    0   0    0    6    0    0    0   0   0    0  208   11    0
19   0    0    0   0    0    0    0    0    0   2   12   32    0    0    0   0   9    0    0  170    0
20   0    0    0   0    0    0    0    0    0   0    0    5    0    0    0   0   0    0    0    9  211
```

> Chart placeholder: add the MLP confusion matrix heatmap.

### CNN Confusion Matrix

The CNN is the best model so far, but the confusion matrix makes it clear where it still struggles. It is really strong on a lot of classes, but faults `0`, `9`, and `15` are still a real issue, and in this version they mostly collapse into class `3`.

```text
    0    1    2    3    4    5    6    7    8   9    10   11   12   13   14  15   16   17   18   19   20
0    0    0    0  212    0    0    0    0    0  13    0    0    0    0    0   0    0    0    0    0    0
1    0  225    0    0    0    0    0    0    0   0    0    0    0    0    0   0    0    0    0    0    0
2    0    0  225    0    0    0    0    0    0   0    0    0    0    0    0   0    0    0    0    0    0
3    0    0    0  225    0    0    0    0    0   0    0    0    0    0    0   0    0    0    0    0    0
4    0    0    0    0  225    0    0    0    0   0    0    0    0    0    0   0    0    0    0    0    0
5    0    0    0    0    0  225    0    0    0   0    0    0    0    0    0   0    0    0    0    0    0
6    0    0    0    0    0    0  225    0    0   0    0    0    0    0    0   0    0    0    0    0    0
7    0    0    0    0    0    0    0  225    0   0    0    0    0    0    0   0    0    0    0    0    0
8    0    0    0    2    0    0    0    0  217   0    2    0    0    0    0   0    4    0    0    0    0
9    0    0    0  206    0    0    0    0    0  19    0    0    0    0    0   0    0    0    0    0    0
10   0    0    0    3    0    0    0    0    0   0  212    0    0    0    0   0   10    0    0    0    0
11   0    0    0    0    1    0    0    0    0   0    0  224    0    0    0   0    0    0    0    0    0
12   0    0    0    0    0    0    0    0    0   0    0    0  223    0    0   0    0    0    2    0    0
13   0    0    0   10    0    0    0    0    2   1    0    0    0  212    0   0    0    0    0    0    0
14   0    0    0    0    0    0    0    0    0   0    0    0    0    0  225   0    0    0    0    0    0
15   0    0    0  211    0    0    0    0    0  14    0    0    0    0    0   0    0    0    0    0    0
16   0    0    0   17    0    0    0    0    0   1    0    0    0    0    0   0  207    0    0    0    0
17   0    0    0    7    0    0    0    0    0   0    0    0    0    0    0   0    0  218    0    0    0
18   0    0    0   17    0    0    0    0    0   0    0    0    0    0    0   0    0    0  208    0    0
19   0    0    0    0    0    0    0    0    0   0    0    0    0    0    0   0    0    0    0  225    0
20   0    0    0   15    0    0    0    0    0   0    0    0    0    0    0   0    0    0    0    0  210
```

> Chart placeholder: add the CNN confusion matrix heatmap.
>
> Chart placeholder: add a zoomed heatmap for faults `0`, `3`, `9`, `15`, and `16`.

### GNN Confusion Matrix

The current GNN result needs a giant asterisk next to it. This is not some clean final graph model. It is literally me vibe coding using `SAGEConv` in PyTorch Geometric without fully understanding how a graph neural network works yet, and then applying that to the TEP windows.

Even with that caveat, the result still matters. The current graph setup does not beat the simpler models. A lot of classes collapse into just a few predictions, especially around classes `9`, `10`, `12`, `17`, and `19`.

```text
    0    1    2   3   4   5    6   7   8    9    10  11   12  13  14  15  16   17  18   19  20
0    0    0    0   0   0   0    0   0   0  179   37   0    0   0   0   0   8    1   0    0   0
1    0  200    0   0   0   2    0   0   3    0    0   0   20   0   0   0   0    0   0    0   0
2    0   18  205   0   0   0    0   0   0    0    0   0    0   0   0   0   0    0   0    2   0
3    0    0    0   0   0   0    0   0   0  179   37   0    0   0   0   0   8    1   0    0   0
4    0    0    0   0   0   0    0   0   0  164   58   0    0   0   0   0   2    1   0    0   0
5    0    7    0   0   0  10    0   0  10   71   71   0   24   0   0   0   1   24   0    7   0
6    0    0    0   0   0   0  192   0   1    0    0   0   21   0   0   0   0   11   0    0   0
7    0   56    0   0   0  21    0   0   2    0   62   0   70   0   0   0   0    2   0   12   0
8    0   50    6   0   0   5    0   0  13    6   13   0  113   0   0   0   0   19   0    0   0
9    0    0    0   0   0   0    0   0   0  176   41   0    0   0   0   0   7    1   0    0   0
10   0    0    0   0   0   4    0   0   0   85   80   0    0   0   0   0   9   41   0    6   0
11   0    0    0   0   0   0    0   0   0  164   58   0    0   0   0   0   2    1   0    0   0
12   0   13    0   0   0   9    6   0   8    0    7   0  162   0   0   0   0    7   5    8   0
13   0   56    0   0   0   9    3   0  21   12    7   0  102   0   0   0   0   12   3    0   0
14   0    0    0   0   0   0    0   0   0   45  162   0    0   0   0   0   3   11   0    4   0
15   0    0    0   0   0   0    0   0   0  180   34   0    0   0   0   0   8    3   0    0   0
16   0    0    0   0   0   0    0   0   0  140   71   0    0   0   0   0   6    8   0    0   0
17   0    0    0   0   0   0    0   0   0   13    3   0    0   0   0   0   1  208   0    0   0
18   0    1    0   0   0   0   58   0   2   21    1   0   63   0   0   0   0   13  66    0   0
19   0    0    0   0   0   0    0   0   0    0    8   0    0   0   0   0   0   18   0  199   0
20   0    0    0   0   0   0    0   0   0   96   88   0    0   0   0   0  22   19   0    0   0
```

> Chart placeholder: add the GNN confusion matrix heatmap.
>
> Chart placeholder: add a side-by-side heatmap comparison of MLP, CNN, and GNN.

## What These Results Say Right Now

- The CNN is the best model here, with `84.13%` top-1 accuracy.
- The MLP is still pretty good at `69.71%` top-1 accuracy with a much simpler input.
- The current GNN is weak at `32.11%` top-1 accuracy, when I actually create a GNN model myself, I hope this accuracy will be similar or higher than the CNN.
- All of these runs only used `3` epochs, so I do expect at least some of these accuracies to go up with longer training.
- The jump from MLP to CNN is big enough that I think time structure clearly matters here.
- Faults `0` and `15` are hard for every model in this setup.
- Fault `9` is also still pretty messy for the MLP and CNN.

## Charts To Add Later

This page will look a lot better once I actually add the visuals.

- top-k grouped bar chart: compare top-1, top-2, and top-3 across the three models
- per-fault grouped bar chart: compare all `21` faults side by side
- three confusion matrix heatmaps: one each for MLP, CNN, and GNN
- hard-fault zoom chart: show just faults `0`, `3`, `9`, `15`, and `16`
- optional runtime chart: compare training time and evaluation time once I have those numbers recorded cleanly

## Next

- [Go back to the main overview](/projects/tep-fault-diagnosis/)
- [Go back to the models page](/projects/tep-models/)
