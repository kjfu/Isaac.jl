
# Isaac.jl

[![Build Status](https://travis-ci.org/cortner/Isaac.jl.svg?branch=master)](https://travis-ci.org/cortner/Isaac.jl)

<!-- [![Coverage Status](https://coveralls.io/repos/cortner/Isaac.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/cortner/Isaac.jl?branch=master)

[![codecov.io](http://codecov.io/github/cortner/Isaac.jl/coverage.svg?branch=master)](http://codecov.io/github/cortner/Isaac.jl?branch=master) -->

The aim of this package is to develop some non-standard experimental Newton type solvers for solving nonlinear systems, optimisation, but most importantly for saddle search.

At present a small selection of Jacobian-Free Newton-Krylov solvers is implemented:
* `nsoli` : a translation of (a subset of) [Tim Kelley's](http://www4.ncsu.edu/~ctk/) `nsoli.m` [Matlab code](http://www4.ncsu.edu/~ctk/newton/SOLVERS/nsoli.m) to Julia (with permission). Any bugs or errors are of course my own.
* `nsolimod` : A *Modulated Jacobian-Free Newton-Krylov Solver* : instead of computing general critical points of an energy or roots of a nonlinear system this solver computes only critical points of a specific spectrum signature. For example only minima, or only index-1 saddles.
* `nkminim` : a wrapper for `nsolimod` choosing better method parameters for minimisation, specifically a better suited line-search.
* `nsolistab` : an `nsolimod`-type solver which doesn't require gradient structure but so far it does not support arbitrary modulations.

## Getting Started

Since the package is not registered, use
```
Pkg.clone("git@github.com:cortner/Isaac.jl.git")
```

There is no documentation, but the inline documentation is fairly extensive;
start with
```
using Isaac
?nsoli
?nsolimod
?nkminim
?nsolistab
```
in the REPL or IJulia


## Examples

See the `tests` directory for more examples.

### Nonlinear Solver `nsoli`

```
using Isaac
f(x) = [x[1] + x[2] + x[1]^2, x[2] + x[1]*x[2]]
x, it_hist, ierr, x_hist = nsoli(rand(2), f)
```

### Minimisation and saddle-search with `nsolimod` and `nkminim`

```
using Isaac, ForwardDiff
E(x) = (x[1]^2 - 1)^2 + (x[2]-x[1]^2)^2
dE(x) = ForwardDiff.gradient(E, x)
# minimisation (index-0 saddle search)
x0, n0 = nsolimod(dE, [0.6,0.1], 0)
# minimisation via nkminim
xmin, nmin = nkminim(E, dE, [0.6,0.1])
# index-1 saddle-search
x1, n1 = nsolimod(dE, [0.4,-0.1], 1)
```


## Should I use `Isaac`?

For most users looking for a robust and well-tested optimiser or nonlinear solver [`Optim.jl`](https://github.com/JuliaNLSolvers/Optim.jl) and [`NLsolve.jl`](https://github.com/JuliaNLSolvers/NLsolve.jl) are probably better choices. That said, the code `nsoli` (or at least its parent `nsoli.m`) is a robust and well-tested code, and always worth comparing against `NLsolve.jl` to decide which will perform better on a specific problem. Moreover, `nkminim` also seems to perform very well on my limited number of test problems.

The second code, `nsolimod` is still very much experimental, and in particular comes with no theoretical convergence guarantee of any kind. However, initial tests indicate that it is both robust and performant, in particular when the energy landscape is not highly nonlinear (but possibly ill-conditioned).

`nsolimod` is written with expensive objectives in mind where each gradient (or function) evaluation far outweighs the cost of the optimisation boilerplate and of the linear algebra. Over time, I hope to optimise the code so that it becomes competitive for cheap objectives.

The main use case for `nsolimod` at the moment is saddle-search. On my tests systems (a few crystalline solids and a few molecules) `nsolimod` outperforms all algorithms I have tested against, both in terms of performance and robustness.  (though I have not yet exhausted the space of all saddle search methods)
