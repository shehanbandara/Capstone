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

# 11/16/2023
## Phone Progress
* Able to get the phone's GPS coordinates and display them to the user.

![](../../assets/images/phoneCoordinates.gif)

**Code Snippet:**
```swift{% raw %}
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    manager = CLLocationManager()
    manager?.delegate = self
    manager?.desiredAccuracy = kCLLocationAccuracyBest
    manager?.requestWhenInUseAuthorization()
    manager?.startUpdatingLocation()
}
    
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let first = locations.first else {
        return
    }
        
    label.text = "\(round(first.coordinate.latitude*1000000000)/1000000000), \(round(first.coordinate.longitude*1000000000)/1000000000)"
}
{% endraw %}```

# 11/18/2023
## Caddy (Raspberry Pi) Progress:
* Able to get the Raspberry Pi's GPS coordinates with the BN-880 GPS.

**Code Snippet:**
```python{% raw %}
if str.find('GGA') > 0:
    msg = pynmea2.parse(str)
    print("Timestamp: %s -- Lat: %s %s -- Lon: %s %s -- Altitude: %s %s -- Satellites: %s" % (msg.timestamp,msg.latitude,msg.lat_dir,msg.longitude,msg.lon_dir,msg.altitude,msg.altitude_units,msg.num_sats))
{% endraw %}```

* Able to get the Raspberry Pi's cardinal direction with the GY-271 Compass.

**Code Snippet:**
```python{% raw %}
sensor = GY271.compass(address=0x0d)
angle = sensor.get_bearing()
print('Heading Angle = {}°'.format(angle))
{% endraw %}```

# 11/20/2023
## Phone & Caddy (Raspberry Pi) Progress:
* Able to send coordinates between the phone and Raspberry Pi on a socket.

![](../../assets/images/phoneRPISocketCoordinates.gif)

**Phone Code Snippet:**
```swift{% raw %}
func sendMessage(message: String) {
    if connected == false {
        return
    }
    guard let data = message.data(using: .utf8) else {
        return
    }
    _ = data.withUnsafeBytes { outputStream?.write($0, maxLength: data.count)}
}

func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let first = locations.first else {
        return
    }
        
    label.text = "\(round(first.coordinate.latitude*1000000000)/1000000000), \(round(first.coordinate.longitude*1000000000)/1000000000)"
    if let labelText = label?.text {
        let encodedMessage = "COORDINATE:\(labelText)"
        sendMessage(message: encodedMessage)
    }
}
{% endraw %}```

**Raspberry Pi Code Snippet:**
```python{% raw %}
message = connectionSocket.recv(1024).decode()
code, content = message.split(":", 1)
if code == ACTION:
    print(f"Incoming action: {content}")
else:
    phone_location = content.split(", ")
    phone_location_lat = float(phone_location[0])
    phone_location_lon = float(phone_location[1])
    print(f"Coordinates: {phone_location_lat}, {phone_location_lon}")
{% endraw %}```

# 11/23/2023
## Caddy (Raspberry Pi) Motor Control:
* Able to successfully control the hoverboard's motors with a Raspberry Pi. The wheels can rotate in both directions and the brake can be successfully applied.
* [Corresponding Electrical Work]({% link docs/projectLogs/electrical.md %}#motor-work-progress-2).

**Code Snippet:**
```python{% raw %}
def init():
    GPIO.setmode(GPIO.BOARD)
    GPIO.setwarnings(False)
    GPIO.setup(29, GPIO.OUT)
    GPIO.setup(31, GPIO.OUT)
    GPIO.setup(32, GPIO.OUT)

def forward(tf): 
    GPIO.output(29, False)
    GPIO.output(31, False)
    motorl = GPIO.PWM(32, 50)
    motorl.start(0)
    motorl.ChangeDutyCycle(20)
    time.sleep(tf)

def reverse(tf): 
    GPIO.output(29, True)
    GPIO.output(31, False)
    motorl = GPIO.PWM(32, 50)
    motorl.start(0)
    motorl.ChangeDutyCycle(20)
    time.sleep(tf)

def brake(tf): 
    GPIO.output(31, True)
    time.sleep(tf)
{% endraw %}```