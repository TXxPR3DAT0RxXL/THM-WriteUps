# [TryHackMe](https://tryhackme.com/dashboard) -> [Lian_Yu](https://tryhackme.com/room/lianyu) Walkthrough 🎯
## Task 1 - Find the Flags 🚩
Welcome to Lian_YU, this Arrowverse themed beginner CTF box! Capture the flags and have fun.

Let's Deploy the machine!  
## Reconnaissance 🕵
Let's start by simple run `Nmap` scan like always ;)
```shell
nmap -sV -p- 10.10.214.206
```
![image](https://user-images.githubusercontent.com/127505784/224450704-333eacc7-f660-46a1-b8e2-f29aeba88ab0.png)

We can see that there is 5 ports open
>21 ftp  
>22 ssh  
>80 http  
>111 rpcbind  
>38651 status  

First i tried "anonymous" ftp login but Without success.  

By looking at first question "What is the Web Directory you found?" We can understand that we want to investegate the http.  
i went to http://10.10.214.206 and found Nothing...  
Let's try using gobuster to find hidden directory:  
```shell
gobuster dir -u http://10.10.214.206 -w /usr/share/dirb/wordlists/big.txt
```
![image](https://user-images.githubusercontent.com/127505784/224451559-0db82cca-8ba5-4de2-9e45-3e3c4f0b7a4a.png)

We found a directory but not the answer to the question.  
so i went to that url and looked in the source code:
 
![image](https://user-images.githubusercontent.com/127505784/224451920-05822b89-6403-4b93-9686-c1de64f1050d.png)

look like we found a name, Let's take a note of that.

Now i used gobuster on that directory:  
```shell 
gobuster dir -u http://10.10.214.206/island/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```
![image](https://user-images.githubusercontent.com/127505784/224452179-eb4620d0-834c-46b4-8949-d0281ea24f31.png)

Look like we found a Subdirectory! and yes that is the answer to the first question.  

i went to that directory:

![image](https://user-images.githubusercontent.com/127505784/224452624-dc6379f7-694b-4c6d-855b-dc9b0bb72478.png)

and that seems like we get a hint that there is a file with the extension .ticket  
lets run gobuster with the -x flag to search for that file:  
```shell 
gobuster dir -u http://10.10.214.206/island/2100 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x .ticket
```
![image](https://user-images.githubusercontent.com/127505784/224452693-96705d40-28d7-4019-8446-059c57d9e4c5.png)

we found a file (second question) let's go look at him:  

![image](https://user-images.githubusercontent.com/127505784/224452810-e4f43c0d-8682-4d19-a566-4ad5546dbc4f.png)

a hind 💡 the key word that we found in that file is base58 encoded string. (question 3)  
after decoding the string i tried to login to the ftp by using the name we found earlier (vigilante) and the decoded string as password:

![image](https://user-images.githubusercontent.com/127505784/224453130-09094c0f-88ae-4165-96e0-93433dcccb00.png)

and it worked! now lets see what is in that ftp by using ls command:

![image](https://user-images.githubusercontent.com/127505784/224453240-ad657039-98ea-4dca-8569-4e521c1ff078.png)

let's get all the 3 files by using get command.  
and before exiting the ftp i used cd .. to look at the /home directory and found that there is another user let's note that.  

![image](https://user-images.githubusercontent.com/127505784/224453533-3858bb81-b9a2-491f-857b-05efd70afbf3.png)

now lets check those images that we got from the ftp. i used exiftool on all of them and notice that "leave_me_alone_png" has a file format error message.  

![image](https://user-images.githubusercontent.com/127505784/224453865-baff6b7f-c0f7-450e-8aca-408a68f14680.png)

by searching in google for png "mugic numbers" we can see that the file need to start with "89 50 4E 47 0D 0A 1A 0A" Let's check that by using Hexeditor.  

![image](https://user-images.githubusercontent.com/127505784/224454082-9a26c12a-44e2-4da1-9383-517614894ae6.png)

we can see that we need to change the start of the hex to the right format.

![image](https://user-images.githubusercontent.com/127505784/224454240-3607686c-e74f-4b98-a190-a50736cf264e.png)

now let's open the image:

![image](https://user-images.githubusercontent.com/127505784/224454311-8d590fb4-bfc6-48f1-afd8-234c948b5fe6.png)

let's take what we want 🤔 ********  

by using steghide command we can see that aa.jpg contains a .zip file with the passphrase ********

![image](https://user-images.githubusercontent.com/127505784/224454711-6d916913-c84a-4073-bce4-190c0b28685b.png)

i used unzip command to extract the files from the zip file.  
and now we have 2 files:  

![image](https://user-images.githubusercontent.com/127505784/224454922-01cd17c1-dd71-46c1-ada9-88a1dec69522.png)

in 1 file there is general info but in the other there is SSH passwword (question 4)

## connect to the machine using SSH
And get user flag, root flag.  
so we have a password and if you remember we have username "slade" from the ftp /home directory. Let's try to connect:

![image](https://user-images.githubusercontent.com/127505784/224455308-fc2fd590-c4fd-42e4-bf96-a5c8e1a7ec83.png)

it worked and we evem got the user flag.  
now lets try to Privesc to get the root flag.  
i typed to following command to see what command slade can run without password.
```shell
sudo -l
```
![image](https://user-images.githubusercontent.com/127505784/224455593-aa0313e1-7624-4ec1-8b2f-a5d5213b8091.png)

we can see that we cam run pkexec.  
so what is it?  
pkexec allows an authorized user to execute PROGRAM as another user.  
if username is not specified, then the program will be executed as the administrative super user, root.

let's run the following command to get root privilege.  
```shell
sudo pkexec /bin/sh
```

![image](https://user-images.githubusercontent.com/127505784/224455834-4fb26f76-e6d6-45f7-b0b4-6620dfe8a4a2.png)

We got the answer to the final question.

![image](https://user-images.githubusercontent.com/127505784/224456268-1ec8b29d-ee2e-4eb6-b950-be3146c1e7ad.png)


 

