---
layout: post
title: A simple analysis of multiplicative weights
---
I give a clean and self-contained presentation of the multiplicative weights framework that is somewhat more direct than the most popular analysis (see the comparison below). The framework provides a simple algorithm for (approximately) solving many important problems (e.g., maximum flow, densest subgraph, linear classification, zero-sum games and even general linear programs) and is a beautiful demonstration of leveraging continuous analysis to solve discrete problems. Warning: this is a technical post.

All of the above problems can be written in the following simple form:

$$\begin{align}
\text{minimize }\ & \max(Ax) \\
\text{such that }\ & x \in K
\end{align}$$

Symbol $A \in \mathbb{R}^{n \times m}$ represents a matrix, $x \in \mathbb{R}^m$ is a vector we are trying to optimize over, $\max : \mathbb{R}^n \to \mathbb{R}$ returns the maximum entry of a vector (e.g., $\max([1, 3, -4]^T) = 3$), and $K$ is some convex set (often called the **easy constraints**).

<details markdown="1">  <!-- markdown means the internals get parsed -->
<summary><b>Example</b>: Casting maximum flow in the above form. <a>(click to expand)</a></summary>
Suppose we are given an uncapacitated directed graph $G = (V, E)$ and we want to compute the maxflow between some $s, t \in V$. Suppose that the optimal value of this problem is $\OPT$ (i.e., there are $\OPT$ edge-disjoint paths between $s$ and $t$).

We cast the problem in the above form. On a high-level, easy constraints $K$ will correspond to unit flows from $s$ to $t$. We first fix the representation of $K$: a flow $x \in K$ is a vector of dimension $\vert E \vert$, one scalar for each directed edge $e \in E$. Precisely, $x_e$ will represent the amount of flow that is pushed through $e$. This determines the representation. Now, define $K$ to be the convex hull of all simple paths from $s$ to $t$. On a side-note, this is equivalent to saying the flow $x \in K$ is conserving in $V \setminus \\{s, t\\}$, pushes 1 unit from $s$, and contains no circulations; equivalency is due to the integrality of the flow polytope. While it might be strange to call this polytope with exponential number of vertices "easy constraints", we only interact with $K$ by optimizing a linear function over $K$. However, optimizing a linear function $f(x) = \inner{c, x} = \sum_{e \in E} c_e \cdot x_e$ over $x \in K$ is exactly the problem of finding a shortest path between $s$ and $t$ with edge costs being $c \in \mathbb{R}^{\vert E \vert}$.

We want the objective $\max(Ax)$ of our form to correspond to the maximum amount of flow going through any edge in the graph (call this the **congestion**). Using the above representation for members of $K$, this means we set $A$ to be the identity matrix $A = I$. Minimizing congestion might seem counterintuitive (or at least unrelated) to maxflow which wishes to maximize the flow from $s$ to $t$. However, we claim that minimizing the congestion over $K$ is equivalent to maxflow. To show one side of the equivalence, if there are $\OPT$ disjoint paths we can find a unit flow with congestion $1/\OPT$. Moreover, one can easily show the converse holds (if there are unit-flows with smaller congestion it leads to more edge-disjoint paths) and hence solving the above problem solves the maximum flow. Of course, the optimal value in the disjoint-path formulation and congestion-minimizing formulation will be reciprocals of each other, but the problems are otherwise equivalent.
</details>

<details markdown="1">
<summary><b>High-level comparison with other analyses:</b> <a>(click to expand)</a></summary>
This post is inspirated by my personal struggles I had a few years back while trying to learn the multiplicative weights framework. Most popular analyses motivate the approach by *the expert prediction* algorithm [AHZ]. While the approach is intuitive by itself, my intuition completely dissapeared when using them to solve problems such as maximum flow. This is because the experts from [AHZ] essentially correspond to dual variables which are largely disconnected from the original (primal) problem. This analysis keeps the entire discussion in the primal. I have not seen this analysis written down anywhere, but I am sure researchers in the area are well-aware of it.

The analysis is essentially equivalent to the Frank-Wolfe method of optimization applied to the log-sum-exp function (denoted below as $\smax$) over $x \in K$. One can say that multiplicative weights is an instance of the Frank-Wolfe method.
</details>

On a high-level, multiplicative weights smoothen the problem by replacing (regular) max with a smooth version of max. The smoothened problem is solved iteratively: the current solution $x_{t-1}$ is updated by linearizing the smooth max around $x_{t-1}$, finding the optimizer of the linearization $h_t$ via any black-box method and updating $x_{t} \gets x_{t-1} + h_t$. The following fact introduces the smooth max and a few of its simple-to-prove properties.

<div class="fact" markdown="1">
**Fact (smooth max):** Define $\smax : \mathbb{R}^n \to \mathbb{R}$ as

$$\mathrm{smax}_{\beta}(x) = \frac{1}{\beta} \ln\left( \sum_{i=1}^n \exp(\beta \cdot x_i) \right)$$

where $\beta$ is some "accuracy" parameter (increasing $\beta$ increases accuracy but decreases smoothness). The following properties hold:

* $\smax$ approximates max:

$$\mathrm{smax}_{\beta}(x) \in [\max(x), \max(x) + \frac{\ln n}{\beta}]$$

* The gradient is some probability distribution over $[n]$:

<center>$$\nabla \mathrm{smax}_{\beta}(x) = ( \frac{1}{Z} \exp(\beta \cdot x_i) )_{i=1}^n$$ where $Z := \sum_{i=1}^n 
\exp(\beta \cdot x_i)$ is the normalization factor</center>

* $\smax_{\beta}$ is $\beta$-smooth with respect to $\dnorm{\cdot}_\infty$:

$$\mathrm{smax}_\beta(x + h) \le \mathrm{smax}_\beta(x) + \inner{\nabla \smax_{\beta}(x), h} + \beta\cdot \dnorm{h}_\infty^2$$

* $$\smax_{\beta}( \pmb{0} ) = \frac{\ln n}{\beta}$$, where $\pmb{0} = [0,0,\ldots,0]$.

</div>

We introduce and compare the original problem, the smoothened version and the linearized version around $x_{t-1}$.

<div style="display:grid;grid-template-columns:1fr 0.3fr 1fr 0.3fr 1fr;">
<div class="cell">
$$\begin{align}
\text{min. }\ & \mathrm{max} (Ax) \\
\text{s.t. }\ & x \in K
\end{align}$$
</div>
<div class="cell">
$$\to$$
</div>
<div class="cell">
$$\begin{align}
\text{min. }\ & \color{red}{\mathrm{s}}\mathrm{max}_{\color{red}{\beta}} (Ax) \\
\text{s.t. }\ & x \in K
\end{align}$$
</div>
<div class="cell">
$$\to$$
</div>
<div class="cell">
$$\begin{align}
\text{min. }\ & \inner{ A^T \nabla \smax_{\beta}(A x_{t-1}), h_t } \\
\text{s.t. }\ & h_t \in K
\end{align}$$
</div>
</div>

In the maxflow problem, the parameter $\beta$ is chosen to be approximately $\eps$. Let's elaborate on the objective of the last problem. Let $\Phi(x) := \smax_{\beta}(Ax)$ be the objective of the smoothened (middle) problem. Then $\inner{ A^T \nabla \smax_{\beta}(A x_{t-1}), h_t }$ is exactly the linearization of $\Phi(x)$ around $x_{t-1}$. This is because $\Phi(x_{t-1} + h_t) - \Phi(x_{t-1})$ is approximately $\inner{\nabla \Phi(x_{t-1}), h_t }$ and $\nabla \Phi(x_{t-1}) = A^T \nabla \smax_{\beta}(A x_{t-1})$ by chain rule.

We can now present the full multiplicative weights algorithm. We repeat the linearize-solve-update loop for a total of $T$ rounds (to be specified later). Solving the linearized problem is done via some black-box method (called the **oracle**) that needs to be supplied to the algorithm upfront. Precisely, the oracle is supplied a vector $p \in \mathbb{R}^n$ and needs to return $\arg\min_{x \in K} \inner{ p, x }$. Luckily, the linearized problem is often much simpler than the original one and can be easily optimized.

How to choose the number of rounds $T$? It will mostly depend on two important parameters: the accuracy $\eps$ we require from the final solution, and a new parameter $\rho$ that we call the **width of the oracle**. The only requirement on the width is that $\rho \ge 1$ must be larger than $$\dnorm{Ax}_\infty$$ for any $x$ that can be returned by the oracle (e.g., $\max_{x \in K} \dnorm{Ax}_\infty$ suffices, but sometimes we can implement the oracle that achieves a better bound).

<div markdown="1" class="algorithm">
**Algorithm (Multiplicative Weights):**
* Input: matrix $A \in \mathbb{R}^{n \times m}$, oracle for the linearized problem, width $\rho$, accuracy $\eps$.
* Define $$\beta := \frac{\eps}{2 \rho^2}$$, $x_0 := 0$, $\Phi(x) := \smax_{\beta}(Ax)$, $$T := \frac{\ln n}{\beta \cdot \eps} = O(\eps^{-2} \rho^2 \ln n)$$.
* For $t = 1 \ldots T$
  * $$h_t := \arg \min_{h \in K} \inner{ \nabla \Phi(x_{t-1}), h }$$ via the oracle
  * $x_t := x_{t-1} + h_t \qquad$ (equivalent: $x_t = \sum_{i=1}^t h_i$)
* Output $(1/T) \cdot x_T$
</div>

<br/>
<details markdown="1">  <!-- markdown means the internals get parsed -->
<summary><b>Example</b>: Multiplicative weights for maximum flow. <a>(click to expand)</a></summary>
Suppose we want to solve maxflow between $s$ and $t$ with $\eps$ relative error. We assume for simplicity that the graph is directed and uncapacitated which allows us to set $\rho = 1$. Set $\beta$ and $T$ accordingly. Let $\OPT$ be the optimal value of the problem when cast in the aforementioned standard form (which is the reciprocal of the number of edge-disjoint paths between $s$ and $t$, note that the relative error is unchanged when taking reciprocals).

Initialize a "congestion" vector $x := [0, 0, \ldots, 0] \in \mathbb{R}^{\vert E \vert}$ that remembers for each edge how many times has it been used. We repeat the following for $T$ rounds: for each directed edge $e$ we compute a cost $c_e := \exp(\beta x_i) > 0$. Normalize this vector of costs by dividing all entries by $$Z := \sum_{e \in E} \exp(\beta x_i)$$ that makes their sum equal to $1$ (note: this is unnecessary, but we do it to stay true to the algorithm). Find the shortest path $P$ between $s$ and $t$ with respect to the edge costs $c$ (e.g., with a Dijkstra). Update $x$ by incrementing $x_e$ for each edge $e$ that was used in the shortest path $P$.

After the above loop terminates, the collection of all shortest paths found throughout the algorithm provide us with $T$ paths that incur congestion of at most $T \cdot (OPT + \eps)$. Scaling by $1/T$, we find a unit-flow that incurs congestion $OPT + \eps$ and we are done.

Note: in the capacitated version we would need to set $\rho := 1 / c_{\min}$, where $c_{\min}$ is the minimum positive edge capacity. This is a significant downside of the method and a long line of research has been developed in order to reduce this width.
</details>

<details markdown="1">  <!-- markdown means the internals get parsed -->
<summary><b>Low-level comparison with other analyses:</b> <a>(click to expand)</a></summary>
Many other authors take a dual approach of the one stated above, in which one maintains a set of "multiplicative weights" or "dual variables" $y \in \mathbb{R}^n$ (one for each constraint of $K$, i.e., row of $A$). The dual roughly tells us how "important" a constraint is (e.g., if $y_3 = 100$ and $y_7 = 1$, it would mean that constraint 3 gets violated much more often and more egregiously than constraint 7). Initially, $y \gets [1, 1, \ldots, 1]$ and then for each $h$ we multiplicatively update $y$ with a value proportional to the violation $Ah$ via the formula $$y_i \gets y_i \cdot \exp\left(\beta (A h)_{i} \right)$$ for $i \in [n]$. This multiplicative reweighting of the constraint importance is the reason behind the method's name. The "linearize+solve" steps now correspond to solving $\min_{h \in K} \inner{y, A h} = \min_{h \in K} \inner{A^T y, h}$. Iteratively performing this procedure yields the same dynamics as the one described in the pseudocode above.

Pattern-matching $\inner{A^T y, h}$ with $\inner{ A^T \nabla \smax_{\beta}(A x_{t-1}), h }$ one can, correctly, presume that the connection between the methods is that $y = \nabla \smax_{\beta}(A x_{t-1})$. This is indeed the case (up to a factor of proportionality that can be ignored), $\nabla \smax(A x_{t-1})$ is a probability distribution proportional to $\exp(\beta A(h_1 + \ldots + h_{t+1}))$ (see property 2 of $\smax$ Fact); $y$ can be easily deduced to have the same value.
</details>


We now prove that the above algorithm works.
<div markdown="1" class="theorem">
**Theroem**: Let $\OPT$ be the value of the optimal solution and $\tilde{x} := x_T/T$ be the output of the algorithm. Then $\tilde{x} \in K$ and $\max(A \tilde{x}) \le OPT + \eps$.
</div>

<div markdown="1" class="proof">
**Proof:** It is sufficient to show $\max(A x_T) \le T \cdot (OPT + \eps)$ since it is equivalent to $\max(A \tilde{x}) = \max(A x_T/T) \le OPT + \eps$, by scaling. Furthermore, note that solving the smoothened version of the problem $\Phi(x) = \smax_{\beta}(Ax)$ is "harder" than solving the original one since $\Phi(x) \ge \max(A x)$ by property 1 of Fact (smooth max). Therefore, it is sufficient to show $\Phi(x_T) \le T \cdot (OPT + \eps)$. Also, note that $x_T/T = \sum_{t=1}^T \frac{1}{T} h_t \in K$ since $K$ is convex and $h_t \in K$.

We track the evolution of $\Phi(x_t) = \Phi(h_1 + h_2 + \ldots + h_t)$ by arguing about the increase $\Phi(x_{t-1} + h_t) - \Phi(x_{t-1})$ in each round. Up to first order terms, the increase is exactly $\inner{\nabla \Phi(x_{t-1}), h_t}$, the linearized problem we are optimizing. We argue that this (approximate) increase is small, $\inner{\nabla \Phi(x_{t-1}), h_t} \le \OPT$. However, this is immediate because the linearized problem is a "relaxation" of the original problem. Let $x_{\ast} \in K$, $\max(A x_{\ast}) = \OPT$ be the optimal solution. By definition of $\Phi$ we have that $$\inner{\nabla \Phi(x_{t-1}), h_t} = \inner{\nabla \smax(x_{t-1}), A h_t}$$ where $\nabla \smax(\cdot)$ is some probability distribution. It is easy to see that $$\inner{p, A x_{\ast}} \le \OPT$$ for any probability distribution $p$, hence the oracle will always return a value not larger than $\OPT$.

We argued that the first-order increase in $\Phi$ is $\inner{\nabla \Phi(x_{t-1}), h_t} \le \OPT$. In reality, we have to account for higher-order errors via property 3 of Fact (smooth max). 

$$\Phi(x_{t-1}+h_t) - \Phi(x_{t-1}) \le \inner{\nabla \Phi(x_{t-1}), h_t} + \beta \dnorm{A h_t}^2_\infty \le \OPT + \frac{\eps}{2}$$

This allows us to effectively bound $\Phi$:

$$\Phi(x_T) \le \Phi(\pmb{0}) + \sum_{t=1}^T\left[ \Phi(x_{t-1} + h_t) - \Phi(x_{t-1}) \right] \le \frac{\ln n}{\beta} + \sum_{t=1}^T \left[ OPT + \frac{\eps}{2} \right]$$

Straighforward algebra with $T = \frac{\ln n}{\beta \cdot \eps}$ gives us that the right-hand side is at most $T \cdot (OPT + \eps)$ and we are done.

</div>

<br/>

A few final points:
* One can often get a better dependence on $\rho$ for special problems. E.g., one can get a linear instead of quadratic dependence of $\rho$ for positive packing and positive covering problems such as maxflow. See [AHZ] for details.
* It would be interesting to rederive matrix multiplicative weights using an analogue of the above analysis.
* Computational aspects (assuming non-negative entries in $A$, constant $\eps$, constant $\rho$). The algorithm is numerically stable. Examining the algorithm, the only part that might need elaboration is the computation of $\nabla \Phi(x_{t-1})$ since its direct computation involves exponentiation. We argue this exponentiation does not give raise to numerical issues.
  <center>$$\nabla \Phi(x_{t-1}) = A^T \nabla \smax_\beta(A x_{t-1})$$</center>
  <center>$$\nabla \smax_{\beta}(A x_{t-1}) = \nabla \smax_{\beta}(A \sum_{j=1}^{t-1} h_{t-1}) = \frac{1}{Z} \left[ \exp\left(\frac{\eps}{2\rho^2} (A \sum_{j=1}^{t-1} h_{j})_i \right) \right]_{i=1}^n$$</center>
  We note that $\dnorm{Ah_j}_{\infty} \le \rho$, hence the term inside $\exp( \cdot )$ is at most $T \cdot \frac{\eps}{2\rho^2} \cdot \rho = \rho \cdot \eps^{-1} \cdot \ln n$. This quantity is $O(\log n)$ (e.g., assuming constant $\eps$, constant $\rho$), hence after exponentiating the argument, the result is at most a polynomial in $n$. Every other operation is a stable addition, multiplication, or division of positive reals, making numerical issues of no concern.
* Computational aspects (assuming non-negative entries in $A$). If one explicitly recalculates $\nabla \Phi(x_{t-1})$, the intermediate numbers involved in the exponentiation might become overwhelmingly large. One possible workaround is to keep a running value of $y_{t-1} := \nabla \smax_{\beta}(x_{t-1})$. One updates $y_{t-1} \to y_t$ with $(y_{t})_i = (y_{t})_i \cdot \exp(\beta (A h_t)_i )$ and normalizing $y_t \gets y_t / Z$ where $Z := \sum_i (y_t)_i$. Then we have $\nabla \Phi(x_t) = A^T y_t$.


References:
* [AHK] Arora, Hazan, Kale: *The Multiplicative Weights Update Method: A Meta-Algorithm and Applications*

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    TeX: {
      Macros: {
        inner: ["{\\left\\langle #1 \\right\\rangle}", 1],
        dnorm: ["{\\vert\\!\\vert #1 \\vert\\!\\vert}", 1],
        eps: "{\\varepsilon}",
        OPT: "{\\mathrm{OPT}}",
        smax: "{\\mathrm{smax}}",
      }
    }
});
</script> 
