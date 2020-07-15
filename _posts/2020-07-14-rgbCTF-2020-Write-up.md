---
layout: post
title: rgbCTF 2020 Write-up
date: 2020-07-14 12:00:00
categories: posts
comments: true
en: true
description: An example post to this theme
keywords: "CTF, rgb, writeups, challenges"
---

## Introduction

| Challenge Name | Category | Solves | Points |
|:--------------:|:--------:|:------:|:------:|
|[Name a more iconic band](#name-a-more-iconic-band) | Beginner | 89 | 411 |
|[I Love Rainbows](#i-love-rainbows) | Cryptography | 408 | 50 |
|[Penguins](#penguins) | Misc | 135 | 295 |
|[Tic Tac Toe](#tic-tac-toe) | Web | 333 | 50 |
|[vaporwave1](#vaporwave1) | [ZTC] | 166 | 190 |

----------------

## Name a more iconic band
#### Category: Beginner | Solves: 89 | Points: 411

----------------

![Name a more iconic band challenge description](https://i.imgur.com/zwX6zYQ.png)

We download the "data.7z" file and extract the contents:

`$ file data_1
data_1: ELF 64-bit LSB core file, x86-64, version 1 (SYSV)`

Running strings on this "ELF" file we become immediately aware that this is not in fact an ELF file.
This file contains a Windows memory coredump.
We will need to [extract](https://www.andreafortuna.org/2017/06/23/how-to-extract-a-ram-dump-from-a-running-virtualbox-machine/) the raw data and then analyze it with [Volatility](https://github.com/volatilityfoundation/volatility/wiki/Installation).
We use objdump with egrep to locate the size and offset of the first LOAD section.

![Name-a-more-iconic-band-objdump](https://i.imgur.com/Oe15Iz5.png)

This section contains the RAM information we care about. We remove the bytes we don't need.

![Name-a-more-iconic-band-offset](https://i.imgur.com/t1lI2o6.png)

Next, we use volatility to determine the image information from our data.raw file.

![name-a-more-iconic-band-volatility-imageinfo](https://i.imgur.com/QmICVDb.png)

Then, we use hivelist to locate the SAM and SYSTEM Virtual memory locations.

![name-a-more-iconic-band-volatility-hivelist](https://i.imgur.com/mBJQKBS.png)

Next, using these locations we dump the NTLM hashes stored in memory from this file.

![name-a-more-iconic-band-volatility-hashdump](https://i.imgur.com/wAEbuNk.png)

Now all we have to do is crack the NTLM hashes, sort the passwords alphabetically, create an MD5 hash of the result, and submit the flag.

![name-a-more-iconic-band-hashcrack](https://i.imgur.com/wVy4YPf.png)

Lastly, we use the following command to generate our flag:

![name-a-more-iconic-band-md5sum](https://i.imgur.com/XGKds6x.png)

{:refdef: .flag}
Flag: `rgbCTF{cf271c074989f6073af976de00098fc4}`
{:refdef}

----------------

## I Love Rainbows

#### Category: Cryptography | Solves: 408 | Points: 50

----------------

![i-love-rainbows](https://i.imgur.com/Qu538xe.png)



We download the rainbows.txt file and list the contents.



![i-love-rainbows-contents](https://i.imgur.com/9kdQ6NS.png)



From the name of the challenge we are given a hint that we may be looking at a rainbow table attack using this list of hashes.

We run hash-identifier on the first two hashes to determine their hash type.



![i-love-rainbows-hash](https://i.imgur.com/a3uZlJz.png)

![i-love-rainbows-sha256](https://i.imgur.com/7793gNB.png)



Okay, so the first hash `4b43b0aee35624cd95b910189b3dc231` is an MD5 hash and the second `cd0aa9856147b6c5b4ff2b7dfee5da20aa38253099ef1b4a64aced233c9afe29` is SHA-256. We will assume the shorter hashes are MD5 and the longer hashes are SHA-256. Assuming these hashes are not [salted](https://doubleoctopus.com/security-wiki/encryption-and-cryptography/salted-secure-hash-algorithm/), we could attempt to crack these locally. I even considered generating my own rainbow tables (yikes) for the sake of the challenge, but instead let's see if we can use an online password cracker that already has rainbow table lists generated and likely has the plain-text results.

We'll use [Crackstation](https://crackstation.net). A limit of 20 hashes can be submitted at once, we'll submit the first 20 hashes and then the last 4 separately.



![i-love-rainbows-crackstation](https://i.imgur.com/l7Q8bR7.png)

![i-love-rainbows-crackstation2](https://i.imgur.com/R14wx8D.png)



Awesome, the plain-text value of these hashes were previously cracked, let's submit our flag!

{:refdef: .flag}
Flag: `rgbCTF{4lw4ys_us3_s4lt_wh3n_h4shing}`
{:refdef}

----------------

## Penguins

#### Category: Misc | Solves: 135 | Points: 295

----------------

![penguins-challenge](https://i.imgur.com/sLCB3ij.png)



We download and unzip the file.



![penguin-unzip](https://i.imgur.com/SXyApI9.png)



Ah, a git challenge. Let's explore the contents and see if we can find any useful information.



![penguin-git](https://i.imgur.com/M4pX0jO.png)

lol..

After some digging around the .git folder I decided to take a look at the git log history.

![penguins-git-history](https://i.imgur.com/7tbkxlT.png)



Commit `57adeae7` looks interesting. Let's check out the "relevant file" change.



![penguins-git-ls](https://i.imgur.com/eExUewD.png)



After checking the new files we find the following:



![penguins-base64](https://i.imgur.com/pjhEmmo.png)



We base64 decode this string with `base64 -d < perhaps_relevant_v2` to receive the following output:

`as yoda once told me "reward you i must"
and then he gave me this ----
rgbctf{d4ngl1ng_c0mm17s_4r3_uNf0r7un473}`

{:refdef: .flag}
Flag: `rgbctf{d4ngl1ng_c0mm17s_4r3_uNf0r7un473}`
{:refdef}

----------------

## Tic Tac Toe

#### Category: Web | Solves: 333 | Points: 50

----------------

![tic-tac-toe](https://i.imgur.com/Hbclev4.png)



We navigate to `http://challenge.rgbsec.xyz:8974` and are presented with the following:

![tic-tac-toe-web](https://i.imgur.com/0Rikkjn.png)



As the player we are given `uwu` as our mark, the script uses `owo`.

I played a few games to determine the behavior of the script running this site. I soon realized that the script was making a logic error when presented with the following condition: In the case that the human player makes a move that sets up a winning condition the following round, the script will prevent the player from winning on the following round even if the script can win on that same exact round. See image below:



![tic-tac-toe-uwus](https://i.imgur.com/5SFcnS9.png)



So, if we place `uwu`on the middle bottom square, the script's logic will prevent us from winning the following move. See image:



![tic-tac-toe-winning-on-next-move](https://i.imgur.com/xiHeYUm.png)



Thus, we are winning on the next move, top middle square.



![tic-tac-toe-base64](https://i.imgur.com/Fdjp25W.png)

We base64 decode this string and retrieve the flag!



![tic-tac-toe-flag](https://i.imgur.com/155646e.png)



Note: I'm sure there was another way to solve this by modifying the javascript, but why work hard?


{:refdef: .flag}
Flag: `rgbCTF{h4h4_j4v42cr1p7_ev3n72_AR3_c00L}`
{:refdef}

----------------

## vaporwave1

#### Category: [ZTC] | Solves: 166 | Points: 190

----------------