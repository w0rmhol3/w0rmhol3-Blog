---
title: 'Wargames2023 CTF MISC Challenge: Splice'
date: 2023-12-17 00:56:09
categories: Write-Up
author: w0rmhol3
tags: CTF
cover: https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/FunnyCover.png?raw=true
---
This year, wargames is filled with very interesting challenges. As for this miscellaneous challenge - Splice, it's a more lighthearted challenge as compared to the other brain-cells consuming challenges. The challenge is to recover 2 QR code that are removed at the middle of the QR image. <!--more-->

![PNG Files](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/PngFiles.png?raw=true)

The challenge provides 3 PNG files, page1, page2, and page3. The first QR is sliced vertically in the middle, the second QR is sliced horizontally in the middle. A third PNG file was given in which is a very dark kinda QR, in which is impossible to scan. So it gives me an idea of using page3.png to fill up removed middle part of both QR.

![GIMP](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/Gimp.png?raw=true)

By using `GIMP`, the PNG files had been opened as layers so that it can be put together instead of being 3 seperated PNG files. `Note: It is important  that page3.png must be the higher layer as it has a default lower opacity as compared to page1.png and page2.png or page3.png is not visible`. So what happened here is that even if its layered together, the QR is not able to be scanned, hence `some parts need to be removed`. What I did was using the rectangular selection tool to highlight the page3.png QR parts that had already been covered by page1.png. 

![Vertical Cut](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/vertical.png?raw=true)

The result of removing the left and right side of the page3.png's QR. Although it has been fairly connected, the darkness of the QR does not match with the 
After that, all I did is `adjust the opacity of the image until it is similarly the same darkness for both page1.png and page3.png.`

![Changing Opacity](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/Opacity.png?raw=true)

![1st QR Done!](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/1stQR.png?raw=true)

After having the QR built, just use any QR scanner and a base64 text is given: `d2dteXs1ZDdiZGVhNjU0NzJjYT`.

![Horizontal Cut](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/Horizontal.png?raw=true)

Process is repeated for the 2nd QR (page2.png), but `removing page3.png QR code horizontally` to adapt to the page2.png QR.

![2nd QR Done!](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/2ndQR.png?raw=true)

After that, I adjust the opacity again and the 2nd QR code is done.

Scanning it will provide another base64 text: `llNjE1ZmNiZDBmOTBhM2I0OX0`.

![Decoding Base64](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/Wargames/Misc/Splice/Cyberchef.png?raw=true)

Using Cyberchef to decode, the flag is shown.
Flag: `wgmy{5d7bdea65472ca9e615fcbd0f90a3b49}`
