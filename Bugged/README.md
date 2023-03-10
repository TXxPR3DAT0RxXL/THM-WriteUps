# [TryHackMe](https://tryhackme.com/dashboard) -> [Bugged](https://tryhackme.com/room/bugged) WriteUp 🎯
## Task 1 - Analyze The Network 📊
John was working on his smart home appliances when he noticed weird traffic going across the network. Can you help him figure out what 
these weird network communications are?

What is the flag?

Let's Deploy the machine!  
## Reconnaissance 🕵
First, i Used `Nmap` to scan, As always...  
```shell
nmap -sV -p- 10.10.27.13
```
![image](https://user-images.githubusercontent.com/127505784/224350675-50e0a069-5fd7-41cd-ac47-0853b05ef8fb.png)

looking at the output of the scan we can see that port 1883 is open and running mosquitto service, But what is it mosquitto? 🤔  
>Mosquitto is an open-source message broker that implements the MQTT (Message Queuing Telemetry Transport) protocol. The MQTT protocol is a lightweight messaging protocol that is designed for use in IoT (Internet of Things) applications, where devices need to communicate with each other over a low-bandwidth, high-latency network.  
The Mosquitto service provides a way for devices to publish and subscribe to messages on a topic. It acts as a central hub for messages, allowing devices to communicate with each other without needing to know each other's IP addresses or network topology. Mosquitto is commonly used in applications such as home automation, industrial automation, and machine-to-machine (M2M) communication.

![image](https://user-images.githubusercontent.com/127505784/224354061-9588d7d3-6850-4f91-9841-96c6690d689a.png)

so we have simple case of IoT communication, To be Honest this is my first time coming acroos MQTT protocol.  
So after some research i found a way to interact with this protocol by using **"Mosquitto command line client"**    

Ok for some reason the machine stopped working for me, so i deployed the machine again.  
Now, Before going Straight to interact i want to try to run Nmap Script scan on port 1883.  
```shell
nmap -sC -sV -p 1883 10.10.83.7
```
![image](https://user-images.githubusercontent.com/127505784/224363579-48049882-5676-4d5c-970d-ad977d0e31ef.png)  

we can see that there some topics identified, even a base64 string  🤔. but before cheacking that out i want to list the topics by using the mosquitto command line client.  
## interacting with MQTT 🫂
To install the Mosquitto client on Kali Linux, you can use the following command:
```shell
sudo apt-get install mosquitto-clients
```
after we have that done we still need to find out what topics there are on the MQTT broker on 10.10.27.13.  
so to do that i used the following command:
```shell 
mosquitto_sub -t "#" -h 10.10.83.7
```
Let's breakdown the command:  
> *mosquitto_sub* --> This is the command to start the Mosquitto subscriber client.  
*-t "#"* --> This option sets the MQTT topic to subscribe to. The "#" character is a wildcard that matches all topics on the MQTT broker.  
*-h 10.10.83.7* --> This option specifies the IP address or hostname of the MQTT broker to connect to.  

So running that command gave us this output:  

![image](https://user-images.githubusercontent.com/127505784/224366063-1ecb95ef-578e-4b3c-a310-637c32476af6.png)

Looking at the output something suspicious caught my eye. The same Base64 encoded string from earlier. Let's decode that.  
we can simple decode that using online tools like [CyberChef](https://gchq.github.io/CyberChef/). But I prefer to do it in the terminal.

![image](https://user-images.githubusercontent.com/127505784/224359292-8878ca6c-e256-439e-929e-46fdcab24866.png)

from that piece of data we can understand that:  
>id = cdd1b1c0-1c40-4b0f-8e22-61b357548b7d --> UUID that may be used to uniquely identify an MQTT client.      
registered_commands = "HELP","CMD","SYS" --> lists the commands that the client is registered to handle.  
pub_topic = U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub --> The MQTT topic that the client should publish messages to.  
sub_topic = XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub --> The topic that the client should subscribe to.  

Now Let's interact with that, maybe tha flag is somewhere there.  
So we have the mosquitto client installed. Now i opend 2 terminals side by side.  
in the first i typed the following commad:  
```shell 
mosquitto_sub -t "U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub" -h 10.10.83.7
```

![image](https://user-images.githubusercontent.com/127505784/224367514-084217f2-f004-463f-a99f-f8a315e28f2f.png)  

When the subscriber client is running, it will wait for messages published to the specified topic on the broker. Any messages published to the topic will be received and displayed on the command line.  

Now in the second Terminal i typed this command:
```shell
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "test_message" -h 10.10.83.7
```
![image](https://user-images.githubusercontent.com/127505784/224368712-0c768e07-5cf4-4dcd-b63a-435bdfefc944.png)  

When the publisher client is running, it will connect to the broker and publish the message to the specified topic. The message will be sent to all subscribers to that topic on the broker.  

And here we go, we got something in the first terminal

![image](https://user-images.githubusercontent.com/127505784/224369486-d9844ccd-5705-4a95-a7e0-71b8dd4ec717.png)  

Another base64 encoded string let's decode that.

![image](https://user-images.githubusercontent.com/127505784/224369849-9e055165-3613-483f-ab68-15e77ed16e69.png)

we can understand that there is a format that need to be followed...  
Let's try to fill the fields:  
we know that the id is "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d".  
from the list of the commands that the client is registered to handle we pick "CMD".  
and for the argument i will try "ls" command.  

That should look like This:  

> {"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "ls"}  

but sending that like this will not work. we already know that we need to send that in base64. like this:
```shell
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQ==" -h 10.10.83.7
```
![image](https://user-images.githubusercontent.com/127505784/224372906-0a7b27ad-346c-498e-bd1b-7641be19a88a.png)  

And after running that we can see that we got another base64 string in the other terminal  

![image](https://user-images.githubusercontent.com/127505784/224372748-3f36f034-9e1c-42ed-aedd-21d3754b6b71.png)

Let's decode that.  

![image](https://user-images.githubusercontent.com/127505784/224373298-b27b398f-056e-4d1f-9bc5-b4a4ee03eeba.png)  

We can see That there is a file called "flag.txt" lets try again to use the format to get that flag by simple using "cat" command.  
> {"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "cat flag.txt"} --> But in base64.
```shell
mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAiY2F0IGZsYWcudHh0In0=" -h 10.10.83.7
```  

After Running That command we will get the base64 encoded flag.txt...  

![image](https://user-images.githubusercontent.com/127505784/224376173-3ec10685-ad83-42dc-91a0-6624a1660a8f.png)  


![image](https://user-images.githubusercontent.com/127505784/224378642-a2c5c8e5-db3e-4b3c-8779-7cc773576995.png)
