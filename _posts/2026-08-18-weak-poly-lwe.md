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

**Practicality Note:** The best case scenario results in a toy problem, with the secret polynomial being of degree 4. Any higher order polynomial (i.e. for most practical applications), would render complete secret recovery impractical with the current approach. {: .notice--warning}

# Background : Poly-LWE and Weak Fields

In this section, I give a brief introduction to the Poly-LWE problem, and then describe instances of number fields which make Poly-LWE vulnerable. 

### Poly-LWE 

Let $f(x)$ be a [monic](https://en.wikipedia.org/wiki/Monic_polynomial) [irriducible](https://en.wikipedia.org/wiki/Irreducible_polynomial) polynomial 

# References

[^1]: Y. Elias, K. E. Lauter, E. Ozman, and K. E. Stange, *Provably Weak Instances of Ring-LWE,* 2015. [arXiv:1502.03708](https://arxiv.org/abs/1502.03708).

[^2]: Kirsten Eisentraeger, Sean Hallgren, and Kristin Lauter, *Weak Instances of PLWE*, Cryptology ePrint Archive, Paper 2014/784, 2014. https://eprint.iacr.org/2014/784