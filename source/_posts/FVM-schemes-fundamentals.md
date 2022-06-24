---
title: Numerical schemes fundamentals
author: Ryan LI
toc: true
declare: true
index_img: /index/fvm_schemes.png
tags:
  - fluid dynamics
  - OpenFOAM
date: 2022-06-18 18:08:24
---

{% note primary %}

This is the **essence** of CFD, advecting with the discontinuities due to inviscid fluid PDEs. Several computational schemes dealing with them are introduced then tested in OpenFOAM on the 1D shockTube case.

{% endnote%}

<!-- more -->

{% note secondary %}

This is a review of my graduate CFD course and the application of the theory to CFD software. The aim is to further the understanding of finite volume method and the `fvscheme` dict in OpenFOAM.

{% endnote%}

**Reference books:**

E.F. Toro, Riemann Solvers and Numerical Methods for Fluid Dynamics, Springer-Verlag.

R.J. LeVeque, Finite Volume Methods for Hyperbolic Problems, Cambridge University Press.

**Overview:**

{% markmap 300px %}

- Scalar conservation laws
  - conservation vs non-conservative
  - 1D simple Riemann problem for Burgers' equation
  - characteristics discontinuities and jump conditions
  - weak solutions and entropy condition
- Numerical schemes for 1D discontinuities
  - practical examples on one conservation law
  - 

- System of conservation laws
  - add complexity, system of conservation laws

{% endmarkmap %}

## Scalar conservation laws

{% note info %}

1-D theory. Examples of 1-D hyperbolic conservation laws. Characteristics discontinuities and jump conditions. Weak solutions and entropy condition. Linear versus non-linear advection

{% endnote%}

### Challenges

As we all know, flow fluids are governed by 3 hyperbolic PDEs (conservation of mass, momentum and energy). These equations are highly non-linear and they can lead to discontinuities even with a smooth initial conditions, which is very difficult to solve numerically. Simple FDM will definitely fail on these discontinuities and shocks.

Recall the incompressible NS equation that we've learned in the kindergarten:
$$
\frac{\partial \mathbf{V}}{\partial t} + \underbrace{\left(\boldsymbol{\nabla}\cdot\mathbf{V}\right)\mathbf{V}}_{\text{convection}} = \frac{\nabla p}{\rho} + \mathbf{g}+ \underbrace{\nu \boldsymbol{\nabla}^2\mathbf{V}}_{\text{diffusion}}
$$
the viscous diffusion term in the equation leads to parabolic equations with smooth solutions, which will save our life. But with very high $Re$, the equation reduces to pure <font color=#75147c>hyperbolic inviscid Euler Equation</font>, and the resultant discontinuity is a nightmare for most of the numerical schemes. 

The presence of discontinuities requires <font color=#75147c>weak solutions</font>, as oppose to <font color=#75147c>strong solutions</font>.

However, the weak solutions give up the uniqueness in math, i.e there exist a large number of solutions that may not be physical acceptable, i.e. extra conditions are needed to justify the solution. They are:

- <font color=#75147c>Rankine-Hugoniot</font> condition deals with the discontinuity
- <font color=#75147c>Entropy</font> condition satisfies the physics

{% note info %}

Just here to remind that we are dealing with inviscid flow, shocks, nothing to do with turbulence.

{% endnote%}

### Scalar conservation laws

#### 1D law

<img src="1D scalar convection.png" alt="1D scalar convection control volume" style="zoom:70%;" />

First let's introduce a simple example to present the problem: consider an 1D control volume $[a,b]$, during the time interval $[t_1,t_2]$ . The scalar conservation law can be stated as that during $[t_1,t_2]$ , change in total conserved quantity in $[a,b]$ equals to the net flux through the boundaries $a$, $b$:

$$
\frac{d}{dt}\int_a^bu(x,t) = -\left[f(u(b,t))-f(u(a,t))\right]
$$
where $u(x,t)$ is called the conserved quantity while $f$ denotes the flux. This equation often describes the <font color=#75147c>transport phenomena</font>.

{% note info %}

You can not create, you can not destroy.

{% endnote %}

#### Integral, differential conservative and primitive forms

<img src="2D scalar convection control volume.png" alt="2D scalar convection control volume" style="zoom:48%;" />

The 1D law can be extended in 2D with the relation, in <font color=#75147c>integral form</font>.
$$
\frac{d}{d t} \int_{\Omega} \mathbf{u} d \Omega+\int_{\Gamma} \mathbf{f}(\mathbf{u}) \cdot \mathbf{n} d \Gamma=0
$$
{% note info %}

The FVM and FEM solves the integral form, while the FDM solves the primitive form.

{% endnote %}

Apply the Gauss divergence theorem the relation can be represented as <font color=#75147c>differential form</font>:
$$
\begin{aligned}
\int_{\Omega}\left\{\frac{\partial \mathbf{u}}{\partial t}+\nabla \cdot \mathbf{f}(\mathbf{u})\right\} d \Omega=0&  \\ 
\Rightarrow 
\color{purple}{
\forall \Omega,\quad \frac{\partial \mathbf{u}}{\partial t}+\nabla \cdot \mathbf{f}(\mathbf{u})=0} &
\end{aligned}
$$
above is called the <font color=#75147c>differential conservative form</font>. The only $u$ changed in time is due to the flux $f$. There is no extra assumptions introduced.

In comparison, with an assumption of $\mathbf{a} =  \frac{d\mathbf{f}}{d\mathbf{u}}$, the equation can be rewritten with the chain rule, as the <font color=#75147c>differential primitive form</font> :
$$
\quad \frac{\partial \mathbf{u}}{\partial t}+\mathbf{a}(\mathbf{u})\nabla \cdot \mathbf{u}=0, \quad \text{where }\mathbf{a} =  \frac{d\mathbf{f}}{d\mathbf{u}}
$$
This form is identical to above in mathematics, but it will lead to problems numerically when dealing with discontinuities (to be discussed later).

#### Rankine-Hugoniot Jump condition

<img src="shock wave.png" alt="1D jump discontinuity illustration" style="zoom:70%;" />

When dealing with discontinuities, the integral form is well defined, but not the differential forms. The differential solvers can't deal with the drastic derivatives so instead they solve an extra <font color=#75147c>jump condition</font> which can be derived from the well-defined integral form (discussed later).

For an 1D jump discontinuity travelling at the speed $s$:
$$
f(u_r)-f(u_l) = s(u_r-u_s)
$$
where $u_r$, $u_l$ represent the speeds just at the left and the right of the discontinuity. 

### 1D Euler equations

Its time to introduce the 1D Euler equations: NS equations with 0 viscosity or heat conduction terms:
$$
\frac{\partial}{\partial t}\left[\begin{array}{l}
\rho \\
\rho u \\
\rho E
\end{array}\right]+\frac{\partial}{\partial x}\left[\begin{array}{l}
\rho u \\
\rho u^{2}+P \\
\rho u\left(E+\frac{P}{\rho}\right)
\end{array}\right]=0
$$
And the conservation form is:
$$
\frac{\partial \mathbf{q}}{\partial t}+\frac{\partial \mathbf{F}(\mathbf{q})}{\partial x}=0
$$
for the quantity vector $\mathbf{q}$ with the flux vector $\mathbf{F}$.

#### Conservation vs non-conservation form

Similar to before, with terms in a form of $\partial_x{uv}$:
$$
\frac{\partial uv}{\partial x} \neq u\frac{\partial v}{\partial x}+ v\frac{\partial u}{\partial x}
$$
When dealing with the discontinuities, the conservative LHS locates the shock directly, while the non-conservative RHS does not. (to be discussed later)

#### Close the Euler equation

Unlike the well-posed NS equation, in Euler equation, we have 4 unknowns but only 3 equations. So an extra state equation, i.e. extra assumption of the gas state is needed. Sometimes it's the idea gas assumption $p = \rho RT$, but for high $Re$ compressible flows, equations of state are required to describe the relation between $p, \rho \text{ and }T$.

{% note primary %}

We are going to use the 1D Euler equations to evaluate the numerical methods.

{% endnote %}

### Analytical solutions of Euler equations

#### Linear Advection Equation

In 1D, the linear advection law for $u(x,t)$ is:
$$
\color{purple}
\frac{\partial u }{\partial t} + a(u)\frac{\partial u }{\partial x} = 0
$$
where $a(u)$ denotes the advection speed. Plus an <font color=#75147c>initial condition</font> $u(x,0)$ and <font color=#75147c>boundary condition</font> (discuss later).

##### solution

This is a simple 2-variable PDE and we can apply the method of characteristics:

1. Imagine a characteristic line $s$, we have the chain rule:
   $$
   \frac{d u(x,t)}{d s} = \frac{d t}{d s}\frac{\partial u}{\partial t} + \frac{d x}{d s} \frac{\partial u}{\partial x}
   $$

2. As a result we can construct, 
   $$
   \begin{aligned}
   \frac{d u(x,t)}{d s} = 0,\quad\frac{d t}{d s}=1, \quad \frac{d x}{d s}= a \\
   \Rightarrow \quad \frac{d u}{0}=\frac{d t}{1}=\frac{d x}{a}= d s
   \end{aligned}
   $$

3. Select the available equations we have the characteristic equation:
   $$
   \begin{aligned}
   \frac{d x}{d t} &= a \\
   \Rightarrow\quad x &= x_0 + a(u)t
   \end{aligned}
   $$
   If $u(x_0,0)=u_0$, then $\frac{dx}{dt}=a(u_0)$, the characteristic equation is therefore:
   $$
   \color{purple}
   x = x_0 + a(u_0)t
   $$

4. And the solution to the problem is:
   $$
   u(x,t) = f(x_0) = f(x-a(u_0)t)
   $$

#### Linear Advection Equation Example

Solve the following advection equation where $a = 0.5$
$$
\frac{\partial u}{\partial t} + a \frac{\partial u}{\partial x }= 0
$$
with initial condition 
$$
u(x,0) = exp(-32x^2)
$$

##### solution

The characteristics are straight lines in the $(x,t)$ plane:
$$
x = 0.5t+x_0
$$
And the solutions are therefore:
$$
u(x,t) = f(x_0) = exp(-32(x-0.5t)^2)
$$
<img src="Linear Advection Solutions.png" alt="Linear Advection Characteristic Lines and Solutions, From my graduate course slides" style="zoom:28%;" />

#### Inviscid Burgers' equation

In 1D, the inviscid Burgers' equation for $u(x,t)$ is:
$$
\color{purple}
\frac{\partial u }{\partial t} + \frac{f(u) }{\partial x} = 0,\quad\text{where }f(u)=\frac{1}{2}u^2
$$
The conservative form for this equation：
$$
\frac{\partial u }{\partial t} + \frac{\partial(\frac{1}{2}u^2) }{\partial x} = 0
$$
and the primitive form:
$$
\frac{\partial u }{\partial t} + u\frac{\partial u }{\partial x} = 0
$$
same form as the 1D linear advection equation with $a(u)=u$.

#### Burgers' equation Example

Solve the advection equation:
$$
\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x }= 0
$$
with initial condition:
$$
u(x,0) = 1-\cos(x)
$$

##### solution

Similarly, the characteristics are described by:
$$
x = x_0 + ut
$$
and the solution:
$$
u = 1-\cos(x-ut)
$$
which is *<u>implicit</u>*. It is can be plotted anyway:

<img src="Inviscid Burgers' equation Solutions.png" alt="Inviscid Burgers' equation Solutions, From my graduate course slides" style="zoom:28%;" />

##### discussion

For non-linear conservation laws, the characteristics may cross within finite time.
This would suggest a multi-valued solution which does not make sense physically.

Where the characteristics start crossing, the solution become discontinuous. And the formation of discontinuities is possible even for smooth initial data. So the differential primitive form of the equations is no longer valid 

$\Rightarrow$ only the integral form can deal with discontinuities. And the differential form can be completed by a <font color=#75147c>jump condition</font> derived from the integral form. 

$\Rightarrow$ we need <font color=#75147c>weak solutions</font>. The mathematical theory of partial differential equations introduces the concept of weak solutions.

### Rankine-Hugoniot condition

#### The Riemann Problem

{% note info %}

In order to understand the behaviour of the solution at discontinuities, it is useful to start with a simplified problem. 

{% endnote %}

The Riemann problem is a conservation law with a single discontinuity.
$$
\color{purple}
\frac{\partial u}{\partial t} + \frac{\partial f(u)}{\partial x} = 0
$$
with
$$
\color{purple}
u(x, 0)= \begin{cases}u_{L} & x \leq 0 \\ u_{R} & x>0\end{cases}
$$
<img src="Riemann problem illustration.png" alt="Riemann problem illustration" style="zoom:48%;" />

#### Shock Path

<img src="shock path control volume.png" alt="shock path control volume" style="zoom:80%;" />

Take the control volume between boundaries $x_L$ and $x_R$, which are taken sufficient close to the shock so that spatial variations of the solution become unimportant. and are taken sufficient apart from the shock so that the boundary will note interfere with the shock motion over time interval $\delta t$.

Recall the integral function:
$$
\frac{d}{dt}\int_{x_L}^{x_R}udx = f(u_L)-f(u_R)
$$
If the position of the shock is $x = X(t)$, with $x_L< X (t) <x_R$,  the values of $u(x,t)$ inside the integral are close to the constants $u_L$ and $u_R$ and we can write:
$$
\begin{aligned}
\frac{d}{dt}\int_{x_L}^{X}u_Ldx + \frac{d}{dt}\int_{X}^{x_R}u_Rdx= f(u_L)-f(u_R)\\
\frac{d}{dt}\left[(x_L-X)u_L + (X-x_R)u_R\right]= f(u_L)-f(u_R)\\
\end{aligned}
$$

Given the shock speed (slope of the shock path) $s = \frac{dX}{dt}$, we have:
$$
\begin{aligned}
s\left(u_L - u_R\right)&= f(u_L)-f(u_R)\\
\color{purple}
s= \frac{dX}{dt}&\color{purple}
= \frac{f(u_L)-f(u_R)}{\left(u_L - u_R\right)}= \frac{f(u_R)-f(u_L)}{\left(u_R - u_L\right)}
\end{aligned}
$$
The equation above is the <font color=#75147c>Rankine-Hugoniot condition</font>, also called the "jump condition".

Correspondingly, <font color=#75147c>weak solutions</font> represents the solutions of the PDE where the solution is smooth and of a Rankine-Hugoniot condition at discontinuities. And they are <font color=#75147c>not unique</font>.

{% note info %}

Strong solution $\Rightarrow$ weak solution

Weak solution $\nLeftarrow$ Strong solution

{% endnote %}

### Non-uniqueness of weak solutions

#### Example of Riemann Problem with Burgers's equation

Consider the [Burgers' equation](#inviscid-burgers-equation) under a [Riemann Problem](#the-riemann-problem):
$$
\begin{aligned}
\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x }&= 0, \\
\text{with }
u(x, 0)&= \begin{cases}1 & x \leq 0 \\ 0 & x>0\end{cases}
\end{aligned}
$$
The characteristics are of the form: $x = x_0 + ut$ 

In $x-t$ plane, the characteristics line:
$$
\begin{cases} x = t - x_0 & x_0 \leq 0 \\ x = x_0 & x_0 > 0\end{cases}
$$
And the according to the Rankine-Hugonoit condition, the speed of the shock is:
$$
s = \frac{f(u_L)-f(u_R)}{\left(u_L - u_R\right)}= \frac{-1/2-0}{\left(1-0\right)} = \frac{1}{2}
$$
and the shock path is:
$$
x = \frac{1}{2} t
$$
therefore,  the solution is:
$$
u(x,t) = \begin{cases} 1 & x \leq \frac{1}{2}t \\ 0 & x> \frac{1}{2}t\end{cases}
$$
<img src="Burgers's equation under Riemann Problem with uL = 1, uR=0.png" alt="General solution (left) and characteristics (right) of Burgers's equation under Riemann Problem. From my graduate course slides" style="zoom:50%;" />

#### Upwind Riemann Problem with Burgers's equation

For the upwind case:
$$
\begin{aligned}
\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x }&= 0, \\
\text{with }
u(x, 0)&= \begin{cases}u_L & x \leq 0 \\ u_R & x>0\end{cases}
\end{aligned}
$$
with $u_L>u_R$.

And the shock is created with a speed:
$$
s = \frac{1}{2}(u_R+u_L)
$$
and the solution:
$$
u(x,t) = \begin{cases} u_L & x \leq \frac{1}{2}t \\ u_R & x> \frac{1}{2}t\end{cases}
$$
<img src="Burgers's equation under Riemann Problem general solution.png" alt="Solution (left) and characteristics (right) of Burgers's equation under Riemann Problem with uL = 1, uR=0. From my graduate course slides" style="zoom:50%;" />

#### Example of non-unique downwind Riemann Problem with Burgers's equation

Reverse the initial condition in the previous example:

$$
\begin{aligned}
\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x }&= 0, \\
\text{with }
u(x, 0)&= \begin{cases}0 & x \leq 0 \\ 1 & x>0\end{cases}
\end{aligned}
$$
the characteristics become:

<img src="Burgers's equation under Riemann Problem with uL = 0, uR=1.png" alt="Solution (left) and characteristics (right) of Burgers's equation under Riemann Problem with uL = 1, uR=0. From my graduate course slides" style="zoom:50%;" />

Solution in the blue area ($0<x<t$) is not defined. So here proposes 2 possible solutions, both are mathematical acceptable:

Solution A:
$$
u(x, t)= \begin{cases}0 & \text { if } \quad \frac{x}{t}<s(=0.5) \\ 1 & \text { if } \quad \frac{x}{t}>s(=0.5)\end{cases}
$$
<img src="Expansion shock solution to the Riemann Problem.png" alt="Expansion shock solution to the Riemann Problem" style="zoom:50%;" />

Solution B:
$$
u(x, t)=\left\{\begin{array}{ccc}
0 & \text { if } & \frac{x}{t}<0 \\
\frac{x}{t} & \text { if } & 0<\frac{x}{t}<1 \\
1 & \text { if } & \frac{x}{t}>1
\end{array}\right.
$$
<img src="Rarefaction wave solution to the Riemann Problem.png" alt="Rarefaction wave solution to the Riemann Problem" style="zoom:50%;" />

{% note info %}

A little spoiler alert there: Solution B is physical, discussed in the [next section](#non-uniqueness-and-entropy-conditions).

{% endnote %}

#### Exact solution to Riemann Problem with Burgers's equation

In conclusion,  

For the Riemann Problem with Burgers's equation:
$$
\begin{aligned}
\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x }&= 0, \\
\text{with }
u(x, 0)&= \begin{cases}u_L & x \leq 0 \\ u_R & x>0\end{cases}
\end{aligned}
$$

- with $u_L>u_R$

  A shock wave is created with a speed:
  $$
  V_s = \frac{u_L+u_R}{2}
  $$
  and the exact solution is:
  $$
  u(x, t)=\left\{\begin{array}{ccc}
  u_L & \text { if } & \frac{x}{t}\leq V_s \\
  u_R & \text { if } & \frac{x}{t}>V_s
  \end{array}\right.
  $$

- with $u_L<u_R$

  the exact solution is the rarefaction wave:
  $$
  u(x, t)=\left\{\begin{array}{ccc}
  u_L & \text { if } & \frac{x}{t}<0 \\
  \frac{x}{t} & \text { if } & 0<\frac{x}{t}<1 \\
  u_R & \text { if } & \frac{x}{t}>1
  \end{array}\right.
  $$
  if $u_L = -u_R$, we have a sonic rarefaction wave.

### Entropy Conditions

Why solution A is wrong? we need to impose additional conditions. There are two ways.

- Add a small diffusion term (2 <sup>nd</sup> order) manually on the RHS to remove the discontinuity. The weak solution then must satisfy:
  $$
  \frac{\partial u^\epsilon}{\partial t} + \frac{\partial f(u^\epsilon)}{\partial x} = \epsilon\frac{\partial^2u^\epsilon}{\partial x^\epsilon}
  $$
  where $\epsilon$ is the <font color=#75147c>viscosity coefficient</font>, it introduces the dissipation, known as the vanishing viscosity concept, into the equation to smooth the solution. A little bit cheating but most of people do this.

- Add the <font color=#75147c>entropy solution</font>

  {% note info %}

  I first hearted Entropy back in my undergraduate thermodynamic course. But I still don't know what is Entropy. So here are some answers:

  In gas dynamics, entropy is a constant physical quantity along particles in smooth flow which can jump to a higher value through a shock.

  The second law of thermodynamics says that entropy can never go down.

  For an evolution equation the information should always flow from the initial data.

  We can see it is very difficult to define it, but in order to translate the defination into the entropy condition, we see entropy as the extra amount of energy that is not available to the system.

  {% endnote %}

  There are again two options of entropy condition:

  - <font color=#75147c>Convex (concave) fluxes / Lax entropy condition</font> 
    $$
    f'(u_L) > s > f'(u_R)
    $$
    and the characteristics must run into the shock, not emerge from it.

  - <font color=#75147c>Oleinik entropy condition</font>

    Similar to Lax entropy condition, it says:
    $$
    \frac{f(u)-f(u_L)}{u-u_L}\geq s \geq \frac{f(u_R)-f(u)}{u_R-u}
    $$

  and Lax and Oleinik are equivalent if $f(u)$ is strictly convex i.e. $f''(u)<0$.

## Numerical representation of discontinuities

{% note info %}

Requirements on numerical schemes. Conservative discretisation: Lax-Wendroff theorem. First versus second order schemes. Representation of discontinuities: physical aspects, shock fitting/capturing.

{% endnote %}

### Problems with Lax Equivalence Theorem

There are 3 fundamental properties of a numerical scheme:

- Consistency: how good you approximate operators and functions.

- Convergence: error between the exact and discrete solutions converges to zero.
- Stability: solution of the difference equation is not too sensitive to small perturbations.



The convergence needs to be evaluated with exact solutions. So we always need analytical solutions to verify it. If we don't have access to the analytical solutions,  we have a <font color=#75147c>Lax Equivalence Theorem </font>says
$$
\text{consistency + stability $\Rightarrow$ convergence}
$$
It is a fundamental convergence theorem but 

- it is valid <font color=#75147c>only for linear PDEs</font> and there is no non-linear equivalent theorem
- this theorem <font color=#75147c>does not tell if the weak solution physically acceptable or not</font>.

For non-linear PDEs, we only have one experience, 

If a scheme is stable on linear PDEs, it will often (not all the time) be stable on non-linear PDEs. If a scheme is unstable on linear, it won't be stable on non-linear.

So the work flow is, 1. Given a non-linear PDE, 2. Linearise it to explore the stabilities of schemes on it. 3. Test the winners on non-linear PDEs.

### 1D linear convection equation

First we consider a <font color=#75147c>linear</font> case, linear convection equation:
$$
\frac{\partial u}{\partial t} + a\frac{\partial u}{\partial x} = 0
$$
with $a$ a positive scalar constant, representing the wave speed. So we have:
$$
f(u) = au
$$

##### Numerical scheme

try to solve it numerically, with finite difference, forward difference in time and central difference in space. we have:
$$
\frac{u^{n+1}_i-u^n_i}{\Delta t} + a \frac{u^n_{i+1}-u^n_{i-1}}{2\Delta x} = 0
$$

$$
u^{n+1}_i = u^n_i - \frac{a\Delta t}{\Delta x}\left(\frac{u^n_{i+1}-u^n_{i-1}}{2} \right) = 0
$$

It is consistency, and the Von Neumann analysis shows it is stable if the CFL condition (Courant number $a\frac{\Delta t}{\Delta x}<1$) is satisfied. It should converge to the exact solution.

##### Numerical practice





## Systems of conservation laws

{% note info %}

Jacobian matrices, linearized equations, conservative and characteristic variables.
Rankine-Hugoniot jump conditions. Boundary conditions.

{% endnote%}



## Numerical schemes for non-linear conservation laws

{% note info %}

It is still an active research area and these are the classical methods:

Centred schemes: one-step and two-step Lax Wendroff, MacCormack predictor-corrector. Artificial dissipation.
Upwind schemes: flux vector and flux difference splitting. Monotone schemes: Godunov and Harten theorems. Exact and approximate Riemann solvers.
High-order upwind schemes: the TVD property. The construction of TVD schemes using slope and flux limiters.
WENO schemes: weighted essentially non-oscillatory methods

{% endnote %}



## Numerical schemes for multi-dimensional problems

{% note info %}

Finite differences and finite volume. Computational domain and boundary conditions.

{% endnote %}



## OpenFOAM demo on shockwave

{% echarts 400 '85%' %}
let legends = ['exact', 'linear', 'upwind', 'linearUpwind', 'QUICK', 'TVD-vanLeer', 'TVD-Minmod', 'TVD-SuperBee']
let xaxis = [0.0005, 0.0015, 0.0025, 0.0035, 0.0045, 0.0055, 0.0065, 0.0075, 0.0085, 0.0095, 0.0105, 0.0115, 0.0125, 0.0135, 0.0145, 0.0155, 0.0165, 0.0175, 0.0185, 0.0195, 0.0205, 0.0215, 0.0225, 0.0235, 0.0245, 0.0255, 0.0265, 0.0275, 0.0285, 0.0295, 0.0305, 0.0315, 0.0325, 0.0335, 0.0345, 0.0355, 0.0365, 0.0375, 0.0385, 0.0395, 0.0405, 0.0415, 0.0425, 0.0435, 0.0445, 0.0455, 0.0465, 0.0475, 0.0485, 0.0495, 0.0505, 0.0515, 0.0525, 0.0535, 0.0545, 0.0555, 0.0565, 0.0575, 0.0585, 0.0595, 0.0605, 0.0615, 0.0625, 0.0635, 0.0645, 0.0655, 0.0665, 0.0675, 0.0685, 0.0695, 0.0705, 0.0715, 0.0725, 0.0735, 0.0745, 0.0755, 0.0765, 0.0775, 0.0785, 0.0795, 0.0805, 0.0815, 0.0825, 0.0835, 0.0845, 0.0855, 0.0865, 0.0875, 0.0885, 0.0895, 0.0905, 0.0915, 0.0925, 0.0935, 0.0945, 0.0955, 0.0965, 0.0975, 0.0985, 0.0995, 0.1005, 0.1015, 0.1025, 0.1035, 0.1045, 0.1055, 0.1065, 0.1075, 0.1085, 0.1095, 0.1105, 0.1115, 0.1125, 0.1135, 0.1145, 0.1155, 0.1165, 0.1175, 0.1185, 0.1195, 0.1205, 0.1215, 0.1225, 0.1235, 0.1245, 0.1255, 0.1265, 0.1275, 0.1285, 0.1295, 0.1305, 0.1315, 0.1325, 0.1335, 0.1345, 0.1355, 0.1365, 0.1375, 0.1385, 0.1395, 0.1405, 0.1415, 0.1425, 0.1435, 0.1445, 0.1455, 0.1465, 0.1475, 0.1485, 0.1495, 0.1505, 0.1515, 0.1525, 0.1535, 0.1545, 0.1555, 0.1565, 0.1575, 0.1585, 0.1595, 0.1605, 0.1615, 0.1625, 0.1635, 0.1645, 0.1655, 0.1665, 0.1675, 0.1685, 0.1695, 0.1705, 0.1715, 0.1725, 0.1735, 0.1745, 0.1755, 0.1765, 0.1775, 0.1785, 0.1795, 0.1805, 0.1815, 0.1825, 0.1835, 0.1845, 0.1855, 0.1865, 0.1875, 0.1885, 0.1895, 0.1905, 0.1915, 0.1925, 0.1935, 0.1945, 0.1955, 0.1965, 0.1975, 0.1985, 0.1995, 0.2005, 0.2015, 0.2025, 0.2035, 0.2045, 0.2055, 0.2065, 0.2075, 0.2085, 0.2095, 0.2105, 0.2115, 0.2125, 0.2135, 0.2145, 0.2155, 0.2165, 0.2175, 0.2185, 0.2195, 0.2205, 0.2215, 0.2225, 0.2235, 0.2245, 0.2255, 0.2265, 0.2275, 0.2285, 0.2295, 0.2305, 0.2315, 0.2325, 0.2335, 0.2345, 0.2355, 0.2365, 0.2375, 0.2385, 0.2395, 0.2405, 0.2415, 0.2425, 0.2435, 0.2445, 0.2455, 0.2465, 0.2475, 0.2485, 0.2495, 0.2505, 0.2515, 0.2525, 0.2535, 0.2545, 0.2555, 0.2565, 0.2575, 0.2585, 0.2595, 0.2605, 0.2615, 0.2625, 0.2635, 0.2645, 0.2655, 0.2665, 0.2675, 0.2685, 0.2695, 0.2705, 0.2715, 0.2725, 0.2735, 0.2745, 0.2755, 0.2765, 0.2775, 0.2785, 0.2795, 0.2805, 0.2815, 0.2825, 0.2835, 0.2845, 0.2855, 0.2865, 0.2875, 0.2885, 0.2895, 0.2905, 0.2915, 0.2925, 0.2935, 0.2945, 0.2955, 0.2965, 0.2975, 0.2985, 0.2995, 0.3005, 0.3015, 0.3025, 0.3035, 0.3045, 0.3055, 0.3065, 0.3075, 0.3085, 0.3095, 0.3105, 0.3115, 0.3125, 0.3135, 0.3145, 0.3155, 0.3165, 0.3175, 0.3185, 0.3195, 0.3205, 0.3215, 0.3225, 0.3235, 0.3245, 0.3255, 0.3265, 0.3275, 0.3285, 0.3295, 0.3305, 0.3315, 0.3325, 0.3335, 0.3345, 0.3355, 0.3365, 0.3375, 0.3385, 0.3395, 0.3405, 0.3415, 0.3425, 0.3435, 0.3445, 0.3455, 0.3465, 0.3475, 0.3485, 0.3495, 0.3505, 0.3515, 0.3525, 0.3535, 0.3545, 0.3555, 0.3565, 0.3575, 0.3585, 0.3595, 0.3605, 0.3615, 0.3625, 0.3635, 0.3645, 0.3655, 0.3665, 0.3675, 0.3685, 0.3695, 0.3705, 0.3715, 0.3725, 0.3735, 0.3745, 0.3755, 0.3765, 0.3775, 0.3785, 0.3795, 0.3805, 0.3815, 0.3825, 0.3835, 0.3845, 0.3855, 0.3865, 0.3875, 0.3885, 0.3895, 0.3905, 0.3915, 0.3925, 0.3935, 0.3945, 0.3955, 0.3965, 0.3975, 0.3985, 0.3995, 0.4005, 0.4015, 0.4025, 0.4035, 0.4045, 0.4055, 0.4065, 0.4075, 0.4085, 0.4095, 0.4105, 0.4115, 0.4125, 0.4135, 0.4145, 0.4155, 0.4165, 0.4175, 0.4185, 0.4195, 0.4205, 0.4215, 0.4225, 0.4235, 0.4245, 0.4255, 0.4265, 0.4275, 0.4285, 0.4295, 0.4305, 0.4315, 0.4325, 0.4335, 0.4345, 0.4355, 0.4365, 0.4375, 0.4385, 0.4395, 0.4405, 0.4415, 0.4425, 0.4435, 0.4445, 0.4455, 0.4465, 0.4475, 0.4485, 0.4495, 0.4505, 0.4515, 0.4525, 0.4535, 0.4545, 0.4555, 0.4565, 0.4575, 0.4585, 0.4595, 0.4605, 0.4615, 0.4625, 0.4635, 0.4645, 0.4655, 0.4665, 0.4675, 0.4685, 0.4695, 0.4705, 0.4715, 0.4725, 0.4735, 0.4745, 0.4755, 0.4765, 0.4775, 0.4785, 0.4795, 0.4805, 0.4815, 0.4825, 0.4835, 0.4845, 0.4855, 0.4865, 0.4875, 0.4885, 0.4895, 0.4905, 0.4915, 0.4925, 0.4935, 0.4945, 0.4955, 0.4965, 0.4975, 0.4985, 0.4995, 0.5005, 0.5015, 0.5025, 0.5035, 0.5045, 0.5055, 0.5065, 0.5075, 0.5085, 0.5095, 0.5105, 0.5115, 0.5125, 0.5135, 0.5145, 0.5155, 0.5165, 0.5175, 0.5185, 0.5195, 0.5205, 0.5215, 0.5225, 0.5235, 0.5245, 0.5255, 0.5265, 0.5275, 0.5285, 0.5295, 0.5305, 0.5315, 0.5325, 0.5335, 0.5345, 0.5355, 0.5365, 0.5375, 0.5385, 0.5395, 0.5405, 0.5415, 0.5425, 0.5435, 0.5445, 0.5455, 0.5465, 0.5475, 0.5485, 0.5495, 0.5505, 0.5515, 0.5525, 0.5535, 0.5545, 0.5555, 0.5565, 0.5575, 0.5585, 0.5595, 0.5605, 0.5615, 0.5625, 0.5635, 0.5645, 0.5655, 0.5665, 0.5675, 0.5685, 0.5695, 0.5705, 0.5715, 0.5725, 0.5735, 0.5745, 0.5755, 0.5765, 0.5775, 0.5785, 0.5795, 0.5805, 0.5815, 0.5825, 0.5835, 0.5845, 0.5855, 0.5865, 0.5875, 0.5885, 0.5895, 0.5905, 0.5915, 0.5925, 0.5935, 0.5945, 0.5955, 0.5965, 0.5975, 0.5985, 0.5995, 0.6005, 0.6015, 0.6025, 0.6035, 0.6045, 0.6055, 0.6065, 0.6075, 0.6085, 0.6095, 0.6105, 0.6115, 0.6125, 0.6135, 0.6145, 0.6155, 0.6165, 0.6175, 0.6185, 0.6195, 0.6205, 0.6215, 0.6225, 0.6235, 0.6245, 0.6255, 0.6265, 0.6275, 0.6285, 0.6295, 0.6305, 0.6315, 0.6325, 0.6335, 0.6345, 0.6355, 0.6365, 0.6375, 0.6385, 0.6395, 0.6405, 0.6415, 0.6425, 0.6435, 0.6445, 0.6455, 0.6465, 0.6475, 0.6485, 0.6495, 0.6505, 0.6515, 0.6525, 0.6535, 0.6545, 0.6555, 0.6565, 0.6575, 0.6585, 0.6595, 0.6605, 0.6615, 0.6625, 0.6635, 0.6645, 0.6655, 0.6665, 0.6675, 0.6685, 0.6695, 0.6705, 0.6715, 0.6725, 0.6735, 0.6745, 0.6755, 0.6765, 0.6775, 0.6785, 0.6795, 0.6805, 0.6815, 0.6825, 0.6835, 0.6845, 0.6855, 0.6865, 0.6875, 0.6885, 0.6895, 0.6905, 0.6915, 0.6925, 0.6935, 0.6945, 0.6955, 0.6965, 0.6975, 0.6985, 0.6995, 0.7005, 0.7015, 0.7025, 0.7035, 0.7045, 0.7055, 0.7065, 0.7075, 0.7085, 0.7095, 0.7105, 0.7115, 0.7125, 0.7135, 0.7145, 0.7155, 0.7165, 0.7175, 0.7185, 0.7195, 0.7205, 0.7215, 0.7225, 0.7235, 0.7245, 0.7255, 0.7265, 0.7275, 0.7285, 0.7295, 0.7305, 0.7315, 0.7325, 0.7335, 0.7345, 0.7355, 0.7365, 0.7375, 0.7385, 0.7395, 0.7405, 0.7415, 0.7425, 0.7435, 0.7445, 0.7455, 0.7465, 0.7475, 0.7485, 0.7495, 0.7505, 0.7515, 0.7525, 0.7535, 0.7545, 0.7555, 0.7565, 0.7575, 0.7585, 0.7595, 0.7605, 0.7615, 0.7625, 0.7635, 0.7645, 0.7655, 0.7665, 0.7675, 0.7685, 0.7695, 0.7705, 0.7715, 0.7725, 0.7735, 0.7745, 0.7755, 0.7765, 0.7775, 0.7785, 0.7795, 0.7805, 0.7815, 0.7825, 0.7835, 0.7845, 0.7855, 0.7865, 0.7875, 0.7885, 0.7895, 0.7905, 0.7915, 0.7925, 0.7935, 0.7945, 0.7955, 0.7965, 0.7975, 0.7985, 0.7995, 0.8005, 0.8015, 0.8025, 0.8035, 0.8045, 0.8055, 0.8065, 0.8075, 0.8085, 0.8095, 0.8105, 0.8115, 0.8125, 0.8135, 0.8145, 0.8155, 0.8165, 0.8175, 0.8185, 0.8195, 0.8205, 0.8215, 0.8225, 0.8235, 0.8245, 0.8255, 0.8265, 0.8275, 0.8285, 0.8295, 0.8305, 0.8315, 0.8325, 0.8335, 0.8345, 0.8355, 0.8365, 0.8375, 0.8385, 0.8395, 0.8405, 0.8415, 0.8425, 0.8435, 0.8445, 0.8455, 0.8465, 0.8475, 0.8485, 0.8495, 0.8505, 0.8515, 0.8525, 0.8535, 0.8545, 0.8555, 0.8565, 0.8575, 0.8585, 0.8595, 0.8605, 0.8615, 0.8625, 0.8635, 0.8645, 0.8655, 0.8665, 0.8675, 0.8685, 0.8695, 0.8705, 0.8715, 0.8725, 0.8735, 0.8745, 0.8755, 0.8765, 0.8775, 0.8785, 0.8795, 0.8805, 0.8815, 0.8825, 0.8835, 0.8845, 0.8855, 0.8865, 0.8875, 0.8885, 0.8895, 0.8905, 0.8915, 0.8925, 0.8935, 0.8945, 0.8955, 0.8965, 0.8975, 0.8985, 0.8995, 0.9005, 0.9015, 0.9025, 0.9035, 0.9045, 0.9055, 0.9065, 0.9075, 0.9085, 0.9095, 0.9105, 0.9115, 0.9125, 0.9135, 0.9145, 0.9155, 0.9165, 0.9175, 0.9185, 0.9195, 0.9205, 0.9215, 0.9225, 0.9235, 0.9245, 0.9255, 0.9265, 0.9275, 0.9285, 0.9295, 0.9305, 0.9315, 0.9325, 0.9335, 0.9345, 0.9355, 0.9365, 0.9375, 0.9385, 0.9395, 0.9405, 0.9415, 0.9425, 0.9435, 0.9445, 0.9455, 0.9465, 0.9475, 0.9485, 0.9495, 0.9505, 0.9515, 0.9525, 0.9535, 0.9545, 0.9555, 0.9565, 0.9575, 0.9585, 0.9595, 0.9605, 0.9615, 0.9625, 0.9635, 0.9645, 0.9655, 0.9665, 0.9675, 0.9685, 0.9695, 0.9705, 0.9715, 0.9725, 0.9735, 0.9745, 0.9755, 0.9765, 0.9775, 0.9785, 0.9795, 0.9805, 0.9815, 0.9825, 0.9835, 0.9845, 0.9855, 0.9865, 0.9875, 0.9885, 0.9895, 0.9905, 0.9915, 0.9925, 0.9935, 0.9945, 0.9955, 0.9965, 0.9975, 0.9985, 0.9995];
let data0 = [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0068466305166028, 0.01517996384993614, 0.023513297183269482, 0.031846630516602824, 0.04017996384993616, 0.04851329718326951, 0.056846630516602846, 0.06517996384993618, 0.07351329718326953, 0.08184663051660288, 0.0901799638499362, 0.09851329718326955, 0.1068466305166029, 0.11517996384993623, 0.12351329718326957, 0.13184663051660292, 0.14017996384993625, 0.1485132971832696, 0.15684663051660286, 0.1651799638499362, 0.17351329718326952, 0.18184663051660288, 0.1901799638499362, 0.19851329718326954, 0.2068466305166029, 0.21517996384993623, 0.22351329718326957, 0.23184663051660293, 0.24017996384993626, 0.2485132971832696, 0.25684663051660295, 0.2651799638499363, 0.2735132971832696, 0.28184663051660297, 0.29017996384993583, 0.2985132971832692, 0.30684663051660255, 0.31517996384993585, 0.3235132971832692, 0.33184663051660257, 0.3401799638499359, 0.34851329718326923, 0.3568466305166026, 0.3651799638499359, 0.37351329718326925, 0.3818466305166026, 0.3901799638499359, 0.3985132971832693, 0.40684663051660264, 0.41517996384993594, 0.4235132971832693, 0.43184663051660266, 0.44017996384993596, 0.4485132971832693, 0.4568466305166027, 0.465179963849936, 0.47351329718326934, 0.4818466305166027, 0.490179963849936, 0.49851329718326937, 0.5068466305166027, 0.515179963849936, 0.5235132971832694, 0.5318466305166027, 0.540179963849936, 0.5485132971832695, 0.5568466305166028, 0.5651799638499361, 0.5735132971832695, 0.5818466305166028, 0.5901799638499361, 0.5985132971832695, 0.6068466305166028, 0.6151799638499361, 0.6235132971832695, 0.6318466305166028, 0.6401799638499361, 0.6485132971832696, 0.6568466305166029, 0.6651799638499362, 0.6735132971832696, 0.6818466305166029, 0.6901799638499362, 0.6985132971832696, 0.7068466305166029, 0.7151799638499362, 0.7235132971832696, 0.7318466305166029, 0.7401799638499362, 0.7485132971832696, 0.756846630516603, 0.7651799638499363, 0.7735132971832697, 0.781846630516603, 0.7901799638499363, 0.7985132971832697, 0.806846630516603, 0.8151799638499363, 0.8235132971832692, 0.8318466305166026, 0.840179963849936, 0.8485132971832692, 0.8568466305166027, 0.8651799638499358, 0.8735132971832693, 0.8818466305166025, 0.890179963849936, 0.8985132971832692, 0.9068466305166027, 0.9151799638499358, 0.9235132971832694, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.9274526028125225, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0];
let data1 = [4.7497782e-21, 1.2236753e-11, 1.4130822e-11, 1.4288661e-11, 3.4924231e-11, 2.9147543e-11, 3.5115528e-11, 4.4161819e-11, 3.215849e-11, 5.8045656e-11, 5.6981576e-11, 7.4404121e-11, 5.3422194e-11, 5.1797341e-11, 3.9958742e-11, 4.3513123e-11, 4.3012595e-11, 2.8551867e-11, 1.0164615e-11, 3.6251191e-11, 1.5676756e-11, 4.3760219e-11, 2.087822e-11, 3.2388469e-11, 1.0404042e-13, 2.2323671e-11, 5.9775439e-12, 1.5878278e-11, 2.5413204e-12, 1.3496131e-11, 9.0037193e-12, 6.4866305e-12, 3.0908672e-12, 9.3806435e-12, 1.8018999e-11, 3.9618166e-11, 3.3207453e-11, 3.636568e-11, 1.6921461e-11, 2.4158606e-11, 1.5541036e-11, 2.1612839e-11, 2.2662135e-12, 3.6244967e-11, 2.1388863e-11, 4.0860982e-11, 2.0243752e-11, 4.4159152e-11, 3.7287927e-11, 4.503816e-11, 4.2794622e-11, 3.2505514e-11, 3.5482004e-11, 3.3629729e-11, 5.203321e-11, 3.059744e-11, 3.5895609e-11, 2.4423153e-11, 2.8592995e-11, 9.0477364e-12, 1.1918963e-11, 1.11291e-11, 9.5233656e-12, 1.9686646e-11, 1.4684259e-11, 1.0456061e-11, 1.326493e-11, 1.6252424e-11, 1.8960364e-11, 1.2528977e-11, 1.0733391e-11, 7.5758314e-12, 1.6704377e-11, 3.1431097e-12, 2.2738832e-12, 1.7303387e-12, 6.4157141e-12, 6.9744867e-12, 1.1955532e-11, 1.2810531e-12, 2.9376966e-12, 5.1365506e-12, 6.5088614e-12, 5.6347441e-12, 8.9720403e-12, 5.0112797e-12, 1.8458058e-11, 3.0491843e-12, 6.0556854e-12, 1.5169225e-12, 1.0354577e-11, 1.6511413e-11, 1.0133936e-12, 1.1652192e-11, 3.9020711e-12, 7.5887253e-12, 1.191296e-11, 1.6945582e-11, 6.168507e-12, 3.7362289e-12, 1.430978e-11, 1.3481458e-11, 1.6208962e-11, 1.5379529e-11, 1.7487792e-11, 1.6227303e-11, 2.3599166e-11, 2.3986428e-11, 2.2877441e-11, 2.8995818e-11, 2.9717098e-12, 1.2812643e-11, 3.9682079e-12, 8.8532164e-12, 1.4168058e-11, 1.9919292e-11, 2.080119e-11, 3.3138871e-11, 2.4218295e-11, 2.2538532e-11, 1.2056572e-11, 2.2702373e-11, 2.0471618e-11, 2.1337621e-11, 9.2228044e-12, 1.1923409e-11, 1.0586334e-12, 7.2505939e-13, 2.3409092e-13, 9.9634254e-12, 1.9138767e-11, 2.1332952e-11, 3.4972806e-11, 1.2183732e-11, 2.5995097e-11, 4.4557306e-12, 1.2136936e-11, 5.6049548e-12, 2.6847761e-11, 8.9601469e-13, 2.9922622e-11, 2.5058955e-11, 1.8621566e-11, 9.110872e-12, 1.7125318e-11, 1.9371191e-11, 1.1349964e-12, 1.0734169e-12, 3.3798683e-12, 2.0197289e-11, 9.8479361e-12, 8.9555895e-12, 4.0984808e-12, 1.52756e-11, 2.0106143e-11, 4.3018931e-12, 3.1851594e-11, 1.9488014e-11, 2.5790573e-11, 1.2311893e-11, 1.3165891e-11, 4.6352448e-12, 1.1384311e-11, 5.0336217e-12, 4.7395075e-12, 1.0127489e-11, 2.3451887e-11, 2.9249694e-11, 2.1945746e-11, 2.2225633e-11, 8.1962946e-12, 3.4067677e-12, 1.1086306e-11, 1.9424656e-11, 1.276318e-11, 4.0311213e-11, 4.1533465e-11, 2.0941134e-11, 3.2442712e-11, 1.8077688e-11, 3.7180997e-11, 1.4652802e-11, 1.8750616e-11, 1.1626293e-11, 3.7798014e-12, 3.7143316e-12, 1.4391923e-11, 4.6232401e-12, 1.3459005e-11, 6.2703243e-12, 1.3369415e-11, 2.2801078e-12, 1.1086528e-11, 1.4684592e-12, 1.151425e-11, 1.0592781e-11, 2.7326391e-11, 1.7531254e-11, 1.7685202e-11, 5.8771716e-12, 4.6834857e-12, 1.7814919e-11, 1.283832e-13, 2.6063234e-11, 8.4499487e-12, 3.4453048e-11, 2.7307051e-11, 2.5739331e-11, 2.6164607e-11, 1.5238919e-11, 2.7857709e-11, 2.8230743e-11, 2.3899394e-11, 4.5124193e-11, 2.07684e-11, 2.5994874e-11, 1.2769738e-11, 7.5410401e-12, 4.1377182e-12, 9.3713065e-12, 7.1242115e-12, 1.8334232e-11, 3.5548474e-11, 1.4479624e-11, 2.4772844e-11, 3.0732159e-11, 2.7721323e-11, 2.8178611e-11, 2.0658579e-11, 3.5832584e-11, 3.0355346e-11, 2.908852e-11, 2.9166884e-12, 2.6237079e-11, 1.2974706e-11, 2.4185616e-11, 1.540943e-11, 2.5027499e-11, 5.3102848e-12, 2.6987816e-11, 2.0652577e-11, 3.3621837e-11, 2.9035055e-11, 3.0026996e-11, 1.9403536e-11, 2.2531196e-11, 2.1464448e-11, 1.056188e-11, 1.4717828e-11, 4.7268359e-12, 1.2038342e-11, 8.5823334e-12, 1.8288437e-11, 1.9149215e-11, 8.5026358e-12, 1.7856269e-11, 6.8202046e-12, 1.0678592e-12, 1.3995325e-11, 1.7243698e-11, 2.3217129e-11, 1.1335292e-11, 1.1618179e-11, 5.3917608e-12, 2.3009937e-11, 1.0752732e-11, 3.0797629e-11, 1.3905401e-11, 2.3099861e-11, 8.8703342e-12, 1.6012664e-11, 1.2647023e-11, 1.0079582e-11, 1.6073021e-11, 2.722613e-12, 2.6127481e-11, 1.3952975e-11, 5.9724308e-12, 7.044625e-12, 1.5309391e-11, 1.3658194e-11, 8.9471418e-12, 2.529249e-11, 4.328459e-12, 2.3591385e-11, 5.059743e-12, 1.1537704e-11, 1.247918e-11, 9.452338e-12, 2.4481175e-11, 2.0175058e-11, 2.3537142e-11, 6.6256846e-11, 6.1706634e-11, 1.2504701e-10, 1.3924097e-10, 2.4660067e-10, 3.2732035e-10, 5.094611e-10, 7.2167552e-10, 1.0844473e-09, 1.5062619e-09, 2.1630591e-09, 3.0315697e-09, 4.2715788e-09, 5.9608509e-09, 8.3437843e-09, 1.1625095e-08, 1.6157491e-08, 2.237713e-08, 3.0939687e-08, 4.2591514e-08, 5.8554935e-08, 8.0188008e-08, 1.0956633e-07, 1.4918876e-07, 2.027e-07, 2.7453773e-07, 3.7079022e-07, 4.9929447e-07, 6.7032184e-07, 8.9726275e-07, 1.1974227e-06, 1.5931029e-06, 2.1130083e-06, 2.7938138e-06, 3.6823602e-06, 4.8380822e-06, 6.336126e-06, 8.27101e-06, 1.0761312e-05, 1.3954958e-05, 1.8035783e-05, 2.3230923e-05, 2.9820083e-05, 3.8145661e-05, 4.8625074e-05, 6.1764365e-05, 7.8173982e-05, 9.8586305e-05, 0.00012387544, 0.00015507864, 0.0001934198, 0.00024033476, 0.0002974976, 0.00036684837, 0.00045062101, 0.00055137138, 0.00067200382, 0.00081579642, 0.00098642278, 0.0011879702, 0.0014249526, 0.0017023168, 0.002025442, 0.0024001302, 0.0028325882, 0.0033293991, 0.0038974847, 0.0045440572, 0.005276562, 0.0061026118, 0.0070299137, 0.0080661894, 0.0092190931, 0.010496126, 0.011904552, 0.013451315, 0.015142962, 0.016985572, 0.018984691, 0.021145281, 0.023471678, 0.025967555, 0.028635909, 0.031479046, 0.034498587, 0.037695476, 0.041070007, 0.044621845, 0.04835007, 0.052253211, 0.056329295, 0.060575892, 0.064990167, 0.069568924, 0.074308664, 0.079205625, 0.084255833, 0.089455144, 0.094799285, 0.10028389, 0.10590454, 0.11165678, 0.11753617, 0.12353828, 0.12965873, 0.13589322, 0.14223751, 0.14868746, 0.15523903, 0.1618883, 0.16863145, 0.1754648, 0.18238479, 0.18938797, 0.19647105, 0.20363084, 0.21086428, 0.21816844, 0.22554052, 0.23297781, 0.24047775, 0.24803786, 0.2556558, 0.2633293, 0.27105622, 0.27883449, 0.28666216, 0.29453734, 0.30245824, 0.31042314, 0.31843041, 0.32647848, 0.33456586, 0.3426911, 0.35085286, 0.35904979, 0.36728075, 0.37554428, 0.38383987, 0.39216506, 0.40052173, 0.40890302, 0.41731947, 0.42574855, 0.43422869, 0.44269034, 0.45124763, 0.45971518, 0.46837279, 0.47681342, 0.48558203, 0.49399581, 0.50280125, 0.51131554, 0.51987967, 0.52886102, 0.53664995, 0.54668062, 0.55314771, 0.56469983, 0.56982903, 0.5827048, 0.58713623, 0.59985871, 0.60559039, 0.61618948, 0.62421812, 0.63250928, 0.64190155, 0.64973676, 0.65867379, 0.66728178, 0.67543557, 0.68436249, 0.69251268, 0.70116246, 0.70940041, 0.71803499, 0.72591586, 0.73465007, 0.7421754, 0.75064661, 0.75785453, 0.76589282, 0.77257401, 0.78002765, 0.78616575, 0.79272695, 0.79838116, 0.80388596, 0.80906539, 0.81353354, 0.81825345, 0.82185202, 0.82608446, 0.82915378, 0.83282197, 0.83569667, 0.83878624, 0.84172396, 0.84429782, 0.84742436, 0.84965847, 0.85300593, 0.85521123, 0.85880565, 0.86132943, 0.86516801, 0.86828684, 0.8723899, 0.87638194, 0.88097185, 0.88619249, 0.89157407, 0.89817, 0.90433351, 0.91190589, 0.91831494, 0.92592112, 0.9316993, 0.9380621, 0.94212677, 0.94587885, 0.94717707, 0.94742498, 0.94767951, 0.94780527, 0.94803564, 0.94820734, 0.94830276, 0.94855132, 0.94874752, 0.94888718, 0.94845965, 0.94872855, 0.94879579, 0.94921478, 0.94945263, 0.94948663, 0.95001979, 0.9507142, 0.94977329, 0.95073316, 0.95280442, 0.94948042, 0.95051732, 0.95976221, 0.94700999, 0.92703429, 0.92353142, 0.92139847, 0.92149921, 0.92694969, 0.93111875, 0.93134918, 0.93179552, 0.9317036, 0.93019943, 0.92913487, 0.92907245, 0.92909259, 0.9291059, 0.9291442, 0.92915769, 0.92922451, 0.92924203, 0.92911433, 0.92899133, 0.92898409, 0.92899886, 0.92900657, 0.92900604, 0.92900255, 0.92900176, 0.92900006, 0.92899903, 0.92899627, 0.92899873, 0.92900462, 0.9290154, 0.92902768, 0.92904411, 0.92906035, 0.9290786, 0.92909653, 0.9291185, 0.92914032, 0.92916527, 0.92919056, 0.9292208, 0.92925171, 0.92928876, 0.92933076, 0.92937489, 0.92942012, 0.92947584, 0.92953381, 0.9295991, 0.92966776, 0.92974198, 0.92981911, 0.92989924, 0.92998185, 0.93006832, 0.93015723, 0.930241, 0.93032401, 0.93040384, 0.93048168, 0.93054887, 0.93061051, 0.93066871, 0.93073158, 0.93079108, 0.93084464, 0.93089778, 0.93094338, 0.93099075, 0.93103015, 0.93107199, 0.93110975, 0.93114968, 0.93118829, 0.93122538, 0.93126367, 0.93129719, 0.93133572, 0.93136851, 0.93140868, 0.93144301, 0.93148121, 0.93151575, 0.93155226, 0.93158905, 0.93162476, 0.93166393, 0.93169731, 0.93173935, 0.93177342, 0.9318156, 0.93185112, 0.93189142, 0.93192729, 0.93196446, 0.93200387, 0.93203979, 0.93208172, 0.93211479, 0.93215424, 0.93218187, 0.93221582, 0.93224005, 0.9322631, 0.93227917, 0.93228247, 0.93227121, 0.93224052, 0.93218199, 0.93208667, 0.93194139, 0.93172852, 0.9314245, 0.93099797, 0.93040702, 0.92959603, 0.92849123, 0.92699517, 0.92497967, 0.92227668, 0.91866727, 0.91386765, 0.90751276, 0.89913639, 0.88814896, 0.87381251, 0.85521555, 0.83125048, 0.80060084, 0.76175515, 0.71307468, 0.65297275, 0.58034371, 0.49543106, 0.40111096, 0.30400643, 0.21385279, 0.13973194, 0.085672249, 0.050029818, 0.028248231, 0.01561467, 0.0085251238, 0.0046229516, 0.0024979795, 0.0013473218, 0.00072604692, 0.00039108398, 0.00021061359, 0.00011341244, 6.1068151e-05, 3.2881948e-05, 1.7704928e-05, 9.5328448e-06, 5.1327199e-06, 2.7634876e-06, 1.487836e-06, 8.0098752e-07, 4.3120483e-07, 2.3210337e-07, 1.2492882e-07, 6.7187527e-08, 3.6127854e-08, 1.9352349e-08, 1.0404359e-08, 5.5210503e-09, 2.9529626e-09, 1.5346396e-09, 7.9428198e-10, 3.6861033e-10, 1.2419183e-10, 4.3436841e-11, 1.3257637e-11, 1.4870166e-11, 1.0206285e-11, 2.4490177e-11, 2.4370081e-11, 1.9556357e-11, 3.4978943e-11, 2.4217574e-11, 2.8594202e-11, 2.1797559e-11, 2.6848701e-11, 2.4967454e-11, 4.3702007e-11, 1.6981616e-11, 1.2151798e-11, 3.6970075e-12, 3.3598392e-11, 6.8825519e-12, 2.0689279e-11, 1.3582629e-11, 3.5937048e-11, 2.1156788e-11, 3.1031754e-11, 3.8754867e-11, 5.7551244e-11, 1.2017272e-11, 3.1834355e-12, 7.7792771e-12, 5.2228499e-12, 8.825401e-12, 2.2278166e-11, 2.3018056e-11, 1.0137468e-11, 2.6215477e-11, 1.8272039e-11, 1.0712309e-11, 7.3875772e-12, 1.0804989e-11, 8.8561465e-12, 3.9837509e-11, 2.3096973e-11, 6.3352043e-11, 2.9827795e-11, 7.8074697e-11, 3.5060857e-11, 6.8725291e-11, 6.4160861e-11, 4.8119258e-11, 4.3734306e-11, 5.0335154e-11, 8.2433787e-11, 2.8126692e-11, 5.3756784e-11, 1.8430428e-11, 2.9340196e-11, 3.4874053e-11, 1.5347553e-11, 1.4757284e-11, 2.7799813e-11, 5.3875659e-12, 1.7271756e-11, 2.7111536e-11, 1.7808525e-11, 1.3187377e-11, 6.0115774e-12, 2.0383045e-11, 3.0349248e-11, 6.6302612e-12, 1.3277616e-11, 1.5641245e-11, 2.5366146e-11, 1.2476235e-11, 1.1086805e-11, 9.6080238e-12, 2.9373495e-11, 5.0011937e-12, 3.4103085e-12, 7.4496232e-12, 2.1377889e-11, 1.5513601e-11, 3.2097745e-11, 2.0315671e-11, 1.5045981e-11, 6.3766385e-13, 5.3094257e-12, 2.071847e-11, 6.0062497e-12, 5.9049116e-12, 4.2392826e-11, 1.8420105e-11, 2.1885356e-11, 8.4259315e-12, 2.8205943e-11, 2.5683036e-12, 2.7446629e-11, 3.480124e-12, 3.6805801e-13, 3.8837336e-11, 8.7038619e-12, 2.6307602e-11, 1.5869117e-11, 7.3966789e-13, 1.2958395e-11, 1.394325e-11, 4.6496739e-12, 1.5903303e-11, 2.8199838e-11, 2.5214417e-11, 8.5717784e-12, 2.6207041e-11, 6.0112444e-12, 4.0859325e-11, 2.4191602e-11, 3.8050274e-11, 2.4840254e-11, 2.8883342e-11, 2.9577836e-12, 8.4301493e-12, 1.0224377e-11, 2.481528e-11, 2.9430213e-11, 1.1148851e-11, 2.2952014e-11, 5.6299781e-12, 2.6766898e-11, 4.9046285e-12, 5.1555983e-11, 1.0087076e-11, 4.4939041e-11, 1.4179447e-11, 2.3963397e-11, 2.3591898e-12, 1.9854821e-11, 1.3202472e-11, 3.1101791e-11, 1.3970666e-11, 2.8291852e-11, 3.9392531e-11, 1.6398339e-12, 1.9717077e-12, 3.3493946e-11, 1.4251482e-11, 2.225519e-11, 3.5135667e-11, 3.6623549e-11, 5.2043471e-11, 2.0132197e-11, 1.4937762e-11, 7.5046765e-12, 1.3253197e-11, 5.6778167e-12, 1.659369e-11, 7.4506221e-12, 8.1147027e-12, 1.4139045e-11, 2.3932651e-12, 4.2783971e-12, 2.1105952e-11, 5.3867889e-12, 1.8312441e-11, 9.8047062e-12, 9.3391949e-12, 3.6215311e-12, 1.3048523e-11, 4.0050618e-11, 1.8217429e-11, 1.684487e-11, 2.7428204e-11, 1.2258796e-11, 2.2140089e-11, 2.0840342e-11, 4.178391e-12, 1.6649631e-11, 1.2855725e-11, 3.9175537e-12, 1.789277e-11, 2.852261e-11, 3.6098545e-11, 6.5570048e-12, 3.2203301e-11, 9.8111439e-12, 1.2932422e-11, 2.8110487e-12, 1.2827977e-11, 7.5176629e-12, 2.5976061e-11, 2.329077e-11, 3.8056933e-11, 4.690975e-11, 3.7367213e-11, 5.2457148e-11, 3.9460349e-11, 4.9403576e-11, 6.3205752e-11, 7.2271235e-11, 4.9367614e-11, 4.5474479e-11, 1.1120325e-11, 1.9082965e-11, 9.2018946e-12, 2.3865167e-11, 4.1185538e-11, 3.977624e-11, 1.6751302e-11, 8.4957471e-12, 3.4906463e-11, 2.2628576e-11, 2.4532244e-11, 4.0878083e-11, 4.5074011e-11, 1.5930275e-11, 8.4807628e-12, 2.2477734e-11, 9.4470816e-12, 3.760119e-11, 2.9787615e-12, 4.1064887e-11, 1.2349257e-12, 2.1885134e-11, 1.5716833e-12, 8.662683e-12, 1.085083e-11, 6.3942867e-12, 1.9193516e-11, 3.2279888e-11, 2.5822778e-11, 4.105412e-11, 1.8547638e-11, 2.2644559e-11, 4.7690819e-11, 2.1394316e-11, 6.6005036e-11, 9.6874959e-12, 1.498449e-11, 1.0487767e-11, 4.0695719e-11, 5.1100795e-12, 3.0456913e-13, 2.4637023e-11, 3.9656476e-11, 2.8627056e-11, 5.2256026e-11, 2.4325572e-11, 3.5351108e-11, 3.4459932e-11, 2.6888215e-11, 3.1180264e-11, 7.3392391e-11, 4.5273468e-11, 2.3238491e-11, 4.5416429e-11, 4.9126201e-12, 6.8129583e-12, 4.7035063e-12, 8.8010932e-12, 3.6449177e-11, 3.57498e-11, 6.8845498e-11, 6.3584799e-11, 6.4005468e-11, 4.7890387e-11, 4.5425864e-11, 3.9753042e-11, 5.3676979e-11, 7.4143047e-11, 5.9449474e-11, 7.5752469e-11, 7.8523892e-11, 5.5389292e-11, 6.6257437e-11, 2.5778935e-11, 4.8051218e-11, 2.9114655e-11, 2.0579061e-11, 3.7228137e-11, 1.9723736e-13, 1.6788374e-11, 1.5427803e-11, 3.2430951e-11, 1.5089269e-11, 8.6060757e-12, 4.2812829e-12, 1.4006628e-11, 3.3064175e-12, 1.1991854e-12, 1.1020541e-11, 1.2967097e-20];
let data2 = [3.1587274e-21, 1.3862829e-11, 1.646295e-11, 3.2144263e-11, 1.5146772e-11, 4.8599543e-11, 1.5956976e-11, 4.5079954e-11, 1.1164003e-11, 2.375367e-11, 8.4472809e-12, 9.1891247e-12, 1.5533811e-11, 1.4210519e-11, 3.0230631e-11, 5.3723755e-11, 1.8964366e-11, 3.6470387e-11, 1.7343736e-11, 2.657321e-11, 1.6595779e-11, 2.8922344e-12, 2.2643239e-12, 7.9208543e-13, 1.0798639e-11, 4.4303875e-12, 1.3237586e-11, 3.0215069e-11, 3.847939e-12, 3.3845368e-11, 1.968798e-11, 2.5638958e-11, 3.1805576e-11, 1.4100588e-11, 4.2181828e-11, 9.1871239e-12, 5.4263521e-11, 2.6657799e-11, 2.9521244e-11, 2.6140709e-11, 3.6060228e-11, 3.9825691e-11, 3.693768e-11, 7.9510883e-12, 5.2393906e-11, 7.6993238e-12, 2.9091077e-11, 9.7627919e-12, 2.3350514e-11, 7.4785714e-12, 3.1269923e-11, 1.6848099e-11, 2.0900673e-11, 3.1482227e-12, 1.9616508e-11, 1.3593057e-11, 3.2383244e-11, 2.9686864e-11, 4.1839584e-11, 2.1039505e-11, 4.4363009e-11, 4.5567699e-11, 5.1585925e-11, 5.0254408e-11, 3.1628285e-11, 4.9054942e-11, 6.1561022e-11, 3.6102133e-11, 2.0040228e-11, 1.4764401e-11, 2.1241917e-11, 2.5835813e-11, 3.5030384e-11, 4.1158653e-11, 3.5126977e-11, 4.6989584e-11, 3.8374905e-11, 5.3902491e-11, 2.2804858e-11, 4.116799e-11, 2.2078131e-11, 2.9711651e-11, 2.7927514e-12, 3.6848534e-11, 4.8283198e-12, 1.5137768e-11, 1.8658358e-12, 5.5483772e-12, 4.8572199e-12, 7.2834956e-12, 4.3929285e-12, 9.2330306e-12, 1.2192069e-11, 1.0106481e-11, 1.2977151e-11, 5.4687908e-13, 2.7688531e-13, 5.1474437e-12, 9.0471806e-12, 7.2589305e-12, 1.1016835e-11, 1.5147217e-11, 7.1979068e-12, 1.6878222e-11, 2.5077518e-12, 1.3794247e-11, 2.3397977e-12, 3.3936292e-11, 4.4505064e-12, 4.141553e-11, 2.6810636e-11, 1.4511192e-11, 3.50435e-11, 2.6215738e-12, 1.6870441e-11, 1.6199848e-11, 7.5394839e-12, 4.3516901e-13, 1.8329341e-13, 2.6110141e-13, 3.3873156e-12, 1.4211298e-11, 5.7610154e-12, 2.4711376e-11, 1.6844209e-11, 1.1221025e-11, 1.4642687e-11, 1.7884391e-11, 5.9851024e-12, 1.7921405e-11, 1.2904123e-11, 1.9490681e-11, 1.8598891e-11, 1.2063686e-11, 1.8508411e-11, 3.2205064e-11, 1.1345184e-11, 1.8744058e-11, 2.3708653e-11, 3.811736e-11, 2.6724824e-12, 4.3599156e-11, 2.0617119e-11, 5.1319933e-11, 3.0725823e-11, 5.1442091e-11, 2.3753115e-11, 6.2107567e-11, 3.1677192e-11, 5.5339939e-11, 3.4040666e-11, 2.5940742e-11, 4.6395909e-11, 3.6795625e-11, 3.4361235e-11, 2.2750948e-11, 3.1149432e-11, 2.7081963e-11, 3.2067789e-11, 8.3646933e-12, 4.675705e-11, 9.3488534e-12, 1.7650189e-12, 1.4368581e-11, 1.7560154e-11, 9.1522214e-12, 5.3022816e-12, 5.1575583e-14, 7.4349989e-12, 4.3805903e-12, 1.007169e-11, 7.4106559e-13, 1.807591e-12, 1.2629572e-11, 4.3775891e-12, 1.3309947e-11, 9.9894355e-12, 1.6843987e-11, 1.3354187e-11, 5.8228171e-12, 3.1425539e-12, 2.8588437e-11, 2.262212e-12, 5.8623881e-12, 7.9196316e-12, 5.8783943e-12, 1.2594558e-11, 2.3195342e-11, 1.5702765e-12, 3.1104303e-11, 8.5607694e-12, 4.3086735e-12, 1.2531867e-11, 1.2554209e-11, 1.8647243e-12, 9.3182859e-12, 5.2836077e-12, 2.6260533e-11, 5.2642671e-13, 2.6149379e-11, 6.5936721e-12, 5.6280749e-12, 1.2302667e-11, 1.50195e-11, 3.4461163e-12, 2.4724603e-11, 1.2550652e-11, 1.7924073e-11, 2.4092246e-11, 3.1099635e-11, 3.2873213e-11, 1.9854823e-11, 1.2604229e-11, 7.9728745e-12, 2.7277039e-11, 8.5136401e-12, 1.9863048e-11, 3.4652904e-11, 1.1901511e-11, 2.5563929e-11, 1.2166726e-11, 2.461745e-11, 1.3254037e-12, 7.839267e-12, 9.9509761e-12, 6.5985629e-12, 3.8159266e-12, 9.5138063e-12, 3.7557921e-12, 1.6593e-11, 1.3511137e-11, 7.505693e-12, 4.7188328e-12, 1.596798e-11, 3.1650071e-12, 5.7689072e-13, 4.2908888e-12, 1.1663197e-11, 6.9723748e-12, 2.1425766e-11, 7.5927268e-12, 1.5739669e-11, 6.1834017e-12, 7.7385613e-13, 2.4608447e-12, 1.8791743e-12, 2.8075349e-12, 1.061968e-12, 8.6929319e-12, 1.971688e-11, 3.1628729e-11, 4.7778446e-11, 3.5523798e-11, 3.331405e-11, 3.4131257e-11, 3.3807687e-11, 3.3272812e-11, 2.7870714e-11, 3.199665e-11, 2.726659e-11, 1.8468729e-11, 3.5013599e-11, 4.2273197e-11, 3.6007874e-11, 3.1525578e-11, 1.2414044e-11, 1.5876389e-11, 3.6948795e-12, 3.3746107e-11, 2.5809358e-11, 2.4112477e-11, 4.4271084e-11, 2.9867156e-11, 3.1420648e-11, 1.7371303e-11, 2.4465836e-11, 4.0992588e-12, 2.6491401e-12, 8.1303801e-12, 8.5713292e-12, 2.3663635e-12, 1.5372526e-11, 2.8357792e-11, 1.1904735e-11, 1.3772349e-11, 5.2911662e-12, 1.6268652e-11, 1.8988709e-11, 4.6916e-12, 1.0875669e-11, 1.3120318e-11, 2.6141042e-11, 7.6080661e-11, 9.5989838e-11, 1.4489528e-10, 2.0154195e-10, 3.0967917e-10, 4.3345436e-10, 6.3358286e-10, 8.8771735e-10, 1.2741231e-09, 1.7645955e-09, 2.5038187e-09, 3.4686766e-09, 4.8671167e-09, 6.7767861e-09, 9.4414534e-09, 1.3159652e-08, 1.8246835e-08, 2.5266961e-08, 3.487059e-08, 4.8017711e-08, 6.5968252e-08, 9.0295516e-08, 1.2330064e-07, 1.6787384e-07, 2.2790804e-07, 3.0853965e-07, 4.1644473e-07, 5.6052828e-07, 7.5213878e-07, 1.0063065e-06, 1.3422031e-06, 1.7847702e-06, 2.3659374e-06, 3.1264883e-06, 4.1185031e-06, 5.4079356e-06, 7.0781943e-06, 9.234058e-06, 1.2006817e-05, 1.5560174e-05, 2.0097201e-05, 2.5868814e-05, 3.3183366e-05, 4.2418174e-05, 5.403248e-05, 6.8582362e-05, 8.6737913e-05, 0.00010930182, 0.00013723087, 0.00017165865, 0.00021392091, 0.00026558174, 0.00032846207, 0.00040466801, 0.00049662036, 0.0006070829, 0.0007391898, 0.00089647036, 0.0010828703, 0.0013027684, 0.0015609871, 0.0018627962, 0.0022139076, 0.0026204618, 0.0030890034, 0.0036264471, 0.0042400316, 0.004937265, 0.0057258587, 0.0066136536, 0.0076085392, 0.0087183668, 0.0099508598, 0.011313523, 0.012813554, 0.014457756, 0.016252461, 0.018203457, 0.020315928, 0.022594403, 0.02504272, 0.027663998, 0.030460627, 0.033434265, 0.036585855, 0.039915638, 0.043423192, 0.047107468, 0.050966832, 0.05499912, 0.059201687, 0.063571465, 0.068105018, 0.072798595, 0.077648185, 0.082649568, 0.087798357, 0.093090051, 0.098520067, 0.10408378, 0.10977655, 0.11559376, 0.12153083, 0.12758323, 0.13374653, 0.14001638, 0.14638853, 0.15285883, 0.15942327, 0.16607796, 0.1728191, 0.17964307, 0.18654633, 0.19352548, 0.20057725, 0.20769849, 0.21488617, 0.22213736, 0.22944927, 0.2368192, 0.24424456, 0.25172287, 0.25925174, 0.26682889, 0.2744521, 0.28211929, 0.28982841, 0.29757751, 0.30536475, 0.3131883, 0.32104646, 0.32893756, 0.33686, 0.34481224, 0.35279281, 0.36080028, 0.36883326, 0.37689044, 0.38497053, 0.39307229, 0.40119452, 0.40933606, 0.41749581, 0.42567269, 0.43386567, 0.44207375, 0.45029603, 0.45853158, 0.46677972, 0.47503945, 0.4833107, 0.49159204, 0.49988463, 0.50818626, 0.5164997, 0.52482283, 0.53315832, 0.54150894, 0.5498738, 0.5582359, 0.5665362, 0.57494469, 0.58311624, 0.59136317, 0.59958707, 0.60776687, 0.6159676, 0.624232, 0.6323957, 0.6405601, 0.64871327, 0.65688165, 0.66508144, 0.673247, 0.68143447, 0.68964666, 0.69798511, 0.70610249, 0.71427091, 0.72235892, 0.73037817, 0.73833991, 0.74625126, 0.75411165, 0.76191705, 0.76966278, 0.77734373, 0.78495423, 0.79248781, 0.79993689, 0.80729313, 0.81454716, 0.82168897, 0.82870754, 0.83559096, 0.84232631, 0.84889968, 0.85529632, 0.86150056, 0.86749612, 0.87326608, 0.87879326, 0.88406045, 0.88905082, 0.89374843, 0.89813868, 0.9022091, 0.90594978, 0.90935421, 0.91241975, 0.91514819, 0.91754603, 0.91962462, 0.9214, 0.92289244, 0.92412584, 0.92512669, 0.92592309, 0.92654356, 0.92701596, 0.92736647, 0.92761863, 0.92779284, 0.92791107, 0.9279828, 0.92801901, 0.92802639, 0.92803187, 0.92801278, 0.92799155, 0.9279374, 0.92788026, 0.92781863, 0.92775834, 0.92770019, 0.9276448, 0.92759277, 0.92754454, 0.92750042, 0.92746044, 0.92742444, 0.92739211, 0.92736302, 0.92733674, 0.92731283, 0.9272909, 0.92727063, 0.92725175, 0.92723405, 0.92721737, 0.92720159, 0.92718663, 0.92717241, 0.92715889, 0.92714602, 0.92713376, 0.92712205, 0.92711087, 0.92710016, 0.92708987, 0.92707996, 0.92707038, 0.92706111, 0.92705212, 0.92704341, 0.92703496, 0.92702678, 0.9270189, 0.92701128, 0.92700388, 0.92699672, 0.92698978, 0.92698311, 0.92697697, 0.92697102, 0.9269644, 0.92695882, 0.92695257, 0.92694634, 0.92694042, 0.92693453, 0.92692853, 0.92692233, 0.92691583, 0.92690895, 0.92690158, 0.92689368, 0.92688513, 0.92687566, 0.9268651, 0.92685345, 0.92684118, 0.92682992, 0.92681909, 0.92680727, 0.92679675, 0.92678714, 0.92677884, 0.92677221, 0.9267674, 0.92676447, 0.92676321, 0.92676338, 0.92676473, 0.92676685, 0.92676937, 0.92677188, 0.92677402, 0.92677561, 0.92677665, 0.92677714, 0.92677547, 0.92677222, 0.92676995, 0.92676664, 0.92676285, 0.92675868, 0.92675421, 0.92674948, 0.92674448, 0.92673922, 0.92673375, 0.92672829, 0.92672287, 0.92671729, 0.92671144, 0.92670515, 0.92670037, 0.9266956, 0.9266888, 0.92668168, 0.92667323, 0.92666665, 0.92666105, 0.92665279, 0.92664626, 0.92663888, 0.92663016, 0.92662086, 0.92661089, 0.9266001, 0.92658832, 0.92657535, 0.92656091, 0.92654466, 0.92652616, 0.92650484, 0.92647999, 0.92645069, 0.92641577, 0.92637373, 0.92632266, 0.92626013, 0.92618306, 0.92608752, 0.92596852, 0.92581972, 0.92563309, 0.92539845, 0.92510291, 0.92473014, 0.92425951, 0.92366491, 0.92291337, 0.92196323, 0.92076189, 0.91924297, 0.91732274, 0.91489558, 0.91182839, 0.90795347, 0.90305973, 0.89688164, 0.88908574, 0.87925393, 0.86686358, 0.85126424, 0.83165196, 0.80704406, 0.77626142, 0.73793377, 0.69055942, 0.63268039, 0.56327591, 0.48250857, 0.39285097, 0.30012158, 0.21309309, 0.14045218, 0.086688654, 0.050828674, 0.028729114, 0.015849886, 0.00861729, 0.0046467978, 0.0024949216, 0.0013366319, 0.0007153302, 0.00038263615, 0.00020462933, 0.00010942248, 5.850937e-05, 3.1284829e-05, 1.6727594e-05, 8.9439202e-06, 4.7819894e-06, 2.5567432e-06, 1.3669313e-06, 7.308132e-07, 3.9068546e-07, 2.0884449e-07, 1.1162505e-07, 5.9670655e-08, 3.1888187e-08, 1.7047214e-08, 9.0987698e-09, 4.8644519e-09, 2.5910582e-09, 1.4090592e-09, 7.3607532e-10, 3.7363817e-10, 1.7609201e-10, 6.9859433e-11, 6.505581e-11, 1.1719474e-11, 4.888612e-11, 2.3601888e-12, 3.4779596e-11, 1.1009219e-11, 1.7066971e-11, 4.3198424e-11, 1.6923455e-11, 2.9291692e-11, 6.9220659e-12, 9.226979e-13, 1.9257449e-11, 2.1512192e-11, 2.9423553e-12, 1.0852717e-11, 3.749397e-13, 2.8956266e-11, 2.7232853e-11, 1.094118e-11, 2.3079547e-11, 3.5580423e-12, 8.7943225e-12, 3.5617162e-11, 3.3684635e-12, 1.2529068e-11, 1.9249346e-11, 3.1177378e-11, 1.2765265e-11, 2.028881e-11, 1.8367383e-12, 3.5656121e-11, 1.7748588e-11, 1.607812e-11, 1.6635646e-11, 2.148844e-11, 1.3817604e-11, 2.4028995e-11, 7.6920353e-12, 2.2070828e-11, 2.0256067e-11, 8.1556597e-12, 4.0443095e-12, 1.4599117e-11, 3.5777882e-11, 1.215202e-11, 3.2232604e-11, 1.05194e-11, 1.4869943e-12, 3.0990797e-12, 2.8712522e-11, 1.0117711e-11, 7.1187483e-12, 1.8899935e-11, 2.1021929e-11, 1.7956037e-11, 3.8355509e-11, 4.8117261e-12, 3.4119731e-13, 6.755352e-12, 2.2086811e-11, 1.8748094e-12, 2.1405527e-11, 2.1825197e-11, 1.3863227e-13, 8.1193644e-12, 6.0514244e-12, 1.3537343e-11, 2.5115964e-11, 1.3615816e-11, 3.6313542e-11, 3.1876534e-12, 5.0612307e-11, 1.4525195e-11, 4.9153061e-11, 5.0610199e-12, 3.5477641e-11, 2.0070373e-11, 1.2292317e-11, 2.257996e-11, 2.6219695e-11, 3.3177057e-11, 3.3918834e-11, 2.0201124e-11, 2.9929022e-11, 2.0404911e-11, 1.7744481e-11, 4.564097e-13, 3.0670022e-12, 6.1093637e-12, 4.9195239e-11, 5.3775764e-12, 3.2420629e-11, 3.2356918e-11, 1.1203127e-11, 2.1044239e-11, 7.0392762e-12, 1.0964045e-11, 2.4331011e-11, 1.1977203e-11, 2.7525213e-11, 2.2182711e-11, 3.4249153e-11, 2.6882887e-12, 3.8139957e-12, 1.3752339e-11, 2.0393922e-11, 1.8775732e-11, 1.6146492e-11, 8.4262645e-12, 2.6534919e-11, 1.1913936e-11, 3.2594002e-11, 1.6710789e-11, 2.4528359e-11, 1.5794528e-12, 3.0703987e-11, 1.9988459e-11, 4.2636903e-11, 2.9170263e-11, 1.5225348e-11, 2.5626762e-11, 4.4411928e-11, 1.589842e-11, 4.5259483e-11, 2.0439208e-11, 2.6341012e-11, 1.347341e-11, 2.1255684e-11, 1.0629729e-11, 3.9437928e-11, 2.9967427e-12, 3.1947237e-11, 6.9956554e-12, 6.1998242e-12, 2.0789284e-12, 2.0116436e-11, 4.647676e-12, 1.6522098e-11, 9.3896974e-12, 6.0516465e-12, 8.3840866e-12, 2.1797115e-11, 1.0969372e-11, 1.4066343e-12, 7.4723771e-12, 1.508716e-11, 2.572588e-11, 3.1789957e-11, 3.7379645e-12, 1.4661163e-11, 3.0160668e-11, 3.8229419e-11, 2.3631412e-11, 3.3269737e-11, 1.1988414e-11, 3.4363256e-11, 2.2416021e-11, 1.1728131e-11, 9.317551e-12, 9.889617e-12, 2.6019238e-11, 5.4338506e-12, 3.9482659e-11, 1.4534407e-11, 5.2454928e-11, 5.7924963e-11, 5.1665202e-11, 7.6479705e-11, 5.8635218e-11, 6.9449863e-11, 4.4034991e-11, 3.6424314e-11, 4.3910566e-11, 3.1681627e-11, 2.2541112e-11, 1.4644958e-11, 1.2643947e-11, 1.8225976e-11, 1.0744275e-11, 1.0612192e-11, 2.0740669e-11, 4.3798572e-11, 3.0136138e-11, 1.6759738e-11, 2.4857014e-11, 5.8160049e-12, 1.1487828e-11, 7.7568561e-12, 1.3198587e-11, 4.9671184e-12, 1.9774128e-11, 4.5795254e-12, 1.0955387e-11, 2.7193117e-11, 2.7553294e-12, 2.3722427e-11, 4.8546809e-12, 1.5726156e-11, 1.0720633e-11, 9.6529766e-12, 2.9908599e-12, 2.6848257e-11, 3.5884548e-13, 3.1516578e-11, 5.297738e-11, 3.3170952e-11, 4.9748325e-11, 1.0841063e-11, 5.7032789e-11, 4.068695e-11, 5.5832715e-11, 3.2919771e-11, 4.6057423e-11, 2.3108183e-11, 4.1595996e-11, 1.170904e-11, 1.0863594e-11, 1.5928055e-11, 2.1407081e-11, 1.4800017e-12, 2.8126359e-11, 4.7800593e-11, 5.4824441e-11, 2.0990407e-11, 5.1780192e-11, 1.1011772e-11, 2.5747635e-11, 1.1905057e-11, 1.7807748e-11, 3.2342156e-11, 9.8929469e-12, 1.1113443e-11, 6.7752201e-12, 1.8121308e-11, 6.0684066e-12, 3.6823562e-11, 3.2676027e-11, 3.8488147e-11, 4.7162818e-11, 2.817775e-11, 4.1453479e-11, 3.4595678e-11, 3.2416078e-11, 5.5118354e-11, 5.5519489e-12, 5.4331847e-11, 3.5331795e-12, 3.1377169e-11, 3.4956633e-12, 2.5582031e-11, 2.1050122e-11, 3.4838313e-11, 6.7041836e-12, 3.0708981e-11, 1.1507141e-11, 1.392205e-12, 2.4646346e-12, 2.9236638e-11, 1.18631e-12, 2.1137697e-11, 1.0209059e-11, 7.883501e-12, 6.1566473e-12, 4.3042588e-12, 3.0403524e-11, 2.3630743e-13, 2.2504595e-11, 1.8515117e-11, 1.5644797e-11, 3.6540082e-11, 3.5870451e-11, 3.4634194e-11, 1.2553487e-12, 2.1674356e-11, 2.6223579e-12, 2.6899536e-12, 1.6752412e-11, 1.2854726e-11, 2.237007e-11, 4.7813135e-12, 1.7285852e-11, 6.1368907e-13, 2.2063613e-11, 2.356861e-20];
let data3 = [6.9688318e-22, 1.644272e-11, 6.314786e-12, 3.2875102e-11, 9.8260368e-14, 2.3936075e-11, 2.694969e-11, 4.2087013e-11, 4.6329106e-11, 2.3309053e-11, 3.7687305e-11, 6.6659224e-12, 4.6838192e-12, 9.6655319e-12, 2.0671807e-11, 2.8959359e-11, 2.2623343e-11, 3.6969137e-11, 2.1388752e-11, 3.5699199e-11, 2.9340507e-11, 1.772255e-11, 8.2035196e-12, 1.8482512e-11, 2.4174501e-11, 1.7265261e-11, 2.2804302e-11, 3.1309939e-12, 6.693711e-12, 2.5666191e-11, 2.6720934e-11, 1.9686869e-11, 6.4094895e-12, 2.6156381e-11, 1.178002e-11, 2.1946524e-11, 1.5986765e-11, 4.5602823e-11, 4.2158819e-11, 5.0142253e-11, 3.0869434e-11, 2.972888e-11, 2.2426155e-11, 3.9049945e-11, 2.5087744e-11, 5.3440423e-11, 3.8887215e-11, 6.2241064e-11, 5.2624773e-11, 7.1175534e-11, 4.66878e-11, 4.4403358e-11, 4.6373456e-11, 4.1789009e-11, 4.2431703e-11, 3.5016267e-11, 4.5389963e-11, 4.1522905e-11, 5.6982577e-11, 4.5454432e-11, 6.0826959e-11, 2.1750226e-11, 2.9669301e-11, 1.1971094e-11, 3.0990037e-11, 1.1845156e-11, 2.3249141e-11, 7.7532336e-12, 1.9204681e-11, 3.0587436e-12, 1.4002994e-11, 2.5361628e-11, 9.4072093e-12, 1.9982205e-12, 5.6142917e-12, 8.5617698e-12, 2.0093693e-11, 1.1226582e-12, 1.1819035e-12, 1.1260151e-11, 7.7041034e-12, 6.2246287e-15, 1.140354e-11, 7.8352655e-13, 1.0221303e-11, 2.1752893e-12, 1.9166222e-11, 2.1780015e-11, 1.1090419e-11, 3.5089073e-11, 1.0938137e-11, 2.4513521e-11, 6.9846017e-12, 8.5136401e-12, 1.1986322e-11, 2.5332061e-12, 1.8469396e-12, 9.5819439e-12, 1.2684482e-11, 4.9295813e-12, 1.0067243e-11, 1.4625681e-11, 3.0977587e-12, 1.4397592e-11, 1.1447335e-11, 2.3799577e-11, 5.6455261e-12, 3.5564147e-11, 2.6856209e-11, 3.0241857e-11, 2.6844426e-11, 2.438536e-11, 1.8498852e-11, 1.5597614e-11, 1.3305279e-11, 8.4438352e-12, 1.2783854e-12, 1.7306833e-11, 2.4847206e-11, 9.7293345e-12, 2.3259701e-11, 6.1117071e-12, 2.1805469e-11, 4.1434982e-12, 2.0206737e-11, 1.071116e-11, 8.3765868e-12, 3.1402197e-12, 9.787357e-12, 1.4450279e-11, 1.1241922e-11, 1.0742283e-11, 4.307562e-12, 1.8921794e-12, 1.7633849e-11, 5.7883593e-12, 1.06129e-11, 8.7511769e-13, 1.374067e-11, 8.8962331e-12, 3.0052673e-11, 5.1190994e-12, 1.807691e-11, 2.3518579e-11, 2.1319836e-11, 4.4795191e-14, 1.0434831e-11, 8.6904865e-12, 2.1587385e-11, 4.3709199e-12, 2.9441991e-11, 1.0720275e-11, 7.4697902e-12, 1.0809754e-11, 1.4696486e-11, 1.8840651e-13, 1.442049e-11, 1.9502353e-11, 2.7316276e-11, 1.128316e-11, 9.0713011e-12, 1.2080803e-11, 3.0674025e-11, 3.5719429e-11, 1.915833e-11, 3.1539917e-11, 1.1264042e-11, 1.2411821e-11, 1.2584888e-12, 1.6417377e-11, 1.2998382e-12, 8.0154466e-12, 6.880339e-12, 2.9339062e-11, 2.0188063e-11, 2.6760505e-11, 3.110786e-11, 3.7842921e-11, 1.6658693e-11, 2.7987093e-11, 2.6125925e-11, 2.0261203e-11, 1.1568827e-11, 1.0462064e-11, 6.349355e-12, 1.5180229e-11, 2.04246e-12, 1.7739112e-12, 2.6350346e-11, 8.2377551e-12, 2.5672971e-11, 2.1312055e-11, 2.3692647e-11, 1.5997325e-12, 4.1161543e-12, 9.3535218e-12, 7.0731917e-12, 2.2783516e-11, 2.787416e-12, 1.0895343e-12, 1.0061575e-11, 5.620961e-12, 5.1491111e-12, 9.7073259e-12, 2.690156e-12, 7.136105e-12, 3.9105189e-12, 2.305262e-11, 9.1106498e-12, 1.7891949e-11, 1.515811e-11, 2.2325894e-11, 1.4907234e-11, 3.9668407e-11, 2.296881e-11, 2.0173391e-11, 2.3783237e-11, 1.961962e-11, 3.8646121e-12, 1.466025e-11, 1.2476068e-11, 4.1588375e-12, 1.4916349e-11, 7.0600755e-12, 1.0899789e-12, 1.9369746e-12, 8.2886638e-12, 1.4640909e-11, 6.0101121e-12, 1.5824368e-11, 6.2542069e-12, 3.9383074e-12, 4.4448375e-12, 1.3129099e-11, 5.8919551e-12, 2.6262423e-12, 9.3675273e-12, 7.495578e-12, 9.0743023e-12, 1.9489792e-12, 4.1585041e-12, 1.1289385e-11, 3.254264e-12, 2.727615e-11, 2.6855208e-11, 3.0241524e-11, 1.5608951e-11, 6.0245621e-13, 1.2463396e-11, 2.1251699e-11, 4.9694857e-12, 1.8488292e-11, 1.7308167e-11, 2.8451051e-12, 3.8551974e-11, 7.7724633e-12, 2.0788852e-11, 1.4494296e-11, 1.0015112e-11, 1.7135544e-12, 2.2126038e-11, 1.0695043e-11, 3.7801571e-11, 1.9514913e-11, 1.7230915e-11, 3.0993482e-11, 1.6909679e-11, 3.4409142e-11, 4.2537633e-12, 2.2227744e-11, 3.6879213e-11, 2.2517635e-11, 2.2170278e-11, 7.1939054e-13, 4.0701364e-12, 2.1823143e-11, 8.1009242e-12, 2.3839704e-11, 5.1708973e-12, 1.1727444e-11, 6.3424634e-12, 1.2382143e-11, 3.5569378e-14, 7.6585302e-12, 2.4730716e-12, 7.9594248e-12, 9.8192583e-12, 1.0134825e-11, 9.5584903e-12, 8.8383217e-12, 2.6082353e-12, 1.8433826e-11, 3.5326054e-11, 4.4339666e-11, 1.2091719e-10, 1.7084625e-10, 2.5967619e-10, 3.634545e-10, 5.5290375e-10, 7.9605286e-10, 1.1043283e-09, 1.5812387e-09, 2.2285323e-09, 3.1469275e-09, 4.4479603e-09, 6.2269566e-09, 8.6930763e-09, 1.2104844e-08, 1.6837257e-08, 2.3295801e-08, 3.2212568e-08, 4.4334733e-08, 6.0893725e-08, 8.337456e-08, 1.1391194e-07, 1.5513682e-07, 2.1066381e-07, 2.8526479e-07, 3.8513473e-07, 5.184716e-07, 6.9585825e-07, 9.3118369e-07, 1.2422554e-06, 1.6522398e-06, 2.1907008e-06, 2.8955934e-06, 3.8152406e-06, 5.0109495e-06, 6.5602045e-06, 8.560473e-06, 1.1133904e-05, 1.4432841e-05, 1.864643e-05, 2.4008378e-05, 3.0806173e-05, 3.939168e-05, 5.0193482e-05, 6.373087e-05, 8.0629924e-05, 0.00010164123, 0.00012766014, 0.00015974814, 0.00019915717, 0.00024735442, 0.00030604959, 0.00037722223, 0.00046314986, 0.0005664359, 0.00069003607, 0.00083728324, 0.0010119086, 0.0012180594, 0.0014603103, 0.0017436691, 0.0020735744, 0.0024558851, 0.0028968594, 0.0034031258, 0.003981642, 0.004639646, 0.0053845953, 0.0062241002, 0.0071658477, 0.0082175215, 0.0093867167, 0.010680855, 0.012107097, 0.013672263, 0.015382752, 0.017244475, 0.019262788, 0.021442444, 0.023787554, 0.026301551, 0.028987178, 0.031846482, 0.034880815, 0.038090856, 0.041476629, 0.045037541, 0.048772415, 0.052679542, 0.056756722, 0.061001318, 0.065410308, 0.069980336, 0.074707762, 0.079588712, 0.084619126, 0.089794796, 0.095111412, 0.1005646, 0.10614993, 0.11186299, 0.11769939, 0.12365474, 0.12972476, 0.13590521, 0.14219195, 0.14858094, 0.15506823, 0.16164999, 0.16832252, 0.1750822, 0.18192558, 0.18884927, 0.19585005, 0.20292479, 0.21007048, 0.21728423, 0.22456326, 0.23190489, 0.23930656, 0.2467658, 0.25428023, 0.2618476, 0.26946571, 0.27713249, 0.28484593, 0.2926041, 0.30040517, 0.30824737, 0.31612901, 0.32404845, 0.33200415, 0.33999461, 0.34801838, 0.3560741, 0.36416044, 0.37227612, 0.38041992, 0.38859065, 0.39678719, 0.40500842, 0.4132533, 0.42152079, 0.42980991, 0.4381197, 0.44644922, 0.45479757, 0.46316387, 0.47154727, 0.47994692, 0.48836204, 0.49679172, 0.50523546, 0.5136919, 0.52216136, 0.53064128, 0.53913374, 0.54763397, 0.55614653, 0.56466445, 0.57319362, 0.58172722, 0.59026928, 0.59881622, 0.60736834, 0.61592511, 0.62448513, 0.63304813, 0.64161313, 0.65017946, 0.65874591, 0.6673123, 0.67587697, 0.68443953, 0.69299866, 0.70155352, 0.71010264, 0.71864521, 0.72717945, 0.73570421, 0.74421761, 0.75271796, 0.76120312, 0.76967072, 0.77811816, 0.78654216, 0.79493922, 0.8033049, 0.81163417, 0.81992079, 0.82815731, 0.83633452, 0.84444109, 0.85246275, 0.86038162, 0.86817472, 0.8758128, 0.88325778, 0.89046056, 0.89735742, 0.90386671, 0.90988573, 0.91528921, 0.91993422, 0.92366278, 0.92634106, 0.92784161, 0.92829757, 0.9282984, 0.9282713, 0.92826961, 0.92826595, 0.92826551, 0.92826812, 0.92827398, 0.92827656, 0.92827988, 0.92828388, 0.92829462, 0.92829526, 0.928294, 0.92829365, 0.92829214, 0.92828957, 0.92828995, 0.92828473, 0.92825203, 0.92821711, 0.92810413, 0.92797596, 0.92781776, 0.92765406, 0.92748905, 0.92733948, 0.92722197, 0.92714752, 0.92712163, 0.92711871, 0.92711722, 0.92711593, 0.92711253, 0.92710873, 0.92709741, 0.92708225, 0.92705974, 0.92703606, 0.92701366, 0.9269948, 0.92697989, 0.92696832, 0.92695887, 0.92695033, 0.92694183, 0.92693296, 0.92692374, 0.92691443, 0.92690531, 0.92689663, 0.92688847, 0.92688082, 0.92687357, 0.92686657, 0.92685967, 0.92685275, 0.92684576, 0.92683874, 0.92683178, 0.92682498, 0.92681835, 0.92681197, 0.92680587, 0.92680037, 0.92679479, 0.9267895, 0.9267835, 0.92677779, 0.92677267, 0.92676776, 0.92676293, 0.92675801, 0.92675287, 0.92674735, 0.92674134, 0.92673479, 0.92672762, 0.92671977, 0.92671123, 0.92670216, 0.92669224, 0.92668065, 0.92666776, 0.92665596, 0.9266461, 0.92663748, 0.92662607, 0.9266169, 0.92660872, 0.9266022, 0.92659765, 0.9265949, 0.92659375, 0.92659388, 0.92659495, 0.92659702, 0.92659938, 0.926602, 0.92660442, 0.92660629, 0.92660738, 0.92660746, 0.92660629, 0.92660383, 0.9266003, 0.92659594, 0.92659129, 0.92658667, 0.92658204, 0.92657723, 0.92657201, 0.92656638, 0.92656039, 0.92655414, 0.92654714, 0.92653966, 0.92653448, 0.92652956, 0.92652316, 0.92651643, 0.92650962, 0.9265021, 0.92649434, 0.9264865, 0.92647784, 0.9264687, 0.92645885, 0.9264501, 0.9264418, 0.92643157, 0.92642051, 0.92640881, 0.92639628, 0.92638275, 0.92636804, 0.92635188, 0.92633396, 0.92631386, 0.92629103, 0.9262648, 0.92623425, 0.92619825, 0.92615529, 0.92610345, 0.92604026, 0.9259625, 0.92586606, 0.92574562, 0.92559432, 0.92540335, 0.92516138, 0.92485383, 0.92446195, 0.92396167, 0.92332208, 0.92250349, 0.92145494, 0.92011106, 0.91838798, 0.91617815, 0.91334364, 0.9097076, 0.90504345, 0.8990609, 0.89138843, 0.88155109, 0.86894299, 0.85279399, 0.83213126, 0.80573957, 0.77213127, 0.72955327, 0.67609096, 0.60999109, 0.53041726, 0.43882191, 0.34056529, 0.24517568, 0.1632928, 0.1014608, 0.059707787, 0.033806, 0.018665709, 0.010154046, 0.005479122, 0.0029441638, 0.0015787208, 0.00084568651, 0.00045280042, 0.00024238724, 0.00012973898, 6.9440429e-05, 3.7165985e-05, 1.9891759e-05, 1.064624e-05, 5.6978887e-06, 3.049471e-06, 1.6320517e-06, 8.7343127e-07, 4.6743332e-07, 2.5014952e-07, 1.3384814e-07, 7.1615183e-08, 3.8302331e-08, 2.0481096e-08, 1.0948229e-08, 5.8510068e-09, 3.1322059e-09, 1.6631885e-09, 8.57237e-10, 4.4180893e-10, 2.3606094e-10, 1.0653837e-10, 4.8445804e-11, 3.8261163e-11, 4.5305767e-12, 3.0781683e-11, 1.9434374e-11, 1.4308533e-11, 1.6763067e-11, 1.0900001e-11, 7.5461885e-12, 2.7021852e-11, 5.7774899e-12, 7.0599212e-12, 6.0658537e-12, 1.4065233e-11, 2.1357799e-11, 2.7991834e-11, 3.1688949e-13, 5.4280789e-12, 1.650667e-11, 4.0891069e-11, 5.6260599e-11, 3.8895053e-11, 2.5336844e-11, 3.9860262e-11, 1.6378804e-11, 1.7282189e-11, 1.3859227e-11, 1.4629087e-13, 8.4004028e-12, 2.5132947e-11, 2.0514684e-11, 5.5260427e-11, 5.613007e-11, 8.2709276e-11, 4.497345e-11, 5.2929541e-11, 5.7110263e-11, 6.9544209e-11, 7.4599235e-11, 3.1559311e-11, 6.120341e-11, 2.6694862e-11, 2.024641e-11, 1.515409e-11, 1.5751907e-11, 1.5740807e-11, 4.6726499e-12, 4.2641121e-11, 3.1802278e-11, 5.9698102e-11, 2.4635248e-12, 1.1014436e-11, 5.2965502e-12, 1.8254279e-11, 1.4623092e-11, 3.7850483e-11, 1.9546146e-12, 4.8512512e-11, 1.8027185e-11, 4.4816947e-11, 1.7458116e-11, 6.9470286e-11, 4.9698711e-11, 6.6345123e-11, 3.3800291e-11, 7.1974213e-11, 3.7158654e-11, 8.4838374e-11, 1.3432009e-11, 8.3507327e-11, 3.8679835e-11, 6.5401114e-11, 1.2362687e-11, 7.8441534e-11, 2.1142137e-11, 3.1710818e-11, 5.3928159e-11, 5.3182054e-11, 4.5290117e-11, 2.5674045e-11, 5.923259e-11, 4.5728657e-11, 5.7957485e-11, 2.930157e-11, 6.1021046e-11, 4.9183585e-11, 8.4027003e-11, 2.2707049e-11, 5.8156498e-11, 4.7588149e-11, 5.6304331e-11, 2.8830176e-11, 5.8142069e-11, 2.744907e-11, 2.3922773e-11, 5.1634678e-12, 3.7483424e-11, 2.7852314e-11, 1.821976e-12, 1.8758099e-14, 7.2568256e-12, 5.5144882e-11, 2.234554e-11, 3.6724e-11, 2.7349841e-11, 3.4800685e-11, 5.3074278e-12, 1.20751e-11, 1.5684422e-11, 3.8380594e-11, 2.5202207e-11, 5.6174357e-11, 1.8072359e-11, 2.6781216e-11, 1.4482462e-11, 8.3793138e-12, 1.3220897e-11, 1.6602569e-12, 3.0230928e-11, 3.3592287e-11, 5.5116578e-11, 6.3377683e-11, 7.4435962e-11, 5.0469568e-11, 4.3236495e-11, 3.3662436e-11, 1.3699395e-11, 5.1504926e-11, 4.373253e-11, 3.3496388e-11, 2.4427687e-11, 3.8436202e-11, 3.5448894e-11, 3.257369e-11, 2.7977294e-11, 2.8831508e-11, 1.2698557e-11, 2.5001751e-11, 1.3384836e-11, 1.971075e-11, 1.2994357e-11, 1.3088148e-11, 1.6582923e-11, 8.1139258e-12, 3.5938158e-11, 1.4750514e-11, 9.3928053e-12, 1.7985562e-12, 1.590963e-11, 4.7555628e-12, 3.5563773e-12, 1.7015469e-12, 8.9185254e-12, 1.5699961e-11, 5.2750174e-12, 1.1218555e-11, 1.7849038e-12, 1.03296e-11, 4.5309097e-12, 2.6763013e-11, 4.5335736e-12, 1.0454357e-11, 3.0471342e-11, 3.211051e-11, 6.2507818e-11, 2.9848218e-11, 6.2961897e-11, 7.356532e-11, 6.8332703e-11, 1.0839231e-10, 7.1992638e-11, 8.1141477e-11, 8.1786356e-11, 7.4219744e-11, 6.9918038e-11, 5.0034581e-11, 3.6349171e-11, 3.1050956e-11, 4.2421795e-11, 2.0902277e-11, 5.2895244e-11, 1.9525611e-11, 3.3844467e-12, 4.1533062e-11, 1.4276123e-12, 1.8104215e-11, 4.7198225e-12, 3.7571221e-11, 1.4317191e-11, 2.9038957e-11, 7.6348731e-12, 4.6844152e-12, 1.2681686e-11, 3.2787799e-12, 1.2783024e-11, 1.5131891e-12, 2.1837961e-11, 1.92186e-11, 2.6969464e-12, 3.3476409e-11, 1.9815418e-11, 4.5208647e-11, 3.9951056e-11, 5.1660762e-11, 2.2907727e-11, 9.1554989e-12, 1.1622131e-11, 7.3245325e-13, 1.5328351e-12, 1.4989152e-11, 1.7662567e-12, 4.8820966e-12, 4.0439765e-12, 5.3799072e-13, 8.3456824e-12, 1.4397107e-12, 1.3843466e-11, 1.4267576e-11, 2.6067188e-11, 7.6877066e-12, 1.781463e-12, 1.741505e-13, 7.2234163e-12, 5.1667977e-12, 9.0937859e-12, 1.0709645e-11, 1.1865432e-11, 3.2584679e-12, 2.2026874e-11, 3.0745489e-14, 2.639551e-11, 1.0207727e-11, 1.4503662e-12, 1.5962685e-11, 2.4756564e-11, 7.4725991e-12, 1.5719829e-11, 5.8238855e-13, 9.3606168e-12, 1.618523e-12, 3.9659917e-11, 1.3356755e-11, 3.2485339e-11, 4.5040491e-12, 9.7542039e-13, 1.4630528e-11, 2.525604e-11, 2.6123684e-11, 5.2356587e-11, 4.3091204e-11, 5.4287227e-11, 1.4033045e-11, 4.9993291e-11, 2.206439e-11, 5.5926728e-11, 3.1548655e-11, 4.8612296e-11, 4.2891746e-11, 3.7626385e-11, 6.0400808e-11, 4.8195178e-11, 4.3433289e-11, 4.8251453e-11, 4.4334343e-11, 3.9629061e-11, 5.6315542e-11, 5.0820089e-11, 2.6441018e-11, 7.3309811e-11, 2.3628415e-11, 2.9492814e-11, 2.7083565e-11, 4.0127648e-11, 2.8342355e-11, 5.2944525e-11, 4.2551659e-11, 3.3472302e-11, 3.773072e-11, 4.7997164e-11, 2.9040733e-11, 5.0418622e-11, 3.3453877e-11, 1.0104946e-12, 2.723363e-12, 4.1511973e-12, 1.8644792e-20];
let data5 = [8.8755469e-22, 1.8272653e-11, 1.0733613e-11, 2.7953635e-11, 4.0261304e-11, 4.0519627e-11, 2.6162828e-11, 1.8507411e-11, 3.7509347e-11, 1.2686038e-12, 4.4012317e-11, 1.0528311e-11, 3.9979306e-11, 7.1194322e-13, 3.1690309e-11, 1.1239254e-11, 2.5174778e-11, 9.4864624e-12, 3.0043224e-11, 7.6723133e-12, 4.0573981e-11, 8.6146793e-12, 1.1601506e-11, 9.6370764e-12, 2.5131094e-11, 1.7846932e-12, 8.4477253e-13, 1.0806197e-11, 3.1424205e-11, 1.6773181e-12, 4.5035937e-11, 1.64286e-13, 2.3256255e-11, 4.8283198e-12, 2.2983149e-11, 1.3398093e-11, 8.522977e-12, 1.1698655e-11, 8.3994846e-12, 1.9472119e-11, 9.1014239e-12, 1.4485737e-11, 2.5002155e-11, 2.4439826e-11, 1.9087858e-11, 1.7320616e-11, 1.3126209e-11, 1.3104534e-11, 1.6955919e-11, 1.3026837e-11, 4.9831132e-11, 2.7873382e-11, 2.1826589e-11, 1.7490015e-11, 1.2527421e-11, 1.3321285e-11, 8.621682e-12, 1.0819202e-11, 6.0263407e-12, 2.0526639e-11, 1.3794247e-11, 6.3775882e-12, 8.5345371e-12, 1.1445557e-12, 1.5747005e-11, 7.6131792e-12, 1.3336847e-11, 3.2547086e-12, 1.1789246e-11, 2.1615173e-11, 1.3106757e-11, 2.0772179e-11, 5.7986967e-12, 2.3010048e-11, 4.2337555e-12, 3.3382188e-11, 2.4994819e-11, 2.4757838e-11, 9.6001732e-12, 2.5441437e-11, 1.2652692e-11, 7.9859907e-12, 1.3192346e-11, 1.3035396e-11, 1.2489296e-12, 1.3751119e-11, 8.4677333e-13, 2.0941467e-13, 9.1730073e-12, 1.1213022e-11, 3.2567093e-12, 1.4686371e-11, 9.470345e-13, 8.1800661e-12, 9.6435234e-12, 8.1576129e-13, 8.6654768e-12, 7.5295912e-13, 2.0637238e-11, 8.718942e-12, 1.378202e-11, 1.1651748e-11, 7.9914372e-12, 2.2589885e-12, 4.1193777e-13, 3.5602716e-13, 2.0938799e-11, 2.5269481e-11, 1.4724497e-11, 2.2813083e-11, 2.8044115e-11, 3.1476003e-11, 5.1240791e-11, 3.116077e-11, 4.8701916e-11, 1.4031005e-12, 4.8881208e-11, 1.3291496e-11, 3.5871044e-11, 1.2820202e-11, 2.3258256e-11, 2.273772e-12, 2.8150934e-11, 1.1085417e-12, 1.6619122e-11, 4.5584372e-12, 4.3841473e-12, 2.1139322e-12, 3.0644124e-12, 1.1742783e-11, 4.1777338e-12, 1.7036172e-11, 5.4652338e-12, 3.793529e-11, 1.9054957e-11, 2.8574321e-11, 1.9084857e-11, 8.7654045e-12, 9.5515988e-12, 3.4541194e-12, 1.4369359e-11, 2.913465e-12, 1.5769125e-11, 2.4605112e-12, 1.4840209e-12, 2.4542532e-11, 7.2936106e-12, 2.9159214e-11, 3.3566371e-12, 1.9202681e-11, 1.3955643e-11, 1.1239477e-11, 2.3841371e-11, 1.4453503e-11, 1.159417e-11, 6.5890036e-12, 2.2701706e-11, 3.6958577e-11, 3.4684249e-11, 2.9639512e-11, 2.7414648e-11, 4.862233e-11, 2.9069958e-11, 3.5153987e-11, 3.4429039e-11, 2.6555981e-11, 4.0850644e-11, 3.0675804e-11, 5.1219449e-11, 4.77441e-12, 2.655398e-11, 3.1386969e-11, 3.4213511e-11, 3.624141e-11, 1.6622679e-11, 2.5780791e-11, 2.4026665e-11, 3.4672912e-11, 3.4961802e-11, 4.0297207e-11, 1.2481292e-11, 2.0136821e-11, 1.155471e-11, 6.5982295e-12, 1.4902899e-11, 5.9878813e-12, 8.4771815e-12, 3.4797627e-11, 1.4078135e-11, 1.2861551e-11, 1.4217078e-11, 2.1404981e-12, 1.2843099e-11, 6.4858527e-13, 1.2323787e-11, 1.2841877e-11, 3.1050505e-11, 4.7659733e-11, 1.9660414e-11, 5.4314429e-11, 2.442382e-11, 5.7484994e-11, 1.7082524e-11, 2.6238524e-11, 2.5719323e-11, 3.2529301e-12, 1.2547429e-11, 2.1220464e-12, 1.3222358e-11, 5.7659062e-12, 2.8632788e-11, 6.9821563e-12, 8.9501429e-12, 3.6701033e-11, 9.392537e-12, 2.6822418e-11, 2.0672807e-11, 3.2724488e-11, 2.9320499e-11, 1.7357964e-11, 2.0273208e-11, 5.9623159e-13, 3.8840641e-12, 1.1987434e-11, 1.1992325e-11, 1.167987e-11, 1.5016833e-11, 6.9690401e-12, 1.071016e-11, 5.2178044e-12, 1.2835207e-11, 3.3371851e-12, 1.4056015e-11, 1.9366189e-11, 1.4011331e-11, 2.3000156e-11, 5.0676683e-11, 3.700215e-12, 3.983914e-11, 2.8753501e-11, 4.2976137e-11, 3.1558369e-11, 4.0128364e-11, 2.7488788e-11, 4.5937731e-11, 3.1377521e-11, 3.7956298e-11, 2.371899e-11, 3.973121e-11, 1.0394926e-11, 2.6689255e-11, 1.6875999e-11, 2.4361129e-11, 5.4975797e-12, 3.1841145e-11, 1.5992656e-11, 1.7503243e-11, 1.3029505e-11, 1.2633018e-11, 1.9364521e-11, 9.046736e-12, 4.2479833e-12, 3.8152596e-12, 3.0043891e-12, 1.8084357e-11, 1.1009721e-11, 2.5575378e-11, 2.2657022e-11, 2.4589551e-11, 2.361206e-11, 2.7337952e-11, 2.7915287e-11, 7.6091777e-12, 8.3942603e-12, 2.236024e-11, 1.2721719e-11, 1.8846431e-11, 2.3197232e-11, 1.0804974e-11, 1.9368745e-11, 4.2606548e-12, 7.4951334e-13, 1.3374862e-11, 4.3480221e-12, 1.0584111e-11, 1.3272933e-12, 1.8003215e-11, 1.7960198e-11, 2.2397699e-11, 3.3285484e-11, 3.7689861e-11, 5.3303481e-11, 9.9480416e-11, 7.7390503e-11, 1.7477177e-10, 2.272775e-10, 3.0180055e-10, 4.3190998e-10, 6.1628025e-10, 8.5127097e-10, 1.2009967e-09, 1.6950411e-09, 2.3772034e-09, 3.3206575e-09, 4.6423437e-09, 6.4769579e-09, 9.0289383e-09, 1.2518788e-08, 1.7366293e-08, 2.3993656e-08, 3.3123488e-08, 4.5587198e-08, 6.2621576e-08, 8.5697329e-08, 1.1700797e-07, 1.5926845e-07, 2.1624124e-07, 2.9268777e-07, 3.9509373e-07, 5.31683e-07, 7.1341492e-07, 9.5435111e-07, 1.2728411e-06, 1.6923936e-06, 2.2433017e-06, 2.9642088e-06, 3.9044963e-06, 5.1265713e-06, 6.7095317e-06, 8.7526142e-06, 1.1380275e-05, 1.4747547e-05, 1.9046987e-05, 2.4516235e-05, 3.1447689e-05, 4.0198829e-05, 5.120511e-05, 6.499374e-05, 8.2200111e-05, 0.0001035856, 0.0001300579, 0.0001626927, 0.00020275786, 0.0002517386, 0.00031136464, 0.00038363743, 0.00047085867, 0.0005756576, 0.00070101771, 0.00085030094, 0.0010272691, 0.0012361002, 0.0014814008, 0.0017682101, 0.0021019974, 0.0024886507, 0.0029344562, 0.0034460673, 0.0040304647, 0.0046949055, 0.0054468639, 0.0062939633, 0.0072439014, 0.0083043699, 0.0094829708, 0.010787131, 0.012224019, 0.013800462, 0.015522872, 0.017397174, 0.019428746, 0.02162237, 0.023982191, 0.026511687, 0.029213656, 0.032090202, 0.03514275, 0.038372053, 0.041778217, 0.045360735, 0.04911852, 0.053049949, 0.057152909, 0.061424846, 0.06586281, 0.070463511, 0.075223362, 0.080138533, 0.085204989, 0.090418541, 0.09577488, 0.10126961, 0.1068983, 0.11265649, 0.11853973, 0.12454359, 0.13066372, 0.13689581, 0.14323565, 0.14967913, 0.15622222, 0.16286103, 0.16959178, 0.17641082, 0.1833146, 0.19029974, 0.19736295, 0.20450109, 0.21171115, 0.21899023, 0.22633556, 0.23374448, 0.24121448, 0.24874313, 0.25632813, 0.26396728, 0.27165847, 0.27939973, 0.28718915, 0.29502494, 0.30290538, 0.31082886, 0.31879385, 0.32679891, 0.33484267, 0.34292387, 0.35104133, 0.35919394, 0.36738069, 0.37560068, 0.3838531, 0.39213724, 0.40045251, 0.40879849, 0.41717482, 0.42558146, 0.43401829, 0.442486, 0.45098432, 0.45951561, 0.46807855, 0.47667818, 0.48531154, 0.49398337, 0.50269278, 0.51142777, 0.52019435, 0.52894898, 0.53767062, 0.54619037, 0.55499055, 0.56387567, 0.57088999, 0.58186201, 0.5861534, 0.60021703, 0.60134738, 0.61785314, 0.61911379, 0.63651665, 0.63678505, 0.65784138, 0.6578795, 0.68603009, 0.68845669, 0.71563473, 0.7134741, 0.71776554, 0.72108445, 0.72489199, 0.72923484, 0.73324986, 0.73173841, 0.73678235, 0.73840361, 0.74979991, 0.75581631, 0.77310827, 0.7704936, 0.78042619, 0.78220028, 0.7940975, 0.79340896, 0.80760385, 0.81136595, 0.80745767, 0.80743041, 0.80738714, 0.8133333, 0.81768179, 0.83020585, 0.83392377, 0.84706516, 0.84821172, 0.84909587, 0.85421756, 0.85347514, 0.86521386, 0.86563354, 0.88757134, 0.88371381, 0.89109113, 0.89310991, 0.89495336, 0.89530943, 0.90088965, 0.9056695, 0.90624354, 0.90462222, 0.90913183, 0.90974483, 0.91466033, 0.91320631, 0.91956892, 0.91835912, 0.92343729, 0.92382604, 0.92693279, 0.92690753, 0.92814953, 0.92854157, 0.92852411, 0.92780793, 0.92735543, 0.92691682, 0.92688725, 0.9267973, 0.92683193, 0.92684918, 0.92670024, 0.92654014, 0.92619538, 0.92608007, 0.92586642, 0.92592779, 0.92550746, 0.92549336, 0.92519671, 0.92510795, 0.92486953, 0.92480406, 0.92477448, 0.92478989, 0.92478722, 0.92486421, 0.92485254, 0.92500294, 0.92502212, 0.92520048, 0.92511055, 0.92517063, 0.92517352, 0.92527893, 0.92528055, 0.92535972, 0.92536251, 0.92539352, 0.92544885, 0.92554893, 0.92559441, 0.92564226, 0.92546138, 0.92548614, 0.92552699, 0.92546651, 0.92542342, 0.92534365, 0.92535393, 0.92537345, 0.92523031, 0.9252568, 0.92505143, 0.9249375, 0.92487305, 0.92485538, 0.92463872, 0.92422622, 0.92384466, 0.92351762, 0.92330762, 0.92339761, 0.92297597, 0.92342119, 0.92266706, 0.92357812, 0.92389217, 0.92388786, 0.92420779, 0.92468576, 0.92435758, 0.92440925, 0.92437358, 0.92438647, 0.92439618, 0.9243746, 0.92442331, 0.92447101, 0.92455341, 0.92467416, 0.92476762, 0.92486624, 0.92492277, 0.92502235, 0.92511518, 0.92534355, 0.92523595, 0.92525984, 0.92497355, 0.92495751, 0.92494348, 0.92493844, 0.92494037, 0.9249994, 0.92507841, 0.92526106, 0.92523889, 0.92552905, 0.92556646, 0.92580527, 0.92587979, 0.92601391, 0.9260997, 0.92623408, 0.92650175, 0.92670573, 0.92672, 0.9264975, 0.92642097, 0.92643383, 0.92653644, 0.92657195, 0.92656547, 0.92653991, 0.92654973, 0.92677317, 0.92675649, 0.92710071, 0.92713023, 0.92744911, 0.92763419, 0.92775855, 0.92771742, 0.92759541, 0.92761257, 0.92770025, 0.92775435, 0.92774143, 0.92765611, 0.92784568, 0.92794316, 0.92835903, 0.92857216, 0.92895711, 0.92898694, 0.92887355, 0.92885884, 0.92890899, 0.92882222, 0.92849467, 0.92815199, 0.92705378, 0.92504573, 0.92195214, 0.91754425, 0.91146947, 0.90331449, 0.89244729, 0.87812935, 0.85940026, 0.83510999, 0.80387083, 0.7640899, 0.7140365, 0.65203482, 0.57693332, 0.48907501, 0.39176494, 0.29237997, 0.20176242, 0.12922921, 0.077781305, 0.04470325, 0.024914804, 0.013636957, 0.0073884694, 0.0039820163, 0.0021403102, 0.0011488503, 0.0006162604, 0.00033046778, 0.0001771866, 9.4995694e-05, 5.0928843e-05, 2.7303412e-05, 1.4637492e-05, 7.8471456e-06, 4.2068227e-06, 2.2552263e-06, 1.2090172e-06, 6.4813081e-07, 3.4746604e-07, 1.8628374e-07, 9.9837153e-08, 5.3508618e-08, 2.8684448e-08, 1.5368512e-08, 8.2270942e-09, 4.4169899e-09, 2.3592016e-09, 1.2645366e-09, 6.5020614e-10, 3.5471392e-10, 1.6778529e-10, 1.0050526e-10, 5.1615698e-11, 1.0800772e-11, 1.7813076e-11, 4.995311e-11, 3.1669195e-11, 3.3961344e-11, 1.1708041e-11, 2.5872614e-11, 1.0639829e-11, 1.9704757e-11, 4.8069532e-12, 1.7319594e-11, 2.2914165e-11, 8.8837842e-12, 1.4515871e-12, 4.1505313e-12, 1.3497718e-11, 2.0270718e-11, 3.6340735e-12, 4.1322172e-12, 2.1671026e-11, 5.2655829e-12, 2.0408351e-11, 1.8707248e-11, 2.5831435e-11, 8.2879649e-13, 1.7264541e-11, 2.4574207e-13, 1.8723787e-11, 5.4721549e-11, 5.3244099e-12, 4.0799387e-12, 3.1560754e-11, 1.171348e-11, 1.5634696e-11, 1.064105e-12, 1.6620661e-11, 1.1558643e-11, 1.7717288e-11, 1.1078591e-11, 1.8877403e-11, 2.0996179e-11, 1.401018e-11, 1.9477884e-11, 1.7053762e-11, 2.1874811e-12, 2.624145e-11, 6.7389027e-11, 4.4233116e-11, 2.6371202e-11, 1.7523158e-11, 1.1761985e-11, 1.7927844e-11, 5.71389e-12, 1.9180751e-11, 1.1333324e-11, 1.6342287e-11, 1.9778013e-11, 1.501457e-11, 2.0305681e-11, 2.9919033e-11, 4.83337e-12, 3.7838718e-11, 4.8999667e-12, 4.1883472e-11, 2.0392146e-11, 3.8322654e-11, 6.206606e-11, 4.4075393e-11, 3.2064669e-11, 5.9149122e-11, 5.2948188e-11, 6.3164129e-11, 8.962113e-11, 4.4416146e-11, 1.0008559e-10, 3.4554944e-11, 7.4140605e-11, 1.188985e-11, 6.6591309e-11, 3.9916093e-11, 5.8978968e-11, 4.1150352e-11, 1.2497435e-11, 9.8640881e-13, 1.3660214e-11, 1.4786143e-11, 1.1291035e-11, 2.8940616e-11, 2.9410234e-12, 3.2128269e-11, 2.2550769e-11, 2.3148697e-11, 4.3842193e-11, 2.3608103e-11, 6.6996661e-11, 2.563664e-11, 8.4134335e-11, 2.7177578e-11, 8.6780779e-11, 7.6754528e-11, 9.3666772e-11, 7.6846209e-11, 4.7693483e-11, 3.9477664e-11, 3.5640137e-11, 3.4006408e-11, 1.3857673e-12, 2.7787826e-11, 1.7218922e-11, 8.4016237e-12, 1.8166372e-11, 2.496257e-11, 7.3955687e-13, 2.7269148e-11, 2.6721834e-11, 1.790276e-11, 1.3086927e-11, 4.7405896e-11, 4.2377064e-11, 1.2679133e-11, 4.8221706e-11, 5.814562e-12, 1.7775671e-11, 2.2400926e-12, 5.3610382e-13, 3.6900148e-12, 5.3574531e-11, 1.0308511e-11, 3.1199244e-11, 1.5624707e-11, 1.2150577e-12, 1.1282599e-11, 1.3037423e-12, 3.3328342e-12, 3.5065297e-11, 2.928148e-12, 1.0302739e-11, 1.7476874e-11, 2.517457e-11, 5.6404116e-12, 6.4988436e-11, 3.1622356e-11, 7.7872687e-11, 4.7857644e-11, 7.8295132e-11, 4.5235286e-11, 4.5798916e-11, 2.2290264e-11, 3.8032182e-11, 3.4512877e-11, 5.2836861e-11, 4.524805e-11, 7.8457073e-11, 3.9417283e-11, 6.1127601e-11, 4.0157173e-11, 3.5567658e-11, 2.9153947e-11, 1.1805273e-11, 2.9678064e-11, 5.7686103e-12, 5.7457454e-12, 3.2302197e-11, 4.9529666e-11, 4.6999878e-11, 5.174534e-11, 1.7997327e-11, 2.8981795e-11, 5.713224e-12, 7.3452883e-12, 1.7813853e-11, 8.047995e-12, 2.8741158e-11, 1.1843011e-11, 3.2613315e-11, 3.2346595e-11, 1.4849854e-11, 3.6456059e-11, 3.2138592e-11, 2.9605806e-11, 4.3727425e-12, 3.0369671e-11, 2.1015048e-11, 3.601652e-11, 1.1914491e-11, 1.3487728e-11, 3.9754928e-12, 1.0703429e-11, 7.4307541e-12, 4.1479785e-12, 1.0173985e-11, 2.0516016e-11, 1.3770764e-11, 1.0646267e-11, 1.7800534e-11, 3.2188317e-11, 3.6530203e-11, 6.5765398e-12, 2.0016763e-12, 1.8334085e-12, 1.0266888e-11, 1.4851519e-11, 2.2044189e-11, 6.0457638e-12, 2.5460048e-11, 1.4669155e-11, 2.4177394e-11, 5.8807037e-11, 6.2834032e-12, 6.6183848e-12, 1.6271806e-13, 1.8075134e-11, 2.2461973e-12, 5.3903408e-12, 1.6028505e-11, 4.0313232e-13, 3.2548272e-11, 4.1948181e-12, 1.1798724e-12, 6.5309211e-12, 6.6507952e-12, 8.8736836e-12, 1.6863184e-11, 1.6735319e-11, 1.9874467e-11, 1.2464469e-11, 2.6440019e-12, 9.8318998e-12, 2.0435212e-12, 1.1405914e-11, 1.9396969e-11, 1.3411475e-12, 8.558903e-12, 1.631254e-11, 8.5403669e-12, 4.0904832e-12, 8.2904072e-12, 1.7191951e-12, 5.3472749e-12, 2.5418536e-11, 1.5305265e-11, 7.2793575e-12, 5.020973e-11, 1.3790522e-11, 2.0260951e-12, 7.3462872e-12, 1.5602508e-12, 1.2291651e-11, 1.3710272e-11, 5.3200812e-12, 7.043827e-12, 8.1839633e-12, 3.117982e-11, 1.4296324e-11, 3.3934817e-11, 9.2874715e-12, 1.71975e-11, 1.5451111e-11, 2.1490771e-12, 1.0344473e-11, 4.5709788e-12, 6.2577634e-12, 2.0089797e-11, 4.5736648e-11, 3.6199883e-11, 2.7324757e-11, 5.4422973e-11, 3.1747225e-11, 3.6302997e-11, 4.2841466e-11, 5.7356338e-11, 5.2325509e-11, 5.2084095e-11, 3.44798e-11, 9.686164e-12, 1.4551611e-11, 4.7665845e-11, 4.0686173e-12, 1.1008331e-11, 2.4060184e-11, 1.879005e-11, 2.6862908e-11, 3.4303985e-12, 7.5572879e-12, 1.8099951e-20];
let data4 = [2.9640062e-21, 9.3329583e-12, 2.7505128e-12, 1.2046123e-11, 9.4759025e-13, 1.3365413e-11, 2.8670914e-11, 2.0633236e-11, 2.3636847e-11, 3.4316662e-12, 2.0934353e-11, 2.3455777e-12, 5.9056272e-12, 1.0987934e-11, 1.5653969e-11, 2.2826977e-11, 1.0468955e-11, 6.6778159e-12, 7.2381446e-12, 8.0519052e-12, 1.0072245e-11, 4.5852253e-12, 9.4651207e-12, 1.260145e-11, 3.3677525e-12, 6.1690628e-12, 4.4931896e-12, 2.3751447e-12, 7.724e-12, 1.3185121e-12, 1.6603671e-11, 1.4090473e-11, 8.0091108e-12, 2.9737105e-12, 4.5938954e-12, 1.2772628e-11, 2.4963029e-12, 4.7423975e-12, 1.1312616e-11, 1.2850213e-11, 1.2648024e-11, 1.9927407e-11, 5.9455315e-12, 1.6689038e-11, 2.1524916e-11, 3.2399806e-11, 1.0232752e-11, 7.1952391e-12, 6.7022699e-12, 1.3764457e-11, 1.1613733e-11, 1.7275043e-11, 1.6410819e-12, 2.666369e-12, 3.8416032e-12, 1.6578661e-12, 1.1866276e-11, 6.4615097e-12, 8.1140404e-12, 3.4703479e-12, 1.5422657e-12, 3.8004761e-12, 1.1784577e-12, 1.2324898e-11, 1.3230583e-11, 3.6088461e-12, 1.758383e-11, 1.4494518e-13, 1.2159945e-11, 1.0882004e-12, 1.4816533e-11, 8.6553618e-12, 1.9557374e-11, 2.2128373e-11, 3.168564e-12, 1.1052737e-11, 7.2584859e-12, 1.0890897e-12, 1.6109479e-11, 4.9710419e-12, 1.3393535e-11, 1.7507244e-11, 7.0728583e-12, 3.611725e-11, 7.3771987e-12, 1.8627457e-11, 2.3087856e-12, 2.7133872e-12, 4.2321994e-12, 2.7794684e-11, 6.4452812e-12, 2.5169665e-11, 8.9348037e-12, 1.2712715e-12, 1.5921295e-11, 1.0449725e-11, 6.3035594e-13, 1.2070577e-11, 1.1566381e-11, 3.5177663e-11, 1.4124486e-11, 4.1937956e-11, 1.7787131e-11, 4.9642726e-11, 2.4534752e-11, 3.7980641e-11, 3.7185554e-12, 2.2332341e-11, 4.5134197e-12, 2.1476786e-11, 1.8698596e-11, 1.5798803e-11, 3.1917953e-12, 1.737775e-11, 1.1408987e-11, 7.2948333e-12, 1.8829758e-11, 4.0927008e-12, 3.0558536e-11, 4.9908286e-14, 2.5212015e-11, 1.9438661e-11, 2.7429542e-12, 1.734985e-11, 1.2607119e-12, 1.4012776e-11, 2.0970145e-11, 3.2229629e-11, 2.7516688e-11, 3.1972752e-11, 3.8435373e-11, 1.5694985e-11, 2.1617841e-11, 3.5497899e-11, 2.7793128e-11, 3.0138373e-11, 1.1688318e-11, 4.4370122e-11, 3.9459771e-12, 2.8765617e-11, 5.9882147e-12, 1.1967093e-11, 1.1350186e-11, 2.7869714e-12, 3.1948409e-11, 1.1738559e-11, 3.4116584e-12, 1.1020503e-11, 1.8829537e-13, 1.7159998e-11, 2.3805913e-12, 8.3825892e-12, 7.3541897e-12, 1.4197737e-11, 6.3963733e-12, 1.9021833e-11, 1.0141939e-11, 3.5838587e-11, 1.2702378e-11, 6.4226057e-12, 1.1199794e-11, 1.5978428e-12, 1.847851e-11, 9.7903582e-12, 2.389328e-11, 1.1467788e-12, 1.2559545e-11, 8.4380551e-12, 5.9497553e-12, 8.2975561e-12, 1.7023279e-12, 6.496301e-12, 2.0455723e-12, 3.3916618e-11, 1.1009943e-11, 2.0572546e-11, 7.4501159e-12, 1.6909234e-11, 1.3017278e-12, 2.511242e-11, 1.2519307e-11, 1.3066964e-11, 9.6181802e-12, 2.9619282e-12, 8.4008185e-12, 6.7134964e-12, 6.4966344e-12, 3.0185057e-12, 9.5896135e-12, 5.3267356e-12, 2.0195844e-11, 5.2957235e-12, 1.6285881e-11, 4.8516622e-12, 1.6781518e-11, 4.7132751e-12, 1.7246921e-11, 3.1046836e-11, 3.3466221e-11, 4.5161096e-11, 2.7890166e-11, 4.8981358e-11, 1.9105087e-11, 3.9502787e-11, 4.4925449e-11, 1.3337403e-11, 2.8367462e-11, 3.8099576e-11, 3.391584e-11, 2.2406369e-11, 4.5648174e-11, 3.0683362e-11, 1.1119319e-11, 2.2178503e-11, 3.9817688e-11, 4.3980638e-11, 4.2083457e-11, 3.1904503e-11, 3.5951408e-11, 2.0441495e-11, 3.4121142e-11, 2.3310165e-11, 4.710474e-11, 6.8178703e-12, 1.8060793e-11, 4.0326774e-12, 2.5641959e-11, 9.9446403e-12, 1.5113203e-11, 3.1063176e-12, 1.5865162e-11, 2.2517079e-11, 1.3830149e-11, 1.5724997e-12, 1.6482735e-11, 6.9053487e-12, 2.8235633e-11, 5.6019536e-12, 9.9887685e-12, 1.6630682e-11, 1.2960591e-13, 3.5512682e-12, 1.0279659e-11, 1.9133098e-11, 1.8250644e-11, 2.2588773e-12, 2.7064401e-11, 1.0553543e-11, 1.6625013e-11, 2.5463223e-12, 1.8596112e-11, 4.1845142e-12, 1.1992991e-11, 2.5180225e-11, 1.9251144e-11, 7.6910983e-12, 2.1652188e-11, 2.3425098e-11, 6.2203049e-12, 9.2353648e-12, 5.592172e-12, 1.6659804e-11, 1.4916794e-11, 8.5797769e-12, 2.5034946e-11, 2.176701e-11, 1.3789356e-11, 1.8613452e-11, 6.2498719e-12, 3.7386743e-12, 1.1784355e-11, 5.3595261e-12, 1.5875055e-12, 2.5069737e-11, 1.8245976e-12, 1.5287716e-11, 1.2674923e-12, 1.5658749e-11, 4.4738488e-12, 1.0064131e-11, 2.1842484e-11, 1.1868499e-11, 3.7473221e-11, 5.8434918e-12, 1.7942524e-11, 5.2399241e-12, 2.1522137e-11, 2.8356569e-12, 1.2945806e-11, 7.3661946e-13, 2.1775121e-13, 2.5093969e-11, 2.5642182e-11, 5.9632939e-11, 9.0133341e-11, 1.6135389e-10, 2.3653087e-10, 3.4442833e-10, 5.2040302e-10, 7.5668623e-10, 1.0945064e-09, 1.5515373e-09, 2.1963396e-09, 3.1317283e-09, 4.3789956e-09, 6.1486076e-09, 8.5833479e-09, 1.19644e-08, 1.6609804e-08, 2.3053427e-08, 3.1830126e-08, 4.3877079e-08, 6.0203165e-08, 8.2486321e-08, 1.1266521e-07, 1.5346586e-07, 2.0842465e-07, 2.8222683e-07, 3.8105991e-07, 5.1301255e-07, 6.8860458e-07, 9.2150284e-07, 1.2294442e-06, 1.6352882e-06, 2.1683996e-06, 2.8662775e-06, 3.7768487e-06, 4.960847e-06, 6.4950875e-06, 8.4761234e-06, 1.1025082e-05, 1.4292922e-05, 1.8467174e-05, 2.3779574e-05, 3.0515209e-05, 3.9023052e-05, 4.9728285e-05, 6.3146081e-05, 7.9897671e-05, 0.00010072802, 0.00012652577, 0.00015834491, 0.0001974285, 0.00024523382, 0.00030345928, 0.00037407188, 0.00045933524, 0.00056183756, 0.00068451808, 0.00083069205, 0.0010040722, 0.0012087865, 0.0014493903, 0.0017308722, 0.0020586523, 0.0024385723, 0.0028768758, 0.0033801792, 0.0039554328, 0.0046098717, 0.0053509572, 0.0061863106, 0.0071236385, 0.0081706536, 0.0093349915, 0.010624125, 0.012045281, 0.013605355, 0.01531084, 0.01716775, 0.019181563, 0.021357164, 0.023698808, 0.026210085, 0.028893907, 0.031752493, 0.03478738, 0.037999428, 0.041388851, 0.044955238, 0.048697596, 0.052614388, 0.056703578, 0.060962683, 0.065388817, 0.069978743, 0.074728923, 0.079635565, 0.084694668, 0.08990207, 0.095253481, 0.10074453, 0.10637077, 0.11212778, 0.11801109, 0.12401628, 0.13013899, 0.13637492, 0.14271983, 0.1491696, 0.15572022, 0.16236776, 0.16910843, 0.17593856, 0.18285462, 0.18985317, 0.19693094, 0.20408476, 0.2113116, 0.21860854, 0.2259728, 0.23340171, 0.24089272, 0.24844338, 0.25605136, 0.26371444, 0.27143049, 0.27919747, 0.28701346, 0.2948766, 0.30278512, 0.31073734, 0.31873165, 0.32676651, 0.33484045, 0.34295207, 0.35110001, 0.35928299, 0.36749977, 0.37574917, 0.38403004, 0.39234128, 0.40068184, 0.4090507, 0.41744686, 0.42586937, 0.43431733, 0.44278987, 0.4512863, 0.45980599, 0.46834908, 0.47691568, 0.48550833, 0.49412819, 0.50277771, 0.51145706, 0.52012893, 0.52870413, 0.53745872, 0.54569641, 0.55388735, 0.56171361, 0.5703083, 0.57884008, 0.58839631, 0.5996874, 0.61464584, 0.63299906, 0.65300355, 0.66843974, 0.67260683, 0.67187109, 0.67378822, 0.67461145, 0.68013213, 0.68891798, 0.69906228, 0.71024186, 0.72634787, 0.72348441, 0.72350901, 0.72400258, 0.72931793, 0.73820286, 0.73928349, 0.74211056, 0.74537831, 0.74721972, 0.75008908, 0.75974998, 0.77083627, 0.77855188, 0.78093807, 0.77979187, 0.78006009, 0.78150724, 0.78188001, 0.78078482, 0.78972435, 0.80253125, 0.81073706, 0.81642665, 0.82116291, 0.82579555, 0.83115008, 0.83771477, 0.84467652, 0.85217798, 0.85965702, 0.86636392, 0.87269061, 0.87912294, 0.8857938, 0.89319124, 0.90139898, 0.90847588, 0.91591374, 0.92256061, 0.92905685, 0.93537009, 0.94484613, 0.95172695, 0.96261748, 0.96689615, 0.97052233, 0.96835432, 0.96371894, 0.95353748, 0.94646765, 0.93869513, 0.93300715, 0.92707869, 0.92303702, 0.91664736, 0.91744421, 0.91719413, 0.91974339, 0.92136019, 0.92451412, 0.92669603, 0.92989174, 0.93016046, 0.93012549, 0.93004485, 0.92979559, 0.92964667, 0.92934676, 0.92917201, 0.92833142, 0.92825144, 0.92832679, 0.92894541, 0.928823, 0.9286215, 0.92814636, 0.92772128, 0.92714753, 0.92684216, 0.92646554, 0.92618261, 0.92590351, 0.92572693, 0.92561247, 0.925564, 0.92557124, 0.92559663, 0.92590674, 0.92589498, 0.9258723, 0.92548199, 0.92527586, 0.92514624, 0.92517072, 0.92519455, 0.92517294, 0.92511427, 0.92507073, 0.92508099, 0.9251498, 0.92523651, 0.92531842, 0.92532105, 0.92534056, 0.92546288, 0.92567932, 0.92587718, 0.92603413, 0.92630519, 0.92622569, 0.92618911, 0.9260881, 0.92611963, 0.92617353, 0.92629568, 0.92638431, 0.92644902, 0.92639186, 0.92632513, 0.92625433, 0.92626243, 0.92623303, 0.92622917, 0.92625492, 0.92623275, 0.92630611, 0.92630213, 0.92631903, 0.92629826, 0.92631528, 0.92630902, 0.92627638, 0.92626638, 0.92621553, 0.92630114, 0.92641228, 0.92647963, 0.92653698, 0.92665872, 0.92689599, 0.92705079, 0.92742487, 0.92732687, 0.9273452, 0.9272198, 0.92721382, 0.92703998, 0.92694653, 0.92701832, 0.92741031, 0.92757705, 0.92722499, 0.9266838, 0.92646418, 0.9263108, 0.92630487, 0.92665255, 0.92756567, 0.92795025, 0.92775351, 0.92732948, 0.92723394, 0.92680679, 0.92634006, 0.92634952, 0.92671698, 0.92693975, 0.92722617, 0.92694944, 0.9269312, 0.92720149, 0.92852329, 0.93030914, 0.93027955, 0.92985191, 0.92960996, 0.92935124, 0.92847124, 0.92810666, 0.92809806, 0.92914492, 0.93069296, 0.93146786, 0.93042349, 0.93049376, 0.93052707, 0.93109751, 0.93098899, 0.93069666, 0.93047184, 0.9297293, 0.92802123, 0.92532139, 0.92149254, 0.91629255, 0.90932378, 0.90007065, 0.88787749, 0.87192529, 0.85120664, 0.82450355, 0.79038292, 0.747226, 0.69333741, 0.6272038, 0.5481055, 0.45723604, 0.35909776, 0.26221379, 0.1771453, 0.11142655, 0.066168553, 0.037697689, 0.020907893, 0.011417705, 0.0061818558, 0.0033325675, 0.0017925853, 0.00096317242, 0.00051724591, 0.00027770297, 0.00014907787, 8.0024386e-05, 4.2955751e-05, 2.3057529e-05, 1.2376637e-05, 6.6433276e-06, 3.5659077e-06, 1.9140184e-06, 1.0273475e-06, 5.5140358e-07, 2.9595236e-07, 1.5884068e-07, 8.5240279e-08, 4.5735953e-08, 2.4536113e-08, 1.3110006e-08, 7.0446593e-09, 3.7344034e-09, 2.0400243e-09, 1.0665481e-09, 5.4378742e-10, 2.7004349e-10, 1.5812854e-10, 7.4338287e-11, 3.3561209e-11, 2.8333919e-11, 8.868467e-13, 3.0858713e-12, 3.5963354e-12, 3.4917785e-12, 1.017121e-11, 2.2028983e-11, 4.0346086e-11, 1.7580209e-11, 4.2501933e-11, 1.4422969e-11, 1.8196895e-11, 4.8419165e-12, 1.6136947e-11, 1.9557134e-11, 1.3521138e-11, 2.5140272e-11, 2.0023977e-11, 3.3199034e-11, 4.6221473e-11, 6.582134e-11, 1.3640679e-11, 1.8451073e-11, 2.6909526e-12, 2.8481986e-11, 3.7241345e-11, 2.7911807e-11, 3.1812378e-11, 2.4314584e-11, 3.0230262e-11, 8.3786478e-12, 3.2504762e-12, 2.3156355e-11, 2.5427859e-11, 1.8178248e-11, 2.6479089e-11, 2.9286697e-11, 6.2549886e-12, 2.2235655e-11, 9.7529828e-12, 8.7256168e-12, 2.3332615e-11, 7.4472923e-12, 3.7661016e-11, 3.9459128e-11, 4.3172452e-11, 4.6146551e-11, 7.0674356e-11, 3.8188906e-11, 2.234998e-11, 1.691735e-11, 2.136912e-11, 1.0642604e-11, 2.3709996e-11, 1.7593085e-11, 4.2050074e-11, 5.7920745e-11, 2.3162904e-11, 2.0006773e-11, 4.6649913e-12, 2.0054834e-11, 3.429455e-11, 4.7810915e-12, 1.6627987e-11, 3.419521e-12, 3.9087851e-12, 2.5789369e-11, 8.5253827e-12, 4.5741865e-11, 3.9988016e-12, 4.2880647e-11, 4.4876219e-12, 5.7759137e-11, 1.0324161e-11, 4.5343838e-11, 1.571317e-11, 4.2209574e-11, 3.6910248e-11, 4.0849557e-11, 1.7327808e-11, 6.036085e-11, 3.0158338e-12, 6.5229405e-11, 2.2726917e-11, 1.035959e-10, 1.3112123e-11, 7.8468617e-11, 2.6461552e-11, 9.2848187e-11, 4.8677117e-11, 7.3172067e-11, 3.0248465e-11, 2.8176973e-11, 3.4641409e-13, 4.5031611e-12, 1.198919e-11, 5.4409543e-12, 2.3406093e-11, 3.6406222e-12, 5.9825856e-11, 5.5350777e-11, 6.1321841e-11, 7.566667e-11, 4.7177247e-11, 1.7426815e-11, 3.9422611e-11, 3.2878814e-12, 1.7982232e-12, 2.9966761e-11, 1.5093043e-12, 3.8254837e-11, 6.3659832e-12, 5.8671402e-11, 3.0131144e-11, 3.1009222e-11, 2.9751986e-11, 2.6061527e-12, 1.7776004e-11, 1.8748538e-11, 2.6435912e-11, 8.2280281e-13, 3.9124035e-11, 2.5456274e-11, 6.8715634e-12, 3.539384e-11, 5.4937877e-12, 4.8476439e-11, 3.7622612e-11, 1.7812965e-11, 3.2490666e-11, 1.3793518e-11, 3.0020593e-11, 1.0668244e-11, 3.3360087e-11, 6.4262532e-12, 3.2945411e-11, 5.8499693e-12, 1.1105896e-11, 1.2388216e-11, 1.3703169e-11, 1.0369113e-12, 5.5859132e-12, 1.8173365e-11, 1.6084224e-12, 2.9652202e-11, 1.3880982e-12, 3.0761038e-12, 2.2155628e-12, 1.2362909e-11, 8.4855356e-12, 2.2690733e-11, 1.6411326e-11, 1.5741251e-11, 3.9831626e-11, 9.5729495e-12, 1.8957319e-11, 7.9878359e-12, 7.3632694e-12, 1.4530301e-12, 2.3316964e-11, 1.4322851e-11, 1.6770726e-11, 2.1865821e-11, 3.0555143e-11, 2.9852991e-11, 1.0491763e-11, 3.8597699e-11, 1.0835291e-12, 5.5185839e-11, 7.7102384e-12, 4.2228443e-11, 2.6358549e-11, 7.4350274e-11, 3.3509153e-11, 7.2135821e-11, 3.9783121e-11, 7.1883087e-11, 6.8346466e-11, 1.0074878e-10, 4.945419e-11, 7.1265513e-11, 4.4255093e-11, 4.7498021e-11, 2.0786621e-11, 2.3789135e-11, 1.531259e-11, 4.4703844e-11, 2.086687e-11, 1.5181173e-11, 3.7472991e-11, 5.671046e-12, 1.4036486e-11, 2.6176185e-11, 3.2143364e-11, 2.0150622e-11, 1.6476923e-11, 5.9871808e-11, 4.4838036e-11, 8.377327e-11, 4.2277835e-11, 6.7988065e-11, 2.0570736e-11, 5.2125163e-11, 7.5326471e-12, 4.0375055e-11, 5.1529011e-11, 5.4923447e-12, 5.0088968e-11, 8.3812007e-12, 5.7186738e-11, 4.427485e-11, 4.3078328e-11, 4.2777533e-11, 6.7281806e-11, 2.7674279e-11, 6.5181123e-11, 2.0746774e-11, 4.5545405e-11, 1.2051347e-11, 4.6774892e-11, 3.5612056e-11, 1.0890677e-11, 3.2721868e-11, 2.6763013e-12, 2.1213284e-11, 2.694682e-11, 4.6181182e-11, 3.9528389e-11, 5.5938493e-11, 7.7387863e-11, 4.8343023e-11, 4.6970464e-11, 3.0434492e-11, 2.6478534e-11, 3.7708521e-11, 7.0224384e-11, 5.3415919e-11, 7.0918877e-11, 3.5120794e-11, 4.0638334e-11, 3.359029e-11, 1.6493795e-11, 1.6370813e-11, 1.2935419e-11, 8.2199256e-12, 5.8821577e-12, 2.9659417e-11, 1.4651276e-14, 3.7516168e-12, 2.661539e-12, 1.2588561e-11, 4.1916659e-11, 6.7540424e-11, 2.2157182e-11, 6.5171355e-11, 2.284557e-11, 5.3456321e-11, 6.3659832e-12, 2.8300288e-11, 3.3745682e-12, 3.498305e-11, 6.4931823e-14, 5.2362803e-12, 5.407445e-11, 7.3582746e-12, 3.4267578e-11, 1.3745568e-12, 4.2672865e-11, 1.9696543e-11, 4.5940767e-11, 3.1288928e-11, 1.7027012e-11, 1.6064467e-11, 2.6515051e-11, 1.1543436e-13, 2.1729409e-12, 2.676046e-11, 1.3155522e-11, 7.0774583e-12, 3.9662803e-12, 2.8672231e-11, 1.7701194e-11, 2.2506038e-11, 1.5213916e-11, 2.1611318e-20];
let data7 = [2.8477683e-20, 1.9817254e-11, 2.1635317e-10, 1.809108e-10, 3.3037479e-10, 3.1952347e-10, 8.4635997e-11, 6.7923694e-11, 8.1165126e-11, 3.8541257e-11, 2.1220779e-10, 8.8978506e-11, 1.5024543e-10, 1.0109516e-10, 1.1924985e-10, 1.5972819e-10, 1.3589799e-10, 5.5968904e-11, 5.9218579e-11, 1.4418632e-10, 6.8115023e-11, 1.4655948e-10, 2.5399996e-11, 2.3202602e-10, 1.9469538e-11, 2.6029821e-10, 2.6356826e-10, 2.0576201e-10, 2.9054878e-10, 9.7460678e-11, 5.1290831e-10, 2.0877626e-10, 4.8137136e-10, 2.3414674e-10, 4.9323768e-10, 1.7236458e-10, 2.7703685e-10, 2.1815489e-10, 2.2927159e-10, 2.9775453e-10, 3.7796739e-10, 4.2749079e-10, 3.3366027e-10, 3.1802514e-10, 2.4735136e-10, 3.6288265e-10, 3.0970653e-10, 3.5424183e-10, 2.1602941e-10, 2.4179075e-10, 2.952367e-10, 3.8359151e-10, 4.9834974e-10, 4.3116471e-10, 3.7808902e-10, 3.6538443e-10, 5.1141341e-10, 5.2062858e-10, 5.2599717e-10, 5.2491092e-10, 4.512288e-10, 4.7755334e-10, 3.6242088e-10, 3.3089726e-10, 3.088043e-10, 2.425945e-10, 3.1302036e-10, 1.9574006e-10, 3.6476364e-10, 1.7781339e-10, 3.4295644e-10, 1.4802614e-10, 2.1914047e-10, 1.0554173e-10, 2.5820686e-10, 1.8239021e-10, 2.0159706e-10, 2.0936174e-11, 2.02232e-10, 7.9556056e-11, 2.7654249e-10, 1.1959572e-10, 2.5486029e-10, 7.5607268e-12, 1.6671328e-10, 1.8532479e-10, 1.6790936e-10, 2.5386076e-10, 1.3592029e-10, 1.90552e-10, 1.312331e-10, 2.3270266e-10, 5.3277926e-11, 8.607033e-11, 5.2760181e-11, 2.2919825e-11, 1.2535519e-10, 1.6959737e-10, 1.049109e-10, 2.782024e-11, 6.1551883e-11, 4.8213281e-11, 1.0873528e-10, 8.0136791e-11, 1.4437715e-10, 1.7885345e-11, 2.2846276e-10, 2.3411295e-11, 3.2771467e-10, 1.1241968e-10, 3.5625737e-10, 8.5255682e-11, 2.3003975e-10, 7.3182836e-11, 8.2392924e-11, 8.2526453e-11, 2.8599207e-11, 3.3357279e-11, 3.2393904e-11, 6.0353097e-11, 1.6471975e-11, 2.7220928e-11, 6.7487991e-11, 3.4596827e-11, 1.4466908e-10, 1.0913542e-10, 3.886743e-11, 1.7155191e-10, 1.9649472e-10, 3.0039176e-10, 2.1706912e-10, 1.7937359e-10, 5.2531302e-12, 7.9055027e-12, 1.5239417e-10, 6.7756783e-11, 1.0415159e-10, 2.3479438e-10, 7.9120719e-11, 1.484773e-10, 7.5912829e-11, 1.993482e-11, 2.4164178e-10, 2.9148732e-11, 1.3744549e-11, 2.2941906e-10, 2.774089e-10, 2.1098422e-10, 1.3233328e-10, 1.5952605e-10, 1.9464789e-10, 4.7715155e-11, 1.5999093e-10, 7.649121e-11, 2.1009985e-10, 1.452682e-10, 2.0044788e-10, 7.6618975e-12, 3.6200604e-11, 1.8525448e-10, 9.7877338e-11, 2.8069348e-12, 2.0239038e-11, 1.9796622e-11, 1.3152776e-11, 1.9356596e-10, 2.6752626e-11, 2.3772909e-10, 1.430706e-11, 3.2829193e-10, 3.419622e-11, 4.3060049e-10, 1.0813806e-10, 3.7939735e-10, 9.8199581e-11, 3.2751842e-10, 6.5516255e-11, 6.6882809e-11, 1.0962431e-10, 2.4312811e-10, 3.511733e-10, 1.8942962e-10, 2.2945665e-10, 1.4616836e-10, 4.5135341e-11, 9.7068791e-11, 3.8640586e-11, 1.2930635e-10, 8.1348505e-11, 9.5416082e-11, 1.8570819e-10, 1.1855809e-10, 1.9283651e-10, 1.0581354e-10, 1.0844812e-10, 9.5097879e-11, 1.3290397e-10, 3.5893166e-10, 2.5249281e-11, 5.4991348e-10, 9.0886239e-11, 4.3684268e-10, 1.4355966e-10, 5.3188724e-10, 1.2617057e-10, 3.7926679e-10, 1.3789329e-10, 1.6742864e-10, 4.1086368e-11, 7.0177179e-11, 1.2346572e-11, 5.9132882e-14, 7.8648206e-13, 7.0490142e-12, 3.6924912e-12, 5.6042079e-11, 7.2597309e-11, 2.0615028e-10, 2.7642091e-10, 2.6228259e-10, 9.9983201e-11, 2.1552475e-12, 4.8360516e-11, 2.0368089e-11, 8.7008727e-11, 3.7743871e-11, 1.0942598e-10, 1.6518165e-10, 1.242478e-10, 1.4404817e-10, 1.6390056e-10, 3.4234109e-10, 1.440687e-10, 2.7340136e-10, 5.2112998e-11, 1.0796456e-10, 4.6046917e-11, 1.6697214e-10, 6.8456262e-12, 1.0125462e-10, 4.7842905e-11, 2.1396614e-10, 3.5911615e-11, 2.9679157e-10, 1.3415511e-10, 2.7936568e-10, 2.1178514e-10, 4.3080251e-10, 4.2927147e-10, 5.0658213e-10, 4.6356507e-10, 6.5276704e-10, 4.8260318e-10, 6.6818873e-10, 5.6053824e-10, 5.2016442e-10, 5.9334532e-10, 5.4408245e-10, 5.1616447e-10, 4.7639677e-10, 5.8710287e-10, 5.7388639e-10, 5.4723248e-10, 5.8585824e-10, 6.392226e-10, 7.1603609e-10, 6.2675589e-10, 7.8034298e-10, 6.5468454e-10, 7.7678248e-10, 5.9770753e-10, 7.3561387e-10, 5.091355e-10, 5.7069555e-10, 4.4704192e-10, 4.5896522e-10, 5.2184403e-10, 6.2306429e-10, 5.872771e-10, 6.9971574e-10, 6.8471009e-10, 6.7302519e-10, 4.6299314e-10, 6.3234335e-10, 4.7836513e-10, 7.4206463e-10, 5.6813867e-10, 8.0981043e-10, 6.6875664e-10, 7.3005264e-10, 7.0932665e-10, 6.8507835e-10, 6.2430738e-10, 6.9547164e-10, 6.5583485e-10, 5.1924179e-10, 7.0097115e-10, 5.2655282e-10, 6.563908e-10, 5.5960659e-10, 6.619308e-10, 4.9482656e-10, 5.3791057e-10, 4.4873093e-10, 4.3237599e-10, 3.2823616e-10, 4.3226711e-10, 2.2101743e-10, 2.6649277e-10, 1.6903861e-10, 2.1312479e-10, 1.7537258e-10, 1.7326172e-10, 5.3140063e-11, 1.7270464e-11, 2.6298704e-11, 1.2999494e-11, 1.1822559e-11, 1.4736385e-10, 1.4860663e-10, 1.1970659e-10, 4.6003957e-11, 1.29762e-10, 1.6484604e-10, 2.2191799e-10, 7.8584122e-11, 1.1429746e-10, 2.6841091e-11, 1.3831702e-10, 2.2084829e-10, 1.6325195e-10, 2.4149503e-10, 6.1705914e-11, 3.6353002e-10, 9.8185172e-11, 3.4585628e-10, 6.2212222e-12, 1.398224e-10, 6.0760331e-12, 1.6543115e-10, 4.8727664e-11, 3.6482602e-10, 2.8267297e-10, 4.6532468e-10, 1.9670647e-10, 4.4901017e-10, 1.2678165e-10, 4.1018904e-10, 1.8250552e-09, 4.1534681e-09, 9.4402826e-09, 1.9585692e-08, 4.0855308e-08, 8.2272735e-08, 1.637779e-07, 3.2005018e-07, 6.1731524e-07, 1.1731282e-06, 2.1975343e-06, 4.0552083e-06, 7.3716991e-06, 1.3194619e-05, 2.3245815e-05, 4.0291446e-05, 6.867536e-05, 0.00011505047, 0.00018933885, 0.00030591933, 0.00048498838, 0.00075396411, 0.0011487021, 0.0017141973, 0.0025043679, 0.0035805309, 0.0050081363, 0.006852056, 0.0091706344, 0.012009175, 0.01539368, 0.019325493, 0.023778893, 0.028706448, 0.03405052, 0.039752921, 0.045760023, 0.052024883, 0.058507644, 0.065175076, 0.071999732, 0.078959028, 0.086034375, 0.093210427, 0.10047445, 0.10781579, 0.11522552, 0.12269605, 0.1302209, 0.13779453, 0.1454121, 0.15306944, 0.16076286, 0.16848914, 0.1762454, 0.18402908, 0.19183791, 0.19966982, 0.20752299, 0.21539573, 0.22328651, 0.23119396, 0.23911681, 0.24705388, 0.25500412, 0.26296652, 0.27094019, 0.27892425, 0.28691792, 0.29492045, 0.30293117, 0.31094941, 0.31897457, 0.32700609, 0.33504346, 0.34308619, 0.35113386, 0.35918609, 0.36724252, 0.37530278, 0.3833665, 0.39143312, 0.39950231, 0.40757347, 0.41564732, 0.42372248, 0.43180034, 0.43987062, 0.44793718, 0.45597733, 0.46403705, 0.4720927, 0.48028065, 0.4885131, 0.49695726, 0.5053638, 0.51370983, 0.52166575, 0.52911717, 0.53630063, 0.54350976, 0.55065175, 0.5576987, 0.56515963, 0.57414881, 0.58581203, 0.60106133, 0.61920826, 0.63441176, 0.64444496, 0.65045212, 0.6552307, 0.6599221, 0.6650715, 0.67001146, 0.67061167, 0.67113711, 0.67315375, 0.67797767, 0.68199324, 0.68571826, 0.69335833, 0.70237947, 0.70950038, 0.71685984, 0.7241968, 0.73129851, 0.73900574, 0.75115207, 0.75963027, 0.76905855, 0.78322163, 0.79159857, 0.79245248, 0.79107474, 0.78888644, 0.78973313, 0.79291809, 0.80155304, 0.81233827, 0.82142127, 0.82650645, 0.82950388, 0.83326463, 0.83366361, 0.83176001, 0.83146794, 0.83202915, 0.83384987, 0.83933199, 0.84790874, 0.85610776, 0.86331973, 0.86895249, 0.87437673, 0.87886068, 0.88107721, 0.88256128, 0.88637201, 0.8903003, 0.89170068, 0.8913573, 0.88855065, 0.88565698, 0.88533144, 0.88606156, 0.88671645, 0.88791243, 0.88913495, 0.8891744, 0.88764162, 0.88782382, 0.88955391, 0.89185602, 0.89196857, 0.89173273, 0.89275971, 0.89683466, 0.90014227, 0.90019847, 0.89987414, 0.89742712, 0.89417066, 0.89250583, 0.89142793, 0.89044687, 0.89000899, 0.89008642, 0.8905574, 0.89235167, 0.89543507, 0.89656239, 0.89636287, 0.89559705, 0.89308331, 0.89015943, 0.88955618, 0.88997233, 0.8915667, 0.89541263, 0.89933524, 0.90012366, 0.89982421, 0.89974462, 0.89953149, 0.89852051, 0.89717144, 0.8962689, 0.89562133, 0.89390928, 0.88979206, 0.88653296, 0.88673392, 0.88801113, 0.8919161, 0.89805037, 0.90365521, 0.90506388, 0.90454156, 0.90286096, 0.89988196, 0.89693817, 0.89508255, 0.89482766, 0.89446222, 0.8935049, 0.89149568, 0.89126409, 0.89133746, 0.89155296, 0.891861, 0.89210393, 0.89241619, 0.89270213, 0.89185516, 0.8909092, 0.89204971, 0.89389175, 0.89439182, 0.89561993, 0.89863491, 0.89857581, 0.89790874, 0.89807274, 0.8975778, 0.89576727, 0.89454615, 0.89394598, 0.89344243, 0.89313787, 0.89326778, 0.89456251, 0.89627034, 0.89716934, 0.89706067, 0.89644184, 0.8951939, 0.89524545, 0.8955355, 0.89648327, 0.89645648, 0.89620751, 0.89535335, 0.89266434, 0.89107604, 0.89119897, 0.89146252, 0.89201422, 0.89367523, 0.89515785, 0.89514318, 0.89468989, 0.8936907, 0.89203612, 0.89111186, 0.89147443, 0.89267491, 0.89616509, 0.89965218, 0.90041286, 0.90012019, 0.89926374, 0.89733478, 0.89248269, 0.88868208, 0.88865755, 0.88955456, 0.8911347, 0.89408624, 0.89764004, 0.8976562, 0.89715549, 0.8955327, 0.890238, 0.88822536, 0.88957124, 0.89035779, 0.89214842, 0.89442469, 0.90050315, 0.90013322, 0.89675555, 0.88715024, 0.87840912, 0.86407897, 0.85519389, 0.86034735, 0.87677547, 0.89883519, 0.91619161, 0.914784, 0.91060943, 0.90052024, 0.88406818, 0.85282057, 0.8092647, 0.77598062, 0.76502665, 0.74142894, 0.63055733, 0.2403422, 0.024533632, 0.0019871171, 0.00015601923, 1.2184269e-05, 9.5273907e-07, 7.4937229e-08, 5.953439e-09, 4.8536166e-10, 3.6191332e-10, 3.4116245e-10, 3.3277232e-10, 2.2153077e-10, 3.2855396e-10, 8.620493e-11, 8.181629e-11, 3.5578282e-10, 2.3914764e-11, 4.8847353e-10, 2.7332507e-10, 4.7721934e-10, 3.6431553e-10, 2.2384307e-10, 3.9855979e-10, 3.8645977e-10, 3.5036318e-10, 1.726803e-10, 2.9928474e-10, 1.9461675e-10, 3.7171756e-10, 1.7373108e-10, 3.7679966e-10, 4.8863157e-10, 3.8452728e-10, 4.2056852e-10, 3.3878678e-10, 3.6588632e-10, 3.3653803e-10, 1.6154007e-10, 3.2305978e-10, 2.7894117e-10, 6.7424967e-11, 1.0233268e-11, 4.5023441e-11, 8.146e-11, 1.1299926e-10, 7.1518314e-11, 5.3754675e-11, 7.0855331e-11, 1.2622738e-11, 7.8869489e-13, 1.4741204e-10, 5.0156919e-11, 9.4527367e-11, 3.6979398e-11, 1.6238011e-10, 3.9140888e-10, 9.6569923e-11, 4.8952982e-11, 1.125599e-10, 6.9977177e-11, 9.3216288e-11, 8.5504999e-12, 1.6346375e-10, 3.3393236e-10, 1.215359e-10, 1.2327146e-11, 3.1730176e-11, 3.9845058e-12, 1.4821947e-10, 5.8621841e-11, 4.0178273e-11, 8.2939545e-11, 1.962649e-10, 2.6365975e-10, 4.681282e-10, 3.1397343e-10, 1.725423e-10, 3.385972e-10, 1.1452388e-10, 1.2711137e-10, 1.0347447e-10, 9.8208115e-11, 3.625952e-11, 4.4265293e-11, 1.0968569e-10, 1.833644e-10, 1.8868279e-11, 6.4592718e-11, 1.77071e-10, 5.7097743e-11, 7.5745921e-11, 2.433641e-10, 2.6176833e-10, 1.9938034e-10, 7.8585294e-11, 1.7420073e-10, 1.5080546e-10, 7.284895e-11, 1.3081094e-10, 8.2025894e-11, 4.0778175e-12, 2.7453699e-10, 8.3939684e-11, 8.6642635e-11, 4.5338077e-11, 7.5067079e-12, 1.7119446e-10, 4.3341007e-11, 2.2271518e-11, 8.4149852e-11, 2.1526762e-10, 1.1833779e-10, 3.1246775e-10, 1.4383031e-10, 1.5289642e-10, 2.8087521e-11, 2.2263925e-11, 2.8324233e-10, 2.1168632e-10, 2.0249857e-10, 2.7185546e-10, 2.6738573e-10, 2.8651923e-10, 2.1367628e-10, 2.4703077e-11, 3.8862975e-10, 2.8921211e-10, 1.4230467e-10, 4.1436828e-11, 3.7256349e-11, 3.0538214e-10, 4.5553487e-11, 1.5740043e-10, 2.8861295e-10, 3.3890142e-10, 4.2071341e-11, 2.2142564e-10, 1.0395009e-10, 4.7673354e-10, 1.4339627e-10, 5.8287293e-10, 3.3438255e-11, 5.5037913e-10, 3.3458078e-11, 3.7407164e-10, 3.854368e-10, 2.4011775e-10, 4.5061275e-10, 5.4182422e-10, 3.8275743e-10, 2.9087651e-10, 1.220687e-10, 3.0405937e-10, 7.2546675e-11, 3.2877402e-11, 1.3158057e-10, 3.4092286e-10, 2.2212738e-10, 3.3963692e-10, 2.106994e-10, 4.455069e-10, 2.6300736e-10, 1.3576514e-10, 1.1298729e-10, 3.8279119e-10, 1.8344209e-11, 1.172805e-10, 3.006375e-11, 1.700625e-10, 1.9673638e-10, 1.2044465e-10, 2.5815689e-10, 4.2112812e-10, 5.1276697e-11, 6.7210226e-11, 5.7983457e-11, 1.089576e-10, 3.2672554e-11, 2.7578965e-10, 1.468722e-10, 2.9628548e-10, 3.9326858e-10, 4.8068342e-10, 3.7565781e-10, 3.241546e-10, 2.2195177e-10, 4.566839e-10, 4.1441796e-10, 2.0261894e-10, 3.0922788e-10, 1.225869e-10, 3.1332938e-11, 1.8846258e-10, 4.1431616e-10, 2.4652404e-10, 3.3884929e-10, 4.3479405e-10, 3.2627362e-10, 2.5278098e-10, 3.3640669e-10, 3.5546187e-10, 3.7988848e-10, 5.767901e-11, 7.0454888e-11, 4.3509946e-10, 2.1841129e-10, 4.475982e-10, 2.8222015e-10, 5.4257885e-10, 2.8392429e-10, 6.8963281e-10, 3.1406001e-10, 2.7907842e-10, 2.3343131e-10, 3.4284524e-10, 3.6977625e-10, 2.5203559e-10, 1.8537896e-10, 2.9391517e-10, 7.8858797e-11, 1.3702003e-10, 2.4885264e-10, 2.4011127e-10, 4.10332e-10, 4.5736337e-10, 5.3042462e-10, 6.7751041e-11, 3.1341871e-10, 8.3023325e-11, 1.8255445e-10, 3.4044546e-10, 4.3011966e-11, 2.6184625e-10, 5.8955461e-11, 4.8949175e-10, 2.2769705e-10, 2.7286237e-10, 1.112853e-10, 1.7151728e-10, 8.9115615e-11, 4.1614266e-11, 2.5054228e-10, 1.8822545e-10, 6.7626191e-11, 2.1285241e-11, 5.7091015e-11, 4.433511e-10, 3.1274912e-10, 2.2601838e-10, 6.2294587e-11, 3.1449292e-11, 4.4388366e-10, 1.0949398e-10, 2.5631037e-10, 6.2711095e-11, 5.8124669e-10, 4.1696024e-10, 6.5950828e-10, 1.6011349e-10, 3.2415787e-10, 9.7031873e-11, 1.6982715e-10, 6.3710753e-11, 2.1660822e-10, 5.5002667e-11, 4.6249485e-10, 3.5534175e-10, 4.0977061e-10, 1.1044674e-10, 4.1232996e-10, 1.7613872e-10, 4.7531882e-10, 1.4800221e-10, 5.4628135e-10, 8.7973527e-11, 3.8900734e-10, 4.1719918e-11, 3.5080572e-10, 6.1888695e-12, 5.6905546e-11, 1.0672898e-10, 6.678153e-11, 2.1756497e-10, 9.6035733e-12, 4.9392453e-11, 9.1473808e-11, 4.521796e-11, 1.0588006e-10, 7.3530424e-11, 1.4769092e-10, 1.8944487e-11, 7.2548502e-11, 4.0307835e-10, 2.1403273e-10, 3.4677373e-10, 3.4571903e-10, 2.6744556e-10, 2.3754971e-11, 8.9804591e-12, 1.1953095e-11, 1.2903551e-10, 4.1469275e-11, 1.7462034e-11, 2.2420991e-10, 3.2052671e-11, 1.9999565e-10, 3.9288371e-11, 3.8543651e-12, 3.3792944e-11, 6.5628575e-11, 1.7551484e-10, 1.7002088e-10, 2.8506586e-10, 2.4517918e-10, 3.7002737e-11, 6.9367117e-11, 3.875388e-11, 1.3229345e-11, 1.3489459e-11, 1.9190398e-10, 1.0360143e-11, 1.2232169e-11, 7.9270097e-12, 1.782311e-10, 6.749101e-11, 1.1641828e-10, 1.5052055e-10, 1.0297067e-10, 4.3455144e-12, 1.311254e-10, 2.2328563e-18];
let data6 = [2.4736089e-21, 3.557215e-11, 1.4491517e-11, 3.1926845e-12, 9.8094767e-12, 6.4086002e-12, 1.0635131e-11, 1.7253035e-11, 7.2617091e-13, 7.0434024e-12, 9.7921366e-12, 5.3706415e-12, 2.0734942e-11, 1.8722161e-11, 1.6536423e-12, 2.9744887e-12, 6.7616262e-12, 4.6114578e-12, 1.6889115e-11, 1.3241588e-11, 6.2568747e-12, 5.2398129e-12, 2.3631512e-11, 1.5845265e-11, 3.2267866e-11, 3.7653514e-12, 2.8384247e-11, 5.9523119e-12, 4.0711368e-12, 3.092868e-12, 1.645528e-12, 1.4564101e-11, 1.4103256e-11, 1.1685094e-11, 2.1664859e-11, 3.3110082e-11, 5.1585703e-11, 3.3429429e-11, 6.1477878e-11, 2.8549978e-11, 6.3940724e-11, 4.2923005e-11, 6.7488102e-11, 4.6114911e-11, 7.38638e-11, 6.2954229e-11, 5.8148252e-11, 5.1998196e-11, 6.2571525e-11, 6.3686069e-11, 6.6297973e-11, 5.5396516e-11, 3.8656236e-11, 3.7013154e-11, 2.5425876e-11, 5.928347e-11, 1.4655137e-11, 1.3775906e-11, 7.3402954e-12, 1.9464115e-11, 1.3586944e-11, 3.222585e-12, 3.1207677e-12, 7.4854629e-12, 1.5714993e-12, 1.0029451e-12, 1.8254646e-11, 1.6622901e-11, 2.7060288e-11, 2.7576044e-11, 2.2113367e-11, 3.7476556e-11, 2.8651462e-11, 1.0359023e-11, 1.8422378e-11, 2.0315446e-11, 1.144978e-11, 1.2722942e-11, 1.2541982e-11, 1.7565045e-11, 1.1307058e-11, 1.0880448e-11, 3.1376187e-11, 1.5549373e-11, 2.7908618e-11, 1.3636074e-11, 2.6978256e-11, 2.4016439e-11, 3.3378853e-11, 6.3285692e-12, 9.4646761e-12, 1.1684983e-11, 8.3919261e-12, 7.3647494e-12, 4.2612106e-12, 4.1272697e-12, 5.051073e-12, 6.0162256e-12, 5.2336994e-12, 9.1067593e-12, 8.3596914e-12, 2.3193786e-11, 2.4675139e-12, 1.96822e-11, 7.0787494e-12, 1.5689872e-11, 1.3103645e-11, 1.2901678e-11, 5.1338829e-12, 3.0584879e-11, 2.4636347e-11, 4.084931e-11, 1.8863771e-11, 1.7808806e-11, 1.0670256e-11, 2.2720714e-11, 6.5158642e-12, 1.4889783e-11, 1.3300166e-11, 2.1428323e-11, 5.7082171e-12, 1.5145549e-11, 2.9703648e-11, 3.2818747e-11, 5.7270022e-11, 3.4523965e-11, 3.5275701e-11, 3.0865322e-12, 1.1147552e-11, 1.3170782e-11, 2.0258313e-11, 8.1201539e-12, 1.8948693e-11, 4.3070062e-12, 2.0004436e-12, 2.087144e-12, 1.278541e-11, 1.4894674e-12, 7.7514551e-12, 7.6359658e-12, 4.8564418e-12, 1.2659361e-12, 2.5807135e-11, 1.9168779e-11, 2.2039671e-12, 9.3736407e-13, 3.4295543e-12, 8.5977838e-13, 2.1398089e-11, 4.11093e-12, 2.9779344e-12, 9.9719843e-12, 1.2265875e-12, 2.3258256e-11, 2.7452885e-12, 3.5525243e-11, 4.9341387e-13, 2.516055e-11, 3.9548695e-13, 2.2922348e-11, 1.2079136e-12, 1.486344e-11, 2.8566095e-11, 1.1704213e-11, 1.3646523e-11, 3.290667e-11, 1.1824593e-11, 1.9435993e-11, 1.4976484e-11, 6.4421688e-12, 3.1013157e-12, 1.4837763e-11, 2.4438825e-11, 1.8236083e-11, 1.5704544e-11, 1.0533647e-11, 7.9248558e-12, 8.2939992e-12, 1.8438273e-11, 2.6298214e-11, 3.3018713e-11, 4.3504675e-12, 1.2537536e-11, 1.660645e-12, 1.8246976e-11, 9.6453018e-12, 2.9774898e-12, 4.012225e-12, 1.3658638e-11, 4.5602156e-12, 8.7537333e-12, 1.76214e-11, 1.7859048e-11, 1.0328901e-11, 1.946067e-11, 1.7977872e-11, 1.3260373e-11, 8.0306747e-12, 3.7705756e-12, 1.5674755e-11, 2.0868439e-11, 1.0141161e-11, 9.1175413e-12, 2.7824474e-11, 1.8477955e-11, 3.1111084e-11, 1.8002659e-11, 4.4133253e-11, 2.1554261e-11, 3.4529856e-11, 1.7164445e-12, 4.1950516e-11, 1.9599501e-11, 5.5902602e-11, 2.3968198e-11, 5.3617381e-11, 3.4065898e-11, 5.772631e-11, 4.3247909e-11, 2.910964e-11, 3.540753e-11, 2.7571931e-11, 1.1315284e-11, 1.8592999e-11, 2.5544255e-11, 1.6387143e-11, 1.5040508e-11, 2.2639905e-11, 5.4360002e-12, 8.6666995e-12, 5.6874312e-12, 1.5383751e-13, 8.4142681e-12, 2.35447e-12, 1.4626348e-11, 3.7153319e-12, 1.3556488e-11, 3.5347063e-14, 1.3290495e-11, 1.0186957e-11, 6.1501665e-12, 8.5577683e-13, 1.3072633e-11, 1.0210744e-11, 2.2636236e-11, 2.474839e-11, 2.8420038e-11, 4.5050942e-11, 2.2227633e-11, 3.1792015e-11, 3.6332556e-11, 3.8697586e-11, 4.2348893e-11, 3.2118253e-11, 2.8153046e-11, 2.4457277e-11, 1.9662081e-11, 2.1734997e-11, 6.8804476e-14, 1.1115651e-11, 4.9918277e-12, 1.8982929e-12, 1.1247702e-11, 2.0770401e-11, 3.5807463e-11, 8.5853346e-12, 2.4645239e-11, 2.9784902e-12, 3.769364e-11, 1.2267098e-11, 3.746344e-11, 1.1964203e-11, 3.5184555e-11, 1.8121038e-11, 3.9607495e-11, 3.9379517e-11, 3.7426425e-11, 3.3673968e-11, 4.049684e-12, 2.1577381e-11, 1.3780908e-11, 2.3483121e-11, 2.7510908e-11, 1.4425381e-11, 3.1311828e-11, 3.6918784e-11, 1.8378583e-11, 2.2417929e-11, 9.0246163e-12, 6.4247186e-14, 9.9341918e-12, 7.702325e-12, 4.1842474e-11, 6.886208e-11, 9.5465635e-11, 1.6657403e-10, 2.5517066e-10, 3.8494351e-10, 5.6399439e-10, 8.0096699e-10, 1.1594728e-09, 1.6400004e-09, 2.3573629e-09, 3.2925991e-09, 4.651614e-09, 6.4904372e-09, 9.0606513e-09, 1.2616985e-08, 1.750877e-08, 2.4246208e-08, 3.3509402e-08, 4.6136119e-08, 6.3343107e-08, 8.6727815e-08, 1.1841186e-07, 1.6118977e-07, 2.1885078e-07, 2.9625965e-07, 3.9988425e-07, 5.3816045e-07, 7.2207441e-07, 9.6595904e-07, 1.2882465e-06, 1.7128436e-06, 2.2702853e-06, 2.9998008e-06, 3.9511764e-06, 5.1876766e-06, 6.7891422e-06, 8.8560828e-06, 1.1514183e-05, 1.4920328e-05, 1.9269039e-05, 2.4800646e-05, 3.1810529e-05, 4.0660049e-05, 5.1789098e-05, 6.5730358e-05, 8.3125583e-05, 0.00010474378, 0.00013150143, 0.00016448458, 0.00020497294, 0.00025446541, 0.00031470709, 0.00038771704, 0.00047581629, 0.00058165571, 0.00070824225, 0.00085896345, 0.0010376082, 0.0012483837, 0.0014959262, 0.0017853056, 0.0021220223, 0.0025119947, 0.0029615384, 0.0034773344, 0.0040663876, 0.0047359764, 0.0054935914, 0.0063468671, 0.007303506, 0.0083711979, 0.0095575358, 0.01086993, 0.012315523, 0.013901112, 0.015633065, 0.017517262, 0.019559027, 0.021763082, 0.024133505, 0.026673708, 0.029386413, 0.032273654, 0.035336779, 0.038576467, 0.04199275, 0.045585049, 0.049352209, 0.053292541, 0.05740387, 0.061683583, 0.066128679, 0.070735817, 0.075501368, 0.08042146, 0.085492024, 0.090708839, 0.096067568, 0.1015638, 0.10719306, 0.11295089, 0.11883282, 0.12483441, 0.13095128, 0.13717913, 0.14351373, 0.14995096, 0.15648679, 0.16311732, 0.16983875, 0.17664742, 0.1835398, 0.19051247, 0.19756214, 0.20468567, 0.21188001, 0.21914228, 0.22646967, 0.23385953, 0.24130932, 0.2488166, 0.25637904, 0.26399444, 0.27166068, 0.27937575, 0.28713777, 0.29494488, 0.30279546, 0.31068767, 0.31862067, 0.32659107, 0.33460296, 0.34264136, 0.3507378, 0.35882029, 0.36702912, 0.37510944, 0.38346982, 0.39151817, 0.40002843, 0.40807226, 0.41671242, 0.42477037, 0.43354705, 0.44164824, 0.4504208, 0.45880128, 0.46731437, 0.47564069, 0.48422064, 0.49218235, 0.50114621, 0.50900396, 0.518143, 0.52591298, 0.5352627, 0.54287413, 0.55237234, 0.55994548, 0.56971587, 0.57701518, 0.58720775, 0.59417315, 0.60464393, 0.61157174, 0.62223854, 0.62896741, 0.63994151, 0.6463752, 0.65756992, 0.66373256, 0.67507394, 0.68084696, 0.69222455, 0.69760447, 0.70898071, 0.71394469, 0.7252862, 0.72979015, 0.74095639, 0.74500105, 0.75577082, 0.75958339, 0.77002512, 0.77383606, 0.78382725, 0.78789691, 0.79706106, 0.80153898, 0.80944065, 0.81447886, 0.82092838, 0.82676051, 0.83227023, 0.84016694, 0.84345249, 0.85416931, 0.85538181, 0.86689314, 0.87031671, 0.88113769, 0.88503948, 0.89266435, 0.89726632, 0.90207678, 0.90609692, 0.90963674, 0.91258655, 0.91502049, 0.91694533, 0.9185455, 0.91993565, 0.92127913, 0.92250348, 0.9236348, 0.92449581, 0.92534573, 0.92593932, 0.926508, 0.92681627, 0.92684728, 0.92684988, 0.92681491, 0.92675624, 0.92662322, 0.92641057, 0.92621132, 0.92590999, 0.92573187, 0.92555097, 0.92539485, 0.92529249, 0.9251848, 0.92514504, 0.92503605, 0.9249758, 0.92487104, 0.92480741, 0.92477195, 0.92475021, 0.92474446, 0.92473058, 0.92466689, 0.92464599, 0.92459863, 0.92455818, 0.92454052, 0.9245201, 0.92451419, 0.92451885, 0.92448997, 0.92448865, 0.92451428, 0.92451891, 0.9246874, 0.92476524, 0.92500648, 0.92509177, 0.92524675, 0.92532686, 0.92542361, 0.92548585, 0.9255706, 0.92561766, 0.92567513, 0.92568916, 0.92568702, 0.92568594, 0.92568644, 0.92568731, 0.92568733, 0.92568538, 0.92568545, 0.92568528, 0.92568663, 0.9256866, 0.92568528, 0.92568502, 0.92567256, 0.92566365, 0.92565065, 0.92563364, 0.92563023, 0.92560237, 0.92558626, 0.92558102, 0.92556718, 0.92556095, 0.92553352, 0.92552234, 0.92551443, 0.92551346, 0.92553188, 0.92554215, 0.92553593, 0.92551216, 0.9254981, 0.92548397, 0.92545191, 0.9254438, 0.92546047, 0.92545948, 0.92545321, 0.92546942, 0.92546893, 0.92548178, 0.9254962, 0.92550577, 0.92551064, 0.92551861, 0.92551255, 0.92551154, 0.92550125, 0.9254909, 0.92547777, 0.92546059, 0.9254376, 0.92542317, 0.92541357, 0.92540004, 0.92539003, 0.92539268, 0.92539484, 0.92539319, 0.92538687, 0.92537376, 0.92536217, 0.92533494, 0.92532182, 0.92530103, 0.92529295, 0.92528055, 0.9252808, 0.92528141, 0.92527479, 0.92526564, 0.92523915, 0.92522498, 0.92519033, 0.92516932, 0.92514302, 0.92513574, 0.92510167, 0.92508319, 0.92504168, 0.92502395, 0.92498993, 0.92496413, 0.92494274, 0.92494213, 0.92495041, 0.9249836, 0.92499667, 0.92499782, 0.92498454, 0.92497894, 0.92492624, 0.92489385, 0.92487897, 0.92509758, 0.92504302, 0.92491965, 0.92467987, 0.92444044, 0.92344321, 0.92290962, 0.92093694, 0.92008582, 0.9162359, 0.91463478, 0.90802769, 0.90425492, 0.89399229, 0.88607405, 0.86898745, 0.85620349, 0.82452756, 0.80408067, 0.75438196, 0.71474921, 0.64709112, 0.57709269, 0.48776217, 0.39181642, 0.29379284, 0.20417154, 0.13180417, 0.079941381, 0.046261304, 0.025928858, 0.014247258, 0.0077397344, 0.0041786383, 0.0022487831, 0.0012082535, 0.00064867088, 0.00034811725, 0.00018678768, 0.00010021515, 5.3764932e-05, 2.884384e-05, 1.5473854e-05, 8.3011042e-06, 4.4531444e-06, 2.3888279e-06, 1.2814482e-06, 6.8740462e-07, 3.6877249e-07, 1.9781662e-07, 1.0612857e-07, 5.6907018e-08, 3.0530822e-08, 1.6362161e-08, 8.775146e-09, 4.6521183e-09, 2.5235309e-09, 1.3190337e-09, 7.1714996e-10, 3.3889165e-10, 1.8756574e-10, 7.8574173e-11, 2.5481803e-11, 6.9994292e-12, 1.247901e-11, 4.1408415e-11, 4.7442414e-12, 4.0464184e-11, 1.6622659e-11, 2.5161139e-11, 2.3042919e-11, 2.7545969e-11, 2.1857719e-11, 1.0317612e-11, 3.6101653e-11, 8.1525515e-13, 1.4730091e-12, 4.1200966e-11, 2.1666919e-11, 2.549046e-11, 1.2807665e-11, 1.5924281e-11, 1.1008553e-11, 3.9694991e-11, 2.012476e-11, 2.4571647e-11, 6.2012669e-13, 3.3875435e-11, 1.7536811e-11, 3.5456885e-11, 9.438868e-12, 1.5482856e-11, 1.2117833e-11, 1.2255133e-11, 3.2043691e-11, 4.9047395e-12, 5.9504194e-13, 2.3257249e-11, 9.3155531e-12, 4.0251407e-11, 1.6236398e-11, 2.031778e-11, 5.1861107e-12, 4.9291028e-11, 8.2434564e-12, 9.2215406e-12, 2.4410372e-11, 1.5256538e-11, 5.7015363e-11, 3.3725704e-12, 5.4832432e-11, 5.1818707e-11, 4.5330519e-11, 3.5841038e-11, 3.9551697e-11, 4.6543135e-11, 6.3229727e-11, 4.3508543e-11, 3.6803361e-11, 2.5596571e-11, 5.6237735e-11, 2.0785067e-11, 5.3363974e-11, 4.6250331e-12, 5.7681996e-11, 1.4312085e-11, 3.4661054e-11, 1.451132e-11, 4.1207626e-11, 2.0039405e-11, 5.1141307e-11, 5.7767129e-11, 6.5028949e-11, 6.081504e-11, 6.6510061e-11, 6.9528559e-11, 5.3917837e-11, 3.3359199e-11, 3.1915825e-11, 2.4460763e-11, 2.6618498e-11, 1.9054772e-11, 2.2080929e-11, 2.1238369e-11, 4.8883122e-12, 2.9346301e-11, 3.8926909e-12, 1.8546972e-11, 3.352447e-11, 7.7369881e-12, 2.0904608e-11, 5.2964393e-11, 3.7098162e-11, 3.698628e-11, 5.4720328e-12, 1.7230799e-12, 1.4661718e-11, 3.0158781e-11, 8.727504e-13, 7.4551729e-12, 9.6010311e-12, 9.7009263e-12, 4.6964914e-11, 1.7044439e-11, 3.1317343e-11, 4.2369517e-11, 3.8669623e-11, 1.3587734e-11, 4.0327661e-11, 2.2638121e-11, 2.0275824e-11, 2.1832634e-12, 2.6967021e-11, 3.5043653e-11, 8.9498259e-12, 1.7058757e-11, 3.7238681e-12, 4.9141296e-11, 1.4965288e-11, 6.4129671e-11, 9.1424014e-12, 5.4471811e-11, 1.130724e-11, 1.602795e-11, 3.6961417e-11, 5.9972591e-12, 2.417473e-11, 3.6010304e-11, 2.7460281e-11, 1.8913365e-11, 2.2333219e-12, 9.1835805e-12, 1.0674682e-11, 2.0343309e-11, 1.0621071e-11, 2.5540186e-11, 1.0256232e-11, 4.1173994e-11, 4.7592922e-11, 3.8839778e-11, 5.586668e-11, 4.5768393e-11, 5.4112521e-11, 1.3272288e-11, 1.3262631e-11, 2.1268781e-12, 2.0194243e-11, 4.7091671e-12, 3.7831725e-11, 1.5198377e-11, 9.3552891e-12, 3.8123752e-11, 9.7382204e-12, 2.1738621e-11, 1.7108705e-12, 5.4503888e-12, 3.0085081e-12, 2.2196696e-12, 1.3662545e-11, 1.8426765e-11, 6.4032773e-12, 2.0263392e-11, 3.1608037e-11, 4.4126672e-11, 2.1159896e-11, 3.9597982e-11, 1.1844232e-11, 1.6582368e-11, 9.2536177e-13, 2.2899403e-11, 6.6586758e-12, 1.3248202e-11, 1.8486591e-11, 4.3278784e-11, 3.2563479e-11, 1.3640346e-11, 1.2093747e-11, 3.9193296e-12, 1.6752745e-11, 6.6797648e-12, 1.0561134e-11, 1.7613841e-11, 6.0695166e-12, 3.1665977e-11, 1.3746124e-11, 2.4588629e-12, 2.1681015e-11, 1.1307129e-11, 5.1117999e-11, 1.8270707e-11, 3.9627285e-12, 8.1229163e-12, 3.1189482e-14, 8.6509177e-13, 2.0935576e-11, 1.1765536e-11, 3.1098461e-11, 5.3431792e-11, 4.5414209e-11, 3.6909915e-11, 5.9130031e-11, 3.0514297e-11, 5.9756929e-11, 5.2456704e-11, 3.5227349e-11, 4.0291587e-11, 1.2385774e-11, 1.3110236e-11, 1.0282982e-11, 3.7945717e-12, 1.1037634e-11, 9.5456448e-12, 1.2120386e-11, 3.4301765e-11, 2.9848773e-11, 2.483648e-11, 1.5567101e-11, 1.8956764e-11, 3.6095881e-11, 5.7025684e-12, 1.6939771e-11, 3.0161445e-11, 6.3281338e-12, 1.0585664e-11, 1.3458537e-11, 1.4699567e-11, 2.6314706e-11, 7.8859428e-12, 2.2798398e-11, 1.1018876e-11, 1.6363931e-11, 6.347669e-12, 1.6589916e-11, 2.0036741e-12, 1.6865848e-11, 5.4995594e-12, 5.0944292e-12, 1.3116007e-11, 2.4353432e-11, 3.5570433e-12, 6.7930902e-12, 2.3220066e-12, 9.9536609e-12, 1.8856536e-11, 8.7824461e-12, 7.618224e-12, 1.6572712e-11, 5.5821394e-12, 2.3594118e-12, 1.351692e-11, 1.9621733e-11, 2.3416193e-11, 8.477988e-12, 1.7558232e-12, 1.3480181e-11, 1.6566829e-11, 8.3494562e-12, 1.0927638e-11, 4.0086691e-11, 1.6651962e-11, 2.7324424e-11, 3.4887816e-12, 1.3475297e-11, 2.1159452e-11, 5.2683577e-12, 1.3745236e-11, 3.5893205e-11, 3.4344276e-11, 4.7905261e-12, 2.1948068e-11, 4.4536575e-12, 5.2116617e-11, 3.6534865e-11, 6.9206452e-11, 3.8344298e-11, 4.4512489e-11, 2.1158231e-11, 1.856573e-11, 1.5882214e-12, 1.1754437e-11, 1.8347626e-11, 2.3172338e-11, 2.9199233e-11, 4.1969381e-11, 3.6500567e-11, 4.019036e-11, 5.0911105e-11, 3.4911625e-20];

option = {
  dataZoom: [
    {
      type: 'slider',
      minSpan: 1
    },
    {
      type: 'inside',
      minSpan: 1
    }
  ],
  tooltip: {
      trigger: 'axis',
      formatter: function (params) {
          let newParams = [];
          let tooltipString = [];
          newParams = [...params];
          newParams.sort((a,b) => {return b.value - a.value});
          newParams.forEach((p) => {
              const cont = p.marker + ' ' + p.seriesName + ': ' + p.value + '<br/>';
              tooltipString.push(cont);
          });
          return tooltipString.join('');
      }
  },
 legend: {
    data: legends
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '9%',
    containLabel: true
  },
  // toolbox: {
  //   feature: {
  //     saveAsImage: {}
  //   }
  // },
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data: xaxis
  },
  yAxis: {
    min: 'dataMin',
    type: 'value',
    axisTick: {
      alignWithLabel: true
    },
    scale: true,
  },
  series: [
    {
      name: legends[0],
      type: 'line',
      showSymbol: false,
      data: data0
    },
    
    {
      name: legends[1],
      type: 'line',
      showSymbol: false,
      lineStyle:{
        type:'dashed'  //'dotted'虚线 'solid'实线
      },   
      data: data1
    },
    {
      name: legends[2],
      type: 'line',
      showSymbol: false,
      lineStyle:{
        type:'dashed'  //'dotted'虚线 'solid'实线
      },   
      data: data2
    },
    {
      name: legends[3],
      type: 'line',
      showSymbol: false,
      lineStyle:{
        type:'dashed'  //'dotted'虚线 'solid'实线
      },   
      data: data3
    },
    {
      name: legends[4],
      type: 'line',
      showSymbol: false,
      lineStyle:{
        type:'dashed'  //'dotted'虚线 'solid'实线
      },   
      data: data4
    },
    {
      name: legends[5],
      type: 'line',
      showSymbol: false,
      lineStyle:{
        type:'dashed'  //'dotted'虚线 'solid'实线
      },   
      data: data5
    },
    {
      name: legends[6],
      type: 'line',
      showSymbol: false,
      lineStyle:{
        type:'dashed'  //'dotted'虚线 'solid'实线
      },   
      data: data6
    },
        {
      name: legends[7],
      type: 'line',
      showSymbol: false,
      lineStyle:{
        type:'dashed'  //'dotted'虚线 'solid'实线
      },   
      data: data7
    }
  ]
};
{% endecharts %}



