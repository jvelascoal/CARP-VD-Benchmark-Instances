# CARP-VD Benchmark Instances

Benchmark instances for the **Capacitated Arc Routing Problem with Vehicle
Dependence (CARP-VD)**, a generalization of the CARP in which both the
servicing cost and the deadheading cost of every edge depend on the type of
vehicle that performs the operation.

These instances accompany the paper:

> H.-A. Pérez-Vicente, J. Velasco, L. E. Urbán-Rivero.
> *A Two-stage Matheuristic for the Capacitated Arc Routing Problem with
> Vehicle Dependence.* (Submitted, 2026.)

All instances are derived from two classical CARP benchmarks:

- **gdb** — 23 small instances (7–27 nodes, 11–55 edges), from DeArmon,
  collected in Golden, DeArmon & Baker (1983). All edges are required.
- **egl** — 24 medium/large instances (77–140 nodes, 98–190 edges), from the
  winter gritting application of Eglese and Li (Lancashire, UK). Some edges
  are not required (demand 0).

## Repository layout

| Folder | Contents |
|---|---|
| `gdb_mod_np/` | 23 CARP-VD instances derived from *gdb* with **non-proportional** vehicle-dependent costs |
| `egl_mod_np/` | 24 CARP-VD instances derived from *egl* with **non-proportional** vehicle-dependent costs |
| `sensitivity_scenarios/` | Six variants of both sets used in the sensitivity analysis of the paper |

Each instance consists of two plain-text files:

- `<name>_graph.txt` — the network and costs,
- `<name>_vh.txt` — the fleet.

## File formats

### `<name>_graph.txt`

One line per edge, space-separated:

```
i j c1 d1 c2 d2 q
```

| Field | Meaning |
|---|---|
| `i`, `j` | endpoint vertices of the edge (vertex `0` is the depot) |
| `c1` | servicing cost of the edge for a **type-1** vehicle |
| `d1` | deadheading (traversal without service) cost for a type-1 vehicle |
| `c2` | servicing cost for a **type-2** vehicle |
| `d2` | deadheading cost for a type-2 vehicle |
| `q` | demand of the edge (`q = 0` means the edge is not required; only *egl*) |

Example (`gdb_mod_np/gdb19_graph.txt`, first lines):

```
0 1 9.32 6.52 4 2.8 8
0 3 2.43 1.7 3 2.1 3
0 4 0.95 0.66 1 0.7 5
```

In the three-type scenario (`sensitivity_scenarios/*_T3/`) the format carries
one cost pair per type, the demand always being the last field:

```
i j c1 d1 c2 d2 c3 d3 q
```

### `<name>_vh.txt`

First line: `k1 k2`, the number of vehicles of type 1 and type 2 (`k1 k2 k3`
in the three-type scenario). Then one line per vehicle with its capacity,
grouped by type and in the same order.

Example (`gdb_mod_np/gdb19_vh.txt`): two type-1 vehicles of capacity 30 and one
type-2 vehicle of capacity 21.

```
2 1
30
30
21
```

### Reading the files in Python

```python
import networkx as nx

with open("gdb_mod_np/gdb19_graph.txt", "rb") as fh:
    G = nx.read_edgelist(
        fh, delimiter=" ", nodetype=int,
        data=(("c1", float), ("d1", float),
              ("c2", float), ("d2", float), ("q", float)),
    )
```

## How the instances were generated

### Non-proportional sets (`gdb_mod_np/`, `egl_mod_np/`)

Motivated by municipal solid waste collection, where the cost of operating a
street depends on *which* vehicle serves it, each edge is independently
labeled as one of three street classes, and the vehicle type with the
**largest capacity** receives an edge-specific cost factor
`c(large) = beta * c(small)`:

| Street class | Probability | Factor `beta` |
|---|---|---|
| regular street (economies of scale) | 0.70 | uniform in [0.80, 0.95] |
| narrow street (large vehicle penalized) | 0.25 | uniform in [1.50, 2.50] |
| weight-limited segment (soft prohibition) | 0.05 | uniform in [3.00, 5.00] |

The smallest-capacity type keeps the original costs, deadheading is
`d = 0.7 c` for both types, and demands, fleet sizes, and capacities are
unchanged, so feasibility is preserved. Because `beta` is drawn per edge, the
cost ratio between vehicle types **varies from edge to edge**: no scaling
factor per vehicle type reproduces these instances.

Each `*_np` folder includes a `manifest.csv` with, for every instance, the
random seed, the number of edges per street class, and the range of factors
used.

### Sensitivity scenarios (`sensitivity_scenarios/`)

Six variants of both sets, each obtained by changing a **single** factor with
respect to the configuration above and keeping the same random seeds, so that
the only difference between a scenario and the main set is the parameter under
study:

| Scenario | Factor modified | Values |
|---|---|---|
| `S1` | street-class probabilities | regular 0.85, narrow 0.10, weight-limited 0.05 |
| `S2` | street-class probabilities | regular 0.55, narrow 0.35, weight-limited 0.10 |
| `S3` | deadheading ratio | `d = 0.5 c` |
| `S4` | deadheading ratio | `d = 0.9 c` |
| `S5` | ranges of `beta` | narrow [2.0, 3.0], weight-limited [5.0, 8.0] |
| `T3` | number of vehicle types | 3 (see below) |

Folders are named `<set>_<scenario>`, e.g. `gdb_mod_S3/`, `egl_mod_T3/`, and
each one carries its own `manifest.csv`. All parameters other than the one
listed keep the values of the main sets.

In `T3` a third, intermediate vehicle type is added: capacity
`Q3 = round((Q1 + Q2) / 2)`, a single vehicle, and factor
`beta_med = 1 + (beta - 1) / 2` on each edge, i.e. half the effect of the
largest type. The columns of the first two types are identical to those of the
main set, so the comparison isolates the effect of the extra type.

## Note on `gdb3`

In an earlier version of these instances, `gdb3_vh.txt` declared capacities
`{4, 4, 5}` for the three type-1 vehicles, which was inconsistent with the
uniform capacity per type assumed by the model. It has been corrected to
`{4, 4, 4}`, and the results reported in the paper correspond to the corrected
instance.


## Citing

If you use these instances, please cite the paper above. BibTeX:

```bibtex
@article{PerezVicente2026carpvd,
  author  = {P{\'e}rez-Vicente, Hugo-A. and Urb{\'a}n-Rivero, Luis E. and Velasco, Jon{\'a}s},
  title   = {A Two-stage Matheuristic for the Capacitated Arc Routing Problem with Vehicle Dependence},
  year    = {2026},
  note    = {Submitted}
}
```

Please also acknowledge the original benchmark authors: Golden, DeArmon &
Baker (1983) for *gdb*, and Eglese & Li (1996) for *egl*.

## License

The instances are released under the
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license: you may
use, share, and adapt them for any purpose, provided you give appropriate
credit.

## Contact

Jonás Velasco — jvelasco@cimat.mx
Centro de Investigación en Matemáticas A.C. (CIMAT), Aguascalientes, Mexico.
