---
layout: post
title: The Simplex Algorithm - Part 2
category: Theory
categories: ['simplex','linear programming', 'optimisation']
tags: ['simplex','linear programming', 'optimisation']
---

---
<h2 class="no_toc">Table of Contents</h2>

* TOC
{:toc}

## Recap

In part one of the `Simplex Algorithm` series, we looked at formulating a linear program (LP) and understanding their properties. 

If you haven't read part one yet, please start there as the content will be relevant when building the Simplex algorithm. 

Let's have a look at the Simplex algorithm that assumes a few constraints. 

## The Main Ideas of Simplex 

{% include pseudocode.html id="1" code="
\begin{algorithm}
\caption{Simplex with Constraints}
\begin{algorithmic}
\REQUIRE An initial feasible basis $\mathbf{B}$
    \WHILE{basis $\mathbf{B}$ is changing}
        \FOR{$j \in \{1, 2, ..., n\}$}
            \STATE $\mathbf{D} \gets \mathbf{B} + j - b_{i}$
            \STATE $\mathbf{x}_{B} = \mathbf{A}^{-1}_{B}b$ \\
            
            \COMMENT{If cost of non-degenerate solution is lower assign the new basis}
            \IF{$\mathbf{D}$ is a basis of $\mathbf{A}$}
                \STATE $\mathbf{y}_{D} = \mathbf{A}_{D}^{-1}b$
                \IF{$\mathbf{y}_{D} \geq 0$ and $\mathbf{c}_{D}\mathbf{y}_{D} \leq \mathbf{c}_{B}\mathbf{x}_{B}$}
                    \STATE $\mathbf{B} = \mathbf{D}$
                \ENDIF
            \STATE \textbf{break for}
            \ENDIF
        \ENDFOR 
    \ENDWHILE
\end{algorithmic}
\end{algorithm}
" %}

This algorithm makes a few assumptions that we will tackle throughout this post. That is,

1. An initial feasible basis is given.
2. Every basic feasible solution is non-degenerate.
3. The feasible region is bounded. 

We will relax them one by one, but let's not rush into it. 

First we need to understand how Simplex moves from one basic feasible solution (BFS) to another. 

Let's set up the algebra. 

## Setting up the Vector Equation

Recall a linear program in standard form is defined as

$$
\begin{align}
    \text{min}\left(\textbf{c}\cdot \textbf{x}\right)
\end{align}
$$

$$
\begin{align}
    \textbf{A}\textbf{x} & = \textbf{b} \\
        \textbf{x} & \geq 0
\end{align}
$$

Suppose we start at a BFS. We have chosen a basis (for the LP) $\mathbf{B}$ that represents a set of basic indices for the columns of the constraint matrix $$\mathbf{A}$$. 

The resulting matrix is called a basis matrix $$\mathbf{A}_{B}$$, with the corresponding vector $\mathbf{x}_{B}$. 

From the previous post, the rest of the $\(n-m\)$ $x$ variables are set to zero. 

Let us represent the movement in space with the equation

\begin{align}
    \mathbf{y} & = \mathbf{x} + \theta\mathbf{d}    
\end{align}

where $\mathbf{d}$ is the $j^{\textit{th}}$ basic direction. 

Since we are moving in the $j^{\textit{th}}$ direction, we can normalise $d_{j} = 1$ and have the remaining non-basic indices set to zero. 

The $\theta$ controls how far we move in the direction of $\textbf{d}$, and if the polyhedron is unbounded in that particular direction then $\theta = \infty$ is possible.

This will occur when $\mathbf{d}_{B} \geq 0$, which will allow for $$\mathbf{y}_{B} = \mathbf{x}_{B} + \theta\mathbf{d}_{B} \geq 0$$ to be a basic solution $\forall \theta \geq 0$ because $$\mathbf{y}_{B}$$ is a valid BFS. 

For now, we assume that $\theta$ is strictly positive and thus for the polyhedron to be bounded $\mathbf{d}_{B} < 0$. We will discuss the value of $\theta$ later.

If the polyhedron is bounded, we want to go as far as possible to hit the next vertex, taking care that we actually remain inside the polyhedron (this is important if degenerate solutions lurk around). 

Since we are moving to another BFS, 

$$
\begin{align}
    \mathbf{A}\mathbf{y} & = \mathbf{A}\left(\mathbf{x} + \theta\mathbf{d}\right) \\
        & = \mathbf{b} \\
    \implies \theta\mathbf{A}\mathbf{d} & = \mathbf{A}\mathbf{y} - \mathbf{A}\mathbf{x} \\
        & = \mathbf{0}
\end{align}
$$

since both have tight constraints and have a solution equal to $\mathbf{b}$. Recall that we defined $\theta > 0$ which gives us


$$
\begin{align}
    \mathbf{A}\mathbf{d} & = 0 \\
        & = \sum_{i = 1}^{n} \mathit{A_{i}}d_{i} \\
        & = \sum_{i = 1}^{m} \mathit{A}_{b(i)}d_{b(i)} + A_{j}d_{j}  \\
        & = \mathbf{A}_{B}\mathbf{d}_{B} + \mathit{A}_{j} \\
    \implies \mathbf{d}_{B} & = -\mathbf{A}_{B}^{-1}\mathit{A}_{j}
\end{align}
$$

Where $\mathit{A}_{b(i)}$ is the $b(i)^{\textit{th}}$ column of $\mathbf{A}$.

So how do we calculate $\theta$? The idea is that since the new BFS, $\mathbf{y} > 0$, is non-degenerate by our constraints, we have

$$
\begin{align}
    0 & < \mathbf{A}\mathbf{y} \\
        & = \mathbf{A}\left(\mathbf{x} + \theta\mathbf{d}\right) 
\end{align}
$$

As mentioned before if $d_{i} \geq 0$ then $\theta$ can be any positive value, whereas if $d_{i} < 0$ then if $\theta$ is a vector, then we would want

$$
\begin{align}
    \theta_{i} \leq -\frac{x_{i}}{d_{i}}
\end{align}
$$

and we have to be careful here, because we don't have $\theta_{i}$ as $\theta$ is actually a scalar! Hence we require the value $\theta$ such that the inequality holds for all $d_{i} < 0$. Hence we take the minimum

$$
\begin{align}
    \theta & = \min\left[-\frac{x_{i}}{d_{i}}\right]
\end{align}
$$

where $i \in B$ and $d_{i} < 0$.

Note: There are 3 cases of $x_{i}$ remember? $x_{i}$ is a basic variable and is either non-zero (non-degenerate) or zero (degenerate), , not a basic variable and thus is set to zero or is negative when we solve the constraint and thus gives a feasible solution instead of a basic feasible solution.

Note2: There are also 3 cases for $d_{i}$. Either $d_{i}$ does not belong to the basis, or it does, or it does not belong and is the $j^{\textit{th}}$ direction.  

Note3: Here is where we can see $\theta$ becoming zero when degenerate solutions (i.e. $x_{b(i)} = 0$) arise, which then sets $\theta$ to zero.

## Deriving the Cost Function

Let's try and calculate the difference in cost between the 2 BFS.

$$
\begin{align}
    \mathbf{c}\cdot\mathbf{y} - \mathbf{c}\cdot\mathbf{x} & = \sum_{j\notin\mathbf{B}}\left[c_{j}y_{j}\right] + \mathbf{c}_{B}\cdot\mathbf{y}_{B} - \mathbf{c}_{B}\cdot\mathbf{x}_{B} \\
        & = \sum_{j\notin\mathbf{B}}\left[c_{j}\theta d_{j}\right] + \mathbf{c}_{B}\cdot\left(\theta\mathbf{d}_{B}\right) \\
        & = \sum_{j\notin\mathbf{B}}\left[\theta c_{j} + \theta\left(\mathbf{c}_{B}\cdot -\mathbf{A}_{B}^{-1}\mathit{A}_{j}\right)\right] \\
        & = \sum_{j\notin\mathbf{B}}\left[\theta\left(c_{j} - \mathbf{c}_{B}^{T}\mathbf{A}_{B}^{-1}\mathit{A}_{j}\right)\right] \\
        \text{and we define} \\
        \bar{c_{j}} & = c_{j} - \mathbf{c}_{B}^{T}\mathbf{A}_{B}^{-1}\mathit{A}_{j}
\end{align}
$$

We refer to this as the reduced cost of index $\textit{j}$ w.r.t to the basis $\mathbf{B}$. If $\mathbf{y}$ is a better BFS, then $\bar{c_{j}} < 0$.

Why is this the case? Let's consider the possibilities of elements in $\mathbf{x}$ and $\mathbf{y}$. If we are comparing basis indices, then $j = b(i)$

$$
\begin{align}
    c_{b(i)} - \mathbf{c}_{B}\mathbf{A}_{B}^{-1}\mathit{A}_{b(i)} & = c_{b(i)} - \mathbf{c}_{B}e_{i} \\
        & = c_{b(i)} - c_{b(i)} \\
        & = 0
\end{align}
$$

If we are not comparing basis indices, then $x_{j} = 0$ because $\mathbf{x}$ is a BFS and $y_{j} 
\geq 0$ because the incoming solution is also basic feasible. 

Hence for an arbitary BFS if the cost does not decrease, then it is safe to say, assuming non-degenerate BFS, that we are at the optimal.

## Degenerate BFSs

What happens when there are degenerate solutions? Well it's actually not too bad. Cycling through basis indices that land on the same vertex is okay, because you may be able to find a better solution.

However, there will exist cases where you might end up cycling through a set of BFS indefinitely. 

We will not discuss the __pivoting rules__ which will prevent anti-cycling, but in short we use Bland's rule for pivot selection and we are good. 

If you are itching for a proof, check out Professor Williamson's notes [here]("https://people.orie.cornell.edu/dpw/orie6300/Lectures/lec13.pdf").

## Finding an Initial Basis

The Simplex algorithm described above requires an initial basis.

To find an initial basis, we solve the following auxiliary program

$$
\begin{align}
    \min\left[\sum_{i=1}^{n}y_{i}\right]
\end{align}
$$

subject to

$$
\begin{align}
    \mathbf{A}\mathbf{x} + \mathbf{y} & = \mathbf{b} \\
    x & \geq 0 \\
    y & \geq 0 \\
\end{align}
$$

Initialising this problem is very easy - just let $\mathbf{x} = 0$ and $\mathbf{y} = \mathbf{b}$. When we solve this problem, if the cost if strictly positive it means that we could not find a feasible solution $\mathbf{x}$ such that $\mathbf{A}\mathbf{x} = \mathbf{b}$ with $\mathbf{y} = 0$. Now we have a BFS.

Unfortunately, this is not enough because we now have a basis in this auxiliary program that consists the indices of both $\mathbf{x}$ and $\mathbf{y}$.

If the solution consists of only basis indices that belong to $\mathbf{A}$ i.e. does not use any variables from $\mathbf{y}$ then we are good. Otherwise we take the subset of columns that correspond to the indices of $\mathbf{x}$ and apply the __basis extension theorem__. 

## The Full Simplex Algorithm

Alright. I think we are ready to introduce the Simplex algorithm.

{% include pseudocode.html id="2" code="
\begin{algorithm}
\caption{Simplex}
\begin{algorithmic}
\REQUIRE Solve auxiliary LP for an initial feasible basis $\mathbf{B}$ or return infeasible.
    \WHILE{basis $\mathbf{B}$ is changing}
        \STATE $\mathbf{x}_{B} = \mathbf{A}^{-1}_{B}b$
        \STATE $\bar{c_{j}} = c_{j} - \mathbf{c}_{B}^{T}\mathbf{A}_{B}^{-1}\mathit{A}_{j}$
        \IF{$\exists j$ such that $\bar{c}_{j} < 0 $}
            \IF{$ \mathbf{d}_{B} \geq 0 $}
                \RETURN \"unbounded objective\"
            \ELSE
                \STATE $\mathbf{D} \gets \mathbf{B} + j - b_{i}$
                \COMMENT{We take out the basis variable $b(i)$ that minimises $\theta = -\frac{x_{b(i)}}{d_{b(i)}}$ }
            \ENDIF
        \ENDIF
    \ENDWHILE
\end{algorithmic}
\end{algorithm}
" %}

<!-- {% include pseudocode.html id="2" code="
\begin{algorithm}
\caption{Simplex with Constraints}
\begin{algorithmic}
\REQUIRE Solve auxiliary LP for an initial feasible basis $\mathbf{B}$ or return infeasible.
    \WHILE{basis $\mathbf{B}$ is changing}
        \STATE $\mathbf{x}_{B} = \mathbf{A}^{-1}_{B}b$
        \IF{$\exists j$ such that $\bar{c}_{j} < 0 $}
            \IF{$ \mathbf{d}_{B} \geq 0 $}
                \RETURN \"unbounded objective\"
            \ELSE
                \COMMENT{If cost of non-degenerate solution is lower assign the new basis} \\
                \STATE $\mathbf{D} \gets \mathbf{B} + j - b_{i}$
                \COMMENT{This time, we take out the basis variable $x_{b_{i}}}$}|
            \ENDIF
        \ENDIF
            \IF{$\mathbf{D}$ is a basis of $\mathbf{A}$}
                \STATE $\mathbf{y}_{D} = \mathbf{A}_{D}^{-1}b$
                \IF{$\mathbf{y}_{D} \geq 0$ and $\mathbf{c}_{D}\mathbf{y}_{D} \leq \mathbf{c}_{B}\mathbf{x}_{B}$}
                    \STATE $\mathbf{B} = \mathbf{D}$
                \ENDIF
            \STATE \textbf{break for}
            \ENDIF
    \ENDWHILE
\end{algorithmic}
\end{algorithm}
" %} -->

## Time Complexity

Each iteration is polynomial time, but the number of iterations can be exponential time for specific LPs. 

Proving time complexity requires us to find a pivot rule that takes at most polynomial iterations to find the optimal solution, or to prove that such a pivotal rule does not exist.

This is currently an open problem. 

Despite this, in practice the Simplex generally works well and doesn't run into complexity issues. 

## Conclusion

Congratulations, you've made it all the way to the end. 

That was much harder than 
```python
print("hello world")
```
wasn't it =(. 

There's alot more detail involved, but this is a nice overall of the main ideas.

If you do decide to dig deeper you will encounter an even cooler concept called __duality__ - you may have heard of this when learning about network flows in an algorithms course. Indeed, we can use the duality of linear programs to mathematically justify why the `min-cut max-flow` theorem holds. 

That's for another day though. 

I hope this post has piqued your curiosity to learn more about optimisation!