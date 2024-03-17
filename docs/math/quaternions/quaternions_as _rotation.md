---
layout: default
title: Quaternions as rotation
parent: Quaternions
grand_parent: Math
permalink: /docs/math/quaternions/quaternions_as_rotation
nav_order: 3
---


<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Quaternions as represantetion of the rotation

Just as a unit circle in the complex plane can be used to describe a rotation in one axis, quaternions using 3 imaginary numbers describe a rotation in 3 dimensions.

According to [Euler's rotation theorem](https://en.wikipedia.org/wiki/Euler%27s_rotation_theorem), any complex rotation of any vector $\mathbf{p}\in \mathbb{R}^3$ about any axes can be replaced by a single rotation about the $\mathbf{v}=[v_x,v_y,v_z] $ by an angle $\theta$. Using quaternions, the rotation of a vector $\mathbf{p}$ can be represented by multiplication:

$$
\begin{equation}\mathbf{p}'=\mathbf{q}\mathbf{p}\mathbf{q}^{-1}=\mathbf{q}\mathbf{p}\mathbf{q}^*\label{eq:rotation}\end{equation}
$$

where vector $\mathbf{p}$ is treated as a pure quaternion - $\mathbf{p}= [0,p_x,p_y,p_z]$, and quaternion $\mathbf{q}$ in trigonometric form is:

$$
\begin{equation}    \mathbf{q}=\cos{\frac{\theta}{2}}+[v_xi,v_yj,v_zk]\sin{\frac{\theta}{2}}\label{eq:trigonometric_form_long}\end{equation}
$$


It can now be seen that the imaginary part of the quaternion can be treated as the vector defining the axis of rotation, and the part $Re(\mathbf{q})$ is equal to the cosine of half the angle of rotation about this axis. We can also see that the inverse quaternion is a rotation about the same axis, but in the opposite direction:

$$
\begin{gather}    \mathbf{q}^{-1}=\cos{\frac{-\theta}{2}}+[v_xi,v_yj,v_zk]\sin{\frac{-\theta}{2}}=\cos{\frac{\theta}{2}}-[v_xi,v_yj,v_zk]\sin{\frac{\theta}{2}}  \end{gather}
$$
  
rotation about angle $\theta$, and next $-\theta$ about the same axis $\mathbf{v}$ returns to the original position - as we know:
  
$$
\begin{gather}    \mathbf{q}^{-1}\mathbf{q}=1 \end{gather}
$$


Due to the intuitive division of the quaternion into a real part - describing the angle of rotation and an imaginary part - describing the axis of rotation, it is common to see the quaternion written in the form:

$$
\begin{equation}    \mathbf{q}=q_w+q_xi+q_yj+q_zk\end{equation}
$$


This form of notation will be used, in some places because of its intuitive interpretation.

Just as in the domain of complex numbers, rotations can be summed by multiplying the complex (unit) numbers by themselves, so the quaternions describing rotations in 3D space can be combined by multiplying them. However, due to the non-transitive nature of rotations (and thus of quaternion multiplication), the order of multiplication matters, and the final quaternion of rotation obtained is not a simple sum of successive angles of rotation:

$$
\begin{gather}    \mathbf{q}_3=\mathbf{q}_2\mathbf{q}_1\label{eq:combining rotations}\end{gather}
$$
 
using it and (\ref{eq:rotation}) we can write rotation of vector $\mathbf{p}$ from 1st to 3rd orientation:    

$$
\begin{gather}\mathbf{p}''=\mathbf{q}_3\mathbf{p}\mathbf{q}_3^*=\mathbf{q}_2\mathbf{q}_1\mathbf{p}\mathbf{q}_1^*\mathbf{q}_2^*\end{gather}
$$

due to the separability of multiplication:   

$$
\begin{gather}\mathbf{p}''=\mathbf{q}_2\left(\mathbf{q}_1\mathbf{p}\mathbf{q}_1^*\right)\mathbf{q}_2^*\end{gather}
$$

which is equivalent to 2 separate rotations:

$$
\begin{gather} \begin{split}      \mathbf{p}'=\mathbf{q}_1\mathbf{p}\mathbf{q}_1^*\\\mathbf{p}''=\mathbf{q}_2\mathbf{p}'\mathbf{q}_2^*      \end{split}\end{gather}
$$

# Transformation with quaternion

A common operation performed on vectors is a transformation from one system of reference to another. To preserve uniformity with the transformation matrices, the notation ${}^B_A\mathbf{q}$ will be used. The quaternion so denoted transforms a vector from the coordinate frame (A) to (B) (according to equation (\ref{eq:rotation})). Note that $\mathbf{q}$ does not describe the rotation of the vector from position A to B, but a rotation from position B to A (a situation analogous to the transformation/rotation matrix).
Keeping this in mind and using equation (\ref{eq:combining rotations}), one can write down the formulas describing the merging of the transformation quaternions:

$$
\begin{gather}   {}^{C}_{A}{\mathbf{q}}={}^{C}_{B}{\mathbf{q}}{}^{B}_{A}{\mathbf{q}}\end{gather}
$$

therefore transformations of the vector $\mathbf{p}^{(A)}$ to $\mathbf{p}^{(C)}$ can be written:

$$
\begin{gather}  \mathbf{p}^{(C)}={}^{C}_{A}{\mathbf{q}}\mathbf{p}^{(A)} {}^{C}_{A}{\mathbf{q}^*}= {}^{C}_{B}{\mathbf{q}} {}^{B}_{A}{\mathbf{q}} \mathbf{p}^{(A)} {}^{B}_{A}{q^*}{}^{C}_{B}{q^*}\end{gather}
$$

due to the separability of multiplication can be written:

$$
\begin{gather}\mathbf{p}^{(C)}={}^{C}_{B}{\mathbf{q}}\left({}^{B}_{A}{\mathbf{q}}\mathbf{p}^{(A)} {}^{B}_{A}{\mathbf{q}^*}\right){}^{C}_{B}{\mathbf{q^*}}\end{gather}
$$

which is equivalent to performing successive transformations first from (A) to (B) and then from (B) to (C):

$$
\begin{gather}        \mathbf{p}^{(B)}={}^{B}_{A}{\mathbf{q}}\mathbf{p}^{(A)} {}^{B}_{A}{\mathbf{q}^*}\\     \mathbf{p}^{(C)}={}^{C}_{B}{\mathbf{q}}\mathbf{p}^{(B)} {}^{C}_{B}{\mathbf{q}^*}\end{gather}
$$

It is often necessary to convert Euler angles into quaternions or vice versa. According to the formula (\ref{eq:trigonometric_form_long}), the individual quaternions describing successive Euler angles look as follows:

$$
\begin{gather}\begin{aligned}\mathbf{q}_{\gamma}&=\left[\cos{\frac{\gamma}{2}},0,0,\sin{\frac{\gamma}{2}}\right]\\\mathbf{q}_{\beta}&=\left[\cos{\frac{\beta}{2}},0,\sin{\frac{\beta}{2}},0\right]\\\mathbf{q}_{\alpha}&=\left[\cos{\frac{\alpha}{2}},\sin{\frac{\alpha}{2}},0,0\right]\\ \end{aligned}\\ \nonumber \\ {}^{1}_{0}{\mathbf{q}}={}^{0}_{1}{\mathbf{q}}^*= (\mathbf{q}_{\gamma}\mathbf{q}_{\beta}\mathbf{q}_{\alpha})^*= \left[\begin{array}{c}  \cos{\frac{\alpha}{2}}\cos{\frac{\beta}{2}}\cos{\frac{\gamma}{2}} + \sin{\frac{\alpha}{2}}\sin{\frac{\beta}{2}}\sin{\frac{\gamma}{2}} \\    -\sin{\frac{\alpha}{2}}\cos{\frac{\beta}{2}}\cos{\frac{\gamma}{2}} + \cos{\frac{\alpha}{2}}\sin{\frac{\beta}{2}}\sin{\frac{\gamma}{2}} \\      -\cos{\frac{\alpha}{2}}\sin{\frac{\beta}{2}}\cos{\frac{\gamma}{2}} - \sin{\frac{\alpha}{2}}\cos{\frac{\beta}{2}}\sin{\frac{\gamma}{2}}\\      -\cos{\frac{\alpha}{2}}\cos{\frac{\beta}{2}}\sin{\frac{\gamma}{2}} + \sin{\frac{\alpha}{2}}\sin{\frac{\beta}{2}}\cos{\frac{\gamma}{2}}\end{array}\right]\label{eq:euler to quaternion}\end{gather}
$$


Thus, the Euler angles from the quaternion notation can be obtained according to the formulas:

$$
\begin{gather}\begin{aligned}\gamma&=atan2(2(q_xq_y-q_wq_z),q_w^2+q_x^2-q_y^2-q_z^2)\\ \beta&=\arcsin{(2(-q_xq_z-q_wq_y))}\\ \alpha&=atan2(2(q_yq_z-q_wq_x),q_w^2-q_x^2-q_y^2+q_z^2)\end{aligned}\label{eq:quaternion to euler}\end{gather}
$$


Conversion from quaternion (${}^{1}_{0}{\mathbf{q}}=a+bi+cj+dk$) to transformation matrix is also possible with the following formula:

$$
\begin{gather}\mathbf{R}_{0}^{1}\left({}^{1}_{0}{\mathbf{q}}\right)=\left[\begin{array}{ccc}    a^2+b^2-c^2-d^2 & 2bc-2ad & 2bd+2ac\\    2bc+2ad & a^2-b^2+c^2-d^2 & 2cd-2ab\\    2bd-2ac & 2cd+2ab & a^2-b^2-c^2+d^2    \end{array}\right]\label{eq:quaternion to matrix}\end{gather}
$$


The inverse process ($\mathbf{R\rightarrow q}$) should not be carried out explicitly because of the numerical errors involved. Obtaining the quaternion of the rotation should be done thoughtfully, taking into account these inaccuracies as discussed in this [paper](https://upcommons.upc.edu/bitstream/handle/2117/124384/2068-Accurate-Computation-of-Quaternions-from-Rotation-Matrices.pdf) or [here](https://en.wikipedia.org/wiki/Rotation_matrix#Quaternion).

Some great videos on this topic: [quaternions](https://www.youtube.com/watch?v=d4EgbgTm0Bg), [quaternions rotation](https://www.youtube.com/watch?v=zjMuIxRvygQ)

# Double coverage 

A property of quaternions is that any configuration in 3D space can be expressed by 2 unit quaternions: q and -q. Intuitively, this can be explained as follows:
quaternion  $1,0,0,0$ represents a non-rotated object,
double multiplication with quaternion $[0,i,0,0]$ gives as a product $[-1,0,0,0$, the identical quaternion is obtained when multiplying twice by the quaternion $[0,0,j,0]$, $[0,0,0,k]$ or any other pure quaternion,   
 it can be seen that pure quaternions (e.g. $[0,i,0,0]$, $[0,0,j,0]$, $[0,0,0,k]$) must represent a rotation of $180^{\circ}$, since their double "use" results in the same quaternion ($[-1,0,0,0]$). In 3D space, arriving at an identical position in 2 identical rotations for any axis can only be done by rotating by $180^{\circ}$ twice. This returns the object to its original position.
The above description shows that the quaternion $[1,0,0,0]$ and $[-1,0,0,0]$ describe an identical setting in 3D space. It can also be seen that to return to the original quaternion one must multiply it 4 times by the pure quaternion. Which, as described above, involves rotating the object 2 times in 3D space. Therefore, in some publications, the full rotation of the quaternion is divided into $720^{\circ}$. The quaternions describing a rotation by angle $\theta$ and $360^{\circ}+\theta$ are not identical although they describe the same orientation in 3D space.


