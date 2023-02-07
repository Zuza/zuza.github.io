---
layout: post
title: A Modern Big-O without Notation Abuse
---
**Summary**: The Big-O notation has been one of the most influential concepts in all of computer science. However, Big-O's textbook definition is no longer aligned with its usage by theoretical computer scientists. This is because the traditional notation does not effectively capture their ideas and intuitions. Therefore, I suggest an alternative notation that is more practical, formal, and in sync with current usage. In a nutshell: the $O$ should be interpreted as *a sufficiently-large universal constant* (without any attached infinite process). Several aspects of the new definition are discussed.

*Note:* This article is NOT a primer. If needed, please see a good introduction [here](https://simple.wikipedia.org/wiki/Big_O_notation) or [here](https://en.wikipedia.org/wiki/Big_O_notation).

*Note*: Several people pointed out that the old definition is too entrenched; changing it would be too disruptive and confusing. This is a valid point. Please consider this article a hypothetical proposal meant to serve as a starting point for discussion about overcoming some shortcomings of the current definitions. Many points in this article can be seamlessly adopted without disruption.
 
# Introduction

Computer science adopted the Big-O notation from number theory, where it was pioneered by Bachman and Landau in the 1900s. We review the standard definition.
<div markdown=1 class="theorem">
**The Bachman-Landau definition:** Given functions \\( f, g : \\{1, 2, \ldots\\} \to \mathbb{R} \\) we write \\( f(n) = O( g(n) ) \\) when there exist \\( C > 0, n_0 > 0 \\) such that \\( \vert f(n) \vert \le C \cdot g(n) \\) for all \\( n \ge n_0 \\).

Similarly, $f(n) = \Omega(g(n))$ when there exist \\( C > 0, n_0 > 0 \\) such that \\( \vert f(n) \vert\ {\color{blue}\ge}\ C \cdot g(n) \\) for all \\( n \ge n_0 \\).
</div>

Big-O turned out to be one of the most influential driving forces of algorithm design. Researchers using Big-O were implicitly guided to focus on the big picture: ignore constants and lower-order terms, and only focus on asymptotics (as any finite-value behavior is cropped off).

The success of Big-O has led to its ubiquity throughout computer science. However, the Bachman-Landau definition is impractical and inflexible for modern research. In response, its usage is increasingly informal, colloquial, and non-mathematical. I propose a slight change in definition to help address the failure modes of Bachman-Landau. I will propose a new convention for Big-O, discuss its various aspects and provide illustrative examples.

*Note:* These ideas are certainly not original. However, I have not seen them being explicitly argued anywhere. My aim is to give a formal foundation aligned with current research usage.

### Failure mode 1: unnecessary infinite process

<div markdown=1 class="theorem">
**Fact:** The breadth-first search takes $O(|V| + |E|)$ time on a graph $G = (V, E)$.
</div>

**Issue:** what exactly is the infinite process \\(n \in \\{ 1, 2, \ldots \\}\\) here? As written, the meaning is at best ambiguous as the Bachman-Landau definition only handles a single variable. At worst, the statement is wrong in that sense that parsing it should throw an error. This definition+statement combo can be found in first-year algorithms classes and teaches students that definitions are handwavy and informal. This should be unacceptable.

On an intuitive level, the implied infinite process is "for every infinite sequence of graphs", but this is unwieldy and does not match the intuition that computer scientists want to convey.
The right interpretation is that a constant $C > 0$ exists (it depends on the underlying RAM model) such that the breath-first search (BFS) completes in at most $C \cdot (|V| + |E|)$ time.

### Failure mode 2: informal usage in algorithms

It is customary to use Big-O in algorithms. A typically usage follows:

> Theorem: Algorithm X computes the answer in \\( {\color{blue}{O(n)}} \\) steps.
>
> Algorithm X: \\
> &nbsp;&nbsp; for step in 1, ..., \\( {\color{blue}{O(n)}} \\): \\
> &nbsp;&nbsp;&nbsp;&nbsp; ...

**Issue:** Formally, Algorithm X allows for the loop to run a total of $1$ steps since $1 = O(n)$. However, actually doing this will almost always be wrong. Examples are found [here (alg1)](https://arxiv.org/pdf/2102.06977.pdf), [here (alg3)](https://arxiv.org/pdf/2003.08929.pdf), [here (alg1)](https://papers.nips.cc/paper/2021/file/b035d6563a2adac9f822940c145263ce-Paper.pdf), [here (alg1 & alg2)](https://proceedings.neurips.cc/paper/2018/file/79a49b3e3762632813f9e35f4ba53d6c-Paper.pdf), [here (alg1)](https://epubs.siam.org/doi/pdf/10.1137/1.9781611977073.107), [here (alg1)](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9317991), [here (alg2)](https://arxiv.org/pdf/2206.02348.pdf). The notation was used informally.

Moreover, replacing $O(n)$ with $\Omega(n)$ is also typically wrong, as $0.01 n$ is also not enough steps. Replacing it with $\Theta(n)$ is wrong for the same reason. The intended meaning is to run the loop for $C \cdot n$ steps, where $C > 0$ is a sufficiently-large constant. This is exactly how this proposal redefines it. My hot take is: take a typical computer science paper and reinterpret Big-Os by following this proposal. It will immediately become less wrong than if one uses Bachman-Landau's interpretation.

### Failure mode 3: no Big-O lower bounds or premises

<div markdown=1 class="theorem">
**Fact:** There exists a constant $C > 0$ such that [IID](https://en.wikipedia.org/wiki/Independent_and_identically_distributed_random_variables) sampling of at least $C \cdot \sqrt{n}$ elements from a universe of size $n$ will ensure a collision (drawing of the same element twice) with probability of at least 99%.
</div>

Replacing the first part with *"IID sampling of at least $O(\sqrt{n})$ elements ..."* will often land you in hot water with reviewers because Bachman-Landau does not allow for expressing lower bounds as $O(\ldots)$. Writing *"IID sampling of at least $\Omega(\sqrt{n})$ elements ..."* is plain wrong. The only unimpeachable way to use Big-O in this context would be to write something impractical like:
<div markdown=1 class="theorem">
**Fact:** Let $f(n)$ be the smallest integer such that IID sampling of at least $f(n)$ elements from a universe of size $n$ will ensure a collision with probability at least 99%. Then $f(n) = O(\sqrt{n})$.
</div>
I believe this inflexibility is suboptimal.


As a general rule: Using Big-O in premises (rather than conclusions) or in lower bounds (rather than upper bounds) often leads to issues. Similarly to the example above, consider the following quote ([source](https://dl.acm.org/doi/10.1145/3086465)). What exactly does it mean?

> A fundamental result by Karger [10] states that for any $\lambda$-edge-connected graph with $n$ nodes, independently sampling each edge with probability $p = \Omega(\log (n) / \lambda)$ results in a graph that has edge connectivity $\Omega(\lambda p)$, with high probability.

The most natural interpretation: "Given any infinite family, sampling with any \\( p(n) = \Omega(\log(n) / \lambda) \\) yields connectivity \\( \Omega(\lambda p) \\)" is wrong, as choosing \\( p(n) = 0.01 \ln(n) / \lambda \\) can result in a disconnected graph with probability tending to \\( 1 \\) (e.g., consider a path where consecutive vertices are connected with \\( \lambda \\) parallel edges). The constant replacing the first $\Omega$ must be sufficiently large. The inflexibility of Big-O caused the statement to be imprecise at best.

### Failure mode 4: colloquial usage and lack of formality

On its face, Bachman-Landau does not explicitly define the meaning of statements like \\( O(n) + O(n) = O(n) \\), \\( n^{O(1)} \\), or \\( \exp(-\Omega(n)) \\). As such, researchers attach their own semantics to these. These vary in formality and mean different things to different authors.

At the same time, Big-O is proliferating to the point where many papers never explicitly specify any constants. In an extreme limit, this might lead to a world where papers are written exclusively with Big-Os (due to their inherent practicality), while its usage has become informal, unspecified folklore, and uncoordinated between subareas. In such a world, only domain experts would be able to infer the correct semantics of formal statements, introducing unnecessary barriers to interdisciplinarity in science. At worst, informality and misinterpretations cause wrong statements to be claimed.

On a positive note, such a scenario is easily avoidable: we put forward several proposals on how to formalize Big-Os (like this manuscript!), and papers specify which proposal they follow.

# A Better Definition: A Sufficiently-Large Constant

I propose to change the standard meaning of the Big-O. A statement $P$ using a single Big-O should be interpreted by replacing the $O$ with a sufficiently large constant and evaluating the statement. Specifically:
\\[ \exists C_0 > 0 \quad \forall C > C_0 \quad P(O \gets C), \\]
where $P(O \gets C)$ denotes $P$ with the Big-O being replaced by $C$. For example, if
\\[ P = \text{"BFS completes in at most } O(|V| + |E|) \text{ time."} \\]
Then,
\\[ P(O \gets 7) = \text{"BFS completes in at most } 7(|V| + |E|) \text{ time."} \\]
This statement $P$ is correct as, indeed, (in whichever RAM model) there always exists a sufficiently large constant $C_0 > 0$ such that the statement is true for $C > C_0$. We assume this notation in the rest of the article.

**Digression: Why not "\\( \exists C_0 > 0\ \ P(O \gets C_0) \\)"**?
I believe this would be disconnected from current usage. For example, correct statements would include "\\(4 < O(1) < 5\\)" (choose $O \gets 4.5$), or even "An algorithm with runtime $n^{O(1)}$ is linear." (choose $O \gets 1$).

### Example: Stirling's approximation

<div markdown=1 class="theorem">
**Fact**: For all $n \in \mathbb{N}$ it holds that \\( n! \le O(n^{n + 1/2} e^{-n}) \\).
</div>
    
Note that the relation operator is "$\le$", and replacing it with the customary "$=$" would be incorrect under our definition. The latter was always an abuse of notation. This is a (minor) point of divergence that I believe is necessary for mathematical precision.

We could have also used weaker approximations \\( n! \le n^{n+O(1)} e^{-n} \\) or \\( n! \le \exp( O(n \log n) ) \\) (both when $n \ge 2$), which are now also perfectly formal. Unfortunately, using such approximations in a formal publication will often agitate the reviewers.

# Big-Omega: A Sufficiently-Small Constant

Analogously to Big-O, a statement $P$ using a single Big-Omega should be interpreted as:

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
 
Consider another striking example of the highlighting power of Big-O. Suppose \\(X_1, \ldots, X_n\\) and \\([0,1]\\)-bounded independent random variables and let $X = \sum_{i=1}^n X_i$ be its sum with expectation \\(\E[X] = \mu\\). We can now succinctly showcase several aspects that are quite hard to grasp by only looking at formulas.

**Aspect:** The sum concentrates exponentially in the expectation. 
<div markdown=1 class="theorem">
**Fact**: \\( \Pr[X > 2 \mu] < \exp(- \Omega(\mu)). \\)
</div>
This shows, for example, that randomly throwing $O(n \log n)$ balls into $n$ bins will hit every bin at least once.
<br/><br/>

**Aspect:** When the deviation is close to $1$, the concentration is quadratic in the deviation. 
<div markdown=1 class="theorem">
**Fact**: \\( \Pr[X > (1 + \eps) \mu] < \exp(- \Omega(\eps^2\mu) ) . \\)
</div>
This shows, for example, that polling $O(1/\eps^2)$ people yields an $\eps \mu$ error.
<br/><br/>

**Aspect:** When the deviation is large, the concentration acts like a Poisson variable.
<div markdown=1 class="theorem">
**Fact**: \\( \Pr[X > \beta \mu] < \exp(- \Omega (\beta \ln \beta) \cdot \mu) . \\)
</div>
This shows, for example, that randomly throwing $n$ balls into $n$ bins will leave the most congested bin with at most $O(\frac{\log n}{\log \log n})$ balls.

# Ordering of Quantifiers: Statements with Multiple Big-Os

Statements with multiple Big-Os are inherently imprecise under Bachman-Landau. Indeed, Janson also [warns of potential problems](https://arxiv.org/pdf/1108.3924.pdf):

> I have for many years avoided the notations “O(·) w.h.p.” and “o(·) w.h.p.” on the grounds that these combine two different asymptotic notations in an ambiguous and potentially dangerous way. (In which order do the quantifiers really come in a formal definition?)

I propose to formalize such statements by fixing the order in which the "sufficiently-large constant" is determined. Specifically, we first fix the constant that replaces $O_1$, then the one for $O_2$, and so on. This greatly increases expressive power while maintaining mathematical formality.
<div markdown=1 class="theorem">
**Fact**: Given an algorithm for 3-SAT that always outputs in \\( n^{O_1(1)} \\) time, there exists an algorithm for solving any Hamiltonian cycle instance in \\( n^{O_2(1)} \\) time.
</div>

As a nice bonus, we can recall previous constants by using the same subscript.
<div markdown=1 class="theorem">
**Fact**: if $a \le O_1(1)$ and $b \le O_1(1)$, then $a + b \le O_2(1)$.
</div>

**Issue: Scope.** It would be clunky to use $O_1$ afresh only once in a paper. Therefore, I propose to keep the "scope" (i.e., the lifetime of a Big-Os constant) *minimal*. The scope should be clear from the context, but as a default, each new statement or paragraph should clear old constants. Of course, one can always explicitly save a constant for global use with \\( C_{\mathrm{important}} := O_1 \\).

### Example: "For sufficiently large $n$"

In many subareas of computer science and mathematics one only wants to make statements that hold for sufficiently large values of (the underlying infinite process) $n$. Consider the statement:

\\[ \Omega(n \log n) \ge O(n) . \\]

Within this proposal, such a statement is not true as written (for any ordering of quantifiers). On the other hand, the intended correct statement is:

\\[ \Omega_1(n \log n) \ge O_1(n) \text{ for } n > O_2(1) . \\]

I propose that such intentions be supported: writing "all statements in this paper hold only for sufficiently large $n$" effectively means automatically transforming all statements from the first into the second one. More generally, each statement gets appended with \\(\text{for } n > O_{\text{last}}(1) \\), where the introduced $O$ is last in the quantifier ordering.

### Example: "With high probability"

Theoretical computer scientists often use the statement "*Algorithm $A$ is correct with high probability.*" to mean exactly "*Algorithm $A(O_1)$ is correct with probability at least \\( 1 - 1 / n^{O_1(1)} \\).*" Critically, the algorithm must be tunable in response to $O_1$ to facilitate any requested failure probability like \\( 1 - 1 / n^{100} \\), where $n$ should be clear from the context (most commonly: input size).

The specific failure probability is chosen such that another algorithm $B$ can call $A$ as many as $n^{O_1(1)}$ times and still have *all* of the invocations be correct with probability at least \\( 1 - 1/n^{O_1(1)} \\). This is achieved by tuning $A$ to have an invocation correct with probability \\( 1 - 1 / n^{O_2(1)} \\).

Let's consider an involved example. Customarily, the "with high probability" Big-O preempts every other Big-O.
<div markdown=1 class="theorem">
**Fact**: Given a \\( \lambda \\)-connected graph with $n$ vertices, independently sampling each edge with probabiliy \\( p \ge \frac{O_2(\log n)}{\lambda} \\) will yield a graph that is $O_1(\log n)$ connected with high probability.
</div>

In full formality, the statement could be expanded as follows. Contemporary papers are lax about the ordering of quantifiers in such claims.
<div markdown=1 class="theorem">
**Fact**: Given a \\( \lambda \\)-connected graph with $n$ vertices, independently sampling each edge with probabiliy \\( p \ge \frac{O_3(\log n)}{\lambda} \\) will yield a graph that is $O_2(\log n)$ connected with probability at least \\( 1 - \frac{1}{n^{O_1(1)}} \\).
</div>

# Parameter Dependence

Big-O constants often need to depend on other parameters. For example: $O_{\eps}(1)$ can (arbitrarily) depend on $\eps$. More precisely, the lower bound (denoted as \\( C_0 \\) in the definition) can depend on $\eps$, while the claim must hold for any constant $C > C_0(\eps)$.

### Example: undergraduate real analysis

One can co-opt the new definition to rephrase the standard $\eps-\delta$ definitions from introductory real analysis classes. Such usage, while formally correct, deviates from the customary usages of Big-O.

<div markdown=1 class="theorem">
**Definition**: A sequence $a_n$ tends to 0 when for all $\eps > 0$ it holds that $\vert a_n \vert < \eps$ for all $n > O_{\eps}(1)$.
</div>

A related example follows.
<div markdown=1 class="theorem">
**Definition**: A function \\( f : \mathbb{R} \to \mathbb{R} \\) is uniformly continuous when: For all $x,y$ and $\eps > 0$ we have \\( \vert x - y \vert < \Omega_{\eps}(1) \\) \\( \implies \\) \\( \vert f(x) - f(y) \vert < \eps \\).
</div>


### Example: Ruzsa-Szemeredi

<div markdown=1 class="theorem">
**Fact**: Every $n$-vertex graph with $\Omega_{\eps}(n^3)$ triangles can be made triangle-tree by removing at most $\eps n^2$ edges.
</div>

Notably, the proof of the claim can involve the famous Szemeredi's regularity lemma, where the hidden constant in $\Omega_{\eps}$ is a notoriously fast-growing function of $\eps$.



# Disadvantages of an Alternative Proposal: A Set of Functions

An alternative formalization of Big-O would be to define $O(f(n))$ as a set of functions in the following manner.

\\[ O(f(n)) := \\{ g : \\{ 1, 2, \ldots \\} \to \mathbb{R} \mid \exists C > 0, n_0, \text{ such that } \vert g(n) \vert \le C \cdot f(n)\ \forall n \ge n_0 \\} \\]

Then, one can define $\Omega$ analogously. Binary operations between sets are defined like \\( A + B := \\{ n 
\mapsto a(n)+b(n) \mid a(n) \in A, b(n) \in B\\} \\) (other operations are analogous). Unary operations are defined like \\( \exp(A) := \\{ n \mapsto \exp(a(n)) \mid a(n) \in A \\} \\).

**Pros:** Such a definition would be more in line with the standard Bachman-Landau convention (as compared to this proposal).

**Equal:** Same as in this proposal, statements like \\( O(n) + O(n) = O(n) \\), \\( n^{O(1)} \\), or \\( \exp(-\Omega(n)) \\) are well defined. Issues involving dependence and ordering of quantifiers remain, but can be resolved in similar ways to this proposal.

**Cons:** Most importantly, failure modes 1, 2, and 3 remain. Furthermore, the definition involves custom set operations, which are arguably non-intuitive for anyone not accustomed to Bachman-Landau.

I believe that the cons of this approach outweigh the pros.

# Other aspects

- **How to interpret Big-Theta?** A statement $P$ containing $\Theta(x)$ is correct if we can replace "$\Theta(x)$" with "at most $O(x)$" and "at least $\Omega(x)$" with the statement holding (both must hold). For example, this statement is correct: "The optimal comparison-based sorting algorithm completes in $\Theta(n \log n)$ rounds."

- **How to interpret Little-o and Little-omega?** The Little-o and Little-Omega are attached to an infinite process that needs to be specified or clear from the context. A statement containing (a single) little-o is correct if we can replace it with some (adversarially chosen) sequence $(a_n)_n$ that tends to $0$. For example, \\(o_1(1) + o_2(1) = o_3(1)\\) is correct. Another example, "The quadratic sieve factors $n$ in $n^{o(1)}$ time (as $n \to \infty$)." is correct. Little-omega is analogous, except the sequence must tend to $\infty$. For example, "$P \neq NP$ would imply that the optimal algorithm for 3-SAT needs $n^{\omega(1)}$ time on instances of size $n$ (as $n \to \infty$)" is correct.

<!-- # Other symbols -->

<!-- O = there exists a sufficiently-large constant -->
<!-- Omega = there exists a sufficiently-small constant -->
<!-- little o -->
<!-- strong exponential time hypothesis -->

<!-- ### asymptotic notions -->
<!-- o = asymptotic notion ... tends to 0 ... there exists a sequence that tends to 0 such that -->
<!-- omega ... tends to infinity ... there exists a sequence that tends to infinity such that -->

## Acknowledgement
I would like to thank Bernhard Haeupler for spurring my thoughts about this topic. Furthermore, I thank Roie Levin, Ellis Hershkowitz, and Filip Pavetic for reading the drafts and giving insightful comments.

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
