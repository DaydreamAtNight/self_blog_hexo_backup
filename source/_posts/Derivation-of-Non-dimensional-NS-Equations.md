---
title: Derivation of Non-dimensional NS Equations
author: Ryan LI
toc: true
declare: true
index_img: /index/non-dimensional_NS.png
tags:
  - fluid dynamics
date: 2022-05-20 10:13:03
---

{% note primary %}

Feeling unsafe when deploying CFD algorithms, the best way to alleviate the anxiety is to derive the fundamentals again.

{% endnote%}

<!-- more -->

Introduce dimensionless parameters into the incompressible Navier-Stokes equations to arrive at a non-dimensional form.

### 1 Motivation

To reduce the dimensionality of the problem.

### 2 Dimensional analysis

Given fundamental physical quantities such as length $[L]$, time $[T]$, mass $[M]$ and temperature $[\Theta]$,

- Aera $A$ is $[L^2]$
- Velocity $V$ is $[LT^{-1}]$
- Acceleration $a$ is $[LT^{-2}]$
- Force $F$ is $[MLT^{-2}]$
- Pressure $p$ is $[ML^{-1}T^{-2}]$
- Energy $J$ is $[ML^2T^{-2}]$

The *principle of dimensional homogeneity* states the dimensions on both sides of an equation balance. And it applies to all equations of mechanics.

In dimensional analysis, take Bernoulli equation as an example, there are 4 factors:
$$
\frac{p}{\rho} + \frac{1}{2}V^2 +gz = Const.
$$

- *Dimensional variables*: variables, $p$, $V$ and $z$
- *Dimensional parameters*: fixed throughout experiment although with a dimension, $\rho$, $g$ and $Const$
- *Pure constants*: mathematical manipulations, $\pi$ and $e\approx2.718$

{% note info %}

Special care: 

- *Angles* are dimensionless yet the units are radians

- Some physical quantities are dimensionless by their definition, such as *strain* as a change in length per unit length
- Integration and differentiation change the dimension of the equation as well

{% endnote %}

### 3 Non-dimensionalisation of equations

Any dimensionally homogeneous equation can be non-dimensionalised, This process roughly proceeds as follows:

1. Identify which quantities are *variables* (vary or measured) and *parameters* (fixed per experiment).

2. Identify the number of fundamental dimensions, $N$, involved.

3. Select $N$ parameters to be scaling parameters with which to define dimensionless variables.

   {% note info %}

   Note: There are often multiple choices here and the choice will depend on exactly what we are aiming to show with our data.

   {% endnote %}

4. Scale each variable $u$ by combinations of these scaling parameters $s_i$ to arrive at a non-dimensional form $u^∗$. i.e. write $u^∗ = \alpha(s_i)u$.

5. Substitute into the equation and simplify to arrive at its non-dimensional form.

Some rules for selecting scaling parameters:

- They must *not* form a dimensionless group amongst themselves. For example:
  $$
  S_{0}^{a} V_{0}^{b}=[L]^{a}\left[L T^{-1}\right]^{b}=L^{0} T^{0} \quad \Leftrightarrow \quad a=b=0
  $$

- Do not include the output variables you wish to analyse/plot.

**Example**

Give the falling-body equation, and follow the process
$$
S = S_0+V_0t+\frac{1}{2}gt^2
$$

1. Divide the variables and parameters:
   $$
   \mathrm{Variables:~}S,t,\qquad\mathrm{Parameters:~}S_0,V_0,g
   $$

2. Identify the dimensions:
   $$
   S = [L], \quad t=[T],\quad S_0=[L],\quad V_0=[L/t],\quad g=[L/t^2]
   $$
   And 2 dimensions exist: $[L]$ and $[T]$ 

---

3. There are 3 options of parameters choosing, choose $(S_0,V_0)$ for instance,

4. Non-dimensional variables can be described as:

$$
S^* = \frac{1}{S_0}S  \qquad t^* = \frac{V_0}{S_0}t
$$

5. Subscribe into the function to get a non-dimensional form of it:
   $$
   S^* = 1+t^*+\frac{1}{2}\alpha t^{*2}, \quad \mathrm{with} \quad \alpha = \frac{gS_0}{V_0^2}
   $$
   It is a function of a single dimensionless parameter $\alpha$ identifying the effect of gravity.

---

3. Similarly, choose $(S_0,g)$, we have

4. $$
   S^{**} = \frac{g}{V_0^2}S  \qquad t^{**} = \frac{g}{V_0}t
   $$

5. The non-dimensional form body-drop function is therefore:
   $$
   S^* =  \alpha+ t^{**}+ \frac{1}{2}gt^{**},\quad \mathrm{with} \quad \alpha=\frac{gS_0}{V_0^2}
   $$
   Here, $\alpha$  identifies the effect of $V_0$.

----

The reduction in the number of variables/parameters (2 in this case, from 5 down to 3) equals the number of fundamental dimensions of the problem. This observation was formalised by **Buckingham**.

### 4 Buckingham’s Pi theorem

Proposed by Buckingham in 1914, it is a means of finding dimensionless groups, or $\Pi$s

>If a physical process satisfies the PDH(Principle of Dimensional Homogeneity) and involves $n$ dimensional variables, it can be reduced to a relation between only $k$ dimensionless variables or $\Pi$s. The reduction $j = n - k$ equals the maximum number of variables that do not form a $\Pi$ among themselves and is always less than or equal to the number of dimensions describing the variables.
>
>
>
>Find the reduction $j$, then select $j$ scaling variables that do not form a $\Pi$ among themselves. Each desired $\Pi$ group will be a power product of these $j$ variables plus one additional variable, which is assigned any convenient nonzero exponent. Each $\Pi$ group thus found is independent.

The first part of this theorem describes what sort of a reduction we can achieve for a given equation. The second part describes a methodology for systematically identifying $\Pi$’s

**Example 1** 

Suppose we have that the drag force on a body depends on length of the body, velocity of the flow, density and viscosity of the fluid:
$$
F = f(L,V,\rho,\mu)
$$

1. Dimensions of each variables are:
   $$
   F=\left[M L T^{-2}\right], \quad L=[L], \quad V=\left[L T^{-1}\right], \quad \rho=\left[M L^{-3}\right], \quad \mu=\left[M L^{-1} T^{-1}\right]
   $$
   3 dimensions exist, $j=3$

2. And we expect to find $k = 5-3 = 2 ~\Pi$ groups. We are intersted in how the drag force relates to the velocity, so we choose $(L,\rho,\mu)$, The two $\Pi$s are:
   $$
   \begin{array}{ll}
   \Pi_{1}=L^{a} \rho^{b} \mu^{c} F=L^{0} \rho^{1} \mu^{-2} F & {[1]\left[M L^{-3}\right]\left[M L^{-1} T^{-1}\right]^{-2}\left[M L T^{-2}\right]=\left[M^{0} L^{0} T^{0}\right]} \\
   \Pi_{2}=L^{a} \rho^{b} \mu^{c} V=L^{1} \rho^{1} \mu^{-1} V & {[L]\left[M L^{-3}\right]\left[M L^{-1} T^{-1}\right]^{-1}\left[L T^{-1}\right]=\left[M^{0} L^{0} T^{0}\right]}
   \end{array}
   $$

3. As a result we have the dimensionless coefficients:
   $$
   C_{f}=\frac{\rho F}{\mu^{2}}=f(\mathrm{Re}) \quad \text { with } R e=\frac{\rho L V}{\mu}
   $$
   $C_f$ is the force coefficients and $Re$ is the famous Reynolds number.

**Example 2**

At low velocities (laminar flow), the volume flow $Q$ through a small-bore tube is a function only of the tube radius $R$, the fluid viscosity $\mu$ and the pressure drop per unit tube length $dp/dx$. Using the Pi theorem, find an appropriate dimensionless relationship.

We have:
$$
Q = f(R,\mu,dp/dx)
$$

1. with dimensions:
   $$
   Q=\left[L^{3} T^{-1}\right], \quad R=[L], \quad \mu=\left[M L^{-1} T^{-1}\right], \quad d p / d x=\left[M L^{-2} T^{-2}\right]
   $$

2. 3 dimensions $\Rightarrow$ 3 scaling variables & 1 $\Pi$ group. Choose $(R, \mu, dp/dx)$:
   $$
   \begin{aligned}
   \Pi_{1} &=R^{a} \mu^{b}(d p / d x)^{c} Q \\
   &=[L]^{a}\left[M L^{-1} T^{-1}\right]^{b}\left[M L^{-2} T^{-2}\right]^{c}\left[L^{3} T^{-1}\right] \\
   &=\left[M^{0} L^{0} T^{0}\right] \quad \Leftrightarrow \quad a=-4, b=1, c=-1
   \end{aligned}
   $$

3. Therefore we have:
   $$
   C = \frac{Q\mu}{R^4(dp/dx)}
   $$

### 5 Non-dimensionalisation of the governing equations

Give incompressible, no gravity governing equations. Take the case of open flow past an infinitely long circular cylinder. The continuity and momentum equations are given by:
$$
\begin{aligned}
0 &= \boldsymbol{\nabla}\cdot\mathbf{V} \\
\rho\frac{\mathrm{d}\mathbf{V}}{dt} &= -\boldsymbol{\nabla}p + \mu \boldsymbol{\nabla}^2\mathbf{V}
\end{aligned}
$$
Plus boundary conditions:
$$
\begin{aligned}
\mathrm{Solid~surface:~}&\mathbf{V}=0 \\
\mathrm{Inlet~or~outlet:~}&\mathrm{Known}~\mathbf{V},p
\end{aligned}
$$
Plus the cylinder has diameter $D$ and the flow has free-stream velocity of $U_0$. Recall that one might suppose that our velocity field is a function of the cylinder diameter, the free-stream velocity, as well as the fluid density and viscosity. We have that:
$$
\mathbf{V} = f(\mathbf{x},t,p,D,U_0,\rho,\mu)
$$

1. with dimensions:
   $$
   \begin{aligned}
   \Pi_{1} &=R^{a} \mu^{b}(d p / d x)^{c} Q \\
   &=[L]^{a}\left[M L^{-1} T^{-1}\right]^{b}\left[M L^{-2} T^{-2}\right]^{c}\left[L^{3} T^{-1}\right] \\
   &=\left[M^{0} L^{0} T^{0}\right] \quad \Leftrightarrow \quad a=-4, b=1, c=-1
   \end{aligned}
   $$

2. 3 dimensions and 8 variables $\Rightarrow$ 3 scaling variables & 5 $\Pi$ group. Choose $(D,U_0,\rho)$ since they cannot form a $\Pi$ group, we have:
   $$
   \begin{aligned}
   &\Pi_{1}=U_{0}^{a} D^{b} \rho^{c} V=U_{0}^{-1} V \\
   &\Pi_{2}=U_{0}^{a} D^{b} \rho^{c} \mathbf{x}=D^{-1} \mathbf{x} \\
   &\Pi_{3}=U_{0}^{a} D^{b} \rho^{c} t=U_{0} D^{-1} t \\
   &\Pi_{4}=U_{0}^{a} D^{b} \rho^{c} p=\left[L T^{-1}\right]^{a}[L]^{b}\left[M L^{-3}\right]^{c}\left[M L^{-1} T^{-2}\right]=U_{0}^{-2} \rho^{-1} p \\
   &\Pi_{5}=U_{0}^{a} D^{b} \rho^{c} \mu=\left[L T^{-1}\right]^{a}[L]^{b}\left[M L^{-3}\right]^{c}\left[M L^{-1} T^{-1}\right]=U_{0}^{-1} D^{-1} \rho^{-1} \mu
   \end{aligned}
   $$

3. Therefore:
   $$
   \frac{V}{U_{0}}=f\left(\frac{x}{D}, \frac{t U_{0}}{D}, \frac{p}{U_{0}^{2} \rho}, \frac{\mu}{\rho U_{0} D}\right)
   $$

4. Use above to write down the non-dimensional scaling of the parameters:
   $$
   V^{*}=\frac{1}{U_{0}} V \quad \mathbf{x}^{*}=\frac{1}{D} \mathbf{x} \quad t^{*}=\frac{U_{0}}{D} t \quad p^{*}=\frac{1}{\rho U_{0}^{2}} p \quad \nabla^{*}=D \nabla \quad \mu^{*}=\frac{1}{\rho U_{0} D} \mu
   $$

5. Therefore the non-dimensional form of N-S equations:
   $$
   \begin{aligned}
   0 &= \boldsymbol{\nabla^*}\cdot\mathbf{V^*} \\
   \rho\frac{\mathrm{d}\mathbf{V^*}}{dt^*} &= -\boldsymbol{\nabla^*}p^* + \frac{1}{Re} \boldsymbol{\nabla^{*2}}\mathbf{V^*}
   \end{aligned}
   $$
   where $Re = \frac{\rho U_0D}{\mu}$ and with boundary conditions:
   $$
   \begin{aligned}
   \mathrm{Solid~surface:~}&\mathbf{V^*}=0 \\
   \mathrm{Inlet~or~outlet:~}&\mathrm{Known}~\mathbf{V^*},p^*
   \end{aligned}
   $$

In this case, rather than analysing our flow problem with respect to the four parameters (cylinder radius, free-stream velocity, density and viscosity), we need only analyse it with respect to the single dimensionless Reynolds number.

Besides, the above can be generalised by considering $U_0$ and $D$ to be any characteristic velocity- and length-scale of the specific problem being considered.

### 6 Dimensionless parameters

To conclude we highlight a couple of the dimensionless parameters which can arise in the incompressible Navier-Stokes equations.

There are no dimensionless parameters in the continuity equation. However, there is one in the momentum equation, the Reynolds number
$$
Re = \frac{\rho U_0D}{\mu}
$$
It can be considered as the ratio of *inertial* to *viscous* effects and is widely considered the **most important** parameter in fluid mechanics.

If the free-stream velocity $U_0$ were instead considered to be oscillating rather than constant, and of the form:
$$
U = U_0\cos(\omega t)
$$
then the problem can be considered as:
$$
\mathbf{V} = f(\mathbf{x},t,p,D,U_0,\rho,\mu, \omega)
$$
And the non-dimensionalised $U(t)$, we observe that:
$$
\frac{U}{U_0}=U^*=\cos\left(\frac{\omega D}{U_0}t^*\right)
$$
where $\omega$ is the frequency. We now have an additional dimensionless parameter called the ***Strouhal number*:**
$$
St = \frac{\omega D}{U_0}
$$
There are many other dimensionless parameters which arise in specific types of flow problems.
