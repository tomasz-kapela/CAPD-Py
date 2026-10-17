# Time maps and Poincaré maps

## ITimeMap

`ITimeMap(solver)` transports sets over a prescribed time interval by repeatedly calling the solver, handling
the step control and the last (shortened) step. Variants: `ITimeMap` (for `IOdeSolver`), `IC2TimeMap`
(for `IC2OdeSolver`), `ICnTimeMap` (for `ICnOdeSolver`).

```python
vf = IMap("par:c;var:x,y,z;fun:y,z,c^2-y-0.5*x^2;")
vf.setParameter("c", 0.125)
solver = IOdeSolver(vf, 20)
tm = ITimeMap(solver)

s = C0HOTripletonSet(IVector([0., -0.125, 0.]))
tm(Interval(100.), s)                  # advance the set; returns an enclosure at the final time
```

### Calling a time map

```python
tm(T, s)                  # -> IVector; the set s is advanced by time T
tm(T, s, curve)           # also record the solution in an ISolutionCurve
tm(T, s, dF)              # C1/C2/Cn sets: dF (IMatrix) receives the derivative of the flow
```

Variants with points instead of sets are also available (`tm(T, v)`, `tm(T, v, dF)`, `tm(T, v, dF, H)`, ...,
with optional `ScalarType&` for the actual time and `SolutionCurve&` for the trajectory), including versions
that take an initial derivative matrix and Hessian (`tm(T, v, V, dF)`).

Which sets each time map accepts follows the solver (see [ODE solvers](ode-solvers.md#which-sets-each-solver-accepts));
`ICnTimeMap` does not accept the HO sets.

### Recording the trajectory

```python
solution = ISolutionCurve(0.0)        # start time
tm(T, s, solution)
solution(7.0)                         # enclosure of x(7); evaluate at a point
solution(Interval(3, 4))              # or on a time interval
```

See [Differential algebra](diff-algebra.md#solution-curves).

### Other methods

`getSolver()`, `getVectorField()`, `getOrder()`, `setOrder(n)`, `getStep()`, `setStep(h)`,
`turnOnStepControl()`, `turnOffStepControl()`, `onOffStepControl()`, `stopAfterStep(bool)`, `completed()`,
`getCurrentTime()`.

## Sections

A Poincaré section is the zero set of a scalar function α. All section classes provide `__call__` /
`evalAt` (value of α), `gradient`, and the `computeDP` / `computeDT` helpers used to turn flow derivatives
into derivatives of the Poincaré map.

| Class | Section | Constructor |
|-------|---------|-------------|
| `ICoordinateSection` | `{x : x_i = c}` | `ICoordinateSection(D, i, c=0)` — dimension `D`, coordinate `i` (0-based) |
| `IAffineSection` | hyperplane through `origin` with `normal` | `IAffineSection(origin, normal)` (two `IVector`s) |
| `INonlinearSection` | `{x : α(x) = 0}` | `INonlinearSection("var:x,y;fun:x^2+y^2-1;")` |

`ICoordinateSection` also has `setConstant`, `setDirection`, `getNormalVector`, `isSpecialSection`.
`IAffineSection` has `setOrigin`, `setNormalVector`, `getOrigin`, `getNormalVector`.

### Crossing direction

```python
CrossingDirection.PlusMinus   # section crossed from positive to negative values of alpha
CrossingDirection.MinusPlus   # from negative to positive
CrossingDirection.Both        # either direction (default)
```

## IPoincareMap

`IPoincareMap(solver, section, direction=CrossingDirection.Both, errorTolerance=0.1)` is the rigorous
first-return map. Variants: `IPoincareMap`, `IC2PoincareMap`, `ICnPoincareMap`.

```python
vf = IMap("par:c;var:x,y;fun:y,c*y*(1-x^2)-x;")
vf.setParameter("c", Interval(2) / Interval(10))
solver = IOdeSolver(vf, 20)
section = ICoordinateSection(2, 1)                          # y = 0
pm = IPoincareMap(solver, section, CrossingDirection.PlusMinus)

u = IVector([2.0004136789920905, 0.0])
s0 = C0Rect2Set(u)
print(pm(s0))                                               # image of the set under P
```

### Calling a Poincaré map

| Call | Result |
|------|--------|
| `pm(s)` / `pm(s, n)` | n-th iterate (default 1) on a C0 set |
| `pm(s, returnTime, n)` | also returns a bound on the return time in the `Interval` `returnTime` |
| `pm(s, dF, n)` | C1 set; `dF` (IMatrix) receives the derivative of the **flow** at the return time |
| `pm(s, dF, returnTime, n)` | same with the return time |
| `pm(s, dF, H, n)` | C2 set; also fills the Hessian `H` |
| `pm(s, jet, n)` | Cn set; `jet` (IJet) receives derivatives of the flow |
| `pm(s, x0, A, returnTime, n)` | result expressed in the affine coordinate system `x0 + A*(...)` |

To obtain the derivative of the **Poincaré map** (not of the flow) pass the flow derivative to `computeDP`:

```python
s1 = C1Rect2Set(u)
dF = IMatrix(2, 2)
v = pm(s1, dF, 1)                              # image and derivative of the flow
dP = pm.computeDP(v, dF)                       # derivative of the Poincare map
```

`computeDP` has overloads that also return the gradient of the return time `dT`, take a return time, or
compute second derivatives (`computeDP(Px, dF, H, dP, d2P, ...)`).

### Settings

| Method | Purpose |
|--------|---------|
| `setMaxReturnTime(t)` | an exception is raised if the section is not reached within time `t` (default 1000) |
| `setBlowUpMaxNorm(m)` | `l_1` norm above which the solution is considered to blow up (default 1e10) |
| `setSection(section)`, `getSection()` | |
| `setCrossingDirection(dir)` | |
| `setOrder`, `getOrder`, `setStep`, `getStep` | solver controls |
| `turnOnStepControl`, `turnOffStepControl`, `onOffStepControl` | |
| `getSolver()`, `getVectorField()` | |

### Non-rigorous maps

`IBasicPoincareMap`, `IC2BasicPoincareMap`, `ICnBasicPoincareMap` compute the Poincaré map on points
without rigorous error control; use them for numerical exploration (finding a periodic orbit candidate) and
then verify with `IPoincareMap`.

```python
basic = IBasicPoincareMap(solver, section, CrossingDirection.PlusMinus)
basic(IVector([2.0, 0.0]))
```
