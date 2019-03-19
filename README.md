# MiNiFi MotionSensor Alert and Response Via SMS

## Short Description:

Here is a small Article on how to use MiNiFi & NiFi on a Raspberry Pi to detect Motion and send alerts to your phone via SMS, and on your reply to SMS, it will trigger panic ALARM sound on the Raspberry Pi. [Basically to shoo away an intruder]

## Introduction

- Here is a small demo how to flex MiNiFi+NiFi on a Raspberry Pi to detect Motion and send alerts to your phone via SMS, also on your  SMS reply it will trigger sound ALARM. You can do it from where ever you have cell phone reception.

- Here you can view the screen recording session that Demonstrates how it works!

[![NiFi + Mac Dictation Demo](https://github.com/jobinthompu/MiNiFi-MotionSensor-Alert-and-Response-Via-SMS/blob/master/Resources/images/You-Tube.jpg)](https://youtu.be/eZGntQDVghE "Voice Command with Mac Dictations and NiFi - Click to Watch!")


## Prerequisite

1)	Raspberry Pi 3, a PIR motion sensor and a speaker connected to it. Details on how to connect PIR Motion Sensor to Raspberry Pi can be found in url under references.
2)	Assuming you already have latest version of HDF/NiFi and Minifi downloaded on your Mac and Pi. Else
	
Get Latest version of MiNiFi :
	
```
# wget http://apache.claz.org/nifi/minifi/0.1.0/minifi-0.1.0-bin.tar.gz
```
Get Latest version of MiNiFi ToolKit:
	
```
# wget http://apache.claz.org/nifi/minifi/0.1.0/minifi-toolkit-0.1.0-bin.tar.gz
```
Get Latest version of NiFi:
	
```
# wget http://apache.claz.org/nifi/1.2.0/nifi-1.2.0-bin.tar.gz
```
3)	Untar the files and start NiFi on your local machine and MiniFi on your Raspberry Pi


## Steps:

## Flow on MiNiFi

Download the flow [Pi-MiNiFi-FLow.xml](https://github.com/jobinthompu/MiNiFi-MotionSensor-Alert-and-Response-Via-SMS/blob/master/Resources/flow/Pi-MiNiFi-FLow.xml) and convert it to YAML format which MiNiFi uses (before you deploy make sure you have your local NiFi URL for RPG rather than what I have in there)

```
# /root/minifi-toolkit-0.1.0/bin/config.sh transform Pi-MiNiFi-FLow.xml minifi-0.1.0/conf/config.yml
```

Flow Looks like below:

![alt tag](https://github.com/jobinthompu/MiNiFi-MotionSensor-Alert-and-Response-Via-SMS/blob/master/Resources/images/MiNiFi-Flow.jpg)

### Flow Explained: 


a)  Poll for Sensor output and Sent it to NiFi
	
**GenerateFlowFile** processor triggers every 5seconds to execute a python script **pirtest.py** as below using an **ExecuteStreamCommand** proceesor, result is sent to NiFi running on my local machine via Remote process group.

**pirtest.py** script looks like below:

```
import RPi.GPIO as GPIO
import time
import os
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BOARD)
GPIO.setup(11, GPIO.IN)         #Read output from PIR motion sensor

i=GPIO.input(11)
if i==1:               			#When output from motion sensor is HIGH
      print "Intruder detected",i
```

b) Play Panic Alarm on SMS trigger from NiFi

**listenHTTP** processor hosts and listens for any incoming flowfile, when arrived next processor **ExecuteStreamCommand** executes a python script **alarm.py** as given below which trigger a panic alarm sound to be played.  I used **mpg123** to play the alarm sound, you can install it on your raspberry pi using below command:

```
# sudo apt-get install mpg123
```

**alarm.py** script looks like below:

```
import os
os.system('mpg123 /root/alarm.mp3')
```

## Flow on NiFi

Download the flow [MiNiFi-MotionSensor-SMS-Alert+Response.xml](https://github.com/jobinthompu/MiNiFi-MotionSensor-Alert-and-Response-Via-SMS/blob/master/Resources/flow/MiNiFi-MotionSensor-SMS-Alert%2BResponse.xml) and deploy it after updating your custom hostnames and other details details. It looks like below:

![alt tag](https://github.com/jobinthompu/MiNiFi-MotionSensor-Alert-and-Response-Via-SMS/blob/master/Resources/images/NiFi-Flow.jpg)


### Flow Explained: 

a)	Receiving Sensor Alert from MiNiFi and send SMS
	
An InputPort receives MotionSensor output from MiNiFi, **RouteOnAttribute** processor verifies the output and send it to a **ControlRate** processor only if motion is detected. Control rate processor ensures your phone is not flooded with alerts. **putEmail** processor is configured to send SMS to my phone.

b)	Check for ALARM request and send signal to MiNiFi

**ConsumeIMAP** processor checks for new ALARM request in a specified folder in my mailbox, when received, triggers a flowfile. **RouteOnContent** processor verifies the new mail and route it based on sender and content, feeding it to a **PostHTTP** processor connecting to **listenHTTP** on MiNiFi triggering ALARM.


Now you its time to try out!!

## References:

[Raspberry Pi and PIR Motion Sensor](https://www.raspberrypi.org/learning/parent-detector/worksheet/)

[MiNiFi](https://nifi.apache.org/minifi/system-admin-guide.html)

Thanks,

Jobin George
