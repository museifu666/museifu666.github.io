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

## Name a more iconic band
#### Category: Beginner | Solves: 89 | Points: 411

![Name a more iconic band challenge description](https://i.imgur.com/zwX6zYQ.png)

We download the "data.7z" file and extract the contents:

`$ file data_1
data_1: ELF 64-bit LSB core file, x86-64, version 1 (SYSV)`

Running strings on this "ELF" file we become immediately aware that this is not in fact an ELF file.
This file contains a Windows memory coredump.
We will need to [extract](https://www.andreafortuna.org/2017/06/23/how-to-extract-a-ram-dump-from-a-running-virtualbox-machine/) the raw data that we can analyze with Volatility.
We use objdump with egrep to locate the size and offset of the first LOAD section.

![Name-a-more-iconic-band-objdump](https://i.imgur.com/Oe15Iz5.png)

Using this info we can extract the RAM we will use and remove the rest of the bytes we don't need.

![Name-a-more-iconic-band-offset](https://i.imgur.com/t1lI2o6.png)

Next, we use volatility to determine image information from our data.raw file.

![name-a-more-iconic-band-volatility-imageinfo](https://i.imgur.com/QmICVDb.png)

Then we use hivelist to locate the SAM and SYSTEM Virtual memory locations

![name-a-more-iconic-band-volatility-hivelist](https://i.imgur.com/mBJQKBS.png)

Finally, using these locations we dump the NTLM hashes stored in memory from this file.

![name-a-more-iconic-band-volatility-hashdump](https://i.imgur.com/wAEbuNk.png)

Now all we have to do is crack these NTLM hashes, sort the passwords alphabetically, create an MD5 hash of the result and submit the flag.

![name-a-more-iconic-band-hashcrack](https://i.imgur.com/wVy4YPf.png)

Lastly, we use the following command to generate our flag:

![name-a-more-iconic-band-md5sum](https://i.imgur.com/XGKds6x.png)

Flag: `rgbCTF{cf271c074989f6073af976de00098fc4}`

## I Love Rainbows

#### Category: Cryptography | Solves: 408 | Points: 50

## Penguins

#### Category: Misc | Solves: 135 | Points: 295

## Tic Tac Toe

#### Category: Web | Solves: 333 | Points: 50

## vaporwave1

#### Category: [ZTC] | Solves: 166 | Points: 190

Some example text, okay? It's normal, right? :)

There are many variations of passages of Lorem Ipsum available, but the majority have suffered alteration in some form, by injected humour, or randomised words which don't look even slightly believable. If you are going to use a passage of Lorem Ipsum, you need to be sure there isn't anything embarrassing hidden in the middle of text. All the Lorem Ipsum generators on the Internet tend to repeat predefined chunks as necessary, making this the first true generator on the Internet. It uses a dictionary of over 200 Latin words, combined with a handful of model sentence structures, to generate Lorem Ipsum which looks reasonable. The generated Lorem Ipsum is therefore always free from repetition, injected humour, or non-characteristic words etc.

**bold text**

*italic text*

> And this is a blockquote

~~~
And this is a source code in a block
Yeah, don't use highligth :D
~~~

But if you like `inline code`, it's fine.

And, of course, links: [lampiaosec].

# Lists

* Item x
* Item z

1. Item 01
2. Item 02

## Table of content

* toc
{:toc}


[lampiaosec]: https://lampiaosec.github.io
