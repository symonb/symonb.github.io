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
<iframe src="{{ 'images/plot_error_vs_step.html' }}" width="800" height="600"></iframe>
# Code

Python implementation of above methods:

```python
import numpy as np
import quaternion  # Install using: pip install numpy-quaternion
from scipy.spatial.transform import Rotation as R

def simulate_gyro_measurements(duration, delta_t, angular_velocity_body):

    num_steps = int(duration / delta_t)+1  # Calculate the number of simulation steps
    gyro_measurements_pure_quat = []  # Initialize an empty list to store gyroscope measurements
    for _ in range(num_steps):  # Iterate over each time point
        # Create a pure quaternion from the angular velocity vector
        q_omega = quaternion.quaternion(0, angular_velocity_body[0], angular_velocity_body[1], angular_velocity_body[2])
        gyro_measurements_pure_quat.append(q_omega)  # Append the pure quaternion to the list

    return np.array(gyro_measurements_pure_quat)  # Convert the list to a NumPy array and return it

def propagate_attitude_quaternion_euler(initial_quaternion, gyro_measurements_pure_quat, delta_t):

    current_quaternion = initial_quaternion  # Set the current quaternion to the initial quaternion
    for q_omega in gyro_measurements_pure_quat[1:]:  # Iterate over each gyroscope measurement (excluding the first, which is the initial state)
        # Update the quaternion using the Euler forward method
        q_update = current_quaternion + 0.5 * delta_t * (current_quaternion * q_omega)
        # Normalize the updated quaternion to prevent drift
        q_update_normalized = q_update / np.abs(q_update)
        current_quaternion = q_update_normalized  # Update the current quaternion for the next iteration

    return current_quaternion

def propagate_attitude_quaternion_trapezoidal(initial_quaternion, gyro_measurements_pure_quat, delta_t):
    
    current_quaternion = initial_quaternion  # Set the current quaternion to the initial quaternion
    for q_omega in gyro_measurements_pure_quat[1:]:  # Iterate over each gyroscope measurement (excluding the first, which is the initial state)
        # Define identity quaternion
        identity_quat = quaternion.quaternion(1, 0, 0, 0)
        # Calculate terms for the Trapezoidal method
        term1 = identity_quat + 0.25 * delta_t * q_omega
        term2 = identity_quat - 0.25 * delta_t * q_omega
        term2_inv = term2.inverse()  # Calculate the inverse of term2
        # Update the quaternion using the Trapezoidal method
        q_update = current_quaternion * (term1 * term2_inv)
        # Normalize the updated quaternion to prevent drift
        q_update_normalized = q_update / np.abs(q_update)
        current_quaternion = q_update_normalized  # Update the current quaternion for the next iteration
    
    return current_quaternion

def analytical_attitude_quaternion(time, angular_velocity_body, initial_quaternion):

    # Calculate the rotation angle
    angle = np.linalg.norm(angular_velocity_body) * time
    # Calculate the rotation axis
    if angle > 1e-6:  # Avoid division by zero if angular velocity is very small
        axis = angular_velocity_body / np.linalg.norm(angular_velocity_body)
    else:
        axis = np.array([1, 0, 0])  # Arbitrary axis if no rotation
    # Create a quaternion from the angle-axis representation
    rotation_quat = quaternion.from_rotation_vector(angle * axis)
    # Apply the rotation to the initial quaternion
    analytic_quat = rotation_quat * initial_quaternion
    
    return analytic_quat

def quaternion_to_euler(q):
    """Converts a quaternion to Euler angles (roll, pitch, yaw) in degrees."""
    # Create a Rotation object from the quaternion
    r = R.from_quat([q.x, q.y, q.z, q.w])
    # Convert to Euler angles (in radians)
    roll, pitch, yaw = r.as_euler('xyz', degrees=True)
    return [roll, pitch, yaw]


if __name__ == "__main__":
    # Simulation Parameters
    duration = 10.0  # Total simulation time [s]
    angular_velocity_body = np.array([360,180,90])  # Angular velocity in degrees per second
    initial_quaternion = quaternion.quaternion(1, 0, 0, 0)  # Initial quaternion (no rotation)
    # Define delta_t value (step size)
    delta_t = 0.05
    
    # Convert angular velocity to radians per second
    angular_velocity_body_rad = np.radians(angular_velocity_body)
    # Simulate Gyroscope Measurements
    gyro_measurements_pure_quat = simulate_gyro_measurements(duration, delta_t, angular_velocity_body_rad)
    # Propagate Attitude Quaternion using Euler and Trapezoidal methods
    attitude_quaternions_euler = propagate_attitude_quaternion_euler(initial_quaternion, gyro_measurements_pure_quat, delta_t)
    attitude_quaternions_trapezoidal = propagate_attitude_quaternion_trapezoidal(initial_quaternion, gyro_measurements_pure_quat, delta_t)
    # Calculate Analytical Attitude (Ground Truth)
    analytical_quaternions = analytical_attitude_quaternion(duration, angular_velocity_body_rad, initial_quaternion)
    # Convert quaternions to Euler angles
    euler_angles_euler = quaternion_to_euler(attitude_quaternions_euler)
    euler_angles_trapezoidal = quaternion_to_euler(attitude_quaternions_trapezoidal)
    euler_angles_analytical = quaternion_to_euler(analytical_quaternions)


    print(f'\n\tNumerical methods comparison\n\nSETUP:\n'
          f'delta_t = {delta_t}\n'
          f'duration= {duration}\n'
          f'angular velocities: w_roll:{angular_velocity_body[0]:.1f} w_pitch:{angular_velocity_body[1]:.1f} w_yaw:{angular_velocity_body[2]:.1f} [deg/s]\n\n'
          f'RESULTS:\n'
          f'Quaternion:\n'
          f'Euler forward method: w:{attitude_quaternions_euler.w:.3f} i:{attitude_quaternions_euler.x:.3f} j:{attitude_quaternions_euler.y:.3f} k:{attitude_quaternions_euler.z:.3f}\n'
          f'Trapezoidal method:   w:{attitude_quaternions_trapezoidal.w:.3f} i:{attitude_quaternions_trapezoidal.x:.3f} j:{attitude_quaternions_trapezoidal.y:.3f} k:{attitude_quaternions_trapezoidal.z:.3f}\n'
          f'Analitical:           w:{analytical_quaternions.w:.3f} i:{analytical_quaternions.x:.3f} j:{analytical_quaternions.y:.3f} k:{analytical_quaternions.z:.3f} \n'
          f'\nEuler Angles:\n'
          f'Euler forward method:       roll:{euler_angles_euler[0]:.1f} pitch:{euler_angles_euler[1]:.1f} yaw:{euler_angles_euler[2]:.1f} [deg]\n'
          f'Trapezoidal forward method: roll:{euler_angles_trapezoidal[0]:.1f} pitch:{euler_angles_trapezoidal[1]:.1f} yaw:{euler_angles_trapezoidal[2]:.1f} [deg]\n'
          f'Analitical:                 roll:{euler_angles_analytical[0]:.1f} pitch:{euler_angles_analytical[1]:.1f} yaw:{euler_angles_analytical[2]:.1f} [deg]')
```


# Summary 
As demonstrated, more advanced methods like the trapezoidal method generally provide more accurate results than simpler methods like the Euler forward method. However, the improvement in accuracy may not always be significant, especially for very small time steps or in situations where computational cost is a primary concern. The choice of method depends on the specific application and the trade-off between accuracy and efficiency. For systems that require long-term stability and precise attitude estimation, the trapezoidal method offers a valuable advantage.