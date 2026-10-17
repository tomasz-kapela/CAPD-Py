# Vectors and matrices

## IVector

A vector of intervals with dynamic dimension.

```python
v = IVector([1.0, Interval(-1, 1), 3.0])   # from a list (floats are converted)
z = IVector(3)                             # zero vector
w = IVector("...")                         # from a string (see CAPD docs for the format)
v[0]                                       # 0-based indexing
v[1] = Interval(0, 2)
len(v), v.dimension()
for x in v: ...                            # iteration
```

| Operation | Notes |
|-----------|-------|
| `u + v`, `u - v`, `-v` | component-wise |
| `v * s`, `s * v`, `v / s`, `v + s`, `v - s` | `s` is a scalar (float or `Interval`) |
| `u @ v` | scalar product |
| `+=`, `-=`, `*=`, `/=` | in-place |
| `abs(v)` | component-wise absolute value |
| `<`, `<=`, `>`, `>=`, `==`, `!=` | component-wise comparisons |
| `v.euclNorm()` | Euclidean norm (an `Interval`) |
| `v.normalize()` | normalizes in place if possible and returns `True`, otherwise `False` |
| `v.sorting_permutation()` | permutation (list of ints) that sorts the vector |

An `IVector` can also be created from the set classes and jets (see [Sets](sets.md)), e.g.
`IVector(s)` gives an interval enclosure of the set `s`.

### Free functions on vectors

`leftVector`, `rightVector`, `midVector`, `maxDiam`, `diam`, `intervalHull`, `intersection`,
`intervalBall`, `solveAffineInclusion`, `solveAffineInclusionWithDir`.

`intersection` raises `EmptyIntersectionError` if the intersection is empty.

## IMatrix

A dense matrix of intervals.

```python
A = IMatrix(2, 3)                          # zero 2x3 matrix
B = IMatrix(2, 2, [1., 2., 3., 4.])        # row-major list of entries
C = IMatrix([[1., 2.], [3., 4.]])          # list of rows (rows must have equal length)
I = IMatrix.Identity(3)

B[0][1]            # element via row
B[0, 1]            # element by (row, column) tuple; 0-based
B[1, 0] = 5.0
B.row(0), B.column(1)
B.transpose()
```

| Operation | Notes |
|-----------|-------|
| `A + B`, `A - B`, `-A` | matrix sum/difference |
| `A * s`, `s * A`, `A / s`, `A + s`, `A - s` | scalar operations |
| `A @ B`, `A @ v` | matrix product, matrix-vector product |
| `+=`, `-=`, `*=`, `/=` | in-place |
| comparisons | component-wise |
| `abs(A)` | component-wise absolute value |
| `A.setToIdentity()` | square matrices |

An `IMatrix` can be built from a C1, C2 or Cn set, or from a matrix set, which extracts the enclosure of the
derivative: `D = IMatrix(s1)` (see [Sets](sets.md)).

### Free functions on matrices

`transpose`, `trace`, `secondTrace`, `midMatrix`, `krawczykCorrection`, `krawczykInverse`.

## Row and column views

`B.row(i)` and `B.column(j)` return `IRowVector` and `IColumnVector` objects: views into the matrix that
share its storage (writes through them modify the matrix). They support indexing, iteration, `+`, `-`,
scalar `*` and `/`, `@`, `euclNorm()`, `normalize()`, `clear()`, `dimension()`.

## Linear algebra

`matrixAlgorithms` functions are exposed at module level:

| Function | Purpose |
|----------|---------|
| `gauss(A, b)` | rigorous solution of `A x = b` for nonsingular `A` |
| `inverseMatrix(A)`, `gaussInverseMatrix(A)` | rigorous inverses |
| `krawczykInverse(A)` | Krawczyk-based inverse |
| `orthonormalize(A)` | Gram-Schmidt column orthonormalization with column permutation (extra overloads take a row vector or a second matrix) |
| `QR_decompose(A)` | QR decomposition |
| `croutDecomposition(A)` | Crout decomposition into lower triangular `D` and unit upper triangular `G` |
| `invLowerTriangleMatrix(A)`, `invUpperTriangleMatrix(A)` | inverses of triangular matrices |
| `symMatrixDiagonalize(A)` | near-diagonalization of a symmetric matrix |
| `spectralRadiusOfSymMatrix(A)`, `maxEigenValueOfSymMatrix(A)` | bounds via Gerschgorin after diagonalization |
| `matrixExp(A)` | matrix exponential |

```python
x = gauss(M, IVector([u[0] - x1, u[1] - y1]))   # solve M x = b
```

## Norms

Abstract class `INorm` and implementations, all constructed with no arguments and callable on an `IVector`
or an `IMatrix`:

| Class | Norm |
|-------|------|
| `IEuclNorm` | Euclidean |
| `IMaxNorm` | max (`l_inf`) |
| `ISumNorm` | `l_1` |
| `IEuclLNorm`, `IMaxLNorm`, `ISumLNorm` | corresponding logarithmic norms |

```python
n = IMaxNorm()
n(IVector([1., -3.]))
```

`INorm` can be subclassed in Python (override `__call__`, `show`, `clone`).

## Multiindices and multipointers

Used to address partial derivatives and monomials.

- `Multipointer`: non-decreasing list of variable indices. `(0,0,2,3)` stands for
  ∂⁴/(∂x₀² ∂x₂ ∂x₃).
- `Multiindex`: exponent vector. The same derivative in 5 dimensions is `(2,0,1,1,0)`.

Both can be built from a list of ints: `Multipointer([0, 0, 2, 3])`, `Multiindex([2, 0, 1, 1, 0])`.
Methods include `module()` (order), `factorial()`, `hasNext()`, `index(...)`. Free functions:
`sumMultipointers`, `addIndex`, `push_back`, `hasIndex`, `indexCount`, `removeIndex`,
`power(vector, multiindex_or_multipointer)`.

## Containers

`IContainer`, `IMatrixContainer` and `IntContainer` are the low-level storage classes behind vectors and
matrices (`resize`, `size`, `empty`, `clear`, iteration, `len`). You rarely use them directly.
