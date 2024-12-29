---
title: 'Wargames2024 CTF Game Pwn Challenge: World 1'
date: 2024-12-29 15:17:50
categories: Write-Up
author: w0rmhol3
tags: CTF
cover: https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/pay2win.jpg?raw=true
---
Wargames is an annual year end CTF organized by WGMY team from Malaysia. This year, they bring in some special categories: `Post Quantum Crptography`,  `Block Chain`, as well as `Game Pwn`. I did not solve the first 2 categories, but i manage to have fun and find the flag for one of the `Game Pwn Challenges - World 1`.

![World 1 Game](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/World1-exe.png?raw=true)

For this challenge, you need to find find the flag by winning a standalone RPG game. The game is nicely yet simple designed, and its easy to understand `(I can see why they choose to made this type of game for this challenge)`, like if you ever played ANY Pokemon game before, then yea you know it. This challenge requires you to defeat bosses to get parts of the flag. There's a total of 5 bosses in this game - `Tarnak`, `Sillad`, `Rakan`, `Baran`, and `Antares`. Each boss you defeat, you level up, along with your stats become better, which helps you with your next boss battle.

There are (as much as I tried and understand) 2 methods to solve this challenge.

1. .rmmzsave save data tampering
2. Unpacking the application

I will explain both methods within this blog, as game hacking challenges is something new to me and I wanted to include as much as I can to share with everyone.

Personally, I went for the first method during the ctf so let's start with that.

# .rmmzsave Save Data Tampering

The aim of this game is to save a princess, which is locked up in a castle. You need to defeat bosses to get there. As the level increases the boss gets more difficult to dealt with, and in the end, you get 1 hit K.O. type of boss. As for the challenge criteria, defeating the first 4 bosses grants you 4 parts of the flags.

The first boss - `Tarnak`, is basically what I like to call 'An Intro Boss', that lets you understand the game. You can easily defeat him and get the first part of the flag.

After defeating him, you will then receive an item. Check in your `Item` and use it on the player. The system will display out the first part of the flag.

![1st part of the flag](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/1st-Part-Flag.png?raw=true)

The second boss - `Sillad`, is more of a bitch ass Naruto wannabe that uses `Kage Bunshin no Jutsu` and create 2 more clones to help him fight. This part of the game, you can also defeat the boss up to luck and a little bit of sense. Kill the boss one by one and you can win.

And similar to the previous boss, defeating this boss will directly grant you with the second part of the flag.

![2nd part flag](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/2nd-Part-Flag.png?raw=true)

Now, for the next stage, you will fight with `Rakan`. This boss is a little bit tougher and may defeat you at times. But with some luck, you can still defeat this boss.

But this time, the flag is not given as an item, some players may miss this out and just directly approach the portal and go to the next stage `(There is no return after you go to the next stage, and you need to replay the whole game)`. But if you pay a little bit of your ADHD attention in this part of the game, you can see that there is a chest behind the portal. All you need to do is interact with it and then it will display the third part of the flag.

![3rd part flag](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/3rd-Part-Flag.png?raw=true)

Now, the forth boss - `Baran`. This level features a volcano like level, where there's lava all around and you walk can only walk on the path `(No suiciding)`. Similar to the third boss, you may get killed by Baran if you're unlucky. But still, there's not much reason to start hacking yet here, still playable.

After defeating the boss, you get....nothing again. But this time, there's no chest, nothing to interact with. So, this part of the flag is given in a different method. After defeating the boss, the map changes a little bit, in which if you go back the way you came from, you notice that the path is gone, but you can walk towards the right more. Then you will notice the pattern of the path is actually text. There's your forth flag.

![4th part flag](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/4th-Part-Flag.png?raw=true)

Then the gay ass cunt - `Antares`. This boss is the final boss sitting between you, the flag and the princess. He is the reason this challenge requires you to hack. He is the reason that you dislike RPG games. He is the reason that touching another man is gay. This boss is so petty and gay, in which if his health is low `(we don't see the health bar but I'm assumming it)`, he will start to touch you. He uses his ultimate one hit K.O. move on you - `Death Touch` that deals millions of damage with 1 hit, so you die.

![Death touch and damage](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/Death-Touch.png?raw=true)

`SO, START HACKING.` In this game, you can save your progress, and the game generates an `.rmmzsave` file for you within the `'save'` folder.

![save folder & save game ss](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/Save-Folder.png?raw=true)

The .rmmzsave file can be tampered easily with any [save-editor](https://www.save-editor.com/tools/rpg_tkool_mz_save.html) online tool such as the one I used. So, by uploading your save data into it, you will be able to see all the game details within the save data, which includes your `character's stats`. The one that I found interesting and the one that actually worked is the `paramPlus` array of data within the `ID : 1` tab. I had modified the values 

original: [0,0,0,0,0,0,0,0]

tampered: [999,999,999,999,999,999,999,999]

![save-editor ss](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/Save-Editor.png?raw=true)

After that, all you need to do is download the tampered file, replace the existing file with it, and load the save data. This time, you will see the difference in your stats.

![stats tampered](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/Character-Stats.png?raw=true)

Now, go back and challenge Antares The Touchy Boy, and you will be able to 1 hit K.O. his life.

![1 hit KO boss](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/1-Hit-Kill-Boss.png?raw=true)

After defeating the final boss. An npc and a floating blue thingy will spawn. First interact with the NPC and he will give you bullshit saying the princess is in another castle, but he also gives you a type of code. In which if u decode it its basically `'wgmy'`.

![NPC saying bs](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/NPC.png?raw=true)

Then, interact with the floating blue thingy, and the first thing it does is to ask for a type of password. So, just input the decoded value in, and it will present you with a QR code. The QR code given is the final part of the flag.

![QR Decode](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/QR-Code.png?raw=true)

flag: wgmy{5ce7d7a7140ebabf5cd43effd3fcaac2}

# Unpacking The Application

Just so you know, I'm not a pwn or reversing type of guy, so this is as much explanation you can get from my understanding.

As I refer to other team's writeup to have a better understanding, the application is packed with `Enigma Virtual Box`. Hence, there is a safe way to allow us to gain access to all the data that the application runs on, which is by using `evbunpack`. To setup this tool, just use pip:

```sh
pip install evbunpack
```

Unpacking the application:

```sh
evbunpack 'World I.exe' unpack #unpack is just the output file name i set btw
```

![evbunpack ss](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/evbunpack.png?raw=true)

After having the application unpacked, you will be introduced with all the game files, including the ones that affects the game. There is a `/data` folder that can be found, which consists of all the .json files that can affect the gameplay.

![/data folder](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/Date-Folder.png?raw=true)

This is where you can have your fun. You can modify any logical data within the json file that relates to the game experience into your liking to solve this challenge. You can tamper enemy health, by changing it to 1 hp so that u can 1 hit them with any damage, and they still die; change your weapon stats to buff up your character stats; 

I was worrying do I need to repack it before running or anything, but no, you can just run the game directly and the value you changed will be applied directly into the game.

But since I already solved the game, I just want to have fun, and I found that there are so many fun features you can try while hacking this game, like change your character's class into to sorcerer and have magic powers.

![Sorcerer Character](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/2024/GamePwn/Modified-Character.png?raw=true)

This is fun and interesting experience learning something I don't normally put my hands on. It was really cool that they actually created an RPG game that actually has alot of hidden features once you hacked into it, was really exciting seeing this type of stuff.
