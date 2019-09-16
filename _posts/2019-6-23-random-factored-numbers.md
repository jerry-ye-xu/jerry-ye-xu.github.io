---
layout: post
title: Generating Prime Factored Numbers
category: Theory
categories: ['randomised algorithms', 'prime numbers']
tags: ['randomised algorithms', 'prime numbers']
---

---
<h2 class="no_toc">Table of Contents</h2>

* TOC
{:toc}

<!-- Need this for table of contents above -->
---

## Prime factored numbers

Today we're going to look at generating random numbers as a product of its prime factors. 

I came across this one page gem in an algorithms class and I thought it was so cool, especially because it generates numbers uniformly.

As you might know, simulating random numbers on a computer is not actually random - there is a deterministic algorithm under the hood. 

Now, it is trivial to run `np.random.randint()` in Python to generate a number but in this case we want to know it's prime factors as well.

One could generate a random number and try to prime factorise it, but it turns out that this is a NP-hard problem, i.e. no polynomial time algorithm is known.

However, there does exist an efficient algorithm to test whether a number is prime.

Take care to note the difference.

We cannot generate a number and then factorise it in a reasonable time, so instead we will generate numbers, test their primality and take product of the ones that are prime to achieve our goal. 

Let's look at the algorithm. It is deceptively simple.

## The algorithm

Let $N$ be the upper bound integer from which to sample random numbers $s^{\*} \in \mathbb{N^{+}}$. 

Denote the set of prime factors as $S_{j} = \left\\{ s_{i, j} : i = 1, 2, ... k \right\\}$, where $k$ is the size of $S$ at the $j^{th}$ iteration.

For each iteration:

1. Generate a random sequence $s_{i}, i = 1, 2, ... k$ starting with $s_{i} \in \left\\{1, 2, ..., N \right\\}$, continuing with $s_{i+1} \in \left\\{1, 2, ..., s_{i} \right\\}$ until we sample $s_{k} = 1$. 

2. We take the product of primes $s^{\*} = \prod_{s \in S_{G}}s_{i}$, where $S_{G} \subseteq S$ is the subset of generated numbers. 

3. Output $s^{\*}$ with probability $\frac{s^{\*}}{N}$ if $s^{\*} \leq N$.

Notice that without loss of generality we state that $N \geq s_{1} \geq s_{2} \cdots \geq s_{k} = 1$.  


## Probabilistic justification 

Let's look at the probability of generating multiple $N$'s. It turns out that a single $N$ occurs with probability $\frac{1}{N}$, and multiple $N$'s occur with probability $\frac{1}{N^{m}}\left\(1-\frac{1}{N}\right\)$. The $1-\frac{1}{N}$ represents the moment when $N$ is not chosen, and from then on $N$ can never be chosen again because now we have a smaller sequence to work with. 

In that sense,

$$ 
\begin{align}
	s^{*} & = \prod_{s\in S_{G}}s_{i}^{m_{i}} \\\\
\end{align}
$$ 

where $m_{i}$ is the number of times $s_{i}$ appears. 

Similarly all other chosen numbers $s_{i}$ has probability of being chosen $\frac{1}{s_{i}^{m}}\left(1-\frac{1}{s_{i}}\right)$. 

After some thinking, we realise that the probability of generating these numbers are independent, and thus 

$$ 
\begin{align}
P\left[s_{i}\;\text{occurs $k$ times}\right] & = \prod_{s\in S_{G}}\frac{1}{s_{i}^{m_{i}}}{\left(1-\frac{1}{s_{i}}\right)} \\\\
	& = \frac{1}{s^{*}}\prod_{s\in S_{G}}{\left(1-\frac{1}{s_{i}}\right)} 
\end{align}
$$

Hence the total probability of outputting a number given the entire algorithm is 

$$
\begin{align}
	P\left[\text{outputting} \; s^{*}\right] & = \left(\frac{s^{*}}{N}\right)\left(\frac{1}{s^{*}}\right)\prod_{s\in S_{G}}{\left(1-\frac{1}{s_{i}}\right)} \\\\
		& = \frac{1}{N}\prod_{s\in S_{G}}{\left(1-\frac{1}{s_{i}}\right)} \\\\
\end{align}
$$

Now, we use rejection sampling in step 3 because we want the probability of outputting $s^{\*}$ to be uniform. 

## Time complexity

To understand the time complexity, we look at 2 components. First we need to test each $s_{i}$ for primality, and the length of the sequence also matters. Then we need to know how often this algorithm is going to restart.

Suppose we somehow ended up choosing every number in the sequence, then the probability of choosing each number for the first time is $\frac{1}{s_{i}}$. Hence the number of primality tests needed is

$$
\begin{align}
	O(\log N) & = 1 + \frac{1}{2} + \frac{1}{3} + ... + \frac{1}{N}
\end{align}
$$

Now, we look at the expected number of restarts. First of all, the probability of rejection is 

$$
\begin{align}
	P\left[\text{reject}\right] & = 1 - \sum_{s\in S} P\left[\text{outputting all posssible} \; s^{*}\right]\\\\
		& = 1 - \prod_{s\in S}\left(1-\frac{1}{s_{i}}\right)
\end{align}
$$

Because the output of each $s^{\*}$ is independent, we sum the probabilities. And it turns out that

$$
\begin{align}
	E\left[\text{# of rejections}\right] & = \frac{1}{\prod_{s\in S}\left(1-\frac{1}{s_{i}}\right)} \\\\
			& = O(\log N)
\end{align}
$$

by [Merten's 3rd Theorem]("https://en.wikipedia.org/wiki/Mertens%27_theorems#In_number_theory"). Hence the expected number of restarts is $O(\log N)$.

Hence the running time is $O(\log^{2}N\cdot\text{primality test})$.

Since we're assuming that the primality test is polynomial time, we are done. 

---

## Aftermath

I stumbled across the __Prime Number Theorem (PNT)__ which describes the distribution of prime numbers asymptotically as $N$ approaches infinity. 

I found this Khan academy [video](https://www.khanacademy.org/computing/computer-science/cryptography/comp-number-theory/v/prime-number-theorem-the-density-of-primes) intuitive for understanding the PNT. 

Now I'm curious as to how `np.random.randint()` is implemented =). That might a discussion for another post. 