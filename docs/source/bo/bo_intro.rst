===================
Key Concepts in BO
===================


.. contents:: Table of Contents
    :depth: 2


What can BO do?
===============
Bayesian Optimization (BO), an active learning framework, has seen a rise in its applications in various chemical 
science fields, including catalyst synthesis, high throughput reactions, and computational material discovery. Its 
close variant, kriging, originating in geostatistics, has also been widely applied in process engineering. Fundamentally, 
BO is a sequential global optimization approach consisting of two essential parts: 

1. a surrogate model to approximate the system behavior
2. an acquisition function to suggest new experiments to run. 

The method is designed to balance the exploration of uncertainty and exploitation of current knowledge in the parameter 
space. It often outperforms experts or other algorithms in locating the optima and producing accurate surrogate models. 


Key Concepts and Terminology
============================
To talk more specifically what BO does, we need to introduce additional terminology. We need to talk about:

- surrogate model
- gaussian process
- acquisition function
- multi-objective optimization

Surrogate Model
----------------
A surrogate model is the method used when the response cannot be easily measured, so a model to easily estimate the response 
is used instead. Most engineering design problems require experiments and/or simulations to evaluate design objective and 
constraint functions as a function of design variables. In BO, the surrogate model used is oftern a gaussian process.


Gaussian Process (GP)
---------------------
A GP model constructs a joint probability distribution over the variables, assuming a multivariate Gaussian distribution. 
A GP is determined from a mean function :math:`\mu({\bf X})` and a covariance kernel function :math:`\Sigma({\bf X}, {\bf X^{'}})`. `[1]`_
A Matern covariance function is typically used as the kernel function:

.. math::

    {C_{v}(d)=\sigma^{2} \frac{2^{1-\upsilon}}{\Gamma(\upsilon)} {(\sqrt{2} \upsilon \frac{d}{\rho})}^{\upsilon} K_{\upsilon} (\sqrt{2} \upsilon \frac{d}{\rho})}

where :math:`d` is the distance between two points, :math:`\sigma` is the standard deviation, :math:`\upsilon` and 
:math:`\rho` are non-negative parameters, :math:`\Gamma` represents the gamma function, and :math:`K_{\upsilon}` is 
the modified Bessel function. A typical BO procedure starts with finding the next sampling point by optimizing the acquisition 
function over the GP. Once a new observation is obtained from :math:`\hat{f}`, it is added to the previous observations 
and the GP is updated.


Acquisition Function
---------------------
The acquisition function is applied to obtain the new sampling point :math:`\bf X_{new}`. It measures the value of evaluating 
the objective function at :math:`\bf X_{new}`, based on the current posterior distribution over :math:`\hat{f}`. The most 
commonly used acquisition function is expected improvement (EI). The EI is the expectation taken under the posterior 
distribution :math:`\hat{f}` of the improvement of the new observation at :math:`\bf X` over the current best 
observation :math:`f^{*}`:

.. math::

    EI({\bf X})=\mathbb{E}[max(\hat{f}({\bf X})-f^{*},0)]

Aside from the EI, there are also other acquisition functions for single objective BO available in NEXTorch, including 
probability of improvement (PI), upper confidence bound (UCB), and their Monte Carlo variants (qEI, qPI, qUCB).


Multi-objective Optimization (MOO)
----------------------------------
Multi-objective optimization involves more than one objective function that are to be minimized (maximized). The optimal 
is not a single point but a set of solutions that define the best tradeoff between competing objectives. This is common 
in engineering problems. For instance, an increase in selectivity of a product would lead to a decrease in the yield in 
a reaction. In the MOO, the goodness of a solution is determined by the dominance, where :math:`{\bf X_{1}}` dominates 
:math:`{\bf X_{2}}` when :math:`{\bf X_{1}}`  is no worse than :math:`{\bf X_{2}}`  in all objectives, and :math:`{\bf X_{1}}` 
is strictly better than :math:`{\bf X_{2}}` in at least one objective. A Pareto optimal set is the set where all the 
points of the set are not dominated by each other in that set, and the boundary defined by the response of this set is 
the Pareto front. 

Weighted Sum Method
^^^^^^^^^^^^^^^^^^^^

One classic method to perform MOO is the weighted sum method, which scalarizes a set of objectives :math:`\lbrace {\bf Y_{1}},{\bf Y_{2}},...{\bf Y_{i}},...{\bf Y_{M}}\rbrace` 
into a new objective :math:`{\bf Y_{mod}}` by multiplying each objective with a set of user-defined weights 
:math:`\lbrace w_{1},w_{2},...w_{i},...w_{M}\rbrace`, simplifying the MOO into a single-objective optimization, where 
:math:`{\bf Y_{mod}}=\Sigma w_{i} {\bf Y_{i}}`. By obtaining the optimal for multiple sets of chosen weights, one can 
populate the Pareto optimal. This method is simple but sometimes difficult to select proper set of weights to obtain the 
optimal in a desired region of the design space.

.. image:: ../_images/bo/weighted_sum.svg

Expected Hypervolume Improvement (EHVI)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An alternative is to use expected hypervolume improvement (EHVI) as the acquisition function `[9]`_. A hypervolume indicator (HV) 
is used to approximate the Pareto set and EHVI evaluates its EI. 

Given a Pareto set :math:`S^{*}` and a reference point :math:`r\in{\mathbb{R}}^{M}`, the hypervolume (HV) of :math:`S^{*}` 
is the :math:`M`-dimensional Lebesgue measure :math:`\lambda_{M}` of the region weakly dominated by :math:`S^{*}` and 
bounded above by :math:`r`.

.. math::

    HV(S^{*})={\lambda_{M}}(\cup_{p_{i} \in S} \lbrack r, p_{i}\rbrack)

where :math:`p_{i}` is the :math:`i`-th point in the :math:`S^{*}`. Given a new set of points :math:`S^{'}`, the 
hypervolume improvement (HVI) is define as:

.. math::

    HVI(S^{'},S^{*})=HV(S^{*} \cup S^{'})-HV(S^{*})

.. image:: ../_images/bo/hvi.svg

In this regard, for a set of point :math:`\bf X`, the EHVI is the expectation of HVI over the posterior :math:`\hat{f}` 
and can be expressed as:

.. math::

    EHVI({\bf X})=\mathbb{E} \lbrack HVI(\hat{f}({\bf X}), \hat{f}(S^{*}) \rbrack


In NEXTorch, one can use either weighted sum method or Monte Carlo EHVI (qEHVI) as acquisition function to perform MOO.



Useful Papers and Books
============================

Gaussian Processes
------------------
`[1]`_ Rasmussen, C. E. Gaussian Processes in Machine Learning; MIT Press, 2006.

`[2]`_ Görtler, J.; Kehlbeck, R.; Deussen, O. A Visual Exploration of Gaussian Processes. Distill 2019, 4.

Kriging 
----------------------------
`[3]`_ Jones, D. R.; Schonlau, M.; W. J. Welch. Efficient Global Optimization of Expensive Black-Box Functions," , Vol. 13, No. 4, Pp. 455-492, 1998. J. Glob. Optim. 1998, 13, 455–492.

`[4]`_ Jones, D. R. A Taxonomy of Global Optimization Methods Based on Response Surfaces. J. Glob. Optim. 2001, 21, 345–383.

`[5]`_ Forrester, A. I. J.; Sbester, A.; Keane, A. J. Engineering Design via Surrogate Modelling; John Wiley & Sons, Ltd: Chichester, UK, 2008.

BO Tutorial and Reviews
----------------------------
`[6]`_ Brochu, E.; Cora, V. M.; de Freitas, N. A Tutorial on Bayesian Optimization of Expensive Cost Functions, with Application to Active User Modeling and Hierarchical Reinforcement Learning. 2010.

`[7]`_ Shahriari, B.; Swersky, K.; Wang, Z.; Adams, R. P.; De Freitas, N. Taking the Human out of the Loop: A Review of Bayesian Optimization. Proc. IEEE 2016, 104, 148–175.

`[8]`_ Frazier, P. I. A Tutorial on Bayesian Optimization. 2018.


Multi-objective Optimization (MOO)
------------------------------------

`[9]`_ Daulton, S.; Balandat, M.; Bakshy, E. Differentiable Expected Hypervolume Improvement for Parallel Multi-Objective Bayesian Optimization. 2020.


Other BoTorch Papers
----------------------

`[10]`_ Balandat, M.; Karrer, B.; Jiang, D. R.; Daulton, S.; Letham, B.; Wilson, A. G.; Bakshy, E. BoTorch: A Framework for Efficient Monte-Carlo Bayesian Optimization. 2019.

`[11]`_ Letham, B.; Karrer, B.; Ottoni, G.; Bakshy, E. Constrained Bayesian Optimization with Noisy Experiments. Bayesian Anal. 2019, 14, 495–519.


.. _[1]: http://www.gaussianprocess.org/gpml/chapters/RW.pdf
.. _[2]: https://distill.pub/2019/visual-exploration-gaussian-processes/
.. _[3]: https://link.springer.com/article/10.1023/A:1008306431147
.. _[4]: https://link.springer.com/article/10.1023/A:1012771025575
.. _[5]: https://onlinelibrary.wiley.com/doi/book/10.1002/9780470770801
.. _[6]: https://arxiv.org/abs/1012.2599
.. _[7]: https://ieeexplore.ieee.org/document/7352306
.. _[8]: https://arxiv.org/abs/1807.02811
.. _[9]: https://arxiv.org/abs/2006.05078
.. _[10]: https://arxiv.org/abs/1910.06403
.. _[11]: https://arxiv.org/abs/1706.07094