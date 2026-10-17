# Examples

The [`examples/`](../examples) directory contains computer assisted proofs ported from the
[CAPDexercises](https://github.com/CAPDGroup/CAPDexercises) repository. Run any of them with

```bash
python examples/<name>.py
```

after installing the package (or, for a source build, copying the `capdpy*.so` file next to the script).

| Script | System / problem | Features used |
|--------|------------------|---------------|
| [`stable-periodic-orbit.py`](../examples/stable-periodic-orbit.py) | Existence and stability of a periodic orbit of the van der Pol oscillator | `ICoordinateSection`, `IPoincareMap`, `C1Rect2Set`, `computeDP`, derivative `|P'| < 1` |
| [`hyperbolic-periodic-orbit.py`](../examples/hyperbolic-periodic-orbit.py) | Hyperbolic periodic orbit of the Rössler system | `C0TripletonSet`, Newton/interval-Newton step with `gauss`, eigenvalues of `DP` |
| [`finding-system-parameter.py`](../examples/finding-system-parameter.py) | Parameter value for which a trajectory hits a target | `C0HORect2Set`, `C1HORect2Set`, interval Newton in one variable |
| [`find-minimum.py`](../examples/find-minimum.py) | Critical point of a function of the initial parameter, defined through a flow (Newton on g′ using first and second derivatives) | `IC2OdeSolver`, `IC2TimeMap`, `C2Rect2Set`, Taylor coefficients of the vector field |
| [`boundary-value-problem.py`](../examples/boundary-value-problem.py) | Boundary value problem solved by shooting | `ITimeMap`, `C1Rect2Set`, `IMatrix(set)`, `gauss` |
| [`check-collisions.py`](../examples/check-collisions.py) | Trajectory stays at distance ≥ 0.1 from the origin up to time 100 | `ISolutionCurve`, `C0HOTripletonSet`, `euclNorm` |
| [`piecewise-linear-system.py`](../examples/piecewise-linear-system.py) | Composition of Poincaré maps for two linear vector fields; sign change of a coordinate | `IPoincareMap` ×2, `subset`, composition of sets |

## Walkthrough: stable periodic orbit

The program proves that the van der Pol oscillator `x' = y, y' = c·y·(1−x²) − x` with `c = 0.2` has a
stable periodic orbit.

```python
from capdpy import *

vector_field = IMap("par:c;var:x,y;fun:y,c*y*(1-x^2)-x;")
vector_field.setParameter("c", Interval(2.0)/Interval(10.0))
solver = IOdeSolver(vector_field, 20)
section = ICoordinateSection(2, 1)                       # y = 0
poincare_map = IPoincareMap(solver, section, CrossingDirection.PlusMinus)
u = IVector([Interval(2.0004136789920905), Interval(0.0)])
```

1. A numerical approximation `u` of a fixed point of the Poincaré map `P` on `{y = 0}` is given.
2. `poincare_map(C0Rect2Set(u))` shows that `P(u)` is close to `u`.
3. A `C1Rect2Set` computes the flow derivative together with the image; `computeDP` converts it to the
   derivative `P'` of the Poincaré map.
4. With `s = 1e-5·[-1, 1]` the script tests `(v[0] - u[0]).subset(s)` (the image stays within `s` of `u`)
   and `abs(P') < 1` (contraction, hence stability), and prints `Result: True` when both hold.

> **Caveat:** in the script as committed, the line `C1Rect2Set(IVector([u[0]+s, Interval(0.0)]))` creates the
> set around `u + s` but does not assign it, so the second `poincare_map(s1, ...)` call still uses the point set
> `s1` built from `u`. To verify the inclusion for the whole interval `u + s`, assign the new set
> (`s1 = C1Rect2Set(...)`) before evaluating. The script also rebinds the name `subset`, shadowing the
> library function `subset`.

## Notes on the examples

- Parameters are set from interval quotients (`Interval(2.0)/Interval(10.0)`) rather than the float `0.2`, so
  that the stored value is a rigorous enclosure of 1/5.
