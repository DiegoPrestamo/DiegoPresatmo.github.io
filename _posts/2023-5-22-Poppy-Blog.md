---
layout: post
title: Introduction to Poppy Humanoid using V-REP / CoppeliaSim
---

#### The Poppy Project is an open-source platform that intertwines science, technology, art, and education. The Poppy Humanoid is the flagship model of the Poppy Project. Standing at 83 cm tall, the Poppy Humanoid is almost fully 3D printed and uses 25 motors to animate its body. These features make the Poppy Humanoid a cheaper alternative in the world of complex robotic systems. However, $10,000 is still far too much for (most) 21 year-old college students, so I decided to explore the Poppy Humanoid using V-REP, which is now Coppelia Sim.

<p align="center">
  <img src="https://www.poppy-project.org/assets/img/humanoid-skating.jpg" width="800">
</p>



## Brief introduction of V-REP and Poppy: 
V-REP which is now known as Coppelia Sim is a powerful robotics simulator that allows you to import or create various robots, program them as you would in real life, and simulate them visually and quantitatively. V-REP uses **_Lua_**, a lightweight, high-level, multi-paradigm coding language that is mostly used in embedded systems. The Poppy Humanoid is most commonly programmed in Python with its native **_pypot_** library. Since we want to get as close as possible to working with the real Poppy Humanoid, we will use the Python API support that V-REP comes with. This will save us the hassle of learning **_Lua_** syntax and will make our code almost identical to what we would run on a real-life Poppy Humanoid.
## Step 0: Download **_Python_**
You should have **_Python_** already installed. I am running 3.8.10, but I believe any Python 3 version should work. If you do not have **_Python_** installed, go [here](https://www.python.org/downloads/) and follow the instructions.
## Step 1: Downloading V-REP for your appropriate OS
Have you noticed that I keep saying 'V-REP, which is now named Coppelia sim' instead of 'Coppelia sim, formerly known as V-REP'? Indeed we will be using V-REP and not Coppelia Sim. The Poppy documentation specifically states that Windows users should infact not run Coppelia sim but using V-REP 3.3.0 instead. If you are on Linux, you can use the most recent version of Coppelia Sim (last tested with CoppeliaSIM 4.2.0 + python3.6.12 + Ubuntu 20.04). I am using V-REP EDU, since I am a student and do not intend to commercialized any robots (yet!).

### Step 1.5: For MacOS users with the new ARM64 processors
Close the lid, concede your dignity, sell your computer and downgrade to an Intel chip Mac. Jokes aside, I seriously struggled connecting to the remote API server with my M1 Macbook Pro. It seems like certain files were simply not built for the new processors (which is strange since there is an 'Apple Silicon' download in Coppelia sim). Maybe someone more competent than me can figure out the issue, or change the files to fit the ARM64 architecture, but I opted for the practical solution: reach into my backpack and pull out my old windows computer that I keep around for situations such as these.

<p align="center">
  <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/f43a8e14-e316-4317-85b0-390e5c91345f" width="800">
</p>

## Step 2: Scene Synopsis
This is a V-REP scene which I have annotated. I have only highlighted very basic elements of the V-REP scene and I recommend messing with a V-REP scene of your own to get the hang of it.

- The underlined portion is where you have simple controls such as panning in and out, translations in X, Y, Z, rotations about Alpha, Beta, Gamma, and other basic controls. If you want to make a fine tune translation (which are often required, since the regular translation step can be quite large), press **_shift_** while you translate.

- The arrow points to the **_Scene Object Properties_**, an incredibly useful element for fine tuning and specifying how your object will behave in the scene. Here you can change physical properties, dynamic properties, and many other properties.

- The **_Hierarchy_** is where you can see all of the object in the scene and how they relate to each other. If an object is indented under another object, they move together or are related in some way.
<div align="center">
  <p>
    <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/5178c4fb-956f-46d5-b676-7f1169e221f6" width="800">
  </p>
     A Poppy Torso + computer vision scene I am currently working on
</div>

## Step 3: Downloading Appropriate Poppy Libraries














