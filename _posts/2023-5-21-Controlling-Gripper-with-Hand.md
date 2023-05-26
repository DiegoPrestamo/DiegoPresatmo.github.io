---
layout: post
title: Blog post in progress - Arduino Braccio Computer + Vision Project
---
#### The Arduino Braccio is a customizable, Arduino-compatible robotic arm that replicates the functionality of a human arm with its shoulder, elbow, wrist, and a sophisticated gripper. The Braccio is a great entry level robotic arm, due to its small size, contextually low price, and capabilities. Computer vision (CV) is a field of artificial intelligence that enables computers to interpret and understand visual data from the real world. It uses digital images and videos as input and applies techniques to process, analyze, and understand them, enabling machines to perform tasks such as identifying objects, recognizing faces, navigating autonomous vehicles, or even diagnosing diseases. Recently, I have been obsessing over the overlap between the two and the seemingly limitless possibilities. We will be modifying a gesture volume control script to move the gripper.
![GIF demonstration could not load!](https://s12.gifyu.com/images/Screen-Recording-2023-05-25-at-01.10.12-PM-1.gif)
Controlling my Arduino Braccio with my fingers

## What is the point of this?
Why do I want to create a gripper that mimics human movement? First of all, its pretty cool; I have never charmed a snake but I sense the feeling would be similar. Second and most importantly, I have a fascination with the possibility of creating an intelligent robot, one that can learn from humans. This is distinct to most of our current robots, which are programmed to systematically to perform certain tasks given certain conditions and do not really know what they are doing or why. The intelligent robot I envision needs vision so this project gets me closer to that

## Materials: 
- [Arduino Braccio](https://store-usa.arduino.cc/products/tinkerkit-braccio-robot?selectedStore=us)
- Arduino Uno or any Braccio Shield compatible Arduino
- Computer with camera or webcam
## Downloads: We will mostly be modifying previously existing code
- If you do not have Python installed already, go [here](https://www.python.org/downloads/) and follow the instructions.
- Download the [Arduino IDE](https://www.arduino.cc/en/software)


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
This is the modified version which fits our needs: 
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




