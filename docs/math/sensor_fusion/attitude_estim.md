---
layout: default
title: Attitude estimation - gyro+acc+mag
permalink: /docs/math/sensor_fusion/attitude_estim_theory
nav_order: 3
parent: Sensor Fusion
grand_parent: Math
toc: true
---

<!-- comment or image allows {: .no_toc} to work correctly  (don't ask me why) -->

{: .no_toc}

# Attitude estimation with accelerometer&gyro and magnetometer

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Intro

One of the most important information about the mobile robot is its attitude and position. For UAVs especially multi-rotors, the first one is essential for even taking off the ground. Knowing your current orientation (in relation to the Earth) is one of the foundations of most modern systems. However, no sensor measures our attitude directly and we need to use a combination of the few other sensors to get it.

# Attitude from gyro

A gyroscope measures the angular speeds and can be integrated to achieve orientation with respect to some starting point. But because of the drift, it quickly becomes unreliable.

# Attitude from accelerometer or magnetometer

Accelerometer measurements allow us to determine the absolute attitude (in 2 axes) however its measurements are vulnerable to noise and they can not be used without filtering which introduces a notable delay which for UAVs' application becomes unacceptable and accelerometer can not be used alone.

Sensor fusion of these allows us to take their benefits and reduce defects. In this post, we will discuss 3 methods for combining acc and gyro measurements to achieve absolute pitch and roll estimation. A magnetometer can be additionally used to get yaw value (absolute value) but in the basic case (acc&gyro) we will focus on pitch and roll.

# Complementary filter

The most simple version of combining data.

$$
roll = (1-\alpha)roll_{g} + \alpha roll_{a}


$$

## Quaternion version

# Mahony filter

## Quaternion version

# Magdwick filter

## Quaternion version

# Summary
