---
layout: default
title: Software
parent: Project Logs
nav_order: 4
---
# Software
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---

# 11/07/2023
## Software Design Progress
**In general:**
* The server and client network will be connected via Wi-Fi to support extended range​
* The user's phone transmits actions and coordinates​ to the caddy
* The caddy follows the user's requested actions and coordinates in real-time

**Phone:**
* Sends the requested action to the caddy (START/STOP)
* Sends the coordinates of the phone, using the phone’s built in GPS​, to the caddy in real-time
* Receives the distance to caddy in real-time from the caddy

**Caddy (Raspberry Pi):**
* Receives action from phone​
    * If STOP​
        * Turn off all sensors
        * Use minimal motor power to remain stationary​
    * If START​
        * Continuously receive coordinates from the phone​
        * Calculate current coordinates and cardinal direction​
        * Transmit current distance from the phone to the phone​
        * Use appropriate motor power to follow phone

**Obstacle Detection:**
* LiDAR to determine the distance to nearest surface/hazard
* Camera to use YOLOv6 algorithm for image recognition (mAP > 50%)​
    * Differentiate between hazards and areas of interest
    * Recognize potential hazards in peripherals (water, ditches, etc.)

**Control:**
* Controller for converging on correct angle to follow phone
* Controller for speed
    * Track user faster at further distances
    * Stop 5m away from the user

# 11/13/2023
## Software Block Diagrams
### Obstacle Avoidance
![](../../assets/images/obstacleAvoidance.png)
### Obstacle Detection
![](../../assets/images/obstacleDetection.png)
### Phone
![](../../assets/images/phone.png)
### Speed
![](../../assets/images/speed.png)
### Start
![](../../assets/images/start.png)
### Stop
![](../../assets/images/stop.png)
### Turn Direction
![](../../assets/images/turnDirection.png)