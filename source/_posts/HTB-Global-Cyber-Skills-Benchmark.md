---
title: 'HTB Global Cyber Skill Benchmark 2025'
date: 2025-05-28 15:51:12
categories: Write-Up
author: w0rmhol3
tags: CTF
cover: https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HTB2025/HTB.jpg?raw=true
---
We're back at it with another CTF writeup. As now I'm a working professional, I'm eligible to take part in the `HTB Global Cyber Skill Benchmark 2025` under Team ProCheckUp. In this CTF, we were placed 141/796 within the competition. It's actually quite impressive, considering only like 5 of us are playing the CTF. I had the the chance to solve challenges of different categories (ML,RE,Forensics,ICS,Crypto) but I won't be covering everything in this writeup. Here are some of the challenges I found interestingly fun. <!--more-->

![HTB Cert](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HTB2025/HTB_Cert.png?raw=true)

# ML: UPLINK ARTIFACTS
This challenge falls under the Machine Learning category. It presents a CSV file with a bunch of floating numbers and thats about it. So knowing it's ML challenge, it has something to do with the floats, which is either rounding up and convert it into text, or graph plotting (Yes, graph plotting it is). So, after trying out the first approach to convert the content to text, which of course didn't work, I went to graph plotting method. One of the most useful library for this use case in python is the [Python pandas](https://pandas.pydata.org/) library that mainly used for data analysis, that allows data manipulation, and [matplotlib](https://matplotlib.org/stable/tutorials/pyplot.html) which has the graph plotting feature.

This challenge is a bunch of trial and error on the graph plotting. By scripting out (with help of some AI) different views and type of data visualizing methods, I was able to get an idea of which approach I can take.

![Data Visualization 1](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HTB2025/Data_Visualization_1.png?raw=true)

![Data Visualization 2](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HTB2025/Data_Visualization_2.png?raw=true)

As you can see on the graph generated, The red dots assembles a squarish shape, which gives me an idea that It might not be plaintext directly for the flag, but a QR code embedded. After noticing that, I start to only focus on the red dots, which were identified as `'label 1' floats value` within the CSV file. I then proceed to plot the graph with `only label 1's value` and ignore the rest and it got me a QR, but not scannable yet.

![QR Unscannable](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HTB2025/QR_Unscannable.png?raw=tru)

I know everything was correct, just need to make it scannable and then we're done. So I quickly use AI to improve the script by rounding up the values and rendering it to make it scannable.

Working Script:
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

FILE_PATH = './uplink_spatial_auth.csv'
df = pd.read_csv(FILE_PATH)

# Filter for label 1 points only
label1_points = df[df['label'] == 1].copy()  # Use copy to avoid warnings

label1_points['x'] = label1_points['x'].round()
label1_points['y'] = label1_points['y'].round()

# Create a grid for QR code
x_min, x_max = int(label1_points['x'].min()), int(label1_points['x'].max())
y_min, y_max = int(label1_points['y'].min()), int(label1_points['y'].max())

grid_size = int(max(x_max - x_min + 1, y_max - y_min + 1))
qr_grid = np.zeros((grid_size, grid_size))

# Fill the grid with 1s where points exist
for _, point in label1_points.iterrows():
    x = int(point['x'] - x_min)
    y = int(point['y'] - y_min)
    qr_grid[y, x] = 1

plt.figure(figsize=(10, 10))

# Plot the QR code
plt.imshow(qr_grid, cmap='binary', interpolation='nearest')
plt.axis('off')  # Hide axes
plt.title('QR Code View')

# Save with high DPI for better scanning
plt.savefig('qr_code_scannable.png', dpi=300, bbox_inches='tight', pad_inches=0)

plt.figure(figsize=(10, 10))
plt.imshow(qr_grid, cmap='binary', interpolation='nearest')
plt.grid(True, which='both', color='gray', linewidth=0.5)
plt.axis('on')
plt.title('QR Code View with Grid')
plt.savefig('qr_code_with_grid.png', dpi=300, bbox_inches='tight')

plt.show()

print(f"\nQR Code Statistics:")
print(f"Grid size: {grid_size}x{grid_size}")
print(f"Number of black squares: {int(qr_grid.sum())}")
print(f"Number of white squares: {grid_size * grid_size - int(qr_grid.sum())}")
print(f"X range: [{x_min}, {x_max}]")
print(f"Y range: [{y_min}, {y_max}]") 
```

Output:
![Final QRCode](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HTB2025/Final_QR.png?raw=true)

`Flag: HTB{clu5t3r_k3y_l34k3d}`

# RE: TINYPLATFORMER
I don't normally look at reverse engineering, cuz I know I'm bad at it. BUT, special scenario like if its mobile reversing or `GAME HACKING`, then we go! This challenge provide players with a `no extension file` named `TinyPlatformer`. So quickly i just went to check the file type, x64 executable, so I just run and see what it is before actually working on the challenge. It's basically a platformer game, that requires the player to collect the coins within a short period of time to pass the level.

![TinyPlatformer](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/HTB2025/TinyPlatformer.png?raw=true)

But after level 1, its impossible to pass level 2 (idk...maybe skill issue). So I just close the game and only now I notice that it was built through PyGame:

```bash
└─$ ./TinyPlatformer
pygame 2.0.1 (SDL 2.0.14, Python 3.7.5)
Hello from the pygame community. https://www.pygame.org/contribute.html

```
At this point I'm like `OH YEA OK GOOD` my small research on game development and game hacking after WGMY24 have a good influence here hahaha. So now I got to working. First step is to unpack this application. I used one of python standalone library call [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor) that extracts the content within the PyInstaller.

```bash
pyinstxtractor TinyPlatformer
```
```bash
└─$ ls TinyPlatformer_extracted
base_library.zip    libFLAC-bf6d1292.so.8.3.0       libpython3.7m.so.1.0                 libssl.so.1.0.0                  libwebp-582c46b3.so.7.1.0  pyimod01_os_path.pyc    pyi_rth_pkgutil.pyc
FreeSansBold.ttf    libfreetype-2d39c124.so.6.17.1  libreadline.so.6                     libtiff-97e44e95.so.3.8.2        libz-a147dcb0.so.1.2.3     pyimod02_archive.pyc    pyi_rth_subprocess.pyc
libbz2.so.1.0       libjpeg-bd53fca1.so.62.0.0      libSDL2-2-d6813302.0.so.0.14.0       libtinfo.so.5                    libz.so.1                  pyimod03_importers.pyc  PYZ-00.pyz
libcrypto.so.1.0.0  libmikmod-fabcac29.so.2.0.4     libSDL2_image-2-554041b7.0.so.0.2.3  libuuid.so.1                     main.pyc                   pyimod04_ctypes.pyc     PYZ-00.pyz_extracted
lib-dynload         libogg-b51fbe74.so.0.8.4        libSDL2_mixer-2-5dc902ba.0.so.0.2.2  libvorbis-205f0f59.so.0.4.8      pygame                     pyi_rth_inspect.pyc     struct.pyc
libffi.so.6         libpng16-b14e7f97.so.16.37.0    libSDL2_ttf-2-dd80ed71.0.so.0.14.1   libvorbisfile-f207f3a6.so.3.3.7  pyiboot01_bootstrap.pyc    pyi_rth_pkgres.pyc      venv                                                                                                                                                       
```

After extracting it, there are alot of `pyc (python compiled) files` that are basically non-readable bytecodes. We will focus on getting the main.pyc content as I assumed that's where all the important codes are. So what we can do now is to convert the byte codes into readable codes using [uncompyle6](https://github.com/rocky/python-uncompyle6) and leak the code. I faced some installation issue within my vm and wasn't sure what happened but in the end I just installed and run it within python virtual environment.

```bash
└─$ python3 -m venv venv && source venv/bin/activate && uncompyle6 main.pyc
/home/kali/Documents/CTF/TinyPlatformer_extracted/venv/lib/python3.12/site-packages/click/core.py:1193: UserWarning: The parameter --version is used more than once. Remove its duplicate as parameters should be unique.
  parser = self.make_parser(ctx)
/home/kali/Documents/CTF/TinyPlatformer_extracted/venv/lib/python3.12/site-packages/click/core.py:1186: UserWarning: The parameter --version is used more than once. Remove its duplicate as parameters should be unique.
  self.parse_args(ctx, args)
# uncompyle6 version 3.9.2
# Python bytecode version base 3.7.0 (3394)
# Decompiled from: Python 3.12.7 (main, Nov  8 2024, 17:55:36) [GCC 14.2.0]
# Embedded file name: main.py
import pygame, os, sys

def resource_path(relative_path):
    base_path = getattr(sys, "_MEIPASS", os.path.dirname(os.path.abspath(__file__)))
    return os.path.join(base_path, relative_path)


pygame.init()
WIDTH, HEIGHT = (800, 640)
PLAYER_SIZE = 40
SCROLL_AMOUNT = 520
GRAVITY = 0.8
JUMP_ACCEL = 15
MOVEMENT_SPEED = 5
PLATFORM_COLOR = (255, 0, 0)
PLAYER_COLOR = (255, 0, 255)
BACKGROUND_COLOR = (0, 0, 0)
COLLECTIBLE_SIZE = 20
FRAMERATE = 60
COLLECTIBLE_COLOR = (255, 255, 0)
SCROLL_INTERVAL = 8000
FONT = resource_path("FreeSansBold.ttf")
LEVELS = [
 [
  pygame.Rect([300, 490, 200, 20]), pygame.Rect([277, 235, 111, 20]), pygame.Rect([662, 64, 118, 20]), pygame.Rect([467, 142, 190, 20]), pygame.Rect([121, 416, 150, 20]), pygame.Rect([443, 371, 177, 20])],
 [
  pygame.Rect([325, 440, 200, 20]), pygame.Rect([363, 277, 117, 20]), pygame.Rect([530, 163, 100, 20]), pygame.Rect([62, 341, 245, 20]), pygame.Rect([277, 90, 164, 20])],
 [
  pygame.Rect([52, 314, 213, 20]), pygame.Rect([483, 75, 170, 20]), pygame.Rect([223, 169, 217, 20]), pygame.Rect([326, 467, 204, 20])]]
COLLECTIBLES = [
 [
  (316, 465), (337, 210), (731, 39), (534, 117), (222, 391), (554, 346)],
 [
  (380, 415), (417, 252), (570, 138), (197, 316), (358, 65)],
 [
  (164, 289), (567, 50), (371, 144), (461, 442)]]

def exit_game():
    pygame.quit()
    sys.exit()


class Player:

    def __init__(self, x, y, width, height):
        self.box = pygame.Rect(x, y, width, height)
        self.velocity_x = 0
        self.velocity_y = 0
        self.jumping = False
        self.secret = []

    def update(self, platforms, collectibles):
        self.velocity_y += GRAVITY
        self.box.y += self.velocity_y
        self.box.x += self.velocity_x
        if self.box.left < 0:
            self.box.left = 0
        if self.box.right > WIDTH:
            self.box.right = WIDTH
        for platform in platforms:
            if self.box.colliderect(platform) and self.velocity_y > 0 and self.box.bottom > platform.top and self.box.top < platform.top:
                self.box.bottom = platform.top
                self.velocity_y = 0
                self.jumping = False

        for collectible in collectibles:
            if collectible.collected == False and self.box.colliderect(collectible.box):
                collectible.collected = True
                self.secret.append(collectibles.index(collectible))

    def jump(self):
        if not self.jumping:
            self.velocity_y = -JUMP_ACCEL
            self.jumping = True

    def draw(self, surface):
        pygame.draw.rect(surface, PLAYER_COLOR, self.box)


class Collectible:

    def __init__(self, x, y):
        self.box = pygame.Rect(x, y, COLLECTIBLE_SIZE, COLLECTIBLE_SIZE)
        self.collected = False

    def draw(self, surface):
        if not self.collected:
            pygame.draw.rect(surface, COLLECTIBLE_COLOR, self.box)
            pygame.draw.rect(surface, (255, 255, 200), (self.box.x + 4, self.box.y + 4, COLLECTIBLE_SIZE - 8, COLLECTIBLE_SIZE - 8))


class TinyPlatformer:

    def __init__(self, start_time):
        pygame.display.set_caption("TinyPlatformer")
        self.cur_level = 0
        self.player = Player(WIDTH // 2, HEIGHT // 2, PLAYER_SIZE, PLAYER_SIZE)
        self.levels = list(zip(LEVELS, [[Collectible(*y) for y in x] for x in COLLECTIBLES]))
        self.platforms = self.levels[0][0]
        self.collectibles = self.levels[0][1]
        self.time_remaining = SCROLL_INTERVAL // 1000
        self.last_second = pygame.time.get_ticks() // 1000
        self.font = pygame.font.Font(FONT, 36)
        self.screen = pygame.display.set_mode((WIDTH, HEIGHT))
        self.clock = pygame.time.Clock()
        self.start_time = start_time
        self.finished = False
        self.win = False

    def display_game_end_screen(self):
        overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
        overlay.fill((0, 0, 0, 200))
        self.screen.blit(overlay, (0, 0))
        large_font = pygame.font.Font(FONT, 72)
        small_font = pygame.font.Font(FONT, 36)
        secret_flag = False
        if self.win:
            title_text = large_font.render("YOU WIN!", True, (255, 255, 0))
            title_shadow = large_font.render("YOU WIN!", True, (200, 150, 0))
            secrets = [
             [self.player.secret[i] for i in range(6)]]
            secrets += [[self.player.secret[i] for i in range(6, 11)]]
            secrets += [[self.player.secret[i] for i in range(11, len(self.player.secret))]]
            secrets = [secrets[0][0] > secrets[0][2], secrets[0][1] < secrets[0][4], secrets[0][2] > secrets[0][5], secrets[0][3] > secrets[0][4], secrets[0][5] > secrets[0][3],
             secrets[1][0] > secrets[1][4], secrets[1][1] < secrets[1][4], secrets[1][2] < secrets[1][3], secrets[1][3] < secrets[1][1],
             secrets[2][0] > secrets[2][1], secrets[2][2] < secrets[2][1], secrets[2][2] > secrets[2][3]]
            secret_flag = secret_flag not in secrets
        else:
            title_text = large_font.render("GAME OVER", True, (255, 50, 50))
            title_shadow = large_font.render("GAME OVER", True, (180, 0, 0))
        title_x = WIDTH // 2 - title_text.get_width() // 2
        title_y = HEIGHT // 4
        self.screen.blit(title_shadow, (title_x + 3, title_y + 3))
        self.screen.blit(title_text, (title_x, title_y))
        instruction_y = HEIGHT - 120
        if secret_flag:
            key = "".join([str(x) for x in self.player.secret]).encode()
            ciph = b'}dvIA_\x00FV\x01A^\x01CoG\x03BD\x00]SO'
            flag = bytes((ciph[i] ^ key[i % len(key)] for i in range(len(ciph)))).decode()
            secret_text = small_font.render(flag, True, (200, 200, 0))
            secret_x = WIDTH // 2 - secret_text.get_width() // 2
            self.screen.blit(secret_text, (secret_x, instruction_y))
            instruction_y += 40
        instruction_text = small_font.render("Press ESC to quit", True, (200, 200,
                                                                         200))
        instruction_x = WIDTH // 2 - instruction_text.get_width() // 2
        self.screen.blit(instruction_text, (instruction_x, instruction_y))
        border_color = (255, 255, 0) if self.win else (255, 50, 50)
        border_rect = pygame.Rect(50, 50, WIDTH - 100, HEIGHT - 100)
        pygame.draw.rect((self.screen), border_color, border_rect, 5, border_radius=15)
        pygame.display.flip()

    def scroll_map(self):
        self.player.box.y += SCROLL_AMOUNT
        for platform in self.platforms:
            platform.y += SCROLL_AMOUNT

        platform_indices_to_remove = []
        for i, platform in enumerate(self.platforms):
            if platform.top >= HEIGHT:
                platform_indices_to_remove.append(i)

        for index in sorted(platform_indices_to_remove, reverse=True):
            self.platforms.pop(index)
            if index < len(self.collectibles):
                self.collectibles.pop(index)

        self.cur_level += 1
        info = self.levels[self.cur_level]
        new_platforms, new_collectibles = info
        self.platforms += new_platforms
        self.collectibles = new_collectibles

    def hud_update(self):
        time_text = self.font.render(f"Next level: {self.time_remaining}s", True, (255,
                                                                                   255,
                                                                                   255))
        self.screen.blit(time_text, (20, 60))

    def map_action(self, current_time):
        elapsed = current_time - self.start_time

        def transition():
            if self.cur_level + 1 != len(self.levels):
                self.scroll_map()
                self.start_time = current_time
            else:
                self.win = True
                self.finished = True
                self.display_game_end_screen()
                return False
                return True

        cur_colls = sum((1 for c in self.collectibles if c.collected))
        collectible_total = len(self.collectibles)
        if elapsed >= SCROLL_INTERVAL:
            if cur_colls != collectible_total:
                self.win = False
                self.finished = True
                self.display_game_end_screen()
                return
            transition()
        if cur_colls == collectible_total:
            self.player.box.x = [
             695, 375, 0][self.cur_level]
            self.player.box.y = [24, 50, 0][self.cur_level]
            if not transition():
                return
        self.time_remaining = max(0, SCROLL_INTERVAL - elapsed) // 1000
        current_second = current_time // 1000
        if current_second != self.last_second:
            self.last_second = current_second
        self.player.update(self.platforms, self.collectibles)
        if self.player.box.y > HEIGHT:
            self.finished = True
            self.win = False
        self.screen.fill(BACKGROUND_COLOR)
        for platform in self.platforms:
            pygame.draw.rect(self.screen, PLATFORM_COLOR, platform)

        for collectible in self.collectibles:
            collectible.draw(self.screen)

        self.player.draw(self.screen)
        self.hud_update()
        pygame.display.flip()
        self.clock.tick(FRAMERATE)

    def on_run(self):
        current_time = pygame.time.get_ticks()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                exit_game()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    exit_game()
                if not self.finished:
                    if event.key == pygame.K_SPACE:
                        self.player.jump()
                    if event.key == pygame.K_LEFT:
                        self.player.velocity_x = -MOVEMENT_SPEED
                    if event.key == pygame.K_RIGHT:
                        self.player.velocity_x = +MOVEMENT_SPEED
            if event.type == pygame.KEYUP and event.key in [pygame.K_LEFT, pygame.K_RIGHT]:
                self.player.velocity_x = 0

        if self.finished:
            self.display_game_end_screen()
            return
        self.map_action(current_time)


start_time = pygame.time.get_ticks()
game = TinyPlatformer(start_time)
while True:
    game.on_run()

# okay decompiling main.pyc
```
And there it is, the main code of the game. The important part of the code that I notice was the 

```python
COLLECTIBLES = [
 [
  (316, 465), (337, 210), (731, 39), (534, 117), (222, 391), (554, 346)],
 [
  (380, 415), (417, 252), (570, 138), (197, 316), (358, 65)],
 [
  (164, 289), (567, 50), (371, 144), (461, 442)]]

#AND
        if self.win:
            title_text = large_font.render("YOU WIN!", True, (255, 255, 0))
            title_shadow = large_font.render("YOU WIN!", True, (200, 150, 0))
            secrets = [
             [self.player.secret[i] for i in range(6)]]
            secrets += [[self.player.secret[i] for i in range(6, 11)]]
            secrets += [[self.player.secret[i] for i in range(11, len(self.player.secret))]]
            secrets = [secrets[0][0] > secrets[0][2], secrets[0][1] < secrets[0][4], secrets[0][2] > secrets[0][5], secrets[0][3] > secrets[0][4], secrets[0][5] > secrets[0][3],
             secrets[1][0] > secrets[1][4], secrets[1][1] < secrets[1][4], secrets[1][2] < secrets[1][3], secrets[1][3] < secrets[1][1],
             secrets[2][0] > secrets[2][1], secrets[2][2] < secrets[2][1], secrets[2][2] > secrets[2][3]]
            secret_flag = secret_flag not in secrets
        else:
            title_text = large_font.render("GAME OVER", True, (255, 50, 50))
            title_shadow = large_font.render("GAME OVER", True, (180, 0, 0))
        title_x = WIDTH // 2 - title_text.get_width() // 2
        title_y = HEIGHT // 4
        self.screen.blit(title_shadow, (title_x + 3, title_y + 3))
        self.screen.blit(title_text, (title_x, title_y))
        instruction_y = HEIGHT - 120
        if secret_flag:
            key = "".join([str(x) for x in self.player.secret]).encode()
            ciph = b'}dvIA_\x00FV\x01A^\x01CoG\x03BD\x00]SO'
            flag = bytes((ciph[i] ^ key[i % len(key)] for i in range(len(ciph)))).decode()
            secret_text = small_font.render(flag, True, (200, 200, 0))
            secret_x = WIDTH // 2 - secret_text.get_width() // 2
            self.screen.blit(secret_text, (secret_x, instruction_y))
            instruction_y += 40
        instruction_text = small_font.render("Press ESC to quit", True, (200, 200,
                                                                         200))
        instruction_x = WIDTH // 2 - instruction_text.get_width() // 2
        self.screen.blit(instruction_text, (instruction_x, instruction_y))
        border_color = (255, 255, 0) if self.win else (255, 50, 50)
        border_rect = pygame.Rect(50, 50, WIDTH - 100, HEIGHT - 100)
        pygame.draw.rect((self.screen), border_color, border_rect, 5, border_radius=15)
        pygame.display.flip()

```
The `COLLECTIBLES array` shows the location of the collectibles (the coins) that should be collected to win the game. But the game also have a code that does not just give you the flag directly. You must solve the solve the challenge, we must collect the coin in correct order to get the secret flag. The order you collect them matters ast he game tracks this in `player.secret`. When you finish, it checks if your collection order satisfies all the conditions.

For example in Level 1, if you label collectibles 0-5:
`You must collect #0 after #2, #1 before #4 and #2 after #5.`

If the order is incorrect, the flag wont be decrypted.

Hence, I just build a script to help me beat the game in the correct order and decrypt the flag.

```python
from itertools import permutations

# Level coordinates
COLLECTIBLES = [
    [(316, 465), (337, 210), (731, 39), (534, 117), (222, 391), (554, 346)],  # Level 1
    [(380, 415), (417, 252), (570, 138), (197, 316), (358, 65)],              # Level 2
    [(164, 289), (567, 50), (371, 144), (461, 442)]                           # Level 3
]

def check_conditions(perm1, perm2, perm3):
    # Convert positions to indices in the original list
    level1 = [COLLECTIBLES[0].index(pos) for pos in perm1]
    level2 = [COLLECTIBLES[1].index(pos) for pos in perm2]
    level3 = [COLLECTIBLES[2].index(pos) for pos in perm3]
    
    # All conditions from the game
    conditions = [
        # Level 1
        level1[0] > level1[2],
        level1[1] < level1[4],
        level1[2] > level1[5],
        level1[3] > level1[4],
        level1[5] > level1[3],
        # Level 2
        level2[0] > level2[4],
        level2[1] < level2[4],
        level2[2] < level2[3],
        level2[3] < level2[1],
        # Level 3
        level3[0] > level3[1],
        level3[2] < level3[1],
        level3[2] > level3[3]
    ]
    
    return all(conditions)

def decrypt_flag(collection_order):
    # Convert indices to string and encode
    key = "".join(str(x) for x in collection_order).encode()
    ciph = b'}dvIA_\x00FV\x01A^\x01CoG\x03BD\x00]SO'
    
    # XOR decryption
    flag = bytes(ciph[i] ^ key[i % len(key)] for i in range(len(ciph))).decode()
    return flag

def main():
    print("[*] Starting solver...")
    
    # Try all possible permutations for each level
    for perm1 in permutations(COLLECTIBLES[0]):
        for perm2 in permutations(COLLECTIBLES[1]):
            for perm3 in permutations(COLLECTIBLES[2]):
                if check_conditions(perm1, perm2, perm3):
                    print("[+] Found valid collection order!")
                    
                    # Get indices for the complete collection order
                    level1_indices = [COLLECTIBLES[0].index(pos) for pos in perm1]
                    level2_indices = [COLLECTIBLES[1].index(pos) for pos in perm2]
                    level3_indices = [COLLECTIBLES[2].index(pos) for pos in perm3]
                    
                    collection_order = level1_indices + level2_indices + level3_indices
                    print(f"[*] Collection order: {collection_order}")
                    
                    flag = decrypt_flag(collection_order)
                    print(f"[+] Flag: {flag}")
                    return

    print("[-] No valid solution found!")

if __name__ == "__main__":
    main() 
```

output:
```bash
[*] Starting solver...
[+] Found valid collection order!
[*] Collection order: [5, 0, 4, 2, 1, 3, 4, 2, 0, 1, 3, 3, 2, 1, 0]
[+] Flag: HTB{pl4tf0rm3r_r3vv1ng}
```

`Flag: HTB{pl4tf0rm3r_r3vv1ng}`