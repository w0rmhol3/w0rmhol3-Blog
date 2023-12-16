---
title: 'Wargames2023 CTF MISC Challenge: Splice'
date: 2023-12-17 00:56:09
categories: Sharing
author: w0rmhol3
tags: CTF
cover: https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/67dc9b80-b12e-4729-a99e-9d1759dfcec6
---
This year, wargames is filled with very interesting challenges. As for this miscellaneous challenge - Splice, it's a more lighthearted challenge as compared to the other brain-cells consuming challenges. The challenge is to recover 2 QR code that are removed at the middle of the QR image. <!--more-->

![PNG Files](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/bd9f0bc6-93e1-46a3-9647-4b505c4c5f16)

The challenge provides 3 PNG files, page1, page2, and page3. The first QR is sliced vertically in the middle, the second QR is sliced horizontally in the middle. A third PNG file was given in which is a very dark kinda QR, in which is impossible to scan. So it gives me an idea of using page3.png to fill up removed middle part of both QR.

![GIMP](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/d5808cc1-19c7-4ffc-b0ec-59cc5438457b)

By using `GIMP`, the PNG files had been opened as layers so that it can be put together instead of being 3 seperated PNG files. `Note: It is important  that page3.png must be the higher layer as it has a default lower opacity as compared to page1.png and page2.png or page3.png is not visible`. So what happened here is that even if its layered together, the QR is not able to be scanned, hence `some parts need to be removed`. What I did was using the rectangular selection tool to highlight the page3.png QR parts that had already been covered by page1.png. 

![Vertical Cut](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/1685cfa1-f09f-4a44-be83-ced499dbe7b1)

The result of removing the left and right side of the page3.png's QR. Although it has been fairly connected, the darkness of the QR does not match with the 
After that, all I did is `adjust the opacity of the image until it is similarly the same darkness for both page1.png and page3.png.`

![Changing Opacity](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/6e5790d8-74c7-4c2f-8e3e-0c144b09e211)

![1st QR Done!](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/feff8c6f-8224-4c7e-b759-beccb414eb2f)

After having the QR built, just use any QR scanner and a base64 text is given: `d2dteXs1ZDdiZGVhNjU0NzJjYT`.

![Horizontal Cut](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/dfe3a47a-c394-4c6a-bbc9-548ab91bb1b4)

Process is repeated for the 2nd QR (page2.png), but `removing page3.png QR code horizontally` to adapt to the page2.png QR.

![2nd QR Done!](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/b0e850fc-270e-4737-84bd-df8413f66fe2)

After that, I adjust the opacity again and the 2nd QR code is done.

Scanning it will provide another base64 text: `llNjE1ZmNiZDBmOTBhM2I0OX0`.

![Decoding Base64](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/18811cbd-c7ea-4509-b2e0-f511976bed31)

Using Cyberchef to decode, the flag is shown.
Flag: `wgmy{5d7bdea65472ca9e615fcbd0f90a3b49}`
