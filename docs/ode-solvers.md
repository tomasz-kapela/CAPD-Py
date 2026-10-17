# ODE solvers

CAPD uses high-order Taylor methods. A solver is built from a vector field (`IMap`) and the order of the
method; it can then advance points (non-rigorously, in the `IBasic*` classes) or **sets** (rigorously)
by a single time step.

## Classes

| Class | Derivatives available | Rigorous on sets | Notes |
|-------|----------------------|------------------|-------|
| `IOdeSolver` | flow and first derivative (C0, C1 sets) | yes | the default choice |
| `IC2OdeSolver` | up to second derivatives | yes | for `C2Rect2Set` |
| `ICnOdeSolver` | arbitrary order | yes | `CnRect2Set`; uses a different step-control policy |
| `IOdeSolverC2` | as `IOdeSolver` | yes | variant storing a C2 curve |
| `IBasicOdeSolver`, `IBasicOdeSolverC2`, `IBasicC2OdeSolver`, `IBasicCnOdeSolver` | as above | **no** | numerical (non-rigorous) integration of points |

Construction:

```python
vf = IMap("par:c;var:x,y;fun:y,c*y*(1-x^2)-x;")
solver = IOdeSolver(vf, 20)          # vector field, order of the Taylor method
```

`ICnOdeSolver` and the basic solvers also accept an optional step control argument.

The solver keeps a reference to the vector field, so changing a parameter of `vf` afterwards (via
`vf.setParameter`) affects the solver.

## Taking a step

On sets the solver is called like a function and advances the set by **one** time step chosen by the step
control:

```python
solver(s)            # in-place
solver(s, result)    # store the image in `result`
```

Use [`ITimeMap`](time-and-poincare-maps.md) to integrate over a prescribed time interval.

On points (basic solvers):

```python
basic = IBasicOdeSolver(vf, 20)
v1 = basic(v)                  # next point
v1 = basic(v, D)               # next point + derivative of the flow w.r.t. v
v1 = basic(t, v)               # non-autonomous; t is updated in place
```

## Which sets each solver accepts

| Set | `IOdeSolver` | `IC2OdeSolver` | `ICnOdeSolver` |
|-----|:---:|:---:|:---:|
| `C0Rect2Set`, `C0TripletonSet` | ✓ | ✓ | ✓ |
| `C0HORect2Set`, `C0HOTripletonSet` | ✓ | ✓ | – |
| `C1Rect2Set`, `C11Rect2Set` | ✓ | ✓ | ✓ |
| `C1HORect2Set` | ✓ | ✓ | – |
| `C2Rect2Set` | – | ✓ | ✓ |
| `CnRect2Set` | – | – | ✓ |

Calling a solver with an unsupported set raises an exception.

## Common methods

| Method | Purpose |
|--------|---------|
| `getStep()` / `setStep(h)` | time step of the last call / fix the step (**turns step control off**) |
| `adjustTimeStep(h)` | set the step without changing step-control settings |
| `setOrder(n)` | order of the Taylor method |
| `getVectorField()` | the underlying `IMap` |
| `getCurve()` | the internal curve storing the last step's Taylor data |
| `getCoeffNorm(...)` | norm of Taylor coefficients (used by step control) |
| `computeCoefficients`, `computeCoefficientsAtCenter` | compute Taylor coefficients at a point |
| `computeTimeStep(...)` | step size chosen by step control |
| `enclosure(...)`, `Phi`, `JacPhi`, `Remainder`, `encloseC0Map`, `encloseC1Map` (and `encloseC2Map`, `encloseCnMap` for higher solvers) | low-level pieces of the rigorous algorithm |
| `setMask(...)`, `addMultiindexToMask(...)`, `resetMask()` | restrict the derivatives that are computed |

The low-level methods mirror the C++ API; most proofs only need the set-call interface and a time or
Poincaré map.

## Step control

`IOdeSolver`, `IC2OdeSolver` (policy "last terms") and `ICnOdeSolver` (policy "enclosure found") inherit the
step-control interface:

| Method | Purpose |
|--------|---------|
| `turnOnStepControl()`, `turnOffStepControl()`, `onOffStepControl()` | enable/disable |
| `getStepControl()` / `setStepControl(policy)` | policy object |
| `setAbsoluteTolerance(e)`, `getAbsoluteTolerance()` | |
| `setRelativeTolerance(e)`, `getRelativeTolerance()` | |
| `setMaxStep(h)`, `getMaxStep()` | cap on the step |
| `isStepChangeAllowed()` | |

For fixed steps use `solver.setStep(h)`.

## Choosing the order

Typical orders are 10–30 for smooth systems; the repository examples use 15, 20 and `6` (for a second-order
computation with `IC2OdeSolver`). Higher order allows larger steps but costs more per step. The HO set classes
require order ≥ 3 and ≤ 64.
