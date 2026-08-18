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

*Elias et al.* explores weak instances of Poly-LWE [^1], which are susceptible to such algebraic attacks. 

# References

[^1]: Y. Elias, K. E. Lauter, E. Ozman, and K. E. Stange,
"Provably Weak Instances of Ring-LWE," 2015.
[arXiv:1502.03708](https://arxiv.org/abs/1502.03708).