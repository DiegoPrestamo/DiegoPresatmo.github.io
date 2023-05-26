---
layout: post
title: Blog post in progress - Controlling Robotic Gripper with Computer Vision
---
#### The Arduino Braccio is a customizable, Arduino-compatible robotic arm that replicates the functionality of a human arm with its shoulder, elbow, wrist, and a sophisticated gripper. The Braccio is a great entry level robotic arm, due to its small size, contextually low price, and capabilities. 
![GIF demonstration could not load!](https://s12.gifyu.com/images/Screen-Recording-2023-05-25-at-01.10.12-PM-1.gif)
Controlling my Arduino Braccio with my fingers

Computer vision (CV) is a field of artificial intelligence that enables computers to interpret and understand visual data from the real world. It uses digital images and videos as input and applies techniques to process, analyze, and understand them, enabling machines to perform tasks such as identifying objects, recognizing faces, navigating autonomous vehicles, or even diagnosing diseases. Recently, I have been obsessing over the overlap between the two and the seemingly limitless possibilities. We will be modifying a gesture volume control script to move the gripper.

## What is the point of this?
Why do I want to create a gripper that mimics human movement? First of all, its pretty cool; I have never charmed a snake but I sense the feeling would be similar. Second and most importantly, I have a fascination with the possibility of creating an intelligent robot, one that can learn from humans. This is distinct to most of our current robots, which are programmed to systematically to perform certain tasks given certain conditions and do not really know what they are doing or why. The intelligent robot I envision needs vision so this project gets me closer to that

## Materials: 
- [Arduino Braccio](https://store-usa.arduino.cc/products/tinkerkit-braccio-robot?selectedStore=us)
- Arduino Uno or any Braccio Shield compatible Arduino
- Computer with camera or webcam
## Downloads: We will mostly be modifying previously existing code
- If you do not have Python installed already, go [here](https://www.python.org/downloads/) and follow the instructions.
- Download the [Arduino IDE](https://www.arduino.cc/en/software)
- Make sure you download the Braccio library in Arduino

## Step 1: Hand Tracking Module
We will be modifying Murtaza Hassan's gesture volume control project for our needs. It is the perfect base to work off. This is the hand tracking module that makes the gesture volume control project possible. We will save it as **HandTrackingModule.py**
```bash
import cv2
import mediapipe as mp
import time


class handDetector():
    def __init__(self, mode=False, maxHands=2, model_complexity = 1, detectionCon=0.5, trackCon=0.5):
        self.mode = mode
        self.maxHands = maxHands
        self.model_complexity = model_complexity
        self.detectionCon = detectionCon
        self.trackCon = trackCon

        self.mpHands = mp.solutions.hands
        self.mpHands.Hands()
        self.hands = self.mpHands.Hands(self.mode, self.maxHands, self.model_complexity,
                                        self.detectionCon, self.trackCon)
        self.mpDraw = mp.solutions.drawing_utils

    def findHands(self, img, draw=True):
        imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        self.results = self.hands.process(imgRGB)
        # print(results.multi_hand_landmarks)

        if self.results.multi_hand_landmarks:
            for handLms in self.results.multi_hand_landmarks:
                if draw:
                    self.mpDraw.draw_landmarks(img, handLms, self.mpHands.HAND_CONNECTIONS)

        return img

    def findPosition(self, img, handNum=0, draw=True):
        lmList = []

        if self.results.multi_hand_landmarks:
            myHand = self.results.multi_hand_landmarks[handNum]

            for id, lm in enumerate(myHand.landmark):
                # print(id, lm)
                h, w, c = img.shape
                cx, cy = int(lm.x * w), int(lm.y * h)
                lmList.append([id, cx, cy])
                if draw:
                    cv2.circle(img, (cx, cy), 25, (255, 0, 0), cv2.FILLED)

        return lmList

def main():
    prevTime = 0
    currentTime = 0
    cap = cv2.VideoCapture(0)
    detector = handDetector()

    while True:
        success, img = cap.read()
        img = detector.findHands(img)
        lmList = detector.findPosition(img)
        if lmList:
            print(lmList[4])

        currentTime = time.time()
        fps = 1/(currentTime - prevTime)
        prevTime = currentTime

        cv2.putText(img, str(int(fps)), (10,70), cv2.FONT_HERSHEY_PLAIN, 3,
                    (255, 0, 255), 3)

        cv2.imshow("Image", img)
        cv2.waitKey(1)


if __name__ == '__main__':
    main()
```
## Step 2: Modify gesture volume control
This is the modified version which fits our needs. It will communicate to the arduino script in the next step.
```bash
import cv2
import time
import numpy as np
from HandTrackingModule import handDetector
import math
import serial

# arduino serial connection
arduino = serial.Serial('/dev/cu.usbmodem21101', 9600)  # replace 'COM_PORT' with the appropriate port
time.sleep(2) 

# cam parameters
wCam, hCam = 640, 480
cap = cv2.VideoCapture(1) # if your camera is not loading, try different numbers such as '0' or '2'
cap.set(3, wCam)
cap.set(4, hCam)
prevTime = 0
last_sent = time.time()  

detector = handDetector(detectionCon=0.7)
servo_angle = 0
while True:
    success, img = cap.read()
    img = detector.findHands(img)
    lmList = detector.findPosition(img, draw=False)
    if lmList:
        x1, y1 = lmList[4][1], lmList[4][2]  # thumb coordinates
        x2, y2 = lmList[8][1], lmList[8][2]  # pointer finger coordinates
        cx, cy = (x1 + x2) // 2, (y1 + y2) // 2  # center between thumb and pointer

        cv2.circle(img, (x1, y1), 15, (255, 0, 255), cv2.FILLED)
        cv2.circle(img, (x2, y2), 15, (255, 0, 255), cv2.FILLED)
        cv2.line(img, (x1, y1), (x2, y2), (255, 0, 255), 3)
        cv2.circle(img, (cx, cy), 15, (255, 0, 255), cv2.FILLED)

        length = math.hypot(x2 - x1, y2 - y1)

        # Hand range 20 - 280
        # Servo range 0 - 180
        servo_angle = np.interp(length, [20, 280], [73, 0])

        if time.time() - last_sent > 0.2: # 0.2 is the rate at which it is sending the aruino data, you can fine tune this number
            try:
                arduino.write((str(int(servo_angle)) + '\n').encode())  # sending angle to arduino
                print(f'Sent angle: {int(servo_angle)}')  
                last_sent = time.time() 
                time.sleep(0.1)  
            except serial.serialutil.SerialException as e:
                print(f"Serial communication error: {e}")

        if length < 50:
            cv2.circle(img, (cx, cy), 15, (0, 255, 0), cv2.FILLED)

    # displaying servo angle
    cv2.putText(img, f'Angle: {int(servo_angle)}', (40, 450), cv2.FONT_HERSHEY_COMPLEX,
                1, (255, 0, 0), 3)

    # displaying fps and video feed
    currTime = time.time()
    fps = 1 / (currTime - prevTime)
    prevTime = currTime
    cv2.putText(img, f'FPS: {int(fps)}', (40, 50), cv2.FONT_HERSHEY_COMPLEX,
                1, (0, 255, 0), 3)
    cv2.imshow("Img", img)

    cv2.waitKey(1)
```
## Step 3: The Arduino script
An Arduino is a user-friendly, open-source electronic platform that allows anyone to create interactive hardware projects. It's essentially a small computer board that can process inputs from sensors, make decisions based on code, and control various outputs like lights, motors (in our case), or displays. I modified one of the standard Braccio sketches (hence the weird comments which I left for context) to create the following Arduino sketch: 
```bash
/*
  simpleMovements.ino

 This  sketch simpleMovements shows how they move each servo motor of Braccio

 Created on 18 Nov 2015
 by Andrea Martino

 This example is in the public domain.
 */

#include <Braccio.h>
#include <Servo.h>

Servo base;
Servo shoulder;
Servo elbow;
Servo wrist_rot;
Servo wrist_ver;
Servo gripper;

void setup() {
  //Initialization functions and set up the initial position for Braccio
  //All the servo motors will be positioned in the "safety" position:
  //Base (M1):90 degrees
  //Shoulder (M2): 45 degrees
  //Elbow (M3): 180 degrees
  //Wrist vertical (M4): 180 degrees
  //Wrist rotation (M5): 90 degrees
  //gripper (M6): 10 degrees
  Braccio.begin();
  Serial.begin(9600); // Set up serial communication at 9600bps
}

void loop() {
  int angle = 73;  // Default angle for gripper
  if (Serial.available()) {  // If data is available to read
    angle = Serial.parseInt(); // Read it and store it in 'angle'
  }

  /*
   Step Delay: a milliseconds delay between the movement of each servo.  Allowed values from 10 to 30 msec.
   M1=base degrees. Allowed values from 0 to 180 degrees
   M2=shoulder degrees. Allowed values from 15 to 165 degrees
   M3=elbow degrees. Allowed values from 0 to 180 degrees
   M4=wrist vertical degrees. Allowed values from 0 to 180 degrees
   M5=wrist rotation degrees. Allowed values from 0 to 180 degrees
   M6=gripper degrees. Allowed values from 10 to 73 degrees. 10: the toungue is open, 73: the gripper is closed.
  */

                       //(step delay, M1, M2, M3, M4, M5, M6);
  Braccio.ServoMovement(20,           0,  15, 180, 170, 0,  angle);  


}
```
## Step 4: Flash the Arduino sketch onto the arduino that you have attached to the Braccio shield
Hit **Upload** on the Aruino sketch to flash it onto the Uno. Once you do, it will move a bit to settle into position and it will run indefinitely waiting for serial data to come in. The serial data will be coming in from the Python script once we run it.

## Step 5: Run the Python script
Once the Arduino is flashed, we can proceed to the Python script. We will now run it and control the movement of the gripper with our thumb and index finger.





