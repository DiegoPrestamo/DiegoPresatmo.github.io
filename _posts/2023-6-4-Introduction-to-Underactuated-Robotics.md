---
layout: post
title: Introduction Material to Underactuated Robotics - MIT 6.832
tags: [Mathjax, Mathematic]
mathjax: true
---
## Underactuated Robotics or: How I learned to Stop Worrying and Love Dynamics
This week I was emailing back and forth with a professor at Syracuse University when he sent me this ominous link: https://underactuated.csail.mit.edu/

The link took me to Russ Tedrake's grad level MIT course: *Underactuated Robotics*. I have been completely fascinated by the material for the previous four days and nights. I am documenting my learning as I go along because I know teaching it will force me to becoming proficient in the material. I will introduce the topic and the fundamental ideas you need to get started with the course. Although I knew the math, I had a couple of gaps in my knowledge that I needed to fill before getting started. I will likely update this blog post if I keep finidng additional material I find relevant.

## Foreword: 
Most of the material in this blog post (such as examples, pictures, etc.) is directly from Russ Tedrake's course. I have no idea what the copyright landscape looks like for this but the course notes are published, so I will assume I can safely republish my interpretations of the course material. Send any *Cease and Desist* orders to my email and I will (begrudgingly) surrender.

## There is something off about ASIMO:
<div align="center">
  <p>
      <img src="https://www.1999.co.jp/itbig02/10020560k_m.jpg" width="600">
  </p>
        Click <a href="https://www.youtube.com/watch?v=FEXSqsW6rMM&t=6s&ab_channel=KingRoseArchives">here</a> for the demo video
</div>        
The robotics world was shocked when Honda announced they had been secretely developing a humanoid for the previous decade. They introduced their model, ASIMO, an astounding feat of engineering. ASIMO could walk, grip, avoid collisions, amongst other things.

But when we look at ASIMO's demo video, there is something wrong with its movement. Its just not natural. What is it that makes ASIMO's movement robotic, and [Atlas'](https://www.youtube.com/watch?v=tF4DML7FIWk&ab_channel=BostonDynamics) movement much more human-like?

## Our underactuated world:
The fundamental reason comes down to dynamics. In order to achieve the desired control, ASIMO is essentially fighting an endless battle against its natural dynamics. Hence the bent knees and slow movements. The idea goes as follows: if we can use feedback to cancel the system's dynamics, the control problem is trivial. The idea has merit and is useful in many settings. Manipulator robots excel in industrial settings where they are bolted down and dynamic cancellation is necessary. Full control all the time. Any desired acceleration can be achieved at any time. 

However, this control ideology begins to hold us down when we get to more complicated control models. I want to prove this to you. Stand up, and take three steps at your normal gait. Now turn around and take three really slow steps, kind of like ASIMO would. I am willing to bet you found the second go harder. Your body was expending much more energy fighting gravity and the movement was surely not too graceful. Although unnatural, this type of movement is very well understood in control theory and is simple as long as you have enough power.

The idea of exploiting mechanical systems and riding dynamics prevails in nature. An albatross can fly for kilometers without flapping its wings, a rainbow trout can ride up stream currents simply using their anatomical mechanisms, and a gymnast can do backflips with relative ease.
 <div align="center">
  <p>
      <img src="https://i0.wp.com/fyfluiddynamics.com/wp-content/uploads/2018/07/A-Dead-Fish-Swims.gif?fit=400%2C246&ssl=1" width="600">
  </p>
        A trout swims upstream. The fish is dead and only tethered to position it correctly!
</div>      

This type of control occurs when there are less actuators than degrees of freedom. This is known as underactuated control. 

"If you had more research funding, would you be working with fully actuated robotics?" Prof. Manolis Kellis famously jokes with Tedrake in an MIT staff meeting.

Underactuated robotics does not come from a lack of materials (or research funding), it comes from a desire to maximize the control of our robotics at the trade-off of much more complicated control problems.

Now that we have an intuitive idea of underactuated robotics, let us go into the math.

## Prerequisite Skills:
Here is what I have found is necessary to stay afloat so far:

- All of calculus (up to vector / multivariable calculus)
- Differential Equations
- Linear Algebra
- Physics - Kinematics
- Patience - If you are an engineering undergrad like I am, this material will likely demand your full abilities but this stuff is really cool so stick with it

## Fundamental Concepts: 
I found that there were a couple of concepts that I had not been exposed to or used. In this blog post, I will walk you along the following funamental concepts that you need to get started:

- Generalized Coordinates
- Principle of Least Action (POLA)
- Lagrangian / Euler-Lagrange equation
- Control affine
- Robotic Manipulation Equation
- Underactuated vs. Fully-actuated mathematically

Once you understand concepts, the first lecture will make much more sense.

## Generalized Coordinates:
Generalized coordinates are parameters used to completely describe the configuration of a system. For our purposes, generalized coordinates might refer to the angles of each join in the robot. The main advantage of generalized coordinates is that they simplify system analysis. Rather than trying to keep track of the position and orientation of every component of the robot in three-dimensional space, you can instead use a smaller number of generalized coordinates that capture the essential degrees of freedom of the system. They are also quite nice because they align with Lagrangian mechanics which will be covered shortly.

## Principle of Least Action:
A key concept in Lagrangian and Hamiltonian mechanics, POLA states that the path taken by a system between two states is the one for which the action is minimized. Given the kinetic and potential energy of a robot (expressed in terms of the generalized coordinates and their time derivatives), you can apply the principle of least action and the Euler-Lagrange equation to derive the equations of motion.

## Lagrangian / Euler-Lagrange Equation:
The Lagrangian will provide us a powerful alternative to Newton's laws which are in terms of Forces. It can be very difficult, if not impossible to analyze all the forces in a complex system and the Lagrangian method gives us an elegant way to simplify things,

$\\ L = T - V\$ 

where $\ T$ = kinetic energy and $\ V$ = potential energy. 

We will then use the Euler-Lagrange equation to find the equations of motion of our system: 
$$\large \frac{\partial L}{\partial q_i} - \frac{d}{dt} \left(\frac{\partial L}{\partial \dot{q}_i}\right)$$ = 0

where $\ q_i$ are our generalized coordinates. 


#### Finding equations of motion of a double pendulum using Lagrangian and Euler-Lagrange equation:
This derivation can take about an hour if you have never done it but I found it useful doing it once and then using software to find the Lagrangian and equations of motion moving forward. I will not go through every step as that would be a LaTex nightmare but if you ever get stuck, follow along [this](https://www.youtube.com/watch?v=KSsZUn0bfwE&ab_channel=PhysicsExplained) video. 

 <div align="center">
  <p>
      <img src="https://scienceworld.wolfram.com/physics/dimg270.gif" width="200">
  </p>
</div>   

The location of $\ m_1$ and $\ m_2$ is deoted by $\ p_1$ and $\ p_2$ respectively. We simply use $\ q$ = $$\ \begin{bmatrix} \ \theta_1 , \theta_2 \end{bmatrix}^T $$. We could totally work in terms of $\theta$ but getting used to generalized coordinates can't hurt.

$\ p_1$:

$$\ \begin{bmatrix} \ x_1 \\ y_1 \end{bmatrix} $$ = $$\ \begin{bmatrix} \ l_1sin{q_1} \\ -l_1cos{q_1} \end{bmatrix} $$

$\ p_2$:

$$\ \begin{bmatrix} \ x_2 \\ y_2 \end{bmatrix} $$ = $\dot p_1$ + $\dot p_2$
$$\ \begin{bmatrix} \ x_2 \\ y_2 \end{bmatrix} $$ = $$\ \begin{bmatrix} \ l_1sin{q_1} + l_2sin{q_2} \\ -l_1cos{q_1} - l_2cos{q_2} \end{bmatrix} $$

We then take the time derivative of our positions in order to find our velocities, which we need for our kinetic energy: $\ T = \frac{1}{2} mv^2$

$\dot p_1$:

$$\ \begin{bmatrix} \dot x_1 \\ \dot y_1 \end{bmatrix} $$ = $$\ \begin{bmatrix} \ l_1\dot q_1 cos{q_1} \\ l_1\dot q_1sin{q_1} \end{bmatrix} $$

$\dot p_2$:

$$\ \begin{bmatrix} \dot x_1 \\ \dot y_1 \end{bmatrix} $$ = $$\ \begin{bmatrix} \ l_1sin{q_1} \\ -l_1cos{q_1} \end{bmatrix} $$

We can rewrite our kinetic energy equation as $T = \frac{1}{2}m(\dot x^2 + \dot y^2)$

Reducing our $\dot p_1$ and $\dot p_2$ equations we get our final kinetic energy equation: 

$T = \frac{1}{2}m_1l_1^2\dot{q}^2 + \frac{1}{2}m_2l_2^2[l_1^2\dot q_1^2 + l_2^2\dot q_2 + 2l_1l_2\dot q_1\dot q_2cos(q_1-q_2)]$

Finding the potential energy is trivial: 

$\ V = m_1gy_1 + m_2gy_2$

$\hspace{.83cm}   = -m_1gl_1cos{q_1} - m_2g(l_1cos{q_1} + l_2cos{q_2})$

$\hspace{.83cm}   = -(m_1 + m_2)gl_1cosq_1 + m_2g(l_2cosq_2)$

Now we plug $T$ and $V$ into our Lagrangian and we get:

$L = \frac{1}{2}m_1l_1^2\dot{q}^2 + \frac{1}{2}m_2l_2^2[l_1^2\dot q_1^2 + l_2^2\dot q_2 + 2l_1l_2\dot q_1\dot q_2cos(q_1-q_2)] + (m_1 + m_2)gl_1cosq_1 + m_2g(l_2cosq_2)$

Now we move onto Euler-Lagrange to find our motion equations $q_1$ and $q_2$

Deriving the first term $\tau _1$:

$\large \frac{\partial L}{\partial q_1} - \frac{d}{dt} \left(\frac{\partial L}{\partial \dot{q}_1}\right)$ =  0

$\large \frac{\partial L}{\partial q_1}$ = $-m_2l_1l_2\dot q_1\dot q_2sin{(q_1-q_2)} - (m_1 + m_2)gl_1sinq_1$ 

$\large \frac{\partial L}{\partial \dot{q}_1}$ = $m_1l_1^2 \dot q_1 + m_2l_1^2 \dot q_1 + m_2l_1l_2 \dot q_2 cos(q_1 - q_2)$

$\large \frac{d}{dt} \left(\frac{\partial L}{\partial \dot{q}_1}\right)$ = $(m_1 + m_2)l_1^2\ddot q_1 + m_2l_1l_2\ddot q_2cos(q_1 - q_2)$ 

$\hspace{6.8cm} - m_2l_1l_2\dot q_2sin(q_1 - q_2)(\dot q_1 - \dot q_2)$

Deriving the second term $\tau _2$:

$\large \frac{\partial L}{\partial q_2} - \frac{d}{dt} \left(\frac{\partial L}{\partial \dot{q}_2}\right)$ =  0

$\large \frac{\partial L}{\partial q_2}$ = $m_2l_1l_2\dot q_1\dot q_2sin(q_1 - q_2) - m_2gl_2sinq_2$

$\large \frac{\partial L}{\partial \dot{q}_2}$ = $m_2l_2^2\dot q_2 + m_2l_1l_2\dot q_1cos(q_1 - q_2)$

$\large \frac{d}{dt} \left(\frac{\partial L}{\partial \dot{q}_2}\right)$ = $m_2l_2^2\ddot q_2 + m_2l_1l_2\ddot q_1 cos(q_1 - q_2) - m_2l_1l_2\dot q_1sin(q_1 - q_2)(\dot q_1 - \dot q_2)$


Putting everything together, we get our equations of motion: 

$\tau _1 = (m_1 + m_2)l_1^2\ddot q_1 + m_2l_1l_2\ddot q_2cos(q_1 - q_2)+ m_2l_1l_2\dot q_2^2sin{(q_1-q_2)} + (m_1 + m_2)gl_1sinq_1$ 

$\tau _2 = m_2l_2^2\ddot q_2 + m_2l_1l_2\ddot q_1cos(q_1 - q_2) - m_2l_1l_2\dot q_1^2sin(q_1 - q_2) + m_2gl_2sinq_2$

## Robotic Manipulation Equation: 
We can generalize most of our robots into this simple equation: 

$M(q)\ddot q + c(q,\dot q) = \tau g(q) + Bu$

where: 
- M is our generalized mass / inertia matrix
- C is our coriolis terms (or velocity product term) - when one joint is moving, it can induce forces on other joints that are also moving, due to the Coriolis and centrifugal effects
- g is our gravity term - this is assuming that our potential energy only comes from gravity; springs on our robot would throw this off
- B is our actuation matrix 
- u is our torque control input 

B will help us determine whether our system is underactuated or not

## Control Affine:
A control affine system can be written in the following form: 

$\dot x = f(x) + g(x)u$

- $x$ is the state of the system
- $\dot x$ is the time derivative of the system
- $u$ is the control input - this is our actuation (servo torques, wheel's angular velocity, etc.)
- $f(x)$ describes the uncontrolled dynamics - this is what happens when u = 0
- g(x) describes how the control input affects the system

We will see the control system
The key characteristic of a control affine system is that the control input $u$ only multiplies the function $g(x)$ and does not appear inside f or g, this gives us a linear part and a constant, making control affine systems easier to handle than more general nonlinear systems.

We will use the second-order system control in the form of:

$\ddot q = f(q,\dot q, u, t)$

where u is our control vector from earlier.

If there exists a u that achieves any desired acceleration $\ddot q$, we call the system fully actuated. If we cannot, then the system is underactuated. In simpler terms, if we can perfectly control our system and not have to deal with dynamics, then our control problem is simple and fully actuated. If we cannot, then we must lean into the dynamics of our system, making the control problem more difficult but much more exciting. 


## Underactuated vs. Fully Actuated Mathematically:
An underactuated system has less actuators than degrees of freedom. Let us put the manipulator equation from earlier: 

$M(q)\ddot q + c(q,\dot q) = \tau g(q) + Bu$

in terms of $\ddot q$:

$\ddot q = M^{-1}(q)[-C(q,\dot q) + \tau g(q) + Bu]$

If B has full row rank, this indicates that the control inputs are able to influence the degrees of freedom independently, which indicates full actuation. If B does not have full row rank, it means that some control inputs will affect the motion in multiple degrees of freedom, indicating underactuation.

## Conclusion:
I hope this blog post is a nice supplement to Tedrake's online lecture 1 and online textbook. If you spot an error, do not hesitate to send me an email. Keep building.






















