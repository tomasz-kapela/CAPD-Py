# CAPD-Py documentation

CAPD-Py (`uj-capdpy` on PyPI, imported as `capdpy`) is a Python interface to the
[CAPD](http://capd.ii.uj.edu.pl/) library for rigorous numerics and computer assisted proofs in dynamical systems.
All computations are carried out with interval arithmetic, so the results are mathematically rigorous enclosures.

## Contents

| Page | What it covers |
|------|----------------|
| [Getting started](getting-started.md) | Installation, a first computation, general conventions |
| [Intervals](intervals.md) | `Interval` and interval functions |
| [Vectors and matrices](vectors-and-matrices.md) | `IVector`, `IMatrix`, `IHessian`, norms, multiindices, matrix algorithms |
| [Maps and functions](maps.md) | `IMap`, `IFunction`: parsing formulas, parameters, derivatives, Taylor coefficients |
| [Sets](sets.md) | `C0Rect2Set`, `C1Rect2Set`, `C2Rect2Set`, `CnRect2Set`, HO and tripleton variants |
| [ODE solvers](ode-solvers.md) | `IOdeSolver`, `IC2OdeSolver`, `ICnOdeSolver`, step control |
| [Time maps and Poincaré maps](time-and-poincare-maps.md) | `ITimeMap`, sections, `IPoincareMap`, `CrossingDirection` |
| [Differential algebra](diff-algebra.md) | Jets, curves, solution curves, `IHessian`, time jets |
| [Examples](examples.md) | Walkthrough of the programs in `examples/` |

## Naming conventions

CAPD is a C++ template library. CAPD-Py exposes the instantiation over **interval** scalars with a
dynamic dimension, and the class names keep CAPD's convention of an `I` prefix for interval types:

| CAPD-Py | CAPD C++ |
|---------|----------|
| `Interval` | `capd::interval` |
| `IVector`, `IMatrix` | `capd::IVector`, `capd::IMatrix` |
| `IMap`, `IFunction` | `capd::IMap`, `capd::IFunction` |
| `IOdeSolver`, `IC2OdeSolver`, `ICnOdeSolver` | `capd::IOdeSolver`, `capd::IC2OdeSolver`, ... |
| `ITimeMap`, `IPoincareMap` | `capd::ITimeMap`, `capd::IPoincareMap` |
| `C0Rect2Set`, `C1Rect2Set`, ... | `capd::C0Rect2Set`, `capd::C1Rect2Set`, ... |

For anything not described here the
[CAPD documentation](http://capd.ii.uj.edu.pl/documentation/) is the authoritative reference; the Python
classes mirror the C++ ones, and docstrings in the bindings are available through `help(capdpy.<name>)`.

## Indexing

All indices are **0-based**, including coordinates of vectors, matrix rows/columns, and the coordinate of a
`ICoordinateSection`.

## License

GPL-3.0-only. See [LICENSE](../LICENSE).
