# Telecom Pathfinding

Pathfinding experiments on the [SNAP Telecom Graph](https://snap.stanford.edu/data/telecom-graph/) using informed search and local search. The notebook compares:

- **A\*** with a simple admissible unit-cost heuristic
- **ALT**: A\* accelerated with landmark distances and the triangle inequality
- **ALT+**: the stronger of the ALT bound and the unit-cost bound
- **Hill Climbing**: steepest-ascent search, with optional sideways moves and random restarts

The project is designed as an experimental, report-ready comparison of optimal search and local search on a heterogeneous telecom graph containing users, packages, apps, and cells.

[![Open in Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Saku53/Telecom_Pathfinding/blob/main/telecom_pathfinding_v3.ipynb)

## What the notebook does

1. Downloads and discovers the SNAP Telecom Graph edge files.
2. Detects edge types from filenames such as `edge_user_buy_package`.
3. Creates globally unique node IDs for each node type. SNAP IDs are local to their type, so this prevents collisions between, for example, `user 1` and `app 1`.
4. Converts relationship attributes into traversal costs:

   ```text
   cost = 1 + (1 - attribute)
   ```

   Stronger relationships are therefore cheaper, and every edge cost is in `[1, 2]`.
5. Treats relationships as bidirectional and keeps only the largest connected component.
6. Samples reproducible start/goal queries, normally between users.
7. Computes optimal reference paths with SciPy's Dijkstra implementation.
8. Runs every search algorithm on the same queries.
9. Checks that A*, ALT, and ALT+ match the Dijkstra optimum.
10. Exports CSV tables, PNG figures, LaTeX macros, and a packaged results archive.

If the real dataset cannot be downloaded, the notebook can generate a small synthetic graph so the complete workflow remains testable.

## Algorithms

### A*

A* uses `f(n) = g(n) + h(n)` with a weak but admissible heuristic: one unit of remaining cost for every non-goal node. With positive edge costs, it is complete and returns an optimal path.

### ALT and ALT+

ALT precomputes shortest-path distances from a configurable number of landmarks. Its heuristic is:

```text
h(n) = max over landmarks L of |d(L, goal) - d(L, n)|
```

This is an admissible and consistent lower bound from the triangle inequality. Landmarks are selected with farthest-point selection.

ALT+ uses:

```text
max(ALT heuristic, unit-cost heuristic)
```

This remains admissible and is at least as informed as either bound alone.

### Hill climbing

Hill climbing keeps only the current node and chooses a neighbor with a better landmark estimate. It is not complete and is not guaranteed to find an optimal path.

The notebook evaluates:

1. **Hill Climbing**: strict improvement only
2. **HC + sideways**: permits a limited number of equal-quality moves
3. **HC + restarts**: retries with stochastic choices after becoming stuck

## Dataset and graph model

Each graph node is a state. An action traverses an edge to a neighboring node. The goal test succeeds when the current node equals the target node.

The source data is directed by relationship type, but the notebook stores edges in both directions so paths can connect entities through relationships such as:

```text
user -> app -> user
```

Only the largest connected component is retained. This makes every sampled query reachable and allows completeness to be compared meaningfully.

## Running the experiment

### Google Colab

1. Open the notebook using the badge above.
2. Select **Runtime > Run all**.
3. Download the generated results from `/content/results`.

Each algorithm is kept in its own notebook section, so individual sections can also be rerun for development or reporting.

### Configuration

The main settings are defined near the beginning of the notebook:

| Setting | Default | Purpose |
|---|---:|---|
| `N_QUERIES` | `100` | Number of start/goal pairs |
| `QUERY_TYPE` | `"user"` | Node type used for sampled queries; `None` uses any node |
| `N_LANDMARKS` | `16` | Number of ALT landmarks |
| `HC_SIDEWAYS` | `20` | Sideways moves allowed by the sideways variant |
| `HC_RESTARTS` | `30` | Maximum attempts for the restart variant |
| `N_MEM_QUERIES` | `10` | Queries used for memory measurement |
| `SEED` | `42` | Seed for reproducible sampling |
| `MAX_ROWS_PER_EDGE_FILE` | `1_500_000` | Rows read from each edge file |
| `EDGE_WEIGHT_COL` | `None` | Mean all attributes, or select one attribute column |
| `USE_SYNTHETIC_IF_DOWNLOAD_FAILS` | `True` | Generate fallback data if download fails |

The loader expects the first two columns of each edge file to contain source and target IDs. Remaining columns are interpreted as relationship attributes.

## Generated outputs

Results are written to `/content/results`:

- CSV summaries for algorithm performance and accuracy
- CSV data-processing and timing reports
- PNG charts and search-tree visualizations at 300 DPI
- `results_macros.tex` for inserting generated values into a LaTeX report
- A packaged archive for downloading the experiment outputs

The evaluation reports success rate, optimal-path rate, cost ratio relative to Dijkstra, nodes expanded/generated, frontier size, and query time. It also includes Wilson 95% confidence intervals and paired Wilcoxon comparisons on the same queries.

Search is not a classification problem, so the notebook uses these honest equivalents:

| Search measure | Meaning |
|---|---|
| Success rate | Whether the goal was reached |
| Optimal-path rate | Whether the returned cost equals Dijkstra's cost |
| Cost ratio | Returned cost divided by optimal cost |

## Reproducibility and correctness

All random choices use `SEED`, including query sampling, synthetic data generation, landmark selection, and stochastic hill climbing.

Before collecting comparative results, the notebook independently computes Dijkstra reference costs and asserts that A*, ALT, and ALT+ return the same optimal costs for every query. This check is intended to catch implementation or heuristic errors before results are interpreted.

## Repository contents

```text
README.md
telecom_pathfinding_v3.ipynb
```

The notebook is the executable source for the full experiment.

## References

- SNAP Telecom Graph: <https://snap.stanford.edu/data/telecom-graph/>
- Goldberg, A. V. and Harrelson, C. (2005), *Computing the Shortest Path: A\* Search Meets Graph Theory*, Proceedings of SODA.

## License

No license file is currently included in this repository. Add a license before redistributing the code or generated results.
