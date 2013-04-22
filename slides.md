% title: Protein Folding is Easy
% subtitle: Towards MSMs for Conformational Change
% author: Robert T. McGibbon
% author: April 22, 2013
% thankyou: Thanks everyone!
% thankyou_details: And especially Vijay, Christian and Kyle.
% contact: <span>www</span> <a href="http://www.stanford.edu/~rmcgibbo">stanford.edu/~rmcgibbo</a>
% contact: <span>github</span> <a href="http://github.com/rmcgibbo">github.com/rmcgibbo</a>

---
title: A Presentation in Three / Four Acts
build_lists: true

- Resolving conformation slow conformation changes within a folding data set.
- MSMAccelerator: Adaptive sampling
- OpenMM Script Builder
- If time permits: initial results with optimal MSM parameter selection

---
title: Background

- This audience is pretty knowledgeable about Markov state models.
- We're building a numerical approximation for the transfer operator.
- Clustering gives us an indicator function basis over phase space.
- The model a discrete-time Markov jump process.
- Clustering and parameter selection remain significant challenges.

---

title: MSM Parameter Selection

- MSM construction is a mix of supervised and unsupervised learning problems.
- Unsupervised learning is the problem of finding "hidden" structure in unlabeled data, where there's no *right* answer.
- How many conformational states does a protein adopt? The answer is in the eye of the beholder.
- (Parameterizing the transition matrix is supervised.)

---
  
title: Competing Sources of Error

The MSM state decomposition, a *clustering*, is characterized by a bias-variance trade off.

- **Bias:** As you lower the number of states, you introduce systematic error in modeling the dynamics.
- Hamiltonian dynamics are completely Markovian in $\mathbb{R}^{6N}$
- **Variance:** As you raise the number of states, you're increasing subject to statistical noise in the transition matrix estimation.
- How do we balance this trade off?

---

title: Choosing the States' Shape
class: img-top-center

<img height=150 src=figures/gpcr_activation.png />

- Conformational change is characterized by slow *conformationally subtle*
  transitions.
- To resolve these transitions in our models, our states need to be "smaller".
- Increasing the number of states is not the *only* way to lower the bias --
  we can also pick the **shape** of our states more intelligently.

---

title: Protein Motions Aren't Isotropic
subtitle: MSM states shouldn't be either
class: img-top-center

<img height=150 src=figures/gpcr_activation.png />

- Different structural degrees of freedom should be weighted according to their
  discriminatory power (equilibration rate).
- Learn a distance metric for clustering which maximally separates kinetically
close and kinetically distant conformations.

---

title: Large-Margin Learning

<table class="flexbox vcenter"><tr>
  <td><img height=150 src="figures/classifiers.png"></td>
  <td><img height=150 src="figures/classifiers2.png"></td>
</tr></table>

- A common goal in supervised learning is to construct binary classifiers,
  e.g. actives vs. inactives, cats vs. others.
- The "margin" is the distance of the object's score from the the decision threshold.
- Large margin approaches attempt to find a classifier via optimization methods
  that maximize the margins.

---

title: Kinetic Distance Metric Learning (KDML)

- A set of $N$ triplets of structres, $(a, b, c)$, where $a$ and
  $b$ appear close together in a single traj., while $a$ and $c$ don't.
- Find a squared Mahalanobis metrics, and maximize the margin
  between the close and far pairs.

$$ d^{\mathbf{X}}(\vec{a}, \vec{b}) = (\vec{a} - \vec{b})^{T} \mathbf{X} (\vec{a} - \vec{b}) $$

$$  \max_{\mathbf{X},\rho} \left[ \alpha \rho - \frac{1}{N} \sum_i^N \lambda \left(d^\mathbf{X}(\vec{a}_i,\vec{c}_i) - d^\mathbf{X}(\vec{a}_i, \vec{b}_i) - \rho \right) \right]
$$

---
title: Optimization and Constraints

$$ \max_{\mathbf{X},\rho} \left[ \alpha \rho - \frac{1}{N} \sum_i^N \lambda \left(d^\mathbf{X}(\vec{a}_i,\vec{c}_i) - d^\mathbf{X}(\vec{a}_i, \vec{b}_i) - \rho \right) \right] $$

- The matrix $\mathbf{X}$ is constrained to be positive semidefinite.
- Relatively efficient optimization by gradient descent with rank-1 updates naturally maintains p.s.d.
- Shen, C.; Kim, J.; Wang, L. Scalable large-margin Mahalanobis distance metric learning.
  *IEEE* *Trans.* *Neural* *Networks* **2010**, 21, 1524–1530

---
title: KDML Model System
class: img-top-center

<img height=250 src="figures/toy_microstates.png">

- 2D Brownian dynamics, where vertical diffusion constant is 10x greater than
the horizontal diffusion constant.

<span style=font-size:30px>
  $$\mathbf{X} = \begin{pmatrix} 0.9915 & 0.0 \cr 0.0 & 0.0085 \end{pmatrix}$$
</span>

---
title: KDML Model System
subtitle: Timescales

<div class="flexbox vcenter">
<img src="figures/timescales.png">
</div>

KDML distance metric gives more converged timescales with fewer states.

---
title: Fip35 WW Domain (Shaw)
subtitle: Lets look at some <em>real</em> data

<div class="vcenter flexbox">
<div style="width:290px"><img height=150 src="figures/tocfigure.png"></div>
</div>

- Two 100 $\mu s$ trajectories.
- Sampled $k=20,000$ triplets at $t_{close}$ = 2 ns, $t_{far}$ = 20 ns
- Structures projected onto the sine and cosine components of the backbone
  dihedrals.

<footer class="source">Shaw et. al; Atomic-level characterization of the structural dynamics of proteins. <em>Science</em> <strong>2010</strong>, 330, 341–346</footer>

---
title: Fip35 WW Domain (Shaw)
subtitle: n states: 5000, lagtime: 75 ns
class: image

<div class="vcenter flexbox">
<div> <img height=450  src="figures/bars.png"> </div>
</div>

---
title: Unweighted metrics miss slow near-native dynamics

- The folding timescale is remarkably robust to changes in the distance metric.
- New timescales are observed in the 100 ns - 1 μs regime, corresponding 
  to near-native hydrogen bond reorganizations in the turns.

<div class="vcenter flexbox">
<div><img height=250 src="figures/state12.png"></div>
</div>

---
title: Unweighted metrics miss slow near-native dynamics

<div class="vcenter flexbox">
<div><img height=475 src="figures/hbonds_01.png"></div>
</div>



---
title: MSM State Decomposition Outlook

- This **is** the right direction, but not necessarily the right algorithm.
- Numerical optimization of the metric challenging &amp; can be poorly conditioned.
- **Low** **rank** metric = dimensionality reduction.
    - Enables computational access to more sophisticated clusterings.
- **Sparse** metric = feature selection.
    - Promising for biophysical insight.

---
title: MSMAccelerator
subtitle: Distributed Adaptive Sampling
class: segue dark nobackground

---
title: MSMAccelerator Architecture

- Distributed messaging passing design ([ZeroMQ](http://learning-0mq-with-pyzmq.readthedocs.org/en/latest/)).
- Two types of clients can connect to the adaptive server:
    - **Simulator**: Receives initial conditions, propagates dynamics.
    - **Modeler**: Receives trajectory data, builds an MSM.
- Adaptive server maintains weights for a multinomial distribution, from the
  most recent MSM.
- Setup system with serialized OpenMM XML files.
- Code on <a href="http://github.com/rmcgibbo/msmaccelerator2">github</a>.

---
title: Adaptive Sampling Algorithms
subtitle: MSMAccelerator provides the rapid prototyping capability

- How do we switch between sampling strategies?
    - Voelz surpisal strategy / mutual information between state decompositions.
    - Smooth interpolation, simulated annealing
- The explore/exploit tradeoff has strong overlap with the multi-armed bandit
  &amp; probabalistic multi-robot mapping problems.
- Knowledge discovery in Sergio's $\theta / \alpha / \beta$ scheme?

<footer class="source">
  S. Thrun. Exploration in Active Learning <strong>1998</strong> <br/>
  S. Bacallado, S. Favaro, L. Trippa. <strong>2012</strong>
</footer>
---
title: MSMAccelerator

<video class="center" height=350 controls>
  <source src="figures/movie.ogv" type="video/ogg">
</video> 
<div style="padding:0px"></div>

Ala5 (amber99sbiln, implicit) OpenMM 5.1 / MSMBuilder2.6. 1000 rounds
(1μs aggregate) of adaptive sampling (even).


<!-- Script builder selection -->
---
title: OpenMM Script Builder
subtitle: Effortlessly <a href="http://openmm.heroku.com">setup</a> OpenMM simulations
class: segue dark nobackground


<!-- Parameter selection -->
---
title: Optimal MSM Parameter Selection
subtitle: Is there a "right" number of states?

$$ P[X_{0...T-1}] dx^T = \prod_{i=0}^{T-1} T(X_i \rightarrow X_{i+1}) \cdot \prod_{i=0}^{T} p(X_{i} | \sigma(X_{i}))$$

- The likelihood of an MSM includes both contributions from the transition
  matrix and the state space.
    - Probability of observing a given conformation, given the state
- The trivial 1-state model would have a transition matrix likelihood of 1.

---
title: A Model for the Intrastate Distribution
subtitle: Minimal assumptions $\rightarrow$ uniform distribution

We assume that the probability of sampling a given configuration, given that
the system is in state $i$, is uniform over the volume of the state.

$$ p(X_i | \sigma(x_i)) = \begin{cases}
\frac{1}{V(\sigma(x_i))} & \mbox{if $X_i$ in state $\sigma(i)$} \cr
0 & \mbox{otherwise } \end{cases}
$$

---
title: Computing the Volume of States
subtitle: Not easy

Approaches:

- Rejection sampling monte carlo
- Ball walk monte carlo
- Analytic approximation based on the distribution of point-generator distances?

---
title: Müller potential

<div class="vcenter flexbox">
<div>
<img src="https://raw.github.com/rmcgibbo/opt-k/master/paper/figs/kcent_vors.png"
  height=500/>
</div>
</div>

---
title: Müller potential

<div style="float:right; margin-top:-100px; padding-right:80px">
<image src="https://raw.github.com/rmcgibbo/opt-k/master/paper/figs/like_comp.png"
height=600>
</div>

- A model with more states will almost always have a higher empirical likelihood.
- Model selection via cross validation, AIC, or BIC.
- $AIC = 2k - 2 \ln(L)$
- We can also compare different clustering algorithms.

---
title: Optimal MSM Parameter Selection
subtitle: Outlook

- Choosing the lag time is more challenging, and requires a rate matrix
  formalism.
- High dimensional volumes are extremely challenging to compute.
- Our microstate models are undenyably overfit.

<footer class="source">
M. Simonovits; How to compute the volume in high dimension?
<em>Math. Program., Ser. B 9</em> <strong>2003</strong> 97: 337–374
</footer>