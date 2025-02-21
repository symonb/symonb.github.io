---
layout: default
title: Update of quaternion 
parent: Quaternions
grand_parent: Math
permalink: /docs/math/quaternions/quaternion_update
nav_order: 3
---

<!-- comment or image allows {: .no_toc} to work correctly  (don't ask me why) -->

{: .no_toc}

# Update of the quaternion

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Introduction 
This post explores the application of numerical methods for estimating the attitude (orientation) of a rigid body using quaternion representation. We build upon concepts introduced in previous posts on [quaternions as rotations](quaternions_as_rotation) and [numerical methods for ODEs](../numerical_methods/numerical_methods_for_ODEs).
  

In this post we will compare 2 numerical methods - Euler forward and trapezoidal method - in a simulated real-life scenario. The goal is to demonstrate the practical differences between these methods and to provide you with the tools to experiment with attitude estimation yourself. 
*Also, I wanted to add interactive plot to one of my posts and this is the perfect occasion.


# Euler method
The simplest quaternion update with forward Euler method:

$$\begin{gather}
  \mathbf{q}_{k+1} = \mathbf{q}_{k} + \Delta t \dot{\boldsymbol{q}}_k 
\end{gather}
$$

Using the quaternion derivative equation (derived in [quaternions derivative post](quaternions_derivative)): 

$$\begin{gather}
\dot{\boldsymbol{q}}_k=\frac{1}{2} \mathbf{q}_{k} \boldsymbol{\omega}_{b,k}\label{eq:eul_1}
\end{gather}
$$

where:

$\boldsymbol{\omega}_{b,k}$ is a pure quaternion representing the angular velocity of the body in the body frame at time step $k$. This is constructed from gyro measurements.  (A pure quaternion has a zero real part.)


We can rewrite the Euler update as:

$$\begin{gather}
  \mathbf{q}_{k+1} = \mathbf{q}_{k} + \Delta t \frac{1}{2} \mathbf{q}_{k} \boldsymbol{\omega}_{b,k}
\end{gather}
$$

{: .note}
Remember to normalize the quaternion $q_{k+1}$ after each update step to prevent it from drifting away from unit length. Quaternions must have a norm of 1 to represent rotations correctly.


# Trapezoidal method 
The classic trapezoidal method:

$$\begin{gather}
  \mathbf{q}_{k+1} =  \mathbf{q}_{k} +  \frac{1}{2} \Delta t (\dot{\boldsymbol{q}}_k + \dot{\boldsymbol{q}}_{k+1} ) \quad 
\end{gather}
$$

Substituting the quaternion derivative equation
$\ref{eq:eul_1})$:

$$\begin{gather}
  \mathbf{q}_{k+1} =  \mathbf{q}_{k} + \frac{1}{2}\Delta t  (\frac{1}{2} \mathbf{q}_{k} \boldsymbol{\omega}_{b,k} +\frac{1}{2} \mathbf{q}_{k+1} \boldsymbol{\omega}_{b,k+1} ) 
\end{gather}
$$

A key challenge here is that $\boldsymbol{\omega}_{b,k+1}$ (the future rotational speed) is generally unknown. We need to approximate it.

Several approaches can be used to estimate $\boldsymbol{\omega}_{b,k+1}$:

1.  **Rotational Acceleration Prediction:** Calculate or measure rotational acceleration and predict $\boldsymbol{\omega}_{b,k+1}$. This could involve using previous gyro measurements to estimate the acceleration and assuming it remains constant. However, this approach is **highly sensitive to noise**, especially with low-cost gyros.
  
2.  **Accelerometer-Based Angular Acceleration:** Measure linear acceleration with an accelerometer mounted off-center from the center of mass (CoM). After subtracting the **translational acceleration** (you need to estimate it separately) and **centripetal acceleration** (can be calculated with gyro measurements) with the knowledge of the offset from the CoM you can convert the remaining acceleration (**tangential acceleration**) to **angular acceleration**. Then you can estimate $\boldsymbol{\omega}_{b,k+1}$ assuming calculated acceleration. This method is also **susceptible to noise** and may be **overly complex** for simple applications like stabilizing a drone.

3.  **Constant Angular Velocity Assumption:** Assume that the angular velocity remains constant over the small time step: $$\boldsymbol{\omega}_{b,k+1}\approx \boldsymbol{\omega}_{b,k}$$. This seemingly naive assumption is often reasonable in practice because the time step $\delta t$ is typically very small (on the order of milliseconds). During such a short interval, the rotational speed may not change significantly.

Let's proceed with the constant angular velocity assumption ($$\boldsymbol{\omega}_{b,k+1} \approx \boldsymbol{\omega}_{b,k}$$). This simplifies the trapezoidal update to:

$$\begin{gather}
  \mathbf{q}_{k+1} =  \mathbf{q}_{k} + \frac{1}{2}\Delta t  (\frac{1}{2} \mathbf{q}_{k} \boldsymbol{\omega}_{b,k} +\frac{1}{2} \mathbf{q}_{k+1} \boldsymbol{\omega}_{b,k} ) 
\end{gather}
$$

Rearranging to solve for $q_{k+1}$:

$$\begin{gather}
\mathbf{q}_{k+1} - \frac{1}{4} \mathbf{q}_{k+1} \boldsymbol{\omega}_{b,k} =  \mathbf{q}_{k} + \frac{1}{4}\Delta t   \mathbf{q}_{k} \boldsymbol{\omega}_{b,k}
\end{gather}
$$

Finally, we can express $\boldsymbol{q}_{k+1}$ as: 

$$\begin{gather}
\mathbf{q}_{k+1} = \mathbf{q}_{k}  \left( \mathbf{1} + \frac{\Delta t}{4} \boldsymbol{\omega}_{b,k} \right) \left( \mathbf{1} - \frac{\Delta t}{4} \boldsymbol{\omega}_{b,k} \right)^{-1}
\end{gather}
$$

where:

*   $\boldsymbol{1}$ is the identity quaternion $[1, 0, 0, 0]^T$.
*   $\boldsymbol{\omega}_{b,k}$ is a pure quaternion constructed from the gyro measurements at time step $k$.
*   $$(\mathbf{1} - \frac{\Delta t}{4} \boldsymbol{\omega}_{b,k})^{-1}$$ is the inverse of the  quaternion $$\mathbf{1} - \frac{\Delta t}{4} \boldsymbol{\omega}_{b,k}$$.

# Comparison 

The following interactive plots allow you to compare the performance of the Euler forward and trapezoidal methods for a simulated attitude estimation scenario. 


<iframe src="{{ 'images/plot_3d.html' }}" width="800" height="600"></iframe>
<iframe src="{{ 'images/plot_2d_s.html' }}" width="800" height="600"></iframe>
# Code

Python code to play with those two methods 


# Summary 
As demonstrated, more advanced methods like the trapezoidal method generally provide more accurate results than simpler methods like the Euler forward method. However, the improvement in accuracy may not always be significant, especially for very small time steps or in situations where computational cost is a primary concern. The choice of method depends on the specific application and the trade-off between accuracy and efficiency. For systems that require long-term stability and precise attitude estimation, the trapezoidal method offers a valuable advantage.