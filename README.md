
<p align="center"><img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/logo.png' width="60%"></p>

A native set of nodes for Hikvision Cameras, Alarms, Radars, NVR etc.

[![NPM version][npm-version-image]][npm-url]
[![NPM downloads per month][npm-downloads-month-image]][npm-url]
[![NPM downloads total][npm-downloads-total-image]][npm-url]
[![MIT License][license-image]][license-url]
[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)
[![Donate via PayPal](https://img.shields.io/badge/Donate-PayPal-blue.svg?style=flat-square)](https://www.paypal.me/techtoday) 
[![Facebook][facebook-image]][facebook-url]


## DESCRIPTION
This is a set of nodes to handle ISAPI Hikvision messages. It works exclusively with ***HIKVISION*** devices.<br/>
Works with cameras, NVR and also with specialized devices, like Radar (for example DS-PR1-60, DS-PR1-100 and DS-PR1-120).<br/>
Digest authentication: it should work with all devices.<br/>
All nodes are capable of auto reconnect if the connection is lost and are able to actively monitor the connection.<br/>
Be sure to have installed **Node.js v12.3.0** or newer (issue a node -v command in a console, to check it out).<br/>

**Note:** for NVR/DVR, pleas remember to select "Notify Alarm Center" in the event window, otherwise the NVR won't emit any alarm event.<br/>
<img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/NotifyCenter.png' width="20%">

## CHANGELOG
* See <a href="https://github.com/Supergiovane/node-red-contrib-hikvision-ultimate/blob/master/CHANGELOG.md">here the changelog</a>

<br/>

---

## ALARM NODE
The alarm node connects to ***NVR, Camera, Alarm system, Radars, etc..*** and outputs true/false whenever an alarm occurs. <br/>
The node can be configured as **Camera/NVR** (with standard and smart events) or as **Security System** and **Radar** (with specific CID events, designed for these type of security devices)<br/>
Some alarm events supports the alarm start and end events, while others, only the alarm start event. CID alarms ALWAYS support start/end events.<br/>
You can filter for CHANNEL, EVENT and ZONE. For NVR, the ***Channel*** represents the CAMERA number, while for Cameras, represents the sensor number (by default 1, if the camera has only one image sensor). The ***zone*** represents the alarm zone for RADARS AND SECURITY SYSTEM, otherwise the region (for example the intrusion alert region number).<br/>
The RADAR and SECURITY SYSTEM device types, can filter improper/false alams.<br/>

<img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/GenericAlarm.png' width="80%">

You can choose from many different alarms, including: <br/>
- Video Motion Alarm (When motion is detected)
- Local alarm input (it's the device's IO pigtail connector)
- Line crossing (when someone crosses a line)
- CID alarms
- Many more.....



**Flow Messages**

The node outputs a payload on **PIN 1**, true if alarm starts, false when alarm ended. Some alarm type doesn't support the alarm end event, so the node sends only a true payload when alarm occurs.</br>
The node outputs a payload on **PIN 2**, representing a connection error. ***TRUE*** if error, otherwise ***FALSE***</br>
This below is an example of msg output:</br>


**Output PIN 1**
```javascript
// Example of an event from NVR/Camera
msg.payload = {
  "payload": true,
  "topic": "",
  "channelid": "13", // This is the camera number for NVR, or the channel ID for cameras
  "zone": 0, // Zone or Region, see above, the explained difference
  "description": "Motion alarm",
  "_msgid": "386a613.89f259e"
}
```

```javascript
// Example of an event from Security System or Radar
msg.payload = {
{
    "zone": 1, // This is the zone number that fired the alarm
    "payload": true, // true if alarm, otherwise false if alarm ended.
    "alarm": {
        "ipAddress": "192.168.1.25",
        "ipv6Address": "",
        "portNo": 80,
        "protocol": "HTTP",
        "macAddress": "BananaRama",
        "channelID": 1,
        "dateTime": "2012-01-13T03:58:19+01:00",
        "activePostCount": 1,
        "eventType": "cidEvent",
        "eventState": "active",
        "eventDescription": "CID event",
        "CIDEvent": {
            "code": 3103,
            "standardCIDcode": 3130,
            "type": "zoneAlarm",
            "trigger": "2012-01-13T03:58:19+01:00",
            "upload": "2012-01-13T03:58:19+01:00",
            "CameraList": [],
            "NVRList": [
                {
                    "id": 1,
                    "ip": "192.168.1.32",
                    "port": 8000,
                    "channel": 1
                }
            ],
            "zone": 1
        }
    }
    "_msgid": "b07e50f6.86a72"
}
```

**Output PIN 2 (connection error)**
```javascript
msg = {
    "topic": "",
    "errorDescription": "", // This will contain the error rescription, in case of errors.
    "payload": false, // Or TRUE if error
    "_msgid": "dd5b3622.884a78"
}
```
<br/>
<br/>

---

## ANPR (License Plate) NODE
This node works with Hikvision ANPR cameras.<br/>

<img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/ANPR.png' width="80%">

**Flow Messages**

The payload contains the license plate number and the property "plate" contains other useful informations.</br>

**Output PIN 1**
```javascript
msg.payload = {
    "topic":"",
    "payload":"AB123CD", // This is the license plate
    "plate":{
        "captureTime":"20201130T114200+0100",
        "plateNumber":"AB123CD",
        "picName":"202011301142008600", // This is the picture's name of the license plate.
        "country":"ITA",
        "laneNo":"1",
        "direction":"forward",
        "matchingResult":"otherlist"
        }
    }
```

**Output PIN 2 (connection error)**
```javascript
msg = {
    "topic": "",
    "errorDescription": "", // This will contain the error rescription, in case of errors.
    "payload": false, // Or TRUE if error
    "_msgid": "dd5b3622.884a78"
}
```
<br/>
<br/>

---

## PTZ NODE
Just select the preset in the configuration window and recall it by passing ***true*** as payload.<br/>

<img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/PTZ.png' width="80%">

**Flow Messages**

The node outputs ***true*** on PIN 1 if the command is executed, otherwise an error on PIN 2.</br>

**Input**
```javascript
msg.payload = true; // Recalls the preset
```

**Output PIN 1**
```javascript
msg.payload = {
{
    "payload": true, // true after the camera has reached the PTZ preset position
    "_msgid": "b07e50f6.86a72"
}
```

**Output PIN 2 (connection error)**
```javascript
msg = {
    "topic": "",
    "errorDescription": "", // This will contain the error rescription, in case of errors.
    "payload": false, // Or TRUE if error
    "_msgid": "dd5b3622.884a78"
}
```
<br/>
<br/>

---


## PICTURE NODE
This node gets a picture from camera, ready to be shown in the dashboard UI.<br/>
The image can be rotated and resized as well.<br/>
Pass **true** as payload to obtain the image.<br/>
You can, for example, link the ***Alarm node*** to the ***Picture node*** to get an image whenever an alarm occurs.<br/>
**CAUTION**: image handling is a very CPU/GPU consuming job. Use only if you have enough computational resources.<br/>

<img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/picture.png' width="80%">

The ***Template*** node in this example, contains this code:
```javascript
<img src="{{payload}}"/>
```
The ***Dashboard*** node in this example, is a **UI_TEMPLATE** and contains this code:
```javascript
<div ng-bind-html="msg.payload"></div>
```

<img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/pictureproperties.png' width="40%">

**Flow Messages**

The node outputs the image in base64 string format, ready for the UI Dashboard, on PIN 1, otherwise an error on PIN 2.</br>

**Input**
```javascript
// To get the image
msg.payload = true; 
```

```javascript
// Dinamically set the overlay text
node.textoverlay = "Hello new overlay";
msg.payload = true; 
```


**Output PIN 1**
```javascript
msg.payload = {
{
    "payload": image in base64 format, // Ready for the Dashboard UI
    "_msgid": "b07e50f6.86a72"
}
```

**Output PIN 2 (connection error)**
```javascript
msg = {
    "topic": "",
    "errorDescription": "", // This will contain the error rescription, in case of errors.
    "payload": false, // Or TRUE if error
    "_msgid": "dd5b3622.884a78"
}
```
<br/>
<br/>

---

## TEXT OVERLAY NODE
You can set the camera's text overlay.<br/>
There are 4 rows avaiable, to be set from the configuration window or dinamically via msg input from flow.<br/>

<img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/Text.png' width="80%">

**Copy this code and paste it into your flow**

<details><summary>View code</summary>

> Adjust the nodes according to your setup

<code>
[{"id":"7e79800b.afb8a8","type":"hikvisionUltimateText","z":"3f22f0c6.ff1328","name":"Overlay Text","server":"ff0cbde2.8c00b8","row1":"UNO","row1XY":"40,100","row2":"DUE","row2XY":"","row3":"TRE","row3XY":"","row4":"QUATTRO","row5XY":"","x":510,"y":120,"wires":[]},{"id":"3aa8a40f.9a0964","type":"inject","z":"3f22f0c6.ff1328","name":"Go","props":[{"p":"payload"},{"p":"topic","vt":"str"}],"repeat":"","crontab":"","once":false,"onceDelay":0.1,"topic":"","payload":"","payloadType":"date","x":170,"y":120,"wires":[["7e79800b.afb8a8"]]},{"id":"2b4d9297.75ee46","type":"comment","z":"3f22f0c6.ff1328","name":"Set the overlay text","info":"","x":190,"y":80,"wires":[]},{"id":"22dc37f9.b3b86","type":"function","z":"3f22f0c6.ff1328","name":"MSG Override","func":"// Override one or more rows\n// You can use from row1 to row4 to set the text\n// and from row1XY to row4XY to set the position in the format x,y (for example: 100,200)\n\n// Row 1\nmsg.row1=\"Temperature: \" + msg.payload;\nmsg.row1XY=\"100,200\";\n\n// Row 2 (here we leave the position previosly set via the camera menu)\nmsg.row2=\"Sun\";\n\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","x":320,"y":180,"wires":[["7e79800b.afb8a8"]]},{"id":"a91e43a0.ccbb2","type":"inject","z":"3f22f0c6.ff1328","name":"Go","props":[{"p":"payload"},{"p":"topic","vt":"str"}],"repeat":"","crontab":"","once":false,"onceDelay":0.1,"topic":"","payload":"22°c","payloadType":"str","x":170,"y":180,"wires":[["22dc37f9.b3b86"]]},{"id":"ff0cbde2.8c00b8","type":"Hikvision-config","host":"192.168.1.20","port":"80","name":"Viessman","authentication":"digest","protocol":"http","heartbeattimerdisconnectionlimit":"1","deviceinfo":"{\"DeviceInfo\":{\"$\":{\"version\":\"2.0\",\"xmlns\":\"http://www.hikvision.com/ver20/XMLSchema\"},\"deviceName\":\"Viessman\",\"deviceID\":\"b3464000-5074-11b4-82cc-bcbac23d3ac8\",\"deviceDescription\":\"IPCamera\",\"deviceLocation\":\"hangzhou\",\"systemContact\":\"Hikvision.China\",\"model\":\"DS-2CD2085FWD-I\",\"serialNumber\":\"DS-2CD2085FWD-I20190716AAWRD39667116\",\"macAddress\":\"Bananassa\",\"firmwareVersion\":\"V5.6.3\",\"firmwareReleasedDate\":\"build 190923\",\"encoderVersion\":\"V7.3\",\"encoderReleasedDate\":\"build 190910\",\"bootVersion\":\"V1.3.4\",\"bootReleasedDate\":\"100316\",\"hardwareVersion\":\"0x0\",\"deviceType\":\"IPCamera\",\"telecontrolID\":\"88\",\"supportBeep\":\"false\",\"supportVideoLoss\":\"false\",\"firmwareVersionInfo\":\"B-R-G1-0\"}}"}]
</code>
</details>
<br/>
<br/>

**Flow Messages**

The node accepts only an input. Pass anything you like to set the text overlay.</br>
You can override the texts by passing some msg inputs. See the sample below</br>

**Input**
```javascript
// Simply overlay the text set in the config window
msg.payload = true;
return msg;
```

```javascript
// Override one or more rows
// You can use from row1 to row4 to set the text
// and from row1XY to row4XY to set the position in the format x,y (for example: 100,200)
// Row 1
msg.row1="Temperature: " + msg.payload; 
msg.row1XY="100,200";
// Row 2 (here we leave the position previosly set via the camera menu)
msg.row2="Sun"; 
return msg;
```

```javascript
// Delete all 4 rows
msg.row1 = "";
msg.row2 = "";
msg.row3 = "";
msg.row4 = "";
return msg;
```


<br/>
<br/>

---

## RAW ALARM NODE
The RAW alarm node reacts to every message sent. You can use this node when the other nodes doesn't fit your needs. It connects to ***NVR, Camera, Alarm system, Radars etc...*** and outputs the alarm received. <br/>

<img src='https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/RawAlarm.png' width="80%">

**Flow Messages**

The node outputs a payload on **PIN 1** that can vary, depending on the alarm type sent by the connected device.</br>
The node outputs a payload on **PIN 2**, representing a connection error. ***TRUE*** if error, otherwise ***FALSE***</br>
This below is an example of msg output (in this case, a movement detected from a radar)</br>


**Output PIN 1**
```javascript
msg.payload = {
    "topic": "",
    "payload": {
        "ipAddress": "192.168.1.25",
        "ipv6Address": "",
        "portNo": 80,
        "protocol": "HTTP",
        "macAddress": "banana",
        "channelID": 1,
        "dateTime": "2012-01-13T04:32:47+01:00",
        "activePostCount": 1,
        "eventType": "MultiRadarTargetEvent",
        "eventState": "active",
        "eventDescription": "MultiRadar Target Event",
        "MultiRadarTargetEventList": [
            {
                "targetID": 25,
                "isTargetDisappear": false,
                "targetType": "people",
                "Coordinate": {
                    "angle": 101.49,
                    "distance": 24.59
                },
                "speed": -0.2,
                "signalStrength": "strong",
                "TrackedInfoList": [],
                "trackedByIPC": false
            }
        ]
    },
    "_msgid": "dba1850a.2dc5e8"
}
```

**Output PIN 2 (connection error)**
```javascript
msg = {
    "topic": "",
    "errorDescription": "", // This will contain the error rescription, in case of errors.
    "payload": false, // Or TRUE if error
    "_msgid": "dd5b3622.884a78"
}
```

# HIKVISION DEVICES TESTED WITH THE NODE
This is a list of Hikvision models the users own and decided to share because it checked working with Hikvision-Ultimate. <br/>
This is only a partial model list. Hikvision-Ultimate should work with basically ALL Hikvision devices.<br/>
[Click HERE to view the list of tested devices](http://www.supergiovane.it:8080/?read=true&md=noCD&fw=no) 

<br/>
<br/>

![Logo](https://raw.githubusercontent.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/img/madeinitaly.png)

[license-image]: https://img.shields.io/badge/license-MIT-blue.svg
[license-url]: https://github.com/Supergiovane/node-red-contrib-hikvision-ultimate/master/LICENSE
[npm-url]: https://npmjs.org/package/node-red-contrib-hikvision-ultimate
[npm-version-image]: https://img.shields.io/npm/v/node-red-contrib-hikvision-ultimate.svg
[npm-downloads-month-image]: https://img.shields.io/npm/dm/node-red-contrib-hikvision-ultimate.svg
[npm-downloads-total-image]: https://img.shields.io/npm/dt/node-red-contrib-hikvision-ultimate.svg
[facebook-image]: https://img.shields.io/badge/Visit%20me-Facebook-blue
[facebook-url]: https://www.facebook.com/supergiovaneDev