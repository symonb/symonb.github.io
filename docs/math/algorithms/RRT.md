<!-- ---
layout: default
title: RRT algorithm
permalink: /docs/math/algorithms/RRT
parent: algorithms
grand_parent: Math

--- -->

<!-- comment or image allows {: .no_toc} to work correctly  (don't ask me why) -->

[![image]()](){:class="img-responsive"}

{: .no_toc }

# Rapidly-exploring Random Tree (RRT)

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Intro
RRT is a powerful algorithm used for motion planning, especially in high-dimensional configuration spaces. Imagine you're trying to plan a path for a robot arm to reach a target object while avoiding obstacles. The configuration space represents all the possible positions and orientations of the arm's joints. RRT is designed to efficiently explore this space and find a feasible path from a starting configuration to a goal configuration.


## How Does RRT Work?

RRT builds a tree-like structure that gradually expands from the starting configuration. Here's a step-by-step breakdown of the algorithm:

Initialization: Start with a tree containing only the initial configuration (the starting point) as its root node.

Sampling: Randomly sample a configuration q_rand from the entire configuration space. This means generating a random set of joint angles (or whatever parameters define the configuration).

Nearest Neighbor Search: Find the node in the existing tree that is closest to the randomly sampled configuration q_rand. This is often done using a distance metric that considers the difference in joint angles or Cartesian positions. Let's call this nearest node q_near.

Steering (Extension): Try to "steer" or extend the tree from q_near towards q_rand. This is done by moving a small distance (a "step size" delta) from q_near in the direction of q_rand. The result is a new configuration q_new. The distance delta is a parameter of the algorithm, and it controls how far the tree extends in each step.

Collision Check: Before adding q_new to the tree, check if the path between q_near and q_new is collision-free. This means checking if the robot (or the object being manipulated) would collide with any obstacles along that path.

Adding to the Tree: If the path is collision-free, add q_new as a new node to the tree and create an edge connecting it to q_near.

Goal Check: Check if the new node q_new is close enough to the goal configuration. If it is, you've found a path!

Iteration: Repeat steps 2-7 until a path to the goal is found, or until a maximum number of iterations is reached.

Path Extraction: Once a node near the goal is found, trace back the path from that node to the root node (the starting configuration) to obtain the sequence of configurations that define the path.

## Key Concepts Explained Further:

Configuration Space: The set of all possible states of the robot or system being controlled. For example, for a robot arm, it's the set of all possible joint angle combinations.

Distance Metric: A function that measures the "distance" between two configurations. This is crucial for finding the nearest neighbor. The choice of distance metric can significantly impact the algorithm's performance.

Step Size (delta): The maximum distance the tree can extend in each iteration. A smaller step size can lead to smoother paths but may take longer to explore the space.

Collision Detection: A function that determines if a path or configuration is in collision with obstacles. This is a critical component of RRT.

Voronoi Bias: RRT implicitly favors exploration of large, unexplored regions of the configuration space. This is because nodes in these regions are likely to be far away from the existing tree, making them more likely to be selected as the nearest neighbor.

## Advantages of RRT:

**Simplicity:** Relatively easy to implement.

**Efficiency:** Works well in high-dimensional spaces where other planning algorithms struggle.

**Probabilistic Completeness:** Given enough time, RRT will eventually find a solution if one exists (but it doesn't guarantee finding the optimal solution).

**Handles Complex Constraints:** Can handle complex constraints and non-holonomic systems (systems with differential constraints).

## Disadvantages of RRT:

- **Suboptimal Paths:** RRT typically finds a feasible path, but it's not guaranteed to be the shortest or most efficient path.

- **Randomness:** The random sampling can lead to unpredictable performance. Sometimes it finds a solution quickly, other times it takes much longer.

**Narrow Passages:** Can struggle in environments with narrow passages or tight constraints.

**No Guarantee of Optimality:** RRT is a sampling-based algorithm, there is no guarantee that the path it finds is the optimal path.

## In summary:

RRT is a powerful and widely used motion planning algorithm that leverages random sampling to efficiently explore complex configuration spaces. It's particularly well-suited for problems with high dimensionality and complex constraints, although it may not always find the optimal solution. There are several improved versions of this algorithm, that take the weaknesses of RRT into account.

## Implementation

# RRT*

## Advantages

## Disadvantages

## Implementation