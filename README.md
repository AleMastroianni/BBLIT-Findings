# Bugs Bunny: Lost in Time — findings

What is known about how *Bugs Bunny: Lost in Time* (Behaviour Interactive / Infogrames, 1999)
works inside: the level files, the objects and their rules, the collision, the camera, how the PC
version draws against the PlayStation one, the save data and the cheats, with memory addresses
for both platforms. It is written for speedrunners, TAS makers, modders, anyone who wants to build tools
for the game, and anyone who is simply curious about how it works.

It continues the work published by **Ombelll** in
[Bugs-bunny-lost-in-time-reverse-engineered](https://github.com/Ombelll/Bugs-bunny-lost-in-time-reverse-engineered)
(findings 1 to 255 there) **and keeps their form and their numbering**: one long numbered list,
from 256 on. Everything here was found while building the
[BBLIT Viewer](https://github.com/AleMastroianni/BBLIT-Viewer), a level viewer for the game, and
while reading the PC executable to make the viewer draw and behave as the game does.

| file | what it is |
|---|---|
| [FINDINGS.md](FINDINGS.md) | findings **256 to 374**, in order, each with its status, the addresses for PC and PlayStation where there are any, and the test that could have failed; at the end, **Notes from the executable**: what was read in the PC and PlayStation executables and never became a numbered finding |
| [REJECTED_HYPOTHESES.md](REJECTED_HYPOTHESES.md) | what was believed and turned out wrong: the old claim, why it looked convincing, what disproved it, what stands; and the blind checks that missed |

Nothing of the game is in this repository: no level file, no executable, no texture. You need
your own copy of the game to check anything written here. Where a finding was compared with the running game,
the pictures taken for it are not published either.

## Addresses

**What they are.** An address is the place in memory where the game keeps a value while it runs:
Bugs's position, his health, the number of clocks, the level to load next. These pages give the
address next to almost every finding.

**What they are for.** So that you can **repeat a test yourself** and see whether a finding is
true, instead of taking it on trust: open the game, attach the tool to it (in Cheat Engine: the running `bugs.exe`), look at the address, change the value, watch
what happens. On the PC the tool is [Cheat Engine](https://www.cheatengine.org/); on the
PlayStation it is the **RAM Watch** (and the Hex Editor) of the
[BizHawk](https://tasvideos.org/BizHawk) emulator. Nothing here needs a modified game, except where a finding says so.

**Which version they hold for.** Only these two:

| platform | version | how to recognise it |
|---|---|---|
| PC | `bugs.exe` **version 1.0**, the same build [BugsDecomp](https://github.com/quantumdude836/BugsDecomp) calls 1.0 | 772,096 bytes, SHA-256 beginning `74ab71e1` |
| PlayStation | the **North American release** | its executable is `SLUS_008.38` |

On any other build (another PC executable, the European PlayStation discs) the
values exist but sit somewhere else: the addresses of this document will point at the wrong
thing. Ombelll's documents describe another PC build: finding 264 says how the two differ.

**How to read them.**

- **PC: `bugs.exe+offset`.** Type it into Cheat Engine exactly like that ("Add Address Manually":
  `bugs.exe+B39BC`). The executable is loaded at `0x400000`, so `bugs.exe+0xB39BC` and the
  absolute `0x4B39BC` you will meet in the rows about the code are the same place. Some values
  are reached through a pointer: `config` is the 4-byte pointer stored at `bugs.exe+0x12FD00`, and
  "`config+0x10041`" means "read that pointer, add `0x10041`" (in Cheat Engine: tick *Pointer*,
  base `bugs.exe+12FD00`, offset `10041`).
- **PlayStation: an offset in `MainRAM`.** In BizHawk choose the memory domain **MainRAM** and
  type the offset as it is written here without the `0x`, for instance `069662` (BizHawk reads it as hexadecimal). The console's own address of
  the same byte is that offset plus `0x80000000` (`0x80069662`): the rows about the code use that
  form, the rows about testing use the MainRAM one.
- **The size matters.** Each address comes with its size (1, 2 or 4 bytes) and whether it is
  signed. Writing 4 bytes where the game keeps 2 overwrites its neighbour.

**An example: warping to any level.** The game changes level when two values are set: the number
of the level to load (its *LevID*, the same numbers on both platforms) and a small "change level
now" word that the main loop looks at on every tick.

| | PC (`bugs.exe` 1.0) | PlayStation (USA) |
|---|---|---|
| the level to load (LevID) | `config+0x10000`, 4 bytes | MainRAM `0x010000`, 4 bytes (the same offset on both) |
| "change level now" | **`bugs.exe+B39BC`**, 2 bytes: write **1** | MainRAM **`0x069662`**, 2 bytes: write **1** |
| what happens | the level named by the LevID loads at once | according to the code, the word counts up from 1 to 22 while the picture fades, then the level loads |
| status | **CODE + GAME** | **CODE**: found in the executable as the exact twin of the PC's pair, **not observed in the game** |

Write the LevID first, then the 1, and only while the word reads 0. More in findings 326 and 353 (PC) and
N50 (PlayStation).

## How to read a finding

Each row of [FINDINGS.md](FINDINGS.md) has a number, the finding (a headline that states the result,
then its **status**, then the detail) and **the test that could have failed**.

| status | meaning |
|---|---|
| `CODE` | read in the executable: the program's own instructions were read step by step, the addresses are given. It says what the program does, not that somebody watched it happen |
| `GAME` | seen or measured in the running game (PC, or PlayStation in an emulator) |
| `DATA` | read or counted on the level files of the PC disc, usually over all of them |
| `HYPOTHESIS` | a reading that fits what is known and has not been tested; where possible with the test that would prove it wrong |

A finding can carry two (`CODE + GAME` is the strongest), or a different one for each of its
parts. A finding that was later corrected **keeps its row and ends with a pointer** ("Corrected by
341"), so that nobody takes the old fact home; the story of each correction is in
[REJECTED_HYPOTHESES.md](REJECTED_HYPOTHESES.md).

**Blind checks.** Many findings were put through a check on the whole disc: a prediction written
down, with its pass mark, *before* counting. The result is given as it came out: *held*, *missed*,
*missed by little*, *held without power*. A missed check is reported as missed, with what was learnt
from it; the bar is never moved afterwards. The *bar* is the pass mark set before the count; the *null* is
what the same count gives by chance (a random or shuffled comparison); *held without power* means passed,
but on too few cases, or against a null too close to the bar, to mean much.

**References.** A bare number is a finding (1 to 255 are Ombelll's, in their repository; a number written
with an O in front, as O102, is one of theirs; 256 on are here). **N1 … N80** are the working notes of the reading of the executables; ten of them, the ones that never became a finding (N2, N6, N8, N18, N23, N49, N50, N62, N68, N69), are at the end of FINDINGS.md, and a table there says which findings hold the content of the others.

**Conventions.** Positions are 32-bit integers in the game's own units; **Y grows downwards**.
Angles are 12-bit: 4096 is a full turn. The game runs **30 logic ticks a second**. Levels are called
by their name in the game with the file's name in brackets, for instance *Hey... What's up, Dock?
1* (`L03A`).

## Looking for a topic?

The list is in the order the findings were made. By theme:

| topic | findings |
|---|---|
| level files: container, load script opcodes, models and faces, terrain, textures | 256–259, 261, 263, 265, 271, 276, 277, 282, 289, 307, 308, 312, 337 |
| objects: types, steps, states, rules, every condition and action; two objects with one id; how a rule fires and what a clone's life is; what is true at a level's first tick | 272, 274, 308, 311, 314, 315, 321, 329, 331, 334, 357, 366, 367 |
| animation: rigs, parts, records, roles, timing, animated textures | 261, 265, 275, 278–280, 313, 316, 342, 343, 347 |
| what places an object (camera, Bugs, markers), skies, snow, the pause menu; every attachment of one object to another, and the node it hangs from | 267, 295, 304, 316, 339, 340, 346, 369 |
| collision: blocks, ground, walls, ceilings, slides, object boxes; the wall sweep and where Bugs's centre stops against a wall; the upper floor of the King's Square | 283, 284, 286, 287, 292, 298–303, 309, 322, 323, 358, 363, 364 |
| movement: characters that walk, platforms and lifts, see-saws, pushing, carrying and putting down, vehicles, what hurts Bugs | 317–320, 324, 325, 330, 340, 356, 365 |
| Bugs's series (his numbered states: idle, run, jump…): their numbers, their names, the jump-start bit | 302, 325, 362 |
| zones, death zones, teleports, restart points, level changes; the timed levels and their countdowns | 285, 290, 302, 326, 329, 337, 349 |
| **warp** to any level from memory | 326, 353 (PC), N50 (PlayStation) |
| visibility: portals, areas, culling, draw distance, what stops running unseen; what a camera far from Bugs changes | 293–297, 306, 351 |
| the camera: orbit, flips, field of view, the first-person view, the hold countdown, its words in memory | 322, 327, 333, 350, 351, 361, N49 |
| how the PC draws: order of a frame, depth, blending, gamma, sprites, the torch, blend mode 2, textures uploaded once per level | 268, 273, 305–307, 310, 312, 338, 345, 346, 353, N68, N69 |
| texture coordinates | 262, 271, 328, 341 (341 corrects 328) |
| the 8-bit software renderer's own level files (the same levels exported for 256 colours), the missing golden carrot, the 8-bit terrain of *The Greatest Escape 3* (`L04B3`) | 281, 344, 371, 373 |
| save bytes and level bytes: counters, abilities, the key, the switches of a level, the timers, the tables at a level's start | 316, 331, 333, 335, 336, 348, 349, 367, 370 |
| collectibles: how a carrot, a clock or a golden carrot is counted and remembered; what takes one (Bugs's box test, or a distance rule); the ones placed where Bugs cannot walk | 335, 370, 374 |
| *Nowhere* (`MERLIN`): the helper wizards one at a time, the objects kept at the origin, what its first tick makes | 367, 368, 369 |
| **cheats**: the built-in function, its real buttons on PC and PlayStation, the GameShark codes | 335, 336, N50 |
| **memory addresses for tables and tools**: the timers, the first-person ban, the camera's hold and its six words, the video options, the texture words, the random generator, Bugs's heading; the PlayStation's camera block and scratchpad | 349–353, 361, N49, N50 |
| the PC's settings and files: `config.pc`, the Video and Audio pages, the keys, the save slots and their two folders; the music that goes silent below volume 50 and the three places that set it; the PlayStation has no volume setting | 352, 359, 360, 372 |
| PC against PlayStation | 305, 306, 319, 320, 344, 346, 348, 354, 355, 361, 363, 372, N50, N68 |
| glitches: the pole of the *Era selector* (`LS01`), the zips (the detach, the exact corner, the trunk clip), the magic door skip and the long boxes, the gate of the King's Square and its upper floor, camera flips, every zip point of the disc, the collectibles Bugs can take under the floor when his position is written in memory | 294, 319, 320, 322, 354–357, 363–365, 374, N2, N6, N18, N23, N62 |
| texts, level and area names | 291, 332 |
| which PC build, and how the two builds differ | 264, N8 |

## Community

The *Bugs Bunny: Lost in Time* speedrun community is on Discord:
**https://discord.gg/PThM9ucHmu**. Many of these findings started from
its members' observations.

## Credits

- **Ombelll**, whose reverse engineering of the game
  ([repository](https://github.com/Ombelll/Bugs-bunny-lost-in-time-reverse-engineered)) is the
  ground all of this stands on: the file formats, the load script, the first 255 findings.
- **quantumdude836**, for [BugsDecomp](https://github.com/quantumdude836/BugsDecomp), the
  decompilation project of the PC executable: its function table identified the build studied here and named
  the OpenGL imports.

The viewer: <https://github.com/AleMastroianni/BBLIT-Viewer>

Corrections are welcome: open an issue, or say it on the Discord. A finding that is wrong is worth
more disproved than left standing.
