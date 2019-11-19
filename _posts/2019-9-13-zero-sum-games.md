---
layout: post
title: Zero-sum Games and the Minimax Algoritrhm
category: Theory
categories: ['game theory','linear programming', 'zero-sum', 'minimax']
tags: ['game theory','linear programming', 'zero-sum', 'minimax']
---

---
<h2 class="no_toc">Table of Contents</h2>

* TOC
{:toc}

## What is a zero-sum game?

Zero-sum games are how you lose friends. If you gain something, your friend will lose something.

So don't play these types of games with your friends =)

Alright, let's be a little more serious now.

In a  __zero-sum game__, there exists any number of players, a set of strategies for each player and a gain/loss that occurs given some choosen strategy by each player.

We encapsulate the gain/loss outcomes in a __payout matrix__ based on a chosen strategy.

Furthermore, a __zero-sum game__ has a payout matrix where the gains of one player to be exactly balanced out by the losses of the other players. This is why a zero-sum game is considered conflict game. 

For our purposes, we only consider 2 player zero-gum games. 

As we shall see, zero-sum games are really cool mathematical properties which is why they are interesting to study. 

## Scissors-paper-rock

Let's look at a straightforward zero-sum game for some intuition. In a game of scissors-paper-rock there are 2 players, $p_{1}, p_{2}$.

Let $P$ denote the payout matrix of $p_{1}$. Then the payout matrix for $p_{2}$ is $-P$, as consistent with the idea of a conflict game.

The payout matrix is

|   | S  | P  | R |
|:---:|:---:|:---:|:---:|
| S | 0  | 1  | -1|
| P | -1 | 0  | 1 |
| R | 1  | -1 | 0 |

Each player has 3 possible strategies, play $S :=$ scissors, $P :=$ paper or $R :=$ rock. The combination of the 2 player's chosen strategies at each iteration determine the payout. 

Because the payout matrix states all possible outcomes of the combination of strategies given by every player, it quickly balloons to something arbitrarily large if the game becomes more complicated.

Suppose there is a game where you have 2 strategies like below

|   | A  | B  | C  | D  |
|:---:|:---:|:---:|:---:|:---:|
| B |  2 |  2 |  4 |  5 |
| W | -2 | -2 | -4 | -5 | 

It turns out that you would always choose `B` because you no matter what your opponent chose, you have a better payout compared to choosing `W`. Then `B` is the __dominant strategy__.

This is pretty straight forward right?

## Taking Turns

So what happens when you're allowed to take turns? 

If you're playing scissors-paper-rock and you go first, then you'll always have a negative payout if your opponent chooses to maximise their own payout. You can see it if you look at the payout matrix. Every row in $P$ contains a negative payout.

In this case, no single strategy is winning for $p_{1}$.

So that sucks. As you might know from experience (or maybe from your statistics classes), that if you just randomly choose any strategy and have infinite iterations your expected payout will be zero in the long-term. 

In fact, for the game of scissors-paper-rock, we call this situation the mixed equilibria of the game. 

The purpose of this blog post is to show you that a __mixed equilibirum__ exists for all two-player zero-sum games and look at the __minimax algorithm__, which is a particular method of solving zero-sum games.

In fact, a mixed equilibrium exists for all zero-sum games, not just for 2 players. 

## Formalising Zero-sum Games

We define any 2 player zero-sum game as a tuple $\left(\mathcal{A}, \mathcal{B}, \mathcal{P} \right)$ where 

$$
\begin{align}
	\mathcal{A} & = \left\{a_{1}, a_{2}, a_{3}, ..., a_{n}\right\} \\
	\mathcal{B} & = \left\{b_{1}, b_{2}, b_{3}, ..., b_{m}\right\} \\
	P & \in \mathbb{R}^{m\times n}
\end{align}
$$

$\mathcal{A}$ and $\mathcal{B}$ are the two players and $P$ is the payout matrix as stated before. 

If each player selects one strategy $\(a_{i}, b_{j}\)$, then this is a tuple of __pure strategies__ that results in the payout $p_{ij}$. 

## Pure Equilibrium

Let's suppose that the game requires the players to take turns and once both players have chosen a strategy we ask them if they want to deviate from their strategy knowing their opponent's response. 

If both players have no incentive to deviate from the strategy (because their payout is not going to increase), then we say that the solution is a __pure equilibrium__. 

Let's look at a quick example of a game where pure equilibrium exists.

|   | A  | B  | C  | 
|:---:|:---:|:---:|:---:|
| A | 2  |  0 | -2 | 
| B | 3  |  0 |  1 |
| C | 1  | -1 |  2 |

If $p_{1}$ picks `A` then $p_{2}$ picks `C`, then if you ask $p_{1}$ whether they want to deviate, then they will say yes and ask to deviate to strategy `C`. But then if you ask $p_{2}$ if they want to deivate, they will ask to deviate to to strategy `B`.

Hence, `(A, C)` is not an equilibrium solution. Suppose $p_{1}$ picks `B`. Then $p_{2}$ picks `B`. You ask $p_{1}$ whether they want to deviate. Well, it turns out they don't need to! Changing to `A` provides the same utility and `C` is a negative payout. 

Therefore we say that `(B, B)` is the equilibrium solution.

Not all games have a pure equilibrium solution. Just consider scissors-paper-rock. You're always going to want to change your strategy after seeing what you're opponent does because there's a counter to all of their strategies. 

Mathematically, pure equilibrium is reached when the following constraints are satisfied. 

$$
\begin{align}
	p_{ij} & \geq p_{kj} \;\;\; \forall k = \{1, ..., n\} \\
	p_{ij} & \leq p_{ik} \;\;\; \forall k = \{1, ..., m\} \\
\end{align}
$$

Take a moment to make sure you understand the above inequalities. Suppose we choose $p_{2, 2}$. Then $p_{1, 2}$ and $p_{3, 2}$ is no larger than the payout for $p_{1}$. Conversely, the $p_{2, 1}$ and $p_{2, 3}$ have largest payouts and hence greater loss for $p_{2}$. Thus we have reached equilibrium. 

So okay, pure equilibrium does not exist for all zero-sum games.

## Mixed Equilibrium Always Exists for Zero-sum Games

We have looked zero-sum games that, when turned-based, can be a losing proposition for one player if their payout is negative for all pure strategies given that their opponent plays sensibly (i.e. chooses to maximise their gain). 

Now we argue that there always exists a mixed equilibrium for all two-player zero-sum games.

Note that just because a mixed equilibrium exists doesn't mean that both players come out equal - it could be that 1 player always loses no matter what mixed strategy they propose!

To create a mixed equilibrium, both players have to offer __mixed strategies__. A mixed strategy is simply randomising your single strategies proportionately over a large number of iterations. 

You can just think of it as applying a probability distribution over the possible strategies. 

For $\mathcal{A}$, we apply a distribution such that $p(a_{i})$ is the probability of choosing the pure strategy $a_{i}$. Since this is a probability, we have the constraint $\sum_{i=1} p(a_{i}) = 1$ for both players. 

Since we are playing the game a large number of times, we can compute the payout in the long run as an empirical expectation. 

let $$s_{A} := \left\{s_{A, i} \,\rvert\, i = 1, 2, ..., n \right\}$$ be the distribution of probabilities - i.e. the chance that $p_{1}$ picks the $$i^{\textit{th}}$$ strategy, and similarly for $s_{B}$. 

The expected payoff of mixed strategies is then 

$$
\begin{align}
	E\left[\text{Payout}\right] & = \sum_{i=1}^{n}\sum_{j=1}^{m}s_{A, i}s_{B, j}p_{ij}
\end{align}
$$

and similar to the pure equilibrium, we can reach a mixed equilibrium if

$$
\begin{align}
	\sum_{i=1}^{n}\sum_{j=1}^{m}s_{A, i}s_{B, j}p_{ij} & \leq \sum_{i=1}^{n}\sum_{j=1}^{m}s_{A, i}\hat{s}_{B, j}p_{ij} \;\;\; \forall \hat{s}_{B, j} \\
	\text{and} \\
	\sum_{i=1}^{n}\sum_{j=1}^{m}s_{A, i}s_{B, j}p_{ij} & \geq \sum_{i=1}^{n}\sum_{j=1}^{m}\hat{s}_{A, i}s_{B, j}p_{ij}
	\;\;\; \forall \hat{s}_{A, i}
\end{align} 
$$

Essentially what we are saying is that IF both players are rational and make sensible moves, the best way to play the game is to choose the distribution of strategies that ensures the maximum possible minimum payout no matter what your opponent does. This is sometimes called the __security level__.

This way of thinking is called the __minimax decision rule__,

Hence for $p_{1}$ it is

$$
	\underset{s_{A}}{\max}\underset{s_{B}}{\min}\sum_{i=1}^{n}\sum_{j=1}^{m}s_{A, i}s_{B, j}p_{ij} = \underset{s_{A}}{\max}\underset{j}{\min}\sum_{i=1}^{n}s_{A, i}p_{ij}
$$

Unsurprisingly, we can represent this problem as a LP and then show that the duality represents the gain of the opponent and thus their optimal payout is equal. 

The LP is

$$
\begin{align}
	\max \;\; & z \\
	\text{w.r.t} \;\; \sum_{i=1}^{n}s_{A, i}p_{ij} & \geq z \;\;\; j = 1, 2,..., m \\
	\sum_{i=1}^{n}s_{A, i} & = 1 \\
	s_{A, i} & \geq 0 \;\;\; i = 1, 2,..., n \\
	\text{z free}
\end{align}
$$

and the equivalent dual is

$$
\begin{align}
	\min \;\; & z' \\
	\text{w.r.t} \;\; \sum_{j=1}^{m}s_{B, j}p_{ij} & \leq z' \;\;\; i = 1, 2,..., n \\
	\sum_{j=1}^{m}s_{B, j} & = 1 \\
	s_{B, j} & \geq 0 \;\;\; j = 1, 2,..., m \\
	\text{z' free}
\end{align}
$$

This can be seen if you follow the construction of dual LPs on the [Wiki](https://en.wikipedia.org/wiki/Dual_linear_program#Constructing_the_dual_LP) page.

It turns out that the dual is the equivalent LP for 

$$
	\underset{s_{B}}{\min}\underset{s_{A}}{\max}\sum_{i=1}^{n}\sum_{j=1}^{m}s_{A, i}s_{B, j}p_{ij} = \underset{s_{B}}{\min}\underset{i}{\max}\sum_{j=1}^{m}s_{B, j}p_{ij}
$$

Hence by the Strong Duality theorem, 

$$
	\underset{s_{A}}{\max}\underset{s_{B}}{\min}\sum_{i=1}^{n}\sum_{j=1}^{m}s_{A, i}s_{B, j}p_{ij} = \underset{s_{B}}{\min}\underset{s_{A}}{\max}\sum_{i=1}^{n}\sum_{j=1}^{m}s_{A, i}s_{B, j}p_{ij}
$$

The maximum payout for $p_{1}$ after $p_{2}$ chooses a (mixed) strategy that minimises their loss is equivalent to the maximum payout $p_{2}$ can get after $p_{1}$ chooses a strategy that minimises their loss.

The reason why we have $\underset{s_{B}}{\min}$ on the LHS is because we are considering the payout matrix for $p_{1}$.

Finally, the __value__ of the game refers to payout at the mixed equilibrium and we say that the value of the game is well defined because the mixed equilibrium always exists.

## Minimax Theorem

You can condense this idea into the __Minimax theorem__ which states that

$$
\underset{s_{A}}{\max}\underset{s_{B}}{\min}\sum_{i=1}^{n}\sum_{j=1}^{m}s_{i}s_{j}p_{ij} = \underset{s_{B}}{\min}\underset{s_{A}}{\max}\sum_{i=1}^{n}\sum_{j=1}^{m}s_{i}s_{j}p_{ij}
$$

Alongside the minimax theorem there also exists a minimax algorithm, which is based on the same idea that we should choose strategies to minimise our losses given sensible play from the opponent. 

Let's have a look at the __minimax algorithm__.

## Minimax Algorithm

In the Minimax algorithm, we calculate n-moves ahead of the current state we are at and evaluate the new state. Then we backtrack up the algorithm, taking either the best outcome or the worst outcome depending on whose payout we are trying to maximise. 

The reason why we do this is because we are dealing with a turn-based game (similar to what we spoke about before) - if $p_{1}$ chooses some strategy then $p_{2}$ picks a response that minimises the pay out of $p_{1}$ (and thus maximises their own payout).

Let's have a look at an example. Looking at the tree diagram below, suppose our current state is the top node, and player 1 is to move.

Player 1 wants to maximise payout and conversely player 2 wants to minimise payout.

Our minimax algorithm calculates to a depth of 2 and returns the payout shown in the leaf nodes.

![minimax_algo_one](/public/minimax_algo1.JPG)

Now we begin backtracking.

Because player 2 will be the one making the decision, the leaf node with the lowest payout is chosen. The result is shown below.

![minimax_algo_two](/public/minimax_algo2.JPG)

We backtrack one more step, this time taking the maximum possible payout as it was player 1's move.

![minimax_algo_three](/public/minimax_algo3.JPG)

Hence we have an optimal move given our evaluation of the outcomes at $n=2$ depth.

__Alpha-beta Pruning__

Let's quickly talk about how we can prune the search tree with an idea called alpha-beta pruning. Essentially, we stop traversing a certain path if we discover that the evaluation is going to be worse than a previous path we've traversed. 

If you recall the example above, the diagram for which I've put below for easy reference,

![minimax_algo_two](/public/minimax_algo2.JPG)

we actually don't need to traverse the final node on the RHS since player 2 is guaranteed to pick at most 2, and more importantly on the LHS the optimal choice for player 1 is already 3, which is larger than 2.

Hence we don't need to look on that side any further. As you can imagine, we can get a good speedup if something like this occurs on a tree with huge depth.

## Conclusion

Minimax decision rule, minimax theorem, minimax algorithm.

The concept of minimax relies on players being rationale, and duality can be used to prove the existence of a mixed equilibrium for any zero-sum game. 

The most important thing you have learned today is that you don't play zero-sum games with your friends. 
