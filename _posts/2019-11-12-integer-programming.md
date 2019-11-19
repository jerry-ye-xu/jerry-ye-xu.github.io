---
layout: post
title: Integer Programming
category: Theory
categories: ['integer programming','matriod property', 'optimisation']
tags: ['integer programming','matriod property', 'optimisation']
---

---
<h2 class="no_toc">Table of Contents</h2>

* TOC
{:toc}

In this post we look at why bipartite matching problem returns an optimal integer solution even if we relax the integrality constraint. 

## Bipartite Matching

Let's write the bipartite matching problem as a linear program and show that it feasible solutions are integral. 

Suppose we have a bipartite graph $G = (V, E)$ where the vertices are split into 2 sides $(L, R)$. The linear program representation of the maximum cardinality matching problem is:

$$
\begin{align}
    (P) \max \;\; & \left[\sum_{(i, j)\in E} x_{i, j}\right] \\
    \text{w.r.t} \;\; & \sum_{i} x_{i, j} \leq 1 & \forall j\in R \\
                 & \sum_{j} x_{i, j} \leq 1 & \forall i\in L \\
                 & x_{i, j} \geq 0       & \forall i\in L, j\in R
\end{align}
$$

The equivalent dual program is

$$
\begin{align}
    (D) \min \;\; & \left[\sum_{v\in V}y_{v}\right] \\
    \text{w.r.t} \;\; & y_{u} + y_{v} = 1 & \forall (u, v) \geq e \in E \\
                      & y_{v} \geq 0 & \forall v\in V
\end{align}
$$

They are equivalent by the Strong Duality Theorem, and this is essentially Konig's Theorem. We can write the primal (P) more concisely by building constraints based on the degree of each vertex. 

$$
\begin{align}
    (P') \max \;\; & \left[\sum_{(i, j)\in E} x_{i, j}\right] \\
    \text{w.r.t} \;\; & \sum_{(i, j) \in \delta(v)} x_{i, j} \leq 1 & \forall v \in V \\
                 & x_{i, j} \geq 0       & \forall i\in L, j\in R
\end{align}
$$

That was the unweighted version of the problem. 

The first step is to show that the weighted matching problem (WM) has only integral basic feasible solutions. This will of course, apply to the unweighed (or cardinality) matching problem (UWM). 

And yes, we have to assume that the weights are integral as well =).

## BFS of Bipartite Matching is Integral

Alright, so using (P'), let $x'$ be a basic feasible solution, and $$F := \left\{e\in E \; \rvert \; 0 < x_{e} < 1 \right\}$$. Since $F$ is any set of edges such that $x'$ is not integral, we just need to show that $F = \varnothing$. Then $x'$ contains only integral solutions.

Now, suppose the set of edges in $F$ have a cycle. Consider an example below.

![bipartite_integral_proof_even](/public/bipartite_integral_proof_even.jpeg)

Since this is a bipartite graph, any cycle we create MUST have an even number of edges. We can create 2 feasible solution vectors y' and y'' defined as

$$
  y' = 
  \begin{cases}
    x_{e} + \epsilon, & \text{for } e \in M_{1} \\
    x_{e} - \epsilon, & \text{for } 0 \in M_{2} \\
    x_{e}, & \text{otherwise}
  \end{cases}
$$

$$
  y'' = 
  \begin{cases}
    x_{e} - \epsilon, & \text{for } e \in M_{1} \\
    x_{e} + \epsilon, & \text{for } 0 \in M_{2} \\
    x_{e}, & \text{otherwise}
  \end{cases}
$$

where $\epsilon$ is the minimum of the the absolute difference between 0 or 1 from any $x_{e}$. Alternatively you could say it is just arbitrarily small, but defining as the above is more correct.

$y'$ and $y''$ looks like below:
![bipartite_integral_proof_even_cycle](/public/bipartite_integral_proof_even_cycle.jpeg)

Now, $$x' = \frac{y' + y''}{2}$$ hence $x'$ is NOT an extreme point and hence not a BFS. This is a contradiction.

Then consider the case where a cycle is not formed. Then the edges in F form a forest. Consider the example below.

![bipartite_integral_proof_odd](/public/bipartite_integral_proof_odd.jpeg)

This is a forest and the optimal solution is M2 with cardinality 3. Now, we can make a argument similar to the case where $F$ contains a cycle. The end result is that $x'$ is not an extreme point and once again we have a contradiction. 

Using a similar idea, we can prove that the dual problem (vertex cover) also has an optimal integral solution.

Now we turn our attention to figuring out what properties of LPs emit an optimal integral solution.

Before we do that though, we need to look at some matrices and Cramer's rule. 

## Unimodular Matrices

Unimodular matrices are square integer matrices with $\det = \pm 1$. In class we were given a stricter definition for non-square integer matrices. 

A matrix $A\in\mathbb{R}^{m\times n}$ with full-row rank is a unimodular matrix if the determinant of each basis of $A$ is $\pm 1$. This requires a little bit more attention so I'm going to skip it now and come back to it later (maybe).

## Cramer's Rule

Cramer's Rule states that the inverse of some square matrix,  $A^{-1}$, can be expressed as 

$$
    A^{-1} = \frac{\text{adj}(A)}{\det(A)}
$$

Where $adj(A)$ is the adjugate matrix $A$, formed by the transposition of the cofactor matrix $C$ of $A$.

The cofactor matrix is formed by the minor matrix times $-1$ to the power of $i+j$ i.e. $(-1)^{i+j}M_{i, j}$ at the $(i, j)$ location. $M_{i, j}$ is the minor, defined as the determinant of a smaller matrix of A where row $i$ and column $j$ has been removed.

The details are not too important here, the main idea here being that the $\text{adj}(A)$ of of a unimodular matrix $A$ is also integral. 

This means that the inverse is ALSO integral since unimodular matrices have a determinant of $\pm 1$.

## Totally Unimodular Matrices

Totally unimodular matrices are matrices $A\in\mathbb{R}^{m\times n}$ where every square submatrix of $A$ has $$\det(A_{\text{sub}})\in\{-1, 0, 1\}$$.

For example,

$$
\begin{bmatrix} 
  1 & -1 & 0 \\
  0 & 1 &  0 \\
  0 & 1 & -1    
\end{bmatrix} 
$$

## Integral Polyhedra

Assume that $b$ is integral in the LPs discussed below.

The following two statements hold. 

- $A\in\mathbb{R}^{m\times n}$ is unimodular IFF $$\mathcal{P} := \left\{x\in\mathbb{R}^{n} \;\rvert\; Ax=b, x\geq 0\right\}$$
- $A\in\mathbb{R}^{m\times n}$ is totally unimodular IFF $$\mathcal{P} := \left\{x\in\mathbb{R}^{n} \;\rvert\; Ax \leq b, x\geq 0\right\}$$

For (1), we have the following proof. 

Assume that $A$ is unimodular. If $x$ is a bfs, then we know from the Simplex algorithm that 

$$
\mathbf{x}_{B} = \mathbf{A}_{B}^{-1}\mathbf{b}
$$

since we are dealing with a polyhedron defined by the standard LP. $\mathbf{b}$ is integral by assumption, and $$\mathbf{A}_{B}^{-1}$$ is integral because of Cramer's rule.

It must be that $\mathbf{x}$ is integral as it is the product of 2 integers.

Assume that $\mathcal{P}$ is an integer polyhedron. Let $$\mathbf{b} = \mathbf{A}_{B}\mathbf{z} + \mathbf{e}_{i}$$, where $$\mathbf{e}_{i}$$ is the basis vector and $$\mathbf{z}$$ is an integer vector. $$\mathbf{b}$$ is also integer by assumption. 

$$
\begin{align}
    \mathbf{x}_{b} & = \mathbf{A}_{B}^{-1}\mathbf{b} \\
                   & = \mathbf{A}_{B}^{-1}\left[\mathbf{A}_{B}\mathbf{z} + \mathbf{e}_{i}\right] \\
                   & = \mathbf{z} + \mathbf{A}_{B}^{-1}\mathbf{e}_{i} \\
                   & \geq 0 
\end{align}
$$

as $$\mathbf{x}_{B}$$ is a basis vector. Everything in the above equation is integral by definition or assumption, and thus it must also be that $$\mathbf{A}_{B}^{-1}$$ is then integral. 

Since $AA^{-1} = I$, then $\det(A)\det(A^{-1}) = 1$. $$\mathbf{A}_{B}$$ is integral, thus either $$\mathbf{A}_{B}$$ and $$\mathbf{A}_{B}^{-1}$$ are both $1$ or $-1$. 

Hence $A$ must be unimodular. As we are picking arbitrary sets of basis indices to form $\mathbf{A}_{B}$.

## Equitable Bi-colouring of Matrix

A matrix $A\in\mathbb{R}^{m\times n}$ has an equitable bi-colouring if we can partition the columns or rows into 2 classes (e.g. green and blue) such that the absolute value of the difference between the sum of the two classes is less than or equal to one. 

Let $i\in C_{1}$ and $j\in C_{2}$ be the indices for the columns that belong to class 1 and 2 respectively. Then $C_{1}\cup C_{2} = \{1, 2, ..., n\}$ and $C_{1}\cap C_{2} = \varnothing$. Then the matrix has an equitable bi-colouring if

$$
\begin{align}
& \left\lvert \sum_{i \in C_{1}}a_{k, i} - \sum_{j \in C_{2}}a_{k, j} \right\rvert \leq 1  & \forall k\in\{1, 2, ..., m\}  
\end{align}
$$

It turns out that a matrix has a equitable bi-colouring if and only if it is a totally unimodular matrix. 

This is the reason why we can do row partitions for equitable bi-colouring - $A^{T}$ is also TUM.

For some problems, using equitable bi-colouring to prove that the constraint matrix is TUM might be easier than alternative approaches.

Let's look at the bipartite problem again.

We know that the polyhedron for this problem is $$\mathcal{P} := \left\{x\in\mathbb{R}^{\|E\|} \;\rvert\; \underset{e\in \delta(u)}{\sum} x_{e} \leq 1, x_{e} \geq 0, \forall u \in V\right\}$$.

For this problem, we use a row colouring and take the difference over each column instead of a column colouring!

Consider the example below. 

![bipartite_equitable_bicolouring](/public/bipartite_equitable_bicolouring.jpeg)

It's pretty easy to see from the diagram that if we colour any row that has the vertex belonging to the LHS one colour and RHS the other colour we can get an equitable bi-colouring as we sum across the columns.

Another example - flow circulation.

![network_flow_example](/public/network_flow_example.jpeg)

The polyhedron is defined as $$\mathcal{P} := \left\{x\in\mathbb{R}^{n} \;\biggl\lvert\; \underset{e\in f(u)}{\sum_{\text{in}}} x_{e} =  \underset{e\in f(u)}{\sum_{\text{out}}} x_{e}, x_{e} \geq 0, \forall u \in V\right\}$$.

It turns out that you simply colour every single row the same colour, and let the 1 and minus 1's do all the work. 

![network_flow_example_equitable_bicolouring](/public/network_flow_example_equitable_bicolouring.jpeg)

## Subset Systems

LPs with a TUM constraint matrix are not the characteristics that allow an integer optimal solution. In this section we consider a broad class of problems, the \textbf{subset selection problem}. 

These problems are defined by a subset system $(U, I)$ where $U$ is the universal set of all the individual items can could be selected, $I \subseteq 2^{u}$ is the feasible subset solutions of $U$ and finally for any $S \subset T \subseteq U$ if $T$ is feasible then it implies that $S$ is also feasible.

The general LP is therefore

$$
\begin{align}
    \max & \left[c\cdot x\right] \\
    \text{w.r.t} \; & S \in I \\
\end{align}
$$

Relating to the subset system is a rank function. 

A rank function is $r: 2^{U} \rightarrow \mathbb{Z}_{+}$ of the $(U, I)$ where

$$
\begin{align}
    r(S) = \underset{A\subseteq S: A\in I}{\max} |A|
\end{align}
$$

This can seem a bit confusing so let's do a quick example. Given a general graph suppose we want to find a maximum matching.

![matriod_bipartite_matching_example](/public/matriod_bipartite_matching_example.jpeg)

Given the diagram above, suppose that $S$ is the set of edges highlighted in blue. It is clear that we can't have $(v_{1}, v_{2})$ and $(v_{1}, v_{3})$ simultaneously, so in this case $\|A\|$ is 3. The rank function ensures that even if we add arbitrarily large number of elements in $U$ we continue to obey the feasibility governed by $I$.

Clearly, $S \in I$ IFF $r(S) = \|S\|$ and if $r(S) < \|S\|$ then $S$ is not feasible. This can be represented in an LP

$$
\begin{align}
    \max & \left[\sum_{j \in U}c_{j}x_{j}\right] \\
    \text{w.r.t}\; & \sum_{j \in S}x_{j} \leq r(S) & \forall S \subseteq U \\
    & x_{j} \geq 0 & \forall j \in U
\end{align}
$$

## Matroid Property

If a subset system has the matroid property, then for any 2 subsets $S, T \in I$, $\exists x\in T\setminus S$ s.t. $S + x \in I$. In other words, we can find an element in the larger set not in $S$ s.t, their union is still feasible. 

It turns out that subset systems with the matroid property have integral polyhedra. 

E.g. greedy algorithms

<p style="color: red">insert greedy algorithm section</p> 
