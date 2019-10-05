---
layout: post
title: Hamiltonian Monte Carlo for Babies
subtitle: (not really)
tags: [statistics]

---

## Foreword

This semester, the instructor of the intro-level Bayesian Statistics course in my department decided to convert all students 
into using [`stan`](https://mc-stan.org/) for statistical computing. From a Bayesian statistician's perspective, `stan` is a 
really nice tool for MCMC sampling, but it uses something called "Hamiltonian Monte Carlo" (HMC) which is not easy to 
understand or explain.

Well, it definitely took *me* a while to kind of (but not exactly) get what HMC is and why it works so well. AND I had to 
teach a roomful of PhD students about it in a lab session! So, after quite some struggling, I did manage to give a mini 
lecture on this topic, and this blog post is a compilation of my (very short) lecture notes.

Almost all the materials here are from [this nicely written article](https://arxiv.org/abs/1701.02434) by Michael Betancourt and 
[this great YouTube playlist](https://youtu.be/FGQddvjP19w) by Gabriele Carcassi. My "baby" version is obviously much less 
rigorous, but hopefully I can convey the basic message. 

## Conventional MCMC methods sometimes don't work
Many of us (including me) have used and love to use inference methods like Gibbs sampling or Metropolis-Hastings sampling.
These methods are often (relatively) easy to understand, formulate, and programme, but unfortunately they perform unsatisfactorily
when the target density (e.g. posterior density) looks "weird" or "ugly".

So, they work fine when the target density looks nice and smooth like this one:
![A ``nice'' density in 2 dimensions.](https://fanbuduke17.github.io/img/Nice_density.jpeg)

But they *really* struggle when the density looks like this ugly-shaped thing: 
![A ``weird'' density in 2 dimensions.](https://fanbuduke17.github.io/img/Nice_density.jpeg)

(Both pictures are drawn by [Jordan Bryan](https://j-g-b.github.io/), a brilliant colleague of mine.)

For example, Gibbs samplers often suffer from these two major issues:

* High autocorrelation; this leads to "sticky" chains and very low effective sample sizes.
* Getting "stuck"; when the target density has high curvature regions, the sampler tends to get trapped inside.

And that's why certain smart people started searching for alternative sampling methods that can handle tricky densities. 
(~Then they found out the magical Hamiltonian Monte Carlo and built stan~)

## HMC tackles "weird" densities via "nice" Markov Chain transitions

The key idea of MCMC sampling is to "walk around" the parameter space according to the target density; if you do it right and 
long enough, eventually you get a bunch of samples with an empirical distribution just like your target density.

Therefore in MCMC, two things have to work out:
* We need to sample from the "right" region
* We need to "fully explore" the sample space

And correspondingly, a good MCMC sampling method should be able to:
* Quickly find the "right" region; this is **not** just the high density region, but rather a broader area with not-too-low densities
* Efficiently move around the right region; this means it must have a "nice" way to transition from one spot to another

As it turns out, HMC ticks both boxes (hooray!).

It sounds like a myth, but HMC does stand on solid theoretical grounds in differential geometry. Unfortunately "differential geometry"
is something way too dense for an average statistician, but, fortunately, its intuition can be gained from Hamiltonian mechanics.

## HMC and Hamiltonian mechanics

(I'm assuming that you still remember a bit of high school physics...)

Very loosely speaking, Hamiltonian mechanics describes the mechanics in an ideal world where the total volume of mechanic 
energy is **preserved**. 

Imagine we are riding a little shuttle in this cute, ideal world. Let `x` represent our location and `p` 
represent our momentum (this is just mass multiplied by velocity). So, in some sense, the location `x` relates to our **potential**
energy, `V(x)`, and the momentum `p` relates to our **kinetic** energy, \\(K(p,x)\\) (let it somehow depend on the location too). 
Because the mechanic energy is **preserved**, we always have the same sum of the potential and kinetic energies. Let us call it 
`H(p,x)` (the "Hamiltonian"):
\\[ H(p,x) = K(p,x) + V(x)\\]

