# [TryHackMe](https://tryhackme.com/dashboard) -> [Disgruntled](https://tryhackme.com/room/disgruntled) Walkthrough 🎯
## Task 1 - Introduction ✏️
### Hey, kid! Good, you’re here! 👋🏻

Not sure if you’ve seen the news, but an employee from the IT department of one of our clients (CyberT) got arrested by the police.  
The guy was running a successful phishing operation as a side gig.  

CyberT wants us to check if this person has done anything malicious to any of their assets.  
Get set up, grab a cup of coffee, and meet me in the conference room.  

### Connecting to the machine 🔗  
Start the virtual machine in split-screen view by clicking on the green "Start Machine" button on the upper right section of this task. Alternatively, you can connect to the VM using the credentials below via "ssh".

![image](https://user-images.githubusercontent.com/127505784/229293145-837e3412-306d-4636-bf2e-6cbbb5da3c4e.png)

## Task 2 - Pre-requisites

This room requires basic knowledge of Linux and is based on the [Linux Forensics](https://tryhackme.com/room/linuxforensics) room.  
You might consider doing it before you start!  
To succeed in this room, I recommend to go and look at the Cheat sheet attached there. 

## Task 3 - Nothing suspicious... So far 🤨
### The user installed a package on the machine using elevated privileges. According to the logs, what is the **full COMMAND**?  
Lets dig inside **auth.log** to see if there is somethins suspicious:  
i used the following command for that:  
```shell
cat /var/log/auth.log | grep COMMAND 
```
![image](https://user-images.githubusercontent.com/127505784/229294155-be71b556-93cc-49da-98ad-8128b5009092.png)

According to the logs we can see what package was installed and what was the command.  

### Now we were asked What was the present working directory (PWD) when the previous command was run?  

By looking at the full line from the output we can see The "PWD=" that tells us the present working directory.

## Task 4 -  Let’s see if you did anything bad 👀  
### Which user was created after the package from the previous task was installed?  
in the same output from the auth.log file we can see couple lines below that a user created:

![image](https://user-images.githubusercontent.com/127505784/229294678-dd90b968-5968-4258-a626-e7dab18c7ca5.png)

### A user was then later given sudo priveleges. When was the sudoers file updated? (Format: Month Day HH:MM:SS)  
we can see 1 line below that the sudoers file was edited by running "visudo":  

![image](https://user-images.githubusercontent.com/127505784/229294797-b13cdf56-41aa-4140-8acf-af2e0f316377.png)

in the start of the line we can see when that happend.  

### A script file was opened using the "vi" text editor. What is the name of this file?

![image](https://user-images.githubusercontent.com/127505784/229294882-62f5c96c-75d8-406c-96f6-120a5074b319.png)

we can see that a file was edited by "vi" text editor.  

## Task 5 - Bomb has been planted. But when and where?
That bomb.sh file is a huge red flag! While a file is already incriminating in itself, we still need to find out where it came from and what it contains. The problem is that the file does not exist anymore.  

### What is the command used that created the file **bomb.sh**?

we can maybe see that in the ".bash_history" file but first we need to go to the home directory of the user that created the file...  
by looking at the output & filtering the name of the file we can see that:  

![image](https://user-images.githubusercontent.com/127505784/229295270-a04852a0-5217-49a9-9711-5c25be6501d6.png)

### The file was renamed and moved to a different directory. What is the full path of this file now?

for this let's check ".viminfo" file:  

![image](https://user-images.githubusercontent.com/127505784/229295380-66f8c3b1-a3f3-4e88-8ad5-d18fd4e3b2b2.png)

and we can see the new name that was given to the file!

### When was the file from the previous question last modified? (Format: Month Day HH:MM)

for that we need to go to the place where the file is stored:  

![image](https://user-images.githubusercontent.com/127505784/229295482-128b01e4-44f4-4b7a-b66d-efd845264a64.png)

and here we got the answer!

### What is the name of the file that will get created when the file from the first question executes?

for that we need to open the file and see what it does...

![image](https://user-images.githubusercontent.com/127505784/229295542-00d5ccc5-6474-46cb-ae6f-da5ef9ebb87e.png)

## Task 6 - Following the fuse 
So we have a file and a motive. The question we now have is: how will this file be executed?
### At what time will the malicious file trigger? (Format: HH:MM AM/PM)  

![image](https://user-images.githubusercontent.com/127505784/229295628-a10061e3-70e4-4da4-a8ed-289eebd92cb0.png)

by putting this schedule expression in a [site](https://crontab.guru/) that convert it to the AP:PM time expression i got this:

![image](https://user-images.githubusercontent.com/127505784/229295735-5ae7232a-8f20-4a0b-b66c-4ced73d40958.png)

## And that's it for today 😎 
<img align="left" src="https://media.tenor.com/uaWxECFrHIMAAAAC/detective-pikachu-investigation.gif" />
