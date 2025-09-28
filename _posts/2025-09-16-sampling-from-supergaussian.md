---
layout: post
title: Sampling from the 2D Supergaussian Distribution
date: 2025-09-24 11:12:00-0400
description: Formulas for drawing samples from the supergaussian distribution with inversion sampling.
tags:
categories:
related_posts: false
---

![Supergaussian density](/assets/img/tech_notes/supergaussian_samples_3.png){: width="400" }

The 2D supergaussian distribution is useful for describing electron beams generated from lasers that can vary from Gaussian to flat-top in shape.
To simulate these beams in a particle tracking code, samples need to be drawn from the distribution.
In this note, I describe how to generate samples from the 2D supergaussian distribution using inversion sampling.

# Inversion Sampling

A neat, efficient way of generating samples from an arbitrary distribution is inversion sampling.
First, we generate samples over $[0, 1]$ from the uniform distribution ($\rho_u$).
Then, we transform them by the inverse of the cummulative density function of the desired distribution ($\rho_d$).
Let's calculate the resulting density, knowing that densities transform with the determinant of the Jacobian out front.

$$\rho_u(x) \to \left[ \frac{d [\text{CDF}]}{d x^\prime}\right]_{x^\prime}\cdot \rho_u(\text{CDF}(x^\prime)).$$

We can simplfy this.
First off, $\rho_u(\text{CDF}^{-1}(x^\prime))$ is one everywhere since we began with the uniform distribution over $[0, 1]$ and the CDF maps from the domain of our desired distribution onto $[0, 1]$.
The second simplification is that the CDF is just the integral of the PDF of our desired distribution.
That is, $\left[ \frac{d [\text{CDF}]}{d x^\prime}\right]_{x^\prime} = \rho_d(x')$
Therefore, the transformed density is,

$$\rho_u(x) \to \rho_d(x^\prime).$$

We have reproduced the desired distribution.

# Inverse Cummulative Density Function of Supergaussian

There are multiple ways of defining the supergaussian distribution, but for this note I will use the convention,

$$\rho(x, y) = A\cdot\exp\left[-\log(2)\left(4x^2 + 4y^2\right)^n\right].$$

This variant is symmetric in $x$ and $y$ and has a full width at half maximum of one.
I don't include variance and covariance here as they can be trivially applied to the resulting samples through a linear transformation.
I will also handle normalization (ie the term $A$) later.

Note that a transformation to polar coordinates naturally expresses the rotational symmetry of the density.
Doing this and being careful to multiply by the determinant of the Jacobian to preserve the interpretation of $\rho$ as a PDF gives,

$$\rho(r, \theta) = |J|\cdot\rho(x(r, \theta), y(r, \theta)) = A\cdot r\cdot\exp\left[-\log(2)4^nr^{2n}\right].$$

Since the PDF is separable, we can draw samples for $\theta$ (uniformly distributed) and $r$ (distributed accoring to the above equation) separately, then transform into cartesian coordinates.
The values of $\theta$ can be trivially distributed.
To use inversion sampling for $r$, we must calculate the inverse of its CDF

Integrating the previous equation and then normalizing so that $\lim\limits_{r\to\inf} \text{CDF}(r) = 1$ gives,

$$\text{CDF}(r) = 1 - \frac{\Gamma(1 - (n-1)/n, 4^nr^{2n}\log(2))}{\Gamma(1/n)},$$

where $\Gamma(x)$ and $\Gamma(a, x)$ are the complete and upper incomplete gamma functions respectively.

This CDF isn't invertible in terms of common functions.
However, many numerical libraries (such as [scipy](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.invgamma.html) and [boost](https://www.boost.org/doc/libs/1_64_0/libs/math/doc/html/math_toolkit/sf_gamma/igamma_inv.html)) contain methods to efficiently calculate the inverse of the incomplete Gamma functions.
Solving the problem in terms of the inverse regularized gamma function (I use the convention of `gamma_q_inv` from boost), the inverse CDF is,

$$\text{CDF}^{-1}(u) = \frac{1}{2}\left[\cdot\Gamma_q^{-1}(1/n, 1-u)/\log(2)\right]^{1/(2n)}.$$

## Generating Samples in Python

To recap, I have calculated the inverse CDF of the radial part of the supergaussian distribution in polar coordinates.
To generate a sample, we first generate two uniformly distributed random numbers, the first is transformed by the inverse of the CDF we found to get $r$.
The second is rescaled to $[0, 2\pi]$ to give use $\theta$.
The coordinates are the then transformed to cartesian coordinates.

The following python code implements this.

```python
from scipy.special import gammainccinv
import numpy as np

def sample_supergaussian(n, n_samp):
    # Start with uniform random samples
    u = np.random.random(n_samp)
    v = np.random.random(n_samp)

    # Transform by inverse CDF of dist in polar coordinates
    theta = 2*np.pi*u
    r = 0.5*np.power(gammainccinv(1/n, (1-v))/np.log(2), 1/2/n)

    # Transform to Cartesian coordinates
    x = r*np.cos(theta)
    y = r*np.sin(theta)

    return x, y
```

This code gnerates the following samples. Remember that in my convention, $n=1$ is a Gaussian distribution, $n\to\inf$ is flat-top.

![Supergaussian samples](/assets/img/tech_notes/supergaussian_samples_1.png){: width="500" }

Plotting the densities using a histogram also helps to confirm that the method works.

![Supergaussian density](/assets/img/tech_notes/supergaussian_samples_2.png){: width="500" }
