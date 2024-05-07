---
layout: default
title: Quaternions Intro
parent: Quaternions
grand_parent: Math

nav_order: 1
---

<!-- comment or image allows {: .no_toc} to work correctly  (don't ask me why) -->

{: .no_toc}

# Quaternions - Introduction

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Why quaternions?

The quaternion notation is the least intuitive solution, but it has several advantages that make it very commonly used in many fields that deal with orientation in 3D space. First, however, the basic operations in the domain of quaternions as well as the assumptions made in the following chapters will be described. The main advantages will be seen in the next chapters, but this part is to give some awareness about quaternions properties used in the next operations. Although it is not necessary to understand quaternions up smallest detail, in my opinion, it is really pleasant to know how some mathematics, used in projects works. Also, I hope that this presentation of quaternions (of course not complete and probably fully correct in math rigor) will be more approachable for non-mathematicians.

# What are quaternions?

Quaternions are just extensions of complex numbers. They have Real part - $Re(\mathbf{q})$ and Imaginary part - $Im(\mathbf{q})$ but instead of 1, there are 3 imaginary symbols ($i,\ j,\ k$). If the real part is zero then such a quaternion calls a pure quaternion. If the imaginary part is zero it is just a real number.

$$
\begin{equation}
\mathbf{q}=a+bi+cj+dk
\end{equation}
$$

$a,\ b,\ c,\ d$ - real numbers

Multiplication and addition in the quaternion domain is performed identically to multiplication polynomials with 3 variables (i, j, k), where the relations between imaginary units are following:

$$
\begin{gather} i^2=j^2=k^2=ijk=-1\\ \nonumber\\ ij=-ji=k,\ jk=-kj=i,\ ki=-ik=j\end{gather}
$$

Quaternion multiplication is noncommutative but separable for addition. Commutative is multiplication by a scalar. Each quaternion whose norm is different from 0 has its inverse:

$$
\begin{align}   &\forall{\mathbf{q}_1,\mathbf{q}_2,\mathbf{q}_3 \in{\mathbb{H}}} && \begin{split}\mathbf{q}_1(\mathbf{q}_2+\mathbf{q}_3) &= \mathbf{q}_1\mathbf{q}_2+\mathbf{q}_1\mathbf{q}_3 \\ &\ne \\ (\mathbf{q}_2+\mathbf{q}_3)\mathbf{q}_1 &= \mathbf{q}_2\mathbf{q}_1+\mathbf{q}_3\mathbf{q}_1 \end{split} \\ \nonumber \\ & \forall{\mathbf{q}_1,\mathbf{q}_2\in{\mathbb{H}},\lambda\in{\mathbb{R}} } & & \mathbf{q}_1(\lambda\mathbf{q}_2)=\lambda\mathbf{q}_1)\mathbf{q}_2=\lambda(\mathbf{q}_1\mathbf{q}_2)\\\nonumber \\&\forall{\mathbf{q}\in{\mathbb{H}\backslash \{0\}}} \  \exists{\mathbf{q}^{-1}}   & &\mathbf{q}^{-1}\mathbf{q}=1 \label{eq:inverse quaternion}\end{align}
$$

$^* \mathbb{H}$ is a symbol of quaternions space from [William Hamilton](https://en.wikipedia.org/wiki/William_Rowan_Hamilton) - author of quaternions.

In addition, we define the conjugation of a quaternion - a quaternion with opposite signs in the imaginary part. This is usually indicated by an asterisk or a horizontal line above the quaternion. In this blog, we will use the notation $\mathbf{q}^*$:

$$
\begin{align}\begin{split}     \mathbf{q}&=a+bi+cj+dk\\ \nonumber \\ \mathbf{q}^*&=a-bi-cj-dk \end{split}\end{align}
$$

It is worth noting that the conjugation of the multiplication of quaternions is equal to the reverse multiplication of conjugations of these quaternions:

$$
\begin{equation}  (\mathbf{q}_1\mathbf{q}_2)^*=\mathbf{q}_2^*\mathbf{q}_1^*\label{eq:conjugation of multiplication}\end{equation}
$$

The norm of a quaternion is defined analogically to the norm of a 4D vector:

$$
\begin{equation} ||\mathbf{q}||=\sqrt{a^2+b^2+c^2+d^2}=\sqrt{\mathbf{q}\mathbf{q}^*}\end{equation}
$$

Similarly scalar product/dot product:

$$
\begin{equation} \begin{split}  \mathbf{q}_1\cdot \mathbf{q}_2 &=(a_1+b_1i+c_1j+d_1k)\cdot (a_2+b_2i+c_2j+d_2k)=\\ &=a_1a_2+b_1b_2+c_1c_2+d_1d_2\end{split} \end{equation}
$$

As in the case of normal vectors, for quaternions it is possible to determine the cosine of the angle between them:

$$
\begin{equation}    \cos{\theta}=\frac{\mathbf{q}_1\cdot \mathbf{q}_2}{||\mathbf{q}_1||||\mathbf{q}_2||}\label{eq:cos(theta) quaternions in general}\end{equation}
$$

The inverse of a quaternion can be determined as:

$$
\begin{equation} \mathbf{q}^{-1}=\frac{\mathbf{q}^*}{||\mathbf{q}||^2}\end{equation}
$$

Methods of division can also be defined, but due to the non-transitive nature of multiplication, a distinction is made between right and left division:

$$
\begin{gather}    \mathbf{q}_1/\mathbf{q}_2=\mathbf{q}_1\mathbf{q}_2^{-1}\\   \nonumber \\  \mathbf{q}_1\backslash \mathbf{q}_2=\mathbf{q}_2^{-1}\mathbf{q}_1 \end{gather}
$$

By analogy with the complex numbers, quaternions can be written in the trigonometric form:

$$
\begin{equation} \mathbf{q}=a+bi+cj+dk=a+\mathbf{v}=||\mathbf{q}||\left(\cos{\theta}+\frac{\mathbf{v}}{||\mathbf{v}||} \sin{\theta}\right)\label{eq:trigonometric form in general}\end{equation}
$$

Exponential function can be defined in a few ways but using one of them we can write:

$$
\begin{gather}    exp(x)=e^x=\sum_{n=0}^\infty \frac{x^n}{n!} \end{gather}
$$

in place of $x$ will be paste vector $\mathbf{v}$ and an angle $\theta$:

$$
\begin{gather}exp(\theta \mathbf{v})=e^{\theta \mathbf{v}}=\sum_{n=0}^\infty \frac{(\theta \mathbf{v})^n}{n!}= \sum_{n=0}^\infty \frac{\theta^n \mathbf{v}^n}{n!}\end{gather}
$$

treating the vector $\mathbf{v}$ as a pure quaternion, to calculate $\mathbf{v}^n$ it can be shown that:

$$
\begin{gather}\sum_{n=0}^\infty \frac{\theta^n \mathbf{v}^n}{n!}= \sum_{n=0}^\infty\frac{\theta^{2n} \mathbf{v}^{2n}}{(2n)!}+\sum_{n=0}^\infty\frac{\theta^{2n+1}\mathbf{v}^{2n+1}}{(2n+1)!}=\nonumber\\ \nonumber \\   =\sum_{n=0}^\infty \frac{\theta^{2n} (-1)^{n}}{(2n)!}+ \sum_{n=0}^\infty \frac{\theta^{2n+1} \mathbf{v}(-1)^{n}}{(2n+1)!}\end{gather}
$$

the formulas for $\cos{\theta},\ \sin{\theta}$ can be recognized in the above sums. Therefore, it can be written as:

$$
  \begin{gather}e^{\theta \mathbf{v}}=\cos{\theta}+\mathbf{v}\sin{\theta}\label{eq:exponential form quaternions}\end{gather}
$$

The trigonometric form of the quaternion obtained in this way corresponds to that known from equation (\ref{eq:trigonometric form in general}), where the quaternion is the unit quaternion.

The unit quaternions ($||\mathbf{q}|| = 1$) can be identified with rotation (this will be shown in the next section). This is particularly interesting from the point of view of determining the orientation of the drone in space. In addition, the awareness of the unity of the quaternion norm allows to simplification of many notations, which greatly increases readability. Therefore, from this point on, it is assumed that all quaternions used below satisfy the condition $||\mathbf{q}|| = 1$.
The simplifications resulting from this assumption are as follows:

$$
\begin{gather} \mathbf{q}^{-1}=\mathbf{q}^*\\\nonumber \\ \mathbf{q}=\cos{\theta}+\mathbf{v}\sin{\theta} \label{eq:quaternion trigonometric form}\\\nonumber \\ \mathbf{q}_2/ \mathbf{q}_1=\mathbf{q}_2\mathbf{q}_1^{*}\label{eq:division right}\\\nonumber \\ \mathbf{q}_1\backslash \mathbf{q}_2=\mathbf{q}_2^{*}\mathbf{q}_1 \\\nonumber \\ \cos{\theta}=\mathbf{q}_1\cdot \mathbf{q}_2\label{eq:cos(theta) quaternions}\end{gather}
$$

The interpretation of the division (\ref{eq:division right}) is the determination of the quaternion of rotation between quaternions describing 2 orientations in space (from the orientation described by $\mathbf{q}_1$ one can arrive at the orientation $\mathbf{q}_2$ - see [Quaternions as rotations in 3D space](quaternions_as_rotation)).

Due to the numerical errors that occur, the norm of a quaternion may differ from unity. Therefore, to preserve the properties of unit quaternions, normalization must be performed. Its performance does not differ in any way from the normalization of a 4D vector:

$$
\begin{equation} \mathbf{q}=\frac{\mathbf{q}}{||\mathbf{q}||} \label{eq:normalization quaternions}\end{equation}
$$

Those are the main operations and properties of quaternions which will be used in the following posts. I hope it is not so scary and as more often we use it, the more natural it will become.
