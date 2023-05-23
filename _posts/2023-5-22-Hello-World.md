---
layout: post
title: Introduction to Poppy Humanoid using V-REP / CoppeliaSim
---

#### The Poppy Project is an open-source platform that intertwines science, technology, art, and education. The Poppy Humanoid is the flagship model of the Poppy Project. Standing at 83 cm tall, the Poppy Humanoid is almost fully 3D printed and uses 25 motors to animate its body. These features make the Poppy Humanoid a cheaper alternative in the world of complex robotic systems. However, $10,000 is still far too much for (most) 21 year-old college students, so I decided to explore the Poppy Humanoid using V-REP, which is now Coppelia Sim.

<p align="center">
  <img src="https://www.poppy-project.org/assets/img/humanoid-skating.jpg" width="500">
</p>



## Brief introduction of V-REP and Poppy: 
V-REP which is now known as Coppelia Sim is a powerful robotics simulator that allows you to import or create various robots, program them as you would in real life, and simulate them visually and quantitatively. V-REP uses **_Lua_**, a lightweight, high-level, multi-paradigm coding language that is mostly used in embedded systems. The Poppy Humanoid is most commonly programmed in Python with its native **_pypot_** library. Since we want to get as close as possible to working with the real Poppy Humanoid, we will use the Python API support that V-REP comes with. This will save us the hassle of learning **_Lua_** syntax and will make our code almost identical to what we would run on a real-life Poppy Humanoid.


## Step 1: Downloading V-REP for your appropriate OS
Have you noticed that I keep saying 'V-REP, which is now named Coppelia sim' instead of 'Coppelia sim, formerly known as V-REP'? Indeed we will be using V-REP and not Coppelia Sim. The Poppy documentation specifically states that we should infact not run our simulation using Coppelia sim but using V-REP 3.3.0 instead. 
