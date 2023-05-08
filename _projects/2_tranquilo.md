---
layout: page
title: tranquilo
description: An optimizer that enables domain experts to solve noisy optimization problems.
img: assets/img/tranquilo/tranquilo_logo.png
importance: 2
category: work
---

## Motivation

Throughout my PhD I had to solve several difficult optimization problems. The most
difficult one was fitting the parameters of an agent-based [Covid Model](https://www.nature.com/articles/s41598-022-12015-9) to German time Series of infections and fatalities.

There were two main difficulties:
1. The objective function was very noisy
2. The computational bottleneck of the model was inherently serial

None of the optimizers I tried (and I tried many!) worked and I could only solve the
the problem with a lot of manual intervention. To make sure I'll never have to go through
this again, we developed Tranquilo, a derivative free trustregion optimizer for noisy
objective functions that can do function evaluations in parallel.

A particular focus was on making tranquilo useful for domain experts, i.e. scientists who
are experts in some field, but have no formal training in numerical optimization or
parallel programming.

## Derivative Free Trustregion Optimization

Let's start with a short introduction to derivative free trustregion optimization. Instead
of showing the math, I will try a visual explanation of how such an optimizer minimizes the function along with a verbal description. Consider the objective function:

$$f(x, y) = x^2 + y^2$$

The global optimum of this function is at $$x=0, y=0$$.

In general, derivative free trustregion optimizers start by fixing a parameter vector
$$x_0$$ which is the center of the trustregion and a trustregion radius $$\delta_0$$.
They then sample parameter vectors inside the trustregion or close by, evaluate the
objective function on each point in the sample and fit an approximation to the
objective function by interpolation or regression. This approximation is called the
surrogate model.

This surrogate model is then minimized over the trustregion and the argmin of the
surrogate model is the next candidate point. In the plot below, the blue point is the
trustregion center, the red points are newly sampled points and the green star is the
argmin of the surrogate model.

<img src="/assets/img/tranquilo/animation_1.svg" class="rounded" width="700" />

The surrogate model does not only give us a candidate point but also a prediction for
the improvement we can expect. After evaluating the objective function at the candidate
point, we can compare this expected improvement to the actual improvement and get:

$$\rho = \frac{\text{actual improvement}}{\text{expected improvement}}$$

As long as the actual improvement is positive, we keep the new point. Depending on
$$\rho$$ we also adjust the the trustregion radius for the next iteration:

If $$\rho$$ is satisfactorily large, say above $$0.2$$ the model is good and we increase
the radius. If it is small, we decrease the radius. This works (in the absence of noise)
because we typically choose surrogate models with taylor like error bounds, i.e. the
model can approximate the objective function arbitrarily well as long as we make the
radius small enough. Of course, a small radius means slow progress, which is why we
want to keep the radius as large as possible as long as the model is good enough to
point us in the right direction.

In our example, everything went well, we increased the radius and the next iteration
looks as follows:

<img src="/assets/img/tranquilo/animation_2.svg" class="rounded" width="700" />

Note that we did not have to sample any new points which would have led to expensive
new function evaluations. There were enough points in the trustregion to fit a model and
everything went well. Never change a winning team!

This holds until iteration five where the picture looks as follows:

<img src="/assets/img/tranquilo/animation_4.svg" class="rounded" width="700" />

We still have not sampled any new points (except for the candidate points of each
iteration) but now we are at the optimum and the expected improvement of the next
iteration is very small (in fact, almost zero). To make sure this does not come from
the very bad sample quality (all points are almost on a line), we discard some of the
existing points and add a new one:

<img src="/assets/img/tranquilo/animation_5.svg" class="rounded" width="700" />

Since this still does not change the model, the algorithm has converged.

Most real world problems take much longer to converge but the basic steps are the same.
While different optimizers differ in the way they choose points and manage the radius,
the common theme is to re-use points as efficiently as possible in order to save
costly evaluations of the objective function.


## Summary of Contributions

Tranquilo has two main contributions over similar optimizers such as
[bobyqa](https://www.damtp.cam.ac.uk/user/na/NA_papers/NA2009_06.pdf),
[pounders](https://www.mcs.anl.gov/papers/P5120-0414.pdf), and
[PyBobyqa and DFO-LS](https://dl.acm.org/doi/10.1145/3338517)

1. Tranquilo can evaluate the objective function in parallel and has multiple
strategies to use otherwise idle cores productively.
2. Tranquilo can estimate the level of noise in the objective function and evaluate and
adaptively determine how often the objective function needs to be evaluated at each sample
point in order to average out the effects of noise.

## Parallelization

There are two strategies to utilize otherwise idle cores while running in parrallel:
Line search and speculative sampling. Both can be seen best in a plot:

<img src="/assets/img/tranquilo/line_and_speculative_points.svg" class="rounded" width="700" />

The line search is triggered when the candidate point is at the border of the trustregion.
In that case, the model might propose a search direction that would be valid even if
a larger step is taken. So we try out multiple step sizes in parallel.

Speculative sampling asks the question: "Which points would we like to encounter in the
next iteration, if the candidate was accepted". In an ideal case, this eliminates or
reduces the need to sample points in the next iteration.

We benchmark tranquilo against the very efficient serial optimizer DFO-LS on the
Moré-Wild benchmark set which is standard in the literature. We vary the number of cores
(batch size) between 2 and 8. The y-axis shows the share of solved problems, the
x-axis a standardized computational budget in terms of batches.

<img src="/assets/img/tranquilo/bld/figures/profile_plots/parallelization_ls.svg" class="rounded" width="700" />

We see that using more cores can dramatically speed up the optimization. Of course,
at the cost of using more resources.

## Noise Handling

To some extent, the only way of getting rid of noise in the objective function is to
evaluate it multiple times at each point and average the noise out. Many optimizers do
that. The difficult question is: "How many times do we have to evaluate?". Most existing
optimizers leave that task to the user who can specify a (typically increasing) sequence
of numbers of evaluations.

Tranquilo's big contribution is to determine that sequence adaptively. Before we look at
how tranquilo does that, we take a step back to see why it is a hard problem. Below you can
see two plots of the same noisy objective function. The only difference between the two
is the trustregion center. The radius and everything else is kept the same. Even the
noise draws at each sampling point are kept the same.

In the first plot, we are in a steep area of the objective function. Even though we are
unlucky in the sense that the noise draw at the highest point is negative and the one at
the lowest point is positive, the model does a good job at leading us in the right
direction.

<img src="/assets/img/tranquilo/noise_plot_2.svg" class="rounded" width="700" />

The only thing that changes in the second plot is that we are in a flat area of the
objective function. The same noise draws now lead to a model that sends us straight away
from the optimum.

<img src="/assets/img/tranquilo/noise_plot_3.svg" class="rounded" width="700" />

An optimal adaptive noise handling strategy would only do one evaluation per point in the
first case and multiple evaluations in the second case.

The way tranquilo determines how often the objective function needs to be evaluated is
based on a simulation. In the simulation, the surrogate model becomes a stand-in for the
true objective function and an estimate of the noise variance is used to create simulated
noisy evaluations of that criterion function. We then fit simulated surrogate models on
those simmulated samples, optimize them and calculate:

$$\rho_{noise} = \frac{\text{expected improvement from the simulated model}}{\text{actual improvement in the real model}}$$

If $$\rho_{noise}$$ is large, we can reduce the number of evaluations per point. If it
is small, we increase it. Of course, we repeat the simulation multiple times and work
with quantiles of the simulated $$\rho_{noise}$$. The mean would not be robust to
outliers.


This is based on the fundamental insight that the random error
due to noise and the approximation error due to a too large trustregion are similar in
the sense that they both can decrease the surrogate model's ability to predict good
search directions but we should only take steps against these problems when it actually
matters!

To determine the number of function evaluations during the acceptance step, we take
standard approaches from power analysis. Moreover, we ensure that the acceptance sample
is always large enough to get a reliable noise estimate.

We compare tranquilo with multiple versions of DFO-LS on Moré-Wild benchmark set with
a substantial artificial noise term added to each function. The different versions
of DFO-LS use 3, 5 and 10 evaluations at each point. While it is clear that more
sophisticated sequences would improve DFO-LS's performance, there is no good way of
coming up with the perfect sequence in practice.

<img src="/assets/img/tranquilo/bld/figures/profile_plots/noisy_ls.svg" class="rounded" width="700" />

While no optimizer solves all problems within the computational budget of 5000 function
evaluations, we can clearly see that tranquilo has an advantage in speed (it only uses
multiple evaluations if necessary) and robustness (It solves more problems).


## Some more details

- Tranquilo works for scalar objective functions but also can exploit a non-linear
least-squares structure, i.e. an objective function of the form $$F(x) = \sum_if_i(x)^2$$
and this leads to massive speed-ups. All benchmarks shown here compare optimizers that
exploit this structure.
- Parallelization and noise handling can of course be combined, even though smart strategies
for utilizing idle cores become less relevant once tha sample sizes grow large.
- Tranquilo is available as part of [estimagic](https://github.com/OpenSourceEconomics/estimagic)
- Tranquilo can handle lower and upper bounds on the parameters.
- Tranquilo is not just a spanish word that describes the algorithm's ability to make calm and steady progress in the presence of noise but an acronym: <span style="color: #0E4187; font-weight: bold;">T</span>rust<span style="color: #0E4187; font-weight: bold;">R</span>egion <span style="color: #0E4187; font-weight: bold;">A</span>daptive <span style="color: #0E4187; font-weight: bold;">N</span>oise robust <span style="color: #0E4187; font-weight: bold;">QU</span>adrat<span style="color: #0E4187; font-weight: bold;">I</span>c or <span style="color: #0E4187; font-weight: bold;">L</span>inear approximation <span style="color: #0E4187; font-weight: bold;">O</span>ptimizer

A working paper containing all the details will be available soon. Until then, you can
check out the presentation slides.

<iframe src="https://raw.githack.com/janosg/data/main/tranquilo_slides.pdf" width="700" height="425"></iframe>
