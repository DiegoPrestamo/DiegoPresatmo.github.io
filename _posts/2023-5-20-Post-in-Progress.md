---
layout: post
title: Making a Differential Drive Robot Simulator
---

#### A differential drive robot (DDR) is a type of mobile robot with two, seperately motorized, wheels. We will discuss the motion model for a DDR and make a simple simulator that will help us interact with the motion model.

<div align="center">
  <p>
    <img src="https://p54.f0.n0.cdn.getcloudapp.com/items/X6u7WlnP/946209e9-9544-46e5-8480-f5821e4e5af8.gif?source=viewer&v=33f4c17f3843a5cd577129dd440eb80c" width="800">
  </p>
     DDR simulator we will create (GIF format worsens frame rate, real sim is smoother)
</div>

Motion models are mathematical models that describe the behavior of robots. There are two main types: kinematic models and dynamic models. Today we will be focusing on the former.
## Kinematics and Inverse Kinematics:
Kinematic models can be approached in two ways: forward kinematics (or simply kinematics) and inverse kinematics. Forward kinematics takes input values and predicts the path of the robot, whereas inverse kinematics defines a desired path and attempts to find the inputs necessary to achieve this path.

In the context of DDR, a forward kinematic model would use user inputs of linear and angular velocity to model the behavior of the robot. On the other hand, an inverse kinematic model would start with a laid out path that we desire our robot to follow, and would then calculate the necessary linear and angular velocity inputs to achieve such a path.
