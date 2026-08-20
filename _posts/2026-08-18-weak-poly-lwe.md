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

If our guess $g$, is correct, then the collection of error images $\{e_j(\alpha)\}$ follow the distribution $\phi_\alpha(\mathcal{N}_\sigma)$ 

## Verifying Membership of the Error Images in the Error Distribution

Now that we have the images of our errors $\{e_j(\alpha)\}$, we must determine if the errors belong to the distribution our errors were originally sampled from, i.e. $\mathcal{N}_\sigma$. This gives us a way to determine if the pairs $(a_j(x), b_j(x))$ were generated from a Poly-LWE instance or are uniformly sampled from $R_q \times R_q$, which is only possible if our guess $g$ matches the image of the actual secret $s(\alpha)$. 

To do this, we first compute the set $S$ of all possible values of $e(\alpha)$. Since $e(\alpha)$ is defined as 

$$e(\alpha) = e_0 + \alpha e_1 + \alpha^2 e_2 +...+\alpha^{n-1} e_{n-1}$$

Since $\alpha$ is a root of small order $r$ modulo $q$ by assumption, i.e., $\alpha^r\equiv 1 \pmod{q}$, we can simplify this sum in the following 2 cases: 

### Case 1: $n$ is divisible by $r$

Here, calculating $e(\alpha)$ is pretty straightforward. 

$$\begin{aligned}e(\alpha) ={}& (e_0 + e_r + e_{2r}+...) + \alpha (e_1 + e_{r+1} + e_{2r+1}+...) \\&+ ...+ \alpha^{r-1}(e_{r-1} + e_{2r-1} + e_{3r-1}+...)\end{aligned}$$

where each of the coeffecients of $\alpha ^ i$ is $(e_i + e_{i+r}+e_{2i+r}+e_{3i+r}+...e_{n-r+j})$ 
### Case 2: $n$ is not divisible by $r$

When $r$ does not divide $n$, then the first $k$ coefficients of $\alpha^i$ have one extra term than the last $r-k$ coefficients.

We can find $k$ by taking $n \pmod{r}$ and let $q = \lfloor n/r \rfloor$ be the quotient. Then we can compute $e(\alpha)$ as:

$$\begin{aligned} e(\alpha) ={}& \sum_{i=0}^{k-1} \alpha^i \left( \sum_{j=0}^{q} e_{i + j \cdot r} \right) + \sum_{i=k}^{r-1} \alpha^i \left( \sum_{j=0}^{q-1} e_{i + j \cdot r} \right) \end{aligned}$$

**Explanation of the terms:**

- **For the first $k$ groups** (corresponding to powers $\alpha^0, \alpha^1, \dots, \alpha^{k-1}$), the index goes up to $q$, meaning each coefficient contains $q + 1$ terms:
    
	$$e_i + e_{i+r} + e_{i+2r} + \dots + e_{i + q \cdot r}$$
    
- **For the remaining $r-k$ groups** (corresponding to powers $\alpha^k, \dots, \alpha^{r-1}$), the index stops at $q-1$, meaning each coefficient contains $q$ terms:
    
    $$e_i + e_{i+r} + e_{i+2r} + \dots + e_{i + (q-1) \cdot r}$$

We can write a function to compute the set $S$ as follows:

```python
import math
from itertools import product as cart 

def generate_error_sums(n, sigma, truncate_limit, q, r):
    max_val = int(math.ceil(truncate_limit * sigma))
    possible_values = range(-max_val, max_val + 1)

    q_quotient = n // r
    k = n % r
    
    # Case 1: If n is divisible by r, all groups have length q_quotient
    # Case 2: If not divisible, the 'long' groups have length q_quotient + 1
    long_len = q_quotient + 1 if k != 0 else q_quotient

    # Using set comprehensions makes this faster and more Pythonic
    sums_long = list({
        sum(vector(Zmod(q), vec)) 
        for vec in cart(possible_values, repeat=long_len)
    })

    sums_short = []
    if k != 0:
        sums_short = list({
            sum(vector(Zmod(q), vec)) 
            for vec in cart(possible_values, repeat=q_quotient)
        })

    return sums_long, sums_short

def generate_error_set(alpha, q, n, sigma, r):
    # Generate [1, alpha, alpha^2, ..., alpha^{min(n,r)-1}]
    alpha_is = vector(Zmod(q), [pow(alpha, i, q) for i in range(min(n, r))])

    k = n % r

    # s1 corresponds to the groups with an extra term, s2 to the rest
    s1, s2 = generate_error_sums(n, sigma, 4, q, r)

    if k == 0:
        # Case 1: n is divisible by r. All r groups are identical in length.
        possible_coefficients = [s1] * r
    else:
        # Case 2: n is not divisible by r. 
        # The first k coefficients have the extra term (s1).
        # The remaining r-k coefficients have one fewer term (s2).
        possible_coefficients = [s1] * k + [s2] * (r - k)

    # Compute all possible values of e(alpha)
    return {
        alpha_is * vector(coeffs) 
        for coeffs in cart(*possible_coefficients)
    }
```

**Note:** While the attack works on paper, pushing the order past $r > 5$ makes generating the group sums using the Cartesian product (`cart`) computationally impossible. The sheer volume of combinations causes your script to instantly run out of memory or hang indefinitely. This is the primary bottleneck of this attack.
{: .notice--warning}

## Putting it All Together 

Now that we have a rough idea of how the attack unfolds, let's put it all together into pseudocode and then write a function in sage. 

```
Input: A collection of l Poly-LWE samples (a_j, b_j), a root alpha, and valid error set S
Output: The secret guess g for s(alpha), or "NOT PLWE"

1. Initialize an empty list of surviving guesses: G = []

2. For each possible guess g in Z_q (from 0 to q-1):
   a. Assume the guess is valid: potential = True
   
   b. For each sample (a_j, b_j) in our collection:
      - Compute the error image: err = b_j(alpha) - g * a_j(alpha)
      - Check if err belongs to our expected error distribution/set S
      - If err NOT in S:
         - potential = False
         - Break (stop checking remaining samples for this guess)
         
   c. If potential is still True after checking all samples:
      - Append g to G (it survived the filter!)

3. Evaluate final results:
   - If G is empty: Return "NOT PLWE" (the samples are random, not Poly-LWE)
   - If G has a single element: Return g (Success! Secret image recovered)
   - If G has multiple elements: Return "INSUFFICIENT SAMPLES" (need more samples to narrow it down)
```

We want to define this as a function in sage, so that we can try repeating it with all the roots of $f(x)$

```python 
def attack(samples, q, N, sigma, R_q, alpha, alpha_order, D):
    S = generate_error_set(alpha, q, N, sigma, alpha_order)
    
    for g in range(q):
        # A guess 'g' is valid if every sample's error image falls within the set S
        is_valid_guess = all(
            R_q(list(b - g * a))(alpha) in S 
            for a, b in samples
        )
        
        if is_valid_guess:
            return g
            
    return None
```

Performance Note: We make use of an early-exit pattern (break), this implementation skips checking remaining samples the moment a candidate secret guess $g$ produces an error image outside of $S$. Because the vast majority of guesses in $Z_q$ will fail, this optimization drastically reduces execution time, ensuring we don't waste CPU cycles testing invalid guesses against every single sample in our collection.
{: .notice--info}
# Recovering the Complete Secret with Lagrange Interpolation
{: #recover}

# Generating Weak Rings
{: #polygen}

# Modifying Sagemath's Ring-LWE Oracle to Verify the Secret
{: #modi}

# References

[^1]: Y. Elias, K. E. Lauter, E. Ozman, and K. E. Stange, *Provably Weak Instances of Ring-LWE,* 2015. [arXiv:1502.03708](https://arxiv.org/abs/1502.03708).

[^2]: K. Eisentraeger, S. Hallgren, and K. Lauter, *Weak Instances of PLWE*, Cryptology ePrint Archive, Paper 2014/784, 2014. [eprint.iacr.org/2014/784](https://eprint.iacr.org/2014/784).