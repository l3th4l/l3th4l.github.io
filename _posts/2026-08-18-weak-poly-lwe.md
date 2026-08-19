---
title: Exploring Complete Key Recovery in Provably Weak Instances of Poly-LWE
categories:
  - Blog
tags:
  - Cryptography
  - LWE
  - FHE
date: 2019-04-18T15:34:30-04:00
---
One of the fundamental hard problems in modern Post-Quantum Cryptography is Learning With Errors (LWE). Poly-LWE adds algebraic structure to the LWE problem by substituting the vectors in LWE with polynomials. However, this also induces the potential risk of being vulnerable to attacks exploiting its algebraic structure. 

*Elias et al.* [^1] and *Eisentraeger et al.* [^2] explore weak instances of Poly-LWE, which are susceptible to such algebraic attacks. In this article, I will be focusing on Poly-LWE instances from *Elias et al.* [^1], satisfying certain properties which allow us to recover the secret polynomial. 

More specifically, I'd describe `Algorithm 1` from the paper, with accompanying code in `python / sagemath`, along with a demo of the best-case scenario where it allows for the full secret recovery in Poly-LWE. 

**Practicality Note:** The best case scenario results in a toy problem, with the secret polynomial being of degree 4. Any higher order polynomial (i.e. for most practical applications), would render complete secret recovery impractical with the current approach. 
{: .notice--warning}
# Background : Poly-LWE and Weak Quotient Rings

In this section, I give a brief introduction to the Poly-LWE problem, and then describe instances of polynomial rings which make Poly-LWE vulnerable. 
## Poly-LWE 

Let $f(x)$ be an $n$-degree [monic](https://en.wikipedia.org/wiki/Monic_polynomial) [irriducible](https://en.wikipedia.org/wiki/Irreducible_polynomial) polynomial in $\mathbb{Z}\left[x\right]$, and let $q$ be a prime number. In Poly-LWE, all polynomials are computed in the [polynomial](https://courses.csail.mit.edu/6.440/spring08/scribe/lec5.pdf) [quotient ring](https://en.wikipedia.org/wiki/Quotient_ring) $R_q = \mathbb{Z}_q / f(x)$ 

We first sample a random secret polynomial $s(x)$ uniformly from $R_q$. The Poly-LWE system in our setup, hence consists of the ciphertext tuple $\left(a(x), b(x)\right)$ such that $b(x) = a(x)s(x) + e(x) \;\in R_q$, where each time we generate a sample for our Poly-LWE system, $e(x)$ is the error drawn from a [discrete Gaussian distribution](https://di-mgt.com.au/discrete_gaussian.html) with mean of $0$ and variance $\sigma^2$, and $a(x)$ is a random public polynomial, sampled uniformly from $R_q$. 

## Weak Rings

There are specific properties that the ring $R_q$ must satisfy to be vulnerable to the attacks in the following sections. Those are as follows: 

1. $q$ must be a prime such that $f(x)$ factorizes completely modulo $q$. 
	
	This might be counterintuitive to the fact that $f(x)$ is an irreducible polynomial in $\mathbb{Z}[x]$, so let's demonstrate what it means with an example.
	
1. 

# Generating Weak Rings

# References

[^1]: Y. Elias, K. E. Lauter, E. Ozman, and K. E. Stange, *Provably Weak Instances of Ring-LWE,* 2015. [arXiv:1502.03708](https://arxiv.org/abs/1502.03708).

[^2]: K. Eisentraeger, S. Hallgren, and K. Lauter, *Weak Instances of PLWE*, Cryptology ePrint Archive, Paper 2014/784, 2014. [eprint.iacr.org/2014/784](https://eprint.iacr.org/2014/784).