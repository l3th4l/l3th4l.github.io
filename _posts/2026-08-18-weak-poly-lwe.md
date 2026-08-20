---
title: Exploring Complete Key Recovery in Provably Weak Instances of Poly-LWE
categories:
  - Blog
tags:
  - Cryptography
  - LWE
  - FHE
date: 2026-08-18T15:34:30-04:00
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
	
	> **Note:** This might be counterintuitive to the fact that $f(x)$ is an irreducible polynomial in $\mathbb{Z}[x]$, so let's demonstrate what it means with the example below. 
	> 
	> __Example 1:__ Let's take $f(x) =  x^2 + 603$ and the prime $q =  151$. By itself, $f(x)$ cannot be reduced into factors $c(x), d(x)$ such that $f(x) = c(x)d(x)$. 
	> But when we take $f(x) \pmod{151}$, we get 
	>
	> $$f(x) \equiv x^2 + 150 \pmod{151}$$
	> 
	> Which factorizes into 
	>
	> $$f(x) \equiv (x + 1) (x + 150) \pmod{151}$$
	>
	> Let's verify this. 
	>
	> $$\begin{aligned} f(x) &\equiv x^2 + x + 150x + 150 \pmod{151} \\ &\equiv x^2 + 151x + 150   \pmod{151} \\ &\equiv x^2 + 150 \pmod{151} \end{aligned}$$
	{: .notice--info}

1. All theh roots of $f(x)$ must be of small [order](https://crypto.stanford.edu/pbc/notes/numbertheory/order.html) or $\pm 1$.
	
	**Note:** The original paper only requires $f(x)$ to have a single root of small order. This is because the original paper only recovers a single homomorphic image of the secret polynomial $s(x)$ and does not attempt to recover the whole of $s(x)$
	{: .notice--info}

The method fo generating such polynomial rings along with the prime $q$ is described later in the section: [Generating Weak Rings](#generating-weak-rings)
# Poly-LWE in Sagemath 

While `sagemath` doesn't provide us with an implementation of Poly-LWE, it does provide us with a [Ring-LWE Oracle Generator](https://doc.sagemath.org/html/en/reference/cryptography/sage/crypto/lwe.html#sage.crypto.lwe.RingLWE) with the option to provide our own  polynomial for calculating the Quotient Group $R_q$, which effectively turns it into a Poly-LWE Oracle.

Throughout the article, we would work with the parameters $q =  13783771$, and 

$$\begin{aligned}
f(x) ={}& x^4 - 13783770x^3 + 233945232486523x^2 \\
        &- 605837133717152552775x \\
        &+ 605836899771878714937.
\end{aligned}$$

Which factorizes completely mod $q$ into: 

$$f(x) \equiv (x-13783770)(x-8774745)(x-5009025)(x-1) \pmod{q}$$

It is clear from this expression that $13783770, 8774745, 5009025$ and $1$ are roots of $f(x)$. Let's define this in our `sagemath` code:

```python
q = 13783771
F = GF(q)
R_q = PolynomialRing(F, 'x')

N = 4 #degree of the polynomial

f = x^4 - 13783770*x^3 + 233945232486523*x^2 - 605837133717152552775*x + 605836899771878714937
```

For the noise sampler, we would be going with a Discrete Gaussian Sampler with mean $0$, and the variance $\sigma = 3$. Let us use $\mathcal{N}_\sigma$ to denote this distribution. With the parameters now defined, we could create the Poly-LWE instance as follows: 

```python 
from sage.crypto.lwe import RingLWE, DiscreteGaussianDistributionPolynomialSampler

sigma = 3.0

#the noise distribution
D = DiscreteGaussianDistributionPolynomialSampler(ZZ['x'], n=euler_phi(N), sigma=sigma)

PolyLWEInstance = RingLWE(N, q, D, poly = f)
```

**Note:** While it is possible to import the Ring-LWE oracle directly from sagemath, one of the drawbacks is that implementation doesn't reveal the secret polynomial $s(x)$, which might be nedeed for verifying if our attack works. For this purpouse, one can slightly modify sage's implementation to reveal the secret as in the section: [Modifying Sagemath's Ring-LWE Oracle to Verify the Secret](#modi)
{: .notice--info}

For our next stages, we would need to calculate the roots of this polynomial. Although we had already talked about the factorization of $f(x)$ and the roots in the beginning of this section, we now show how we could calculate this for ourselves. 

```python 
f_q = R_q(f) #taking f(x) mod q

roots = [r[0] for r in f_q.roots()]
```

We could also verify for ourselves that all the roots of $f(x)\pmod{q}$ are low-order (i.e. $<< q$) as follows:

```python  
for r in roots:
    print(f"order of the root {r} : {r.multiplicative_order()}")
```

**Output:**
```
order of the root 13783770 : 2
order of the root 8774745 : 3
order of the root 5009025 : 3
order of the root 1 : 1
```

# Decision and Search Poly-LWE problem 

# Attack Setup 

Before we proceed with our attack, we need to draw $m$ samples of the pairs $(a_j(x), b_j(x))$ from our oracle. To generate a single sample we can simply call our `PolyLWEInstance` : 

```python
m = 20

samples = [PolyLWEInstance() for _ in range(m)]
```

The attack then proceeds in the following four stages:

1. Transfer the problem from $R_q$ to $\mathbb{Z}_q$ via a ring homomorphism $\phi:R_q \rightarrow \mathbb{Z}_q$.
2. Loop through the guesses for the possible images $\phi(s(x))$ of the secret.
3. Assuming that the guess at hand is correct, compute image of the error polynomials $\phi(e_j(x))$.
4. Examime the distribution of $\phi(e_j(x))$ to determine if it matches the distribution the errors are sampled from or not.

In the next section ([Recovering the Complete Secret with Lagrange Interpolation](#recover)), we extend this further to enable recovery of the complete secret. 

## Transferring to $\mathbb{Z}_q$ 

The first step of our attack requires us to find a ring homomorphism from $R_q$ to $\mathbb{Z}_q$, which makes the problem of guessing the secret, and validating the distribution of the errors tractable, given a root of small order. 

Given $f(x)$ has no double roots, we specify a root $\alpha = \alpha_i$, where $\alpha_i$ for $i=0, 1, ..., n-1$ are roots of $f(x)$. We define the evaluation homomorphism as : 

$$\phi_{\alpha} : R_q \rightarrow \mathbb{Z}_q \; \; ,  \phi_\alpha(g) = g(\alpha)$$

We then apply this homomorphism to the coordinates of the $m$ samples $(a_j(x),b_j(x))$, giving us $(a_j(\alpha),b_j(\alpha))_{j=1,...,m}$.

## Looping Through the Guesses of the Secret and Computing the Image of the Error

We combine step 2. and 3. since every guess for the secret's image yields a corresponding image of the error in a very straightforward manner.

We loop through all the possible values of $g\in\mathbb{Z}_q$, where each $g$ is considered to be a guess for the image of the secret $s(\alpha)$, i.e., $g=s(\alpha)$

For each guess $g$, we assume that it is a correct guess for $s(\alpha)$ and we compute the image of the errors $e_j(\alpha)_{j=1,...,m}$ as

$$\begin{aligned} e_j(\alpha) & = b_j(\alpha) - a_j(\alpha)g \\
&=b_j(\alpha) - a_j(\alpha)s(\alpha)\end{aligned}$$
If our guess $g$, is correct, then the collection of error images $\left\{e_j(\alpha)\right\}$ follow the distribution $\phi_\alpha(\mathcal{N}_\sigma)$ 

## Verifying Membership of the Error Images in the Error Distribution

Now that we have the images of our erros $\left\{e_j(\alpha)\right\}$, we must determine if the errors belong to the distribution our errors were originally sampled from, i.e. $\mathcal{N}_\sigma$. This gives us a way to determine if the pairs $(a_j(x), b_j(x))$ were generated from a Poly-LWE instance or are uniformly sampled from $R_q \times R_q$, which is only possible if out guess $g$ matches the image of the actual secret $s(\alpha)$. 

To do this, we first compute the set $S$ of all possible values of $e(\alpha)$. Since $e(\alpha)$ is defined as 

$$e(\alpha) = e_0 + \alpha e_1 + \alpha^2 e_2 +...+\alpha^{n-1} e_{n-1}$$

Since $\alpha$ is a root of small order $r$ modulo $q$ by assumption, i.e., $\alpha^r\equiv 1 \pmod{q}$, we can simplify this sum in the following 2 cases: 

### Case 1: $n$ is divisible by $r$

Here, calculating $e(\alpha)$ is pretty straightforward.

$$e(\alpha) = $$

### Case 2: $n$ is not divisible by $r$


# Recovering the Complete Secret with Lagrange Interpolation
{: #recover}

# Generating Weak Rings
{: #polygen}

# Modifying Sagemath's Ring-LWE Oracle to Verify the Secret
{: #modi}

# References

[^1]: Y. Elias, K. E. Lauter, E. Ozman, and K. E. Stange, *Provably Weak Instances of Ring-LWE,* 2015. [arXiv:1502.03708](https://arxiv.org/abs/1502.03708).

[^2]: K. Eisentraeger, S. Hallgren, and K. Lauter, *Weak Instances of PLWE*, Cryptology ePrint Archive, Paper 2014/784, 2014. [eprint.iacr.org/2014/784](https://eprint.iacr.org/2014/784).