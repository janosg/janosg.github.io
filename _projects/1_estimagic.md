---
layout: page
title: estimagic
description: A Python package for nonlinear optimization with or without constraints.
img: assets/img/optimization.svg
importance: 1
category: work
---

## About

Estimagic is a Python package for nonlinear optimization with or without constraints. It is particularly suited to solve difficult nonlinear estimation problems. On top, it provides functionality to perform statistical inference on estimated parameters.

Estimagic grew out of my frustration with existing tools. Some notable features are:

- **Algorithms**: Estimagic wraps [algorithms](https://estimagic.readthedocs.io/en/stable/algorithms.html) from scipy, Nlopt, Pygmo, TAO, and many more.
- **Flexible Parameter Format**: Parameters to optimize over can be specified in many formats, including (nested) dictionaries, NamedTuples, pandas Series or DataFrames, numpy arrays, scalars and mixes thereof. This is inspired by JAX's [pytrees](https://jax.readthedocs.io/en/latest/pytrees.html) and implemented in [pybaum](https://github.com/OpenSourceEconomics/pybaum)
- **Dagnostics**: It is effortless to create plots of the criterion and parameter
history. Or to watch them in a real-time dashboard.
- **Logging**: The progress can be logged in an sqlite database. No information is lost
if the optimization is aborted or crashes.
- **Constraints**: Estimagic can reparametrize many constrained problems such that they
can be solved with any optimizer.


## Documentation

- Get started with [Tutorials](https://estimagic.readthedocs.io/en/stable/getting_started/index.html)
- Become a power user with [How-To-Guides](https://estimagic.readthedocs.io/en/stable/how_to_guides/index.html)
- Brush up the theory with [Explanations](https://estimagic.readthedocs.io/en/stable/explanations/index.html)
- Start right away with the [API Docs](https://estimagic.readthedocs.io/en/stable/reference_guides/index.html)


## In Depth Tutorial

<iframe width="560" height="315" src="https://www.youtube.com/embed/ftlw0rARrtI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>