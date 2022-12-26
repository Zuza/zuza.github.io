---
layout: post
title: A Better Big-O Notation
hidden: true
permalink: okbquvkrklilvtrvcatgixthgzayqjld
---



<!-- [comment]: run by going to zuza.github.io (using br) and then $ bundle exec jekyll serve .. then chrome localhost:4000 -->

<!-- [comment]: I added a line preventing this from being published in index.html ... see the "if" with the title check -->


**Summary**: The Big-O notation has been one of the most influential concepts in all of computer science. However, Big-O's textbook definition is no longer aligned with its usage by theoretical computer scientists. This is because the traditional notation does not effectively capture their ideas and intuitions. Therefore, I suggest an alternative notation that is more practical, formal, and in sync with current usage. In a nutshell: the $O$ should be interpreted as *a sufficiently-large universal constant*. Several aspects of the new definition are discussed.

*Note:* This article is NOT a primer. If needed, please see a good introduction [here](https://simple.wikipedia.org/wiki/Big_O_notation) or [here](https://en.wikipedia.org/wiki/Big_O_notation).

Computer science adopted the Big-O notation from number theory, where it was pioneered by Bachman and Landau in the 1900s. We review the standard definition.
<div markdown="1" class="theorem">
**The Bachman-Landau definition:** Given functions \\( f, g : \\{1, 2, \ldots\\} \to \mathbb{R} \\) we write \\( f(n) = O( g(n) ) \\) when there exist \\( C > 0, n_0 > 0 \\) such that \\( \vert f(n) \vert \le C \cdot g(n) \\) for all \\( n \ge n_0 \\).
</div>

Big-O turned out to be one of the most influential driving forces of algorithm design. Researchers using Big-O were implicitly guided to focus on the big picture: ignore constants and lower-order terms, and only focus on asymptotics (as any finite-value behavior is cropped off).

The success of Big-O has led to its ubiquity throughout computer science. However, the Bachman-Landau definition is impractical and inflexible for modern research. In response, its usage is increasingly informal, colloqiual, and non-mathematical. I propose a slight change in definition to help address the failure modes of Bachman-Landau. I will propose the new convention for Big-O, discuss its various aspects and provide examples to illustrate them.

*Note:* These ideas are certainly not original. However, I have not seen them being explicitly written down.

### Failure mode 1: unnecessary infinite process

<div markdown="1" class="theorem">
**Fact:** The breadth-first search takes $O(|V| + |E|)$ time on a graph $G = (V, E)$.
</div>

**Issue:** what exactly is the infinite process \\(n \in \\{ 1, 2, \ldots \\}\\) here? The implied process is "for every infinite sequence of graphs", but this is a extremely unwieldy and does not correspond to the intuition that computer scientists want to convey.
The right interpretation is that there exists a constant $C > 0$ (that depends on the underlying RAM model) such that the breath-first search completes in at most $C \cdot (|V| + |E|)$ time.

### Failure mode 2: no Big-O lower bounds

<div markdown="1" class="theorem">
**Fact:** There exists a constant $C > 0$ such that randomly sampling at least $C \cdot \sqrt{n}$ elements from a universe of size $n$ will ensure a collision (we draw the same element twice) with probability at least 99%.
</div>

Replacing the first part with *"Randomly sampling at least $O(\sqrt{n})$ elements ..."* will often land you in hot water with reviewers because Bachman-Landau does not allow for expressing lower bounds as $O(\ldots)$. Writing *"Randomly sampling at least $\Omega(\sqrt{n})$ elements ..."* is plain wrong. The only unimpeachable way to use Big-O in this context would be to write something impractical like:
<div markdown="1" class="theorem">
**Fact:** Let $f(n)$ be the smallest integer such that randomly sampling at least $f(n)$ elements from a universe of size $n$ will ensure a collision with probability at least 99%. Then $f(n) = O(\sqrt{n})$.
</div>
I believe this is suboptimal.

### Failure mode 3: colloqiual usage and lack of formality

@@@ TODO


# A Better Definition: A Sufficiently-Large Constant

I propose to change the standard meaning of the Big-O. An expression $P$ using a single Big-O should be interpreted as
\\[ \exists C_0 > 0 \quad \forall C > C_0 \quad P(O \gets C), \\]
where $P(O \gets C)$ denotes $P$ with the Big-O being replaced by $C$. For example, if \\( P = O(|V| + |E|) \\), then \\( P(O \gets 10) = 10(|V| + |E|) \\). We assume this notation in the rest of the article.

### Example: Stirling's approximation

<div markdown="1" class="theorem">
**Fact**: For all $n \in \mathbb{N}$ it holds that \\( n! \le O(n^{n + 1/2} e^{-n}) \\).
</div>

Note that the relation operator is $\le$, and replacing it with the customary $=$ would be incorrect under our definition. This is a (minor) point of divergence that I believe is necessary for mathematial precision.

We could have also used weaker approximations \\( n! \le n^{n+O(1)} e^{-n} \\) or \\( n! \le \exp( O(n \log n) ) \\) (both when $n \ge 2$), which are now also perfectly formal. Unfortunately, using such approximations in a formal publication will often agitate the reviewers.

# Big-Omega: A Sufficiently-Small Constant

Analogously to Big-O, an expression $P$ using a single Big-Omega should be interpreted as:

\\[ \exists \eps_0 > 0 \quad \forall \eps \in (0, \eps_0) \quad P(\Omega \gets \eps), \\]

### Example: tail comparison

As a simple example of the simplifying power, consider the question of "_which distribution decays faster for large $x$ --- a normal or an exponential one (for fixed parameters)?_" To the uninitiated, the formulas might seem complicated at first glance:

$$
\begin{align*}
    \Pr[\mathrm{Normal} = x] = \frac{1}{\sigma \sqrt{2 \pi}} \exp(-\frac{(x - \mu)^2}{2 \sigma^2}) && \qquad \Pr[\mathrm{Exp} = x] = \lambda \exp(-\lambda \cdot x)
\end{align*}
$$

However, the answer is immediate if informed that $\Pr[\mathrm{Normal} = x] \le \exp(- \Omega(x^2))$ and $\Pr[Exp = x] \le \exp(- \Omega(x))$: the normal one decays faster. Big-Omega simplified the problem down to triviality.

### Example: Chernoff bounds
 
Consider another striking example of the highlighting power of Big-O. Suppose \\(X_1, \ldots, X_n\\) and \\([0,1]\\)-bounded independent random variables and let $X = \sum_{i=1}^n X_i$ be its sum with expectation \\(\E[X] = \mu\\). We can now succintly showcase several aspets that are quite hard to grasp by only looking at formulas.

**Aspect:** The sum concentrates exponentially in the expectation. This shows, for example, that randomly throwing $O(n \log n)$ balls into $n$ bins will hit every bin at least once.

<div markdown="1" class="theorem">
**Fact**: \\( \Pr[X > 2 \mu] < \exp(- \Omega(\mu)). \\)
</div>

**Aspect:** When the deviation is close to $1$, the concentration is quadratic in the deviation. This shows, for example, that polling $O(1/\eps^2)$ people yields an $\eps \mu$ error.
<div markdown="1" class="theorem">
**Fact**: \\( \Pr[X > (1 + \eps) \mu] < \exp(- \Omega(\eps^2\mu) ) . \\)
</div>

**Aspect:** When the deviation is large, the concentration acts like a Poisson variable. This shows, for example, that randomly throwing $n$ balls into $n$ bins will leave the most congested bin with at most $O(\frac{\log n}{\log \log n})$ balls.
<div markdown="1" class="theorem">
**Fact**: \\( \Pr[X > \beta \mu] < \exp(- \Omega (\beta \ln \beta) \mu) . \\)
</div>

# Ordering of Quantifiers: Expressions with Multiple Big-Os

Expressions with multiple Big-Os are inherently imprecise under Bachman-Landau. Indeed, Janson also [warns of potential problems](https://arxiv.org/pdf/1108.3924.pdf):

> I have for many years avoided the notations “O(·) w.h.p.” and “o(·) w.h.p.” on the grounds that these combine two different asymptotic notations in an ambiguous and potentially dangerous way. (In which order do the quantifiers really come in a formal definition?)

I propose to formalize such expressions by fixing the order in which the "sufficiently-large constant" is determined. Specifically, we first fix the constant that replaces $O_1$, then the one for $O_2$, and so on. This greatly increases the expressive power while maintaining mathematical formality.
<div markdown="1" class="theorem">
**Fact**: Given an algorithm for 3-SAT that always outputs in \\( n^{O_1(1)} \\) time, there exists an algorithm for solving any Hamiltonian cycle instance in \\( n^{O_2(1)} \\) time.
</div>

As a nice bonus, we can recall previous constants by using the same subscript.
<div markdown="1" class="theorem">
**Fact**: if $a \le O_1(1)$ and $b \le O_1(1)$, then $a + b \le O_2(1)$.
</div>

**Issue: Scope.** It would be clunky to use $O_1$ afresh only once in a paper. Therefore, I propose to keep the "scope" (i.e., the lifetime of a Big-Os constant) *minimal*. The scope should be clear from context, but as a default each new statement or paragraph should clear old constants. Of course, one can always explicitly save a constant for global use with \\( C_{\mathrm{important}} := O_1 \\).

### Example: "with high probability"

Theoretical computer scientists often use the expression "*Algorithm $A$ is correct with high probability.*" to mean exactly "*Algorithm $A(O_1)$ is correct with probability at least \\( 1 - 1 / n^{O_1(1)} \\).*" Critically, the algorithm must be tunable in response to $O_1$ to facilitate any requested failure probability like \\( 1 - 1 / n^{100} \\), where $n$ should be clear from context (most commonly: input size).

The specific failure probability is chosen such that another algorithm $B$ can call $A$ as many as $n^{O_1(1)}$ times and still have *all* of the invocations be correct with probability at least \\( 1 - 1/n^{O_1(1)} \\). This is achieved by tuning $A$ to have an invocation correct with probability \\( 1 - 1 / n^{O_2(1)} \\).

Let's consider an involved example. Customarily, the "with high probability" Big-O preempts every other Big-O.
<div markdown="1" class="theorem">
**Fact**: Given a \\( \lambda \\)-connected graph with $n$ vertices, independently sampling each edge with probabiliy \\( p \ge \frac{O_2(\log n)}{\lambda} \\) will yield a graph that is $O_1(\log n)$ connected with high probability.
</div>

In full formality, the statement could be expanded as follows. Contemporary papers are lax about the ordering of quantifiers in such claims.
<div markdown="1" class="theorem">
**Fact**: Given a \\( \lambda \\)-connected graph with $n$ vertices, independently sampling each edge with probabiliy \\( p \ge \frac{O_3(\log n)}{\lambda} \\) will yield a graph that is $O_2(\log n)$ connected with probability at least \\( 1 - \frac{1}{n^{O_1(1)}} \\).
</div>

# Parameter Dependence

Big-O constants often need to depend on other parameters. For example: $O_{\eps}(1)$ can (arbitrarily) depend on $\eps$. More precisely, the lower bound (denoted as \\( C_0 \\) in the definition) can depend on $\eps$, while the claim must hold for any constant $C > C_0(\eps)$.

### Example: undergraduate real analysis

One can co-opt the new definition to rephrase the the standard $\eps-\delta$ definitions from introductory real analysis classes. Such usage, while formally correct, deviates from the customary usages of Big-O.

<div markdown="1" class="theorem">
**Definition**: A sequence $a_n$ tends to 0 when for all $\eps > 0$ it holds that $\vert a_n \vert < \eps$ for all $n > O_{\eps}(1)$.
</div>

A related example follows.
<div markdown="1" class="theorem">
**Definition**: A function \\( f : \mathbb{R} \to \mathbb{R} \\) is uniformly continuous when: For all $x,y$ and $\eps > 0$ we have \\( \vert x - y \vert < \Omega_{\eps}(1) \\) \\( \implies \\) \\( \vert f(x) - f(y) \vert < \eps \\).
</div>


### Example: Ruzsa-Szemeredi

<div markdown="1" class="theorem">
**Fact**: Every $n$-vertex graph with $\Omega_{\eps}(n^3)$ triangles can be made triangle-tree by removing at most $\eps n^2$ edges.
</div>

Notably, the proof of the claim can involve the famous Szemeredi's regularity lemma, where the hidden constant in $\Omega_{\eps}$ is a notoriously fast-growing function of $\eps$.



# Disadvantages of an Alternative Proposal: A Set of Functions

An alternative formalization of Big-O would be to define $O(f(n))$ as a set of functions in the following manner.

\\[ O(f(n)) := \\{ g : \\{ 1, 2, \ldots \\} \to \mathbb{R} \mid \exists C > 0, n_0, \text{ such that } \vert g(n) \vert \le C \cdot f(n)\ \forall n \ge n_0 \\} \\]

Then, one can define $\Omega$ analogously. Binary operations between sets are defined like \\( A + B := \\{ n 
mapsto a(n)+b(n) \mid a(n) \in A, b(n) \in B\\} \\) (other operations are analogous). Unary operations are defined like \\( \exp(A) := \\{ n \mapsto \exp(a(n)) \mid a(n) \in A \\} \\).

**Pros:** Such definition would be more in line with the standard Bachman-Landau convention (as compared to this proposal).

**Equal:** Same as in this proposal, statements like \\( O(n) + O(n) = O(n) \\), \\( n^{O(1)} \\), or \\( \exp(-\Omega(n)) \\) are well defined.

**Cons:** Most importantly, failure modes 1 and 2 remain. Furthermore, it is unclear how to specify the ordering of quantifiers without redoing much of the work of the current proposal. Furthermore, the definition involves custom set operations, which are arguably non-intuitive for anyone not accustomed to Bachman-Landau.

I believe that the cons outweight the pros.

<!-- # Other symbols -->

<!-- O = there exists a sufficiently-large constant -->
<!-- Omega = there exists a sufficiently-small constant -->
<!-- little o -->
<!-- strong exponential time hypothesis -->

<!-- ### asymptotic notions -->
<!-- o = asymptotic notion ... tends to 0 ... there exists a sequence that tends to 0 such that -->
<!-- omega ... tends to infinity ... there exists a sequence that tends to infinity such that -->

## Acknowledgement
I would like to thank Bernhard Haeupler for spurring my thoughts about this topic.

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    TeX: {
      Macros: {
        inner: ["{\\left\\langle #1 \\right\\rangle}", 1],
        dnorm: ["{\\vert\\!\\vert #1 \\vert\\!\\vert}", 1],
        E: "{\\mathbb{E}}",
        eps: "{\\varepsilon}",
        OPT: "{\\mathrm{OPT}}",
        smax: "{\\mathrm{smax}}",
      }
    }
});
</script> 
