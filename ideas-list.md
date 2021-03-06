# OSQP

OSQP is a numerical optimization package for solving sparse convex quadratic programs. It is written in C, but also provides interfaces to the most popular languages for scientific computing. OSQP is based on the alternating direction method of multipliers (ADMM), which has the following advantages over other state-of-the-art methods:
- **Computational efficiency:** its iterations are computationally cheap
- **Fast convergence for moderate accuracy:** it typically requires few hundreds of iterations to find a good solution
- **Supports warm-starting:** providing a good initial guess reduces computation times considerably
- **Detects infeasible and unbounded problems:** provides certificates of primal and dual infeasibility without modifying the problem



## Table of contents
- [Mentors](#mentors)
- [Project Ideas](#project-ideas)
  * [Quadratic program differentiation](#quadratic-program-differentiation)
  * [User-provided linear algebra](#user-provided-linear-algebra)
  * [Algorithm improvements: acceleration](#algorithm-improvements--acceleration)
  * [Support for embedded systems](#support-for-embedded-systems)



## Mentors

- [Bartolomeo Stellato](https://github.com/bstellato)
- [Goran Banjac](https://github.com/gbanjac)
- [Paul Goulart](https://github.com/goulart-paul)
- [Ian McInerney](https://github.com/imciner2)


## Project Ideas



### Quadratic program differentiation

| **Intensity**  | **Priority**  | **Involves**   | **Mentors**  |
| -------------  | ------------  | -------------  | -----------  |
| Moderate       | Medium        | Differentiate solutions of quadratic programs | [Bartolomeo Stellato](https://github.com/bstellato), [Goran Banjac](https://github.com/gbanjac), and [Paul Goulart](https://github.com/goulart-paul) |

#### Abstract
The goal of this project is to allow OSQP to internally compute derivatives of the optimal solution with respect to the problem parameters. In this way we can differentiate the optimal solutions with respect to the coefficients in the objective and constraints of a quadratic optimization problem (see [1](https://arxiv.org/pdf/1703.00443.pdf), [2](http://reports-archive.adm.cs.cmu.edu/anon/anon/usr/ftp/2019/CMU-CS-19-109.pdf) and [3](https://arxiv.org/pdf/1904.09043.pdf)).

This technique is a powerful tool to use OSQP in differentiable architectures such as neural network layers. These tools have seen a wide range of applications from game theory (see [4](https://arxiv.org/pdf/1805.02777)) to control (see [5](https://web.stanford.edu/~boyd/papers/pdf/learning_cocps.pdf) and [6](https://proceedings.neurips.cc/paper/2018/file/ba6d843eb4251a4526ce65d1807a9309-Paper.pdf)).

With OSQP being able to directly compute derivatives, users could quickly embed it in neural network architectures, and perform end-to-end prediction and optimization tasks from different languages.


#### Technical details
The mathematics behind differentiating quadratic programs can be found in [1](https://arxiv.org/pdf/1703.00443.pdf). For general conic programs, we refer to [3](https://arxiv.org/pdf/1904.09043.pdf).

The main package currently performing automatic differentiation for general convex optimization problems is [cvxpylayers](https://github.com/cvxgrp/cvxpylayers). We have written a basic prototype implementation in [osqp-python/pull/46](https://github.com/oxfordcontrol/osqp-python/pull/46). However, the implementation is not complete and is quite inefficient (numpy). The goal of this project is to:
- finalize the prototype to have a reliable implementation
- port the differentiation functions to C so that they can be interfaced from other interface languages
- interface `osqp-python` with [cvxpylayers](https://github.com/cvxgrp/cvxpylayers)
- add `osqp-python` submodules to directly interface pytorch and jax libraries

#### Helpful experience
- Previous knowledge of [convex optimization](https://en.wikipedia.org/wiki/Convex_optimization)


#### First steps
- Read the [OptNet paper](https://arxiv.org/pdf/1703.00443.pdf)
- Read the [cvxpylayers paper](https://arxiv.org/pdf/1703.00443.pdf)





### User-provided linear algebra

| **Intensity**  | **Priority**  | **Involves**   | **Mentors**  |
| -------------  | ------------  | -------------  | -----------  |
| Moderate       | High          | Allow OSQP to use external and user-provided linear algebra implementations | [Goran Banjac](https://github.com/gbanjac), [Paul Goulart](https://github.com/goulart-paul), and [Bartolomeo Stellato](https://github.com/bstellato) |

#### Abstract

OSQP relies on a relatively small number of linear algebra routines. It uses a custom implementation of these routines, which makes the solver library-free, but its hard-coded linear algebra makes it impossible to run on non-standard hardware architectures such as GPUs. To be able to exploit specialized hardware architectures, one needs to abstract data structures and computationally intensive linear algebra routines, and link a specific implementation during code compilation.


#### Technical details

We will adopt an object-oriented approach to abstract data structures representing vectors and matrices, and define a set of operations on these structures, e.g., vector addition, matrix-vector multiplication etc. We will provide at least two implementations, but will also allow for user-definable implementations. Switching between various implementations will be made easy by setting appropriate flags during code compilation.

Such abstraction will not be used only to run OSQP on different hardware architectures, but also to optimize the solver for speed or memory. For instance, the current data structure representing a symmetric matrix stores only its upper-triangular part to reduce the memory footprint. However, evaluating the matrix-vector product would be easily parallelizable if the full matrix were stored. Hence, if memory is not an issue for a specific application, the user would prefer the latter implementation as it would be faster.

Moreover, we will be able to use existing BLAS libraries, which consist of linear algebra operations that are highly optimized for most common hardware architectures.

#### Helpful experience

- Basic knowledge of ADMM
- Prior work with CMake and BLAS libraries

#### First steps

- Read the paper: https://arxiv.org/abs/1711.08013
- Understand the numerical algorithm implemented by OSQP and the main computational bottlenecks



### Algorithm improvements: acceleration

| **Intensity**  | **Priority**  | **Involves**   | **Mentors**  |
| -------------  | ------------  | -------------  | -----------  |
| Moderate       | High          | Implement the ADMM acceleration to speed up OSQP convergence  | [Paul Goulart](https://github.com/goulart-paul), [Bartolomeo Stellato](https://github.com/bstellato), and [Goran Banjac](https://github.com/gbanjac) |


#### Abstract

First-order methods, like the OSQP solver algorithm, have recently gained popularity due to their simplicity and applicability in a wide range of areas, including neural networks training and large-scale decision-making.
However, in some cases they can converge slowly, thereby requiring more iterations and computing time.
Despite [its smart parameter selection](https://osqp.org/docs/solver/index.html#rho-step-size) that improves convergence, OSQP would greatly benefit from an acceleration scheme.

The main idea of this project is to implement an acceleration scheme in OSQP along the same lines of recent Julia implementation in [COSMO.jl](https://github.com/oxfordcontrol/COSMO.jl). The OSQP acceleration implementation will include safeguards in case of numerical errors and has the potential to significantly reduce the computation time.

#### Technical details

The implementation will be along the lines of [COSMOAccelerators.jl](https://github.com/oxfordcontrol/COSMOAccelerators.jl) which is used to accelerate [COSMO.jl](https://github.com/oxfordcontrol/COSMO.jl).
We will also use the [SCS](https://github.com/cvxgrp/scs) implementation as a reference.

One of our main visions is to have OSQP library-free so that it can be implemented on any platform (from embedded to GPU). Therefore, OSQP will not explicitly depend on any large linear algebra library.
This means any operation involved in the acceleration, including linear system solution, has to rely on internal functions. We will mostly refer to implementation in [COSMOAccelerators.jl](https://github.com/oxfordcontrol/COSMOAccelerators.jl), which, compared to the one in [SCS](https://github.com/cvxgrp/scs), does not require any link to LAPACK.

The project will involve two main steps
- First prototype in Python to check reduction in algorithm steps
- C implementation in core solver

#### Helpful experience
- Basic knowledge of linear algebra and least squares (in particular, [QR decomposition from VMLS book](http://vmls-book.stanford.edu/))
- Basic knowledge of ADMM algorithm


#### First steps
- Read [recent survey on Acceleration Schemes](https://arxiv.org/pdf/2101.09545.pdf) (in particular Chapter 3 on Nonlinear Acceleration)
- Get familiar with acceleration code in [COSMOAccelerators.jl](https://github.com/oxfordcontrol/COSMOAccelerators.jl)
- Read the paper on [safeguards for Anderson acceleration](https://arxiv.org/pdf/1808.03971.pdf)




### Support for embedded systems

| **Intensity**  | **Priority**  | **Involves**   | **Mentors**  |
| -------------  | ------------  | -------------  | -----------  |
| Moderate       | Medium        | Implement code generation in C. Allow multiple code-generated solvers in a single application. | [Ian McInerney](https://github.com/imciner2), [Goran Banjac](https://github.com/gbanjac), [Bartolomeo Stellato](https://github.com/bstellato), and [Paul Goulart](https://github.com/goulart-paul) |

#### Abstract

OSQP is a lightweight C library that does not rely on other external libraries, making it perfect for use in embedded optimization applications. There is currently support for generating embedded code in two of the interfaces (MATLAB and Python), but the generated code only allows for one problem to be embedded in each application. This project will make the code generation a part of the C library itself, so it will be available for all interfaces to use and allow for easier maintenance and updating as the solver evolves. At the same time, the code generation will be redesigned so that it can support multiple problems in a single application (as is found in control systems using both optimization-based estimators and controllers).

#### Technical details

The main task for this project will be moving the code generation system from the interfaces where it currently exists into the C library as an optional component that can be compiled into all the interfaces. This component will handle packaging the problem data into the C files, and then generating the subset of C files the solver needs when embedded.

As part of this, we want to implement problem separation - where the data for each problem lives inside its own variable "namespace" so that multiple problems can then be compiled into the same program without causing multiply-defined variable errors.

#### Helpful experience

- Knowledge of Python or MATLAB to understand the current code generation system
- Prior work with CMake and coding in C

#### First steps

- Read the paper: https://web.stanford.edu/~boyd/papers/pdf/osqp_embedded.pdf
