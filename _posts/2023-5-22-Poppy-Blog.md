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
We will be using **_Python_** to run our simple code. I am running 3.8.10, but I believe any Python 3 version should work. If you do not have **_Python_** installed already, go [here](https://www.python.org/downloads/) and follow the instructions.
## Step 1: Downloading V-REP for your appropriate OS
Have you noticed that I keep saying 'V-REP, which is now named Coppelia sim' instead of 'Coppelia sim, formerly known as V-REP'? Indeed we will be using V-REP and not Coppelia Sim. The Poppy documentation specifically states that Windows users should infact not run Coppelia sim but use [V-REP 3.3.0](https://v-rep-pro-edu.software.informer.com/download/) instead. If you are on Linux, you can use the most recent version of Coppelia Sim (last tested with CoppeliaSIM 4.2.0 + python3.6.12 + Ubuntu 20.04). I am using V-REP EDU, since I am a student and do not intend to commercialized any robots (yet!).

### Step 1.5: For MacOS users with the new ARM64 processors
Close the lid, concede your dignity, sell your computer and downgrade to an Intel chip Mac. Jokes aside, I seriously struggled connecting to the remote API server with my M1 Macbook Pro. It seems like certain files were simply not built for the new processors (which is strange since there is an 'Apple Silicon' download in Coppelia sim). Maybe someone more competent than me can figure out the issue, or change the files to fit the ARM64 architecture, but I opted for the practical solution: reach into my backpack and pull out my old windows computer that I keep around for situations such as these.
<div align="center">
<p align="center">
  <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/f43a8e14-e316-4317-85b0-390e5c91345f" width="800">
</p>
The page should look something like this. Hit 'Download now'.
</div>

## Step 2: Scene Synopsis
This is a V-REP scene which I have annotated. I have only highlighted very basic elements of the V-REP scene and I recommend messing with a V-REP scene of your own to get the hang of it.

- The underlined portion is where you have simple controls such as panning in and out, translations in X, Y, Z, rotations about Alpha, Beta, Gamma, and other basic controls. If you want to make a fine tune translation (which are often required, since the regular translation step can be quite large), press **_shift_** while you translate.

- The arrow points to the **_Scene Object Properties_**, an incredibly useful element for specifying how your object will behave in the scene. Here you can change physical properties, dynamic properties, and many other properties.

- The **_Hierarchy_** is where you can see all of the object in the scene and how they relate to each other. If an object is indented under another object, they move together or are related in some way.
<div align="center">
  <p>
    <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/5178c4fb-956f-46d5-b676-7f1169e221f6" width="800">
  </p>
     A Poppy Torso + computer vision scene I am currently working on
</div>

## Step 3: Downloading Appropriate Libraries (following standard documentation)
As previously mentioned, **_pypot_** is the Poppy library. We will pip install the Poppy Humanoid which should force install **_pypot_** since it is one of its dependencies. Open a Command prompt (windows + cmd) if on Windows, or open your terminal if on MacOS or Linux. Run the following line of code:
```bash
pip install pypot poppy_humanoid
```
As a sanity check, run the following lines of code in your Command Prompt / Terminal
```bash
from pypot.vrep import from_vrep
from poppy.creatures import PoppyHumanoid
```
If they run with no issues, you are likely okay. Else, troubleshoot using [this](https://docs.poppy-project.org/en/installation/install-poppy-softwares.html) and [that](https://notebook.community/matthieu-lapeyre/pypot/samples/notebooks/Controlling%20a%20Poppy%20humanoid%20in%20V-REP%20using%20pypot).

## Step 4: Load a default Poppy Humanoid into V-REP:
To simply load a default Poppy Humanoid into V-REP, open your favorite **_Python_** IDE and run the following lines of code: 
```bash 
from pypot.creatures import PoppyHumanoid # make sure not to use 'poppy.creatures' which has now deprecated and will not work

vrep_port = 19997 
poppy = PoppyHumanoid(simulator='vrep', port = vrep_port)
```
If you are having errors connecting to V-REP, open _**remoteApiConnections.txt**_ file which is in the V-REP path location
  <p>
    <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/732e5253-6e01-4679-85b4-d49d6d34d924" width="800">
  </p>
     I am using 19990 and I created a second port 19991. Yours SHOULD look different, just make sure you match the port number to the port in your script.
</div>


#### When you run your script, you should see the following pop-up:
<div align="center">
  <p>
    <img src="https://docs.poppy-project.org/en/img/vrep/vrep3_2.png" width="800">
  </p>
  Image from Poppy documentation
</div>

Make sure to check the box saying 'Do not show this message again' and re-run you script three times until you no longer get this message. This pop-up blocks **_pypot_** communication, which we do not want.

#### A default Poppy Humanoid should now be loaded in a V-REP scence:
<div align="center">
  <p>
    <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/24a16cac-a35a-4dc7-bdc4-a5e604a97dcd" width="800">
  </p>
</div>

## Step 5: Basic controls of a default Poppy Humanoid
Now that we have loaded a Humanoid onto our simulator, it is time to control it. 

Here is a simple script that moves various joints. These are the basic ways to control the arms. 
```bash 
from pypot.creatures import PoppyHumanoid

vrep_port = 19990


poppy = PoppyHumanoid(simulator='vrep', port = vrep_port)

# right shoulder movement
poppy.r_shoulder_y.goto_position(-30, 0.5, wait=True) # right shoulder movement in y direction, time frame of 0.5 seconds, waits to get to position before proceeding
poppy.r_shoulder_x.goto_position(-60, 0.5, wait=True) # right shoulder movement in x direction
poppy.r_arm_z.goto_position(30, 0.5, wait=True) # right shoulder movement in z direction

# right arm movement
poppy.r_arm_z.goto_position(-30, 0.5, wait=True) 
poppy.r_arm_z.goto_position(30, 0.5, wait=True)  

# return to zero
poppy.r_shoulder_y.goto_position(0, 0.5, wait=True)
poppy.r_shoulder_x.goto_position(0, 0.5, wait=True)
poppy.r_arm_z.goto_position(0, 0.5, wait=True)

# left shoulder movement
poppy.l_shoulder_y.goto_position(-30, 0.5, wait=True)
poppy.l_shoulder_x.goto_position(60, 0.5, wait=True)
poppy.l_arm_z.goto_position(-30, 0.5, wait=True)

# left arm movement
poppy.l_arm_z.goto_position(-30, 0.5, wait=True) 
poppy.l_arm_z.goto_position(30, 0.5, wait=True)  

# return to zero
poppy.l_shoulder_y.goto_position(0, 0.5, wait=True)
poppy.l_shoulder_x.goto_position(0, 0.5, wait=True)
poppy.l_arm_z.goto_position(0, 0.5, wait=True)

#closes poppy communication
poppy.close()
```
And the movement should look something like this: 

![GIF demonstration could not load!](https://i.imgur.com/eHOg7uk.gif)

You now have a version of a Poppy Humanoid that you can control in quite a similar fashion to the real thing. Let us take this one step further.

## Controlling a specific Poppy Humanoid scene:
The problem with only using a default Poppy scene is that we cannot use a specific scene that we have modified unless we want to manually program everything into our script (which might be a legitimate alternative, just not one that I have explored). For example, if I want to attach a vision sensor to Poppy's forehead, I must create that in a scene, so loading a default every time will not help us.

## Step 1: Save a default Poppy scene as a specific '.ttt' file in a specific file path that you will keep handy for the next steps
![GIF demonstration could not load!](https://i.imgur.com/t2V503v.gif)






