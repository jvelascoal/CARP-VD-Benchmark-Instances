# CARP-VD Benchmark Instances

Benchmark instances for the **Capacitated Arc Routing Problem with Vehicle
Dependence (CARP-VD)**, a generalization of the CARP in which both the
servicing cost and the deadheading cost of every edge depend on the type of
vehicle that performs the operation.

These instances accompany the paper:

> H.-A. Pérez-Vicente, L. E. Urbán-Rivero, J. Velasco.
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
| `gdb_mod/` | 23 CARP-VD instances derived from *gdb* with **proportional** vehicle-dependent costs |
| `egl_mod/` | 24 CARP-VD instances derived from *egl* with **proportional** vehicle-dependent costs |
| `gdb_mod_np/` | 23 CARP-VD instances derived from *gdb* with **non-proportional** vehicle-dependent costs |
| `egl_mod_np/` | 24 CARP-VD instances derived from *egl* with **non-proportional** vehicle-dependent costs |
| `generar_instancias_np.py` | Generator used to produce the `*_np` sets from the `*_mod` sets (reproducible by seed) |

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

Example (`gdb_mod/gdb19_graph.txt`, first lines):

```
0 1 4 2.8 4.8 3.36 8
0 3 3 2.1 3.6 2.52 3
0 4 1 0.7 1.2 0.84 5
```

### `<name>_vh.txt`

First line: `k1 k2`, the number of vehicles of type 1 and type 2.
Then `k1 + k2` lines, one per vehicle, with its capacity
(type-1 vehicles first, then type-2).

Example (`gdb_mod/gdb19_vh.txt`): two type-1 vehicles of capacity 30 and one
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

with open("gdb_mod/gdb19_graph.txt", "rb") as fh:
    G = nx.read_edgelist(
        fh, delimiter=" ", nodetype=int,
        data=(("c1", float), ("d1", float),
              ("c2", float), ("d2", float), ("q", float)),
    )
```

## How the instances were generated

### Proportional sets (`gdb_mod/`, `egl_mod/`)

Starting from the original benchmarks, the single-type cost of each edge is
kept as the type-1 servicing cost (`c1`). Deadheading costs are 30% cheaper
than servicing (`d = 0.7 c`). Type-2 costs are obtained by scaling the type-1
costs by a constant factor per instance. Fleet sizes are preserved and
capacities are modified to obtain a heterogeneous fleet that can still meet
the total demand.

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

To regenerate the `*_np` sets (deterministic, same seed = same files):

```bash
python3 generar_instancias_np.py gdb_mod egl_mod --p-narrow 0.25 --p-restricted 0.05 --seed 2026
```

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

The instances and the generator are released under the
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license: you may
use, share, and adapt them for any purpose, provided you give appropriate
credit.

## Contact

Jonás Velasco — jvelasco@cimat.mx
Centro de Investigación en Matemáticas A.C. (CIMAT), Aguascalientes, Mexico.
