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

vrep_port = 19990 # your port will likely be 19997 since I specifically changed mine which I menioned earlier


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

## Step 2: Get creative
Now you can add various peripherals in this scene which you will be able to simulate. Much cooler than a simple default Poppy. Let us add a vision sensor:
- Go to the Menu bar and click **Add**
- Under **Vision sensor** select **Perspective type**. The **Perspective type** vision sensor has a triangular field of view similar to our eyes, the **Orthographic type** projects in a rectangular fashion.
- Go to **Object/item position/orientation**. Under **Position/Translation** change the y-position to **-0.035 m**, the z-position to **0.8 m**, and leave the x-position alone. Then go to the **Orientation/Rotations** and change alpha to **90 deg**
![GIF demonstration could not load!](https://im3.ezgif.com/tmp/ezgif-3-345c2e8240.gif)

#### Step 2.5: Connect the Vision Sensor to Poppy's head visual
If we do not do this step, it is as if the vision sensor and Poppy's head just happen to be at the same place at once. By dragging the vision sensor and dropping it to Poppy's **head_visiual** we are effectively connecting the vision sensor to the head like we would expect
![GIF demonstration could not load!](https://i.imgur.com/FnvzLlL.gif)
* Not shown: Change resolution to 1080 / 1080 or anything that is not the treacherous default 32 / 32 
* Make sure to save your scene anytime you make changes or they will not show up next time you run the scene

## Step 3: Add a renderable object in the field of view:
Let us put something in front of the camera just to make sure we have everything working. We will add a cuboid and put it right in front of Poppy's face.
- Go to the Menu bar and click **Add**
- Select **Cuboid** under **Primitive shape**
- Move the cuboid to be right in Poppy's face
- Change the y-position to **- 0.75** this should be right outside of Poppy's vision sensor volume
![image](https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/fe80be1b-e775-4f01-9cc8-87425f395b4a)

#### Step 3.5 Make sure the object is renderable and still! 
Do not expect something to render on the vision sensor's view (next step) if you did not make it renderable. 
Under **Scene Object Properties** hit the **renderable** box:
<div align="center">
  <p>
    <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/1ceba361-8a78-45ad-a9a1-3647502161e1" width="800">
  </p>
  The **Scene Object Properties** is the magnifying glass which I drew the arrow to at the beginning
</div>

While you are at it, go to **Show dynamic properties dialog** and uncheck **Body is dynamic**. That way the cube will float instead of falling with gravity.
![image](https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/9bc5fa5e-4d50-4a38-b55c-204b1ff8f7bf)
  <p>
    <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/9bc5fa5e-4d50-4a38-b55c-204b1ff8f7bf" width="800">
  </p>
  
## Step 4: Insert a floating view and associate it with the vision sensor
It is nice to be able to see what the vision sensor is seeing. Let us add a floating view to do just that.
 - Right click anywhere in the scene (another way to access the menu)
 - Hover over to **Add** --> **Floating View**
 - In the **Hierarchy** select your vision sensor
 - Right click the floating view --> **view** --> **associate view with selected vision sensor**
 - Run your scene and you should see what the vision sensor is seeing
![GIF demonstration could not load!](https://i.imgur.com/N5JSYmM.gif)

## Step 5: Copy and paste API files in our path
We will need to fetch three files from the V-REP file location and place them in the same path as our current scene
- Sim.py - Found in **_pycache_** file
- simConst.py - Found in **_pycache_** file
- remoteApi.dll - Found in **programming** --> **remoteApiBindings** --> **lib** --> **lib** --> choose 64 bit or 32 bit --> remoteApi.dll
<div align="center">
  <p>
    <img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/assets/103367642/c0dcdbae-4348-4bd2-9485-c19fd9980e18" width="800">
  </p>
  This is my path for the _poppy_humanoid_ file we are working with
</div>

## Step 6: Control this specific scene using a Python script
We have now created a specific scene which we would like to experiment with. We do not want to simply run a default scene. Let us write a Python script to communicate directly with the scene we have just created. The important lesson from this script is the manner in which we are communicating with V-REP.
```bash
from pypot.creatures import PoppyHumanoid
from pypot.vrep.controller import VrepController
import time

# the path to where you have your Poppy scene saved... this is mine and yours should be different
scene_path = r"C:\Users\prest\OneDrive\Documents\Poppy Research\poppy_humanoid.ttt"

# the new way we will connect to V-REP
vrep_host = '127.0.0.1'  # the IP address
vrep_port = 19990  # same port from earlier... again yours will probably be 1997
vrep_scene = scene_path

poppy = PoppyHumanoid(simulator='vrep', scene=vrep_scene, host=vrep_host, port=vrep_port)

# head movement to pan around scene
poppy.head_z.goal_position = 60
time.sleep(1)
poppy.head_z.goal_position = -60
time.sleep(1)
poppy.head_z.goal_position = 0
time.sleep(1)

poppy.close()
```









