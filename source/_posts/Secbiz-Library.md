---
title: 'ASCIS2023 Jeopardy Finals: Secbiz-Library Web Challenge'
date: 2023-10-28 20:58:36
author: w0rmhol3
categories: Write-Up
tags: Web CTF
cover: https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/40327b82-bf06-47ba-bb23-2b42b4c79452
---
The ASCIS CTF is an annual Vietnam CTF challenge that many teams from ASEAN countries will participate. This year, the CTF consists of 3 rounds, Warmup Round which requires teams to at least solve 1 challenge to proceed with the next round; Semi-Finals, and 2 different Finals, in which top20 overall teams will be playing Attack and Defense CTF, while the remaining teams continue the Jeopardy CTF. <!--more--> 

This challenge is a web challenge from the ASCIS CTF Semi-Finals. When first access the webpage, it shows a plain html site with a huge list of html files within it `(rfcxxxx.html)`. 

![Webpage](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/10a34688-28b2-4024-a6a0-9269012cbaaf)

A python file can be downloaded to view the backend of the website. At first, through the overall review, it `seems to me like its LFI vulnerability`, and I had spent certain period of time to try to do path traversal attacks. Later on, it was discovered that the `DEBUG_MODE_ENABLED` requires a value and by default is set to None. Hence it raise suspicion to our team. 

![Source Code](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/28afc1ec-2c62-443e-9963-5da9dca89964)

Through analysing and testing, it was found out that there is possibility to execute `Server Side Template Injection (SSTI) attack` onto the system. Through the help of hacktricks SSTI cheatsheet, i was able to found that the SSTI type is Jinja2. To ease the testing process, I had used burpsuite to intercept the network traffic and had sent it to repeater to test out the attack.

![Types Of SSTI](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/0bf210a7-7249-4db3-958d-c3abe6c62401)

By injecting `{{7*7}}?DEBUG_MODE_ENABLED=1` the POC for SSTI is success as it outputs as 49.  

![POC](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/eb1cee51-8c24-4183-8b2a-7c8a83ea865d)

So, going through the cheatsheet, i was able to find a suitable injection that can be done to run bash command. Using `{{namespace.__init__.__globals__.os.popen('id')}}`, it outputs the user details within the system.

![id](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/19c52a40-d5b9-4684-a391-6bb018d9536c)


Then by changing the `'id'` to other command I was able to inject commands through SSTI. At first I was trying to list out where the files is within the system by using `'ls'`, but nothing much was found beside a prestart.sh that I did not find much in help.

![ls](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/520267b8-747c-44d9-82fe-46243ecc1cf3)


My team member gave me an idea to `run 'env' command to list out all the environment variables`, and it was found out that a variable call `FLAG_DIR` provides the directory to the flag.

![env](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/9839ecb5-afa7-412b-a92a-0495c264e777)


So by injecting `'cat /opt/y5oyqodQ3BCjCdVSxQuL'` I was able to retrieve the flag.

![Solution](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/4b7aefa5-08b0-42cf-bc27-7c51e1b93192)


Flag: ASCIS{fr0m_jinja2_with_l0v3_w2sLxD9EnmmFRTtB8joX}


