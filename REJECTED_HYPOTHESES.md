# Rejected hypotheses, and corrected findings

The companion of [FINDINGS.md](FINDINGS.md) (findings 256…374, which continue Ombelll's numbering,
and the notes N1…N80 of the reading of the PC and PlayStation executables). It takes the name and the form of
Ombelll's own file of the same name, for the same reason: history is not silently rewritten.

Two kinds of entry, kept apart on purpose.

**Corrected**: a finding that was written down, believed, and then falsified by a later one. Each
entry records four things: the old claim, why the evidence looked convincing at the time, the
falsifying evidence, and the corrected reading. In [FINDINGS.md](FINDINGS.md) the old row keeps its
number and ends with a pointer to the row that corrects it; here is the story in between.

**Rejected**: a hypothesis (one of these findings', a player's, or one the community has repeated for years) that was
tested and found false, often before it ever became a finding. Each entry says what the hypothesis
was, why it looked convincing, what falsified it, and what stands instead.

The last part, [Blind checks that missed](#blind-checks-that-missed), is a table of every prediction
that was counted on the whole disc and did not reach the pass mark written down for it, followed by
the checks that held, so that both sides can be seen.

References: findings by bare number (`341`), notes of the reading of the executable as `N41`,
Ombelll's findings as `O16`. Status words as in FINDINGS.md: `CODE` read in the executable, `GAME`
seen or measured in the running game, `DATA` read or counted on the level files, `HYPOTHESIS` not
tested. PC addresses are for `bugs.exe` version 1.0; PlayStation addresses for the USA disc,
`SLUS_008.38`. Lengths are in game units.

---

## REJECTED — finding 260: "the texture table is cumulative across files"

**The hypothesis.** 125 faces of *Hey... What's up, Dock? 1* (`L03A`) name 8 texture ids that the
level does not register. The game's table of texture slots is not cleared on a level change, so the
ids were taken to come from files loaded earlier: six from the common file of the dock levels, two
from the title file. "An extractor that looks at a single file cannot draw that level; load order
matters."

**Why it looked convincing.** It rested on a true reading of the code (there is no routine that
zeroes the table), and the count closed: 456 slots = 442 from the level, 10 from the common file, 4
from the title. Every one of the 8 ids was found in a companion file.

**The falsifying evidence.** The proof had no power: almost every file of the disc registers slots
3–7 and the pair used by Bugs's head (the eighth, 307, Merlin's eye, by `title` alone), so *any* companion file would have filled them. And on screen
the result was wrong: a barrel strap where Bugs's eyes should be, an olive fan where the sun's halo
should be (269). Finding 275 then counted the whole disc: 520 objects of type 2 or 20 carry opcode
`0x0B`, none of the 520 slots they name is registered by their own file, and together they cover 518
of the 545 ids that faces use without the file registering them (7,875 faces). The 27 left belong to
models no object places: drawn faces that name an empty slot are 0 of 277,588.

**What stands.** The 8 ids are **animated texture slots, filled every tick by objects of the same
level** (275): the sun's halo, the water, the ripples, Bugs's eyes (in every file his head uses 8
consecutive ids and the 5th and 7th are never registered), Merlin's eye. 269, which had asked where
the halo's textures come from, is closed by the same finding. That the table survives a level change
is still true in the code; nothing needs it to draw a level.

---

## CORRECTED — finding 275, two points: "type 20 plays its frames from first to last" and "a type 2 animator plays its frame list"

**Old claim.** A type 20 texture animator loops over the frames (first, last, duration) of its
opcode `0x42`; a type 2 one plays the (frame, duration) pairs of its sequence. Read on the data and
right on screen for the level it was made on.

**Why it looked convincing.** The three words of `0x42` look like first, last, duration, and for
most animators first…last is the whole resource, so the picture agreed.

**The falsifying evidence.** The handlers, read in the code (313; type 20 `0x445670`, type 2
`0x445750`). Type 20 **never reads "last"**: the frame starts at the first word, lasts the third
word's ticks, goes up by one and wraps to 0 after the last record of the frames resource. Cycles on
the disc are written from 0 or from 1, so a reader that trusts first…last never shows frame 0 of a
cycle written from 1: 47 of 129 animators would show other frames than the game. Type 2 is a state
machine like type 14's: it starts in state 2 if it has one, else 1, and plays the animation of that
state's first step; 137 of 391 have up to 14 states (Bugs's blinks and looks), and for 110 of 391
the sequence a naive reader picks is not the one the game starts with.

**Corrected reading.** 313. What 275 has right stays: the slot gets the texture id of the frame's
`0x64` record, type 2 takes (frame, duration) pairs, type 20 has no sequence.

---

## REJECTED, then settled the other way — finding 266: "semi-transparent `0x4A` / `0x4E` faces are drawn opaque"

**The hypothesis.** Modes `0x4A` and `0x4E` carry no UV, and the word where the UV modes keep their
blend mode falls among the colours. A reader that takes the blend mode only from UV modes draws a
semi-transparent `0x4A` / `0x4E` face opaque without saying so. Was that a fault?

**Why it looked convincing.** It was a real gap in the reader, and the sea of the dock levels is
made of exactly these faces.

**The falsifying evidence, first round (`DATA`).** In the three dock levels 0 of 4,899 such faces
have the semi-transparent bit: for them the question does not arise, and the hypothesis was rejected
as a fault of those levels. But the census of the playable levels found 30 such faces elsewhere, and
what the game does with them stayed open.

**What stands (`CODE`, 307 and 312).** On the PC the hypothesis describes **what the game itself
does**. The pass that marks which blend copies of a texture are needed marks the texture of a `0x4A`
/ `0x4E` face *opaque only*, overwriting, without looking at the face's flag; the blend byte of
these modes is at `+6`. Walking the disc as the code does: 20 semi-transparent faces of this kind in
the models (*Train your Brain! 3* (`L05A3C`) 12, *The Carrot Factory 1, 5, 2, 4* 2 each) and 234 in the terrain
of five files; none of their textures is blended by a UV face of the same level (check held), so
**the PC draws all of them opaque**. Not compared with the PlayStation.

---

## CORRECTED in three steps — texture coordinates: 262 "(size − 1)", then 328 "byte / 255, wrapped", then 341 "clamped to 0.01…0.99"

**Old claim, step one (262).** The pixel coordinate is `u / 255 × (size − 1)`, sampled at the texel
centre. Taken from a function Ombelll's documents had read in the executable.

**Why it looked convincing.** The function is real and was read right (`0x41CDD0` in version 1.0).

**The falsifying evidence, step one (328, `CODE`).** That function is called only by the **software
renderer**'s drawers (`0x4140B0`, `0x4143E0`). The OpenGL path, the one players use, takes the
coordinate from two tables (`0x467F50`, `0x468350`) as **byte / 255 over the whole texture**, with
`GL_REPEAT`, `GL_LINEAR`, no mipmaps. 328 concluded that at a coordinate of exactly 0 or 1 the
filter mixes the edge texel with the opposite edge: a half-texel line at the border of every sprite
and of every face that reaches 0 or 255.

**The falsifying evidence, step two (341, `CODE`).** Players do not see that line on the skies, and
the capability function was read again to its end. The first reading had stopped at the code of each
driver profile and **missed the last loop of the function**: at `0x4228F4`, unless the renderer is
the software one, every entry of both tables above 0.99 becomes 0.99 and every entry below 0.01
becomes 0.01 (constants at `0x45CA24`, `0x45CA28`). Also read again: a card whose vendor string is
"NVIDIA Corporation" gets profile 4, whose own clamp is [0, 1], that is nothing.

**Corrected reading.** On every OpenGL card the coordinate is **`clamp(byte / 255, 0.01, 0.99)`,
then repeat, linear, one level**; `ATI` / `RAGE PRO` keep their tighter [4.5 / 255, 250.5 / 255].
The opposite edge still mixes in by `max(0, 0.5 − 0.01 × N)` for a side of N texels: nothing from 50
texels up, 18% on 32, 34% on 16. The blind check of 341 missed (see the table below): for sides of
16 texels and under the reading predicts a faint mix that players do not report.

**And the v axis (271 corrects 262).** 262 counted v from the top of the image, as is usual. Four
defects that looked independent (lettering on the crates upside down, the 14 billboards of the sky
upside down, the white band missing on the ship, stray triangles at the stern) all went away with
`v' = 255 − v`, and a count that could have failed agrees: on the vertical faces of the terrain, v
from the top puts the texture upside down 696 times against 238 in *Hey... What's up, Dock? 1* (`L03A`) (748 against 245 in
*Nowhere* (`MERLIN`), 1,065 against 854 in *Wabbit on the run! 1* (`L01A`)). A slip on the way, kept here because it
delayed the answer: the lettering had first been reported as upright, from a crop in which the first
letter was cut off. Looked at whole, it was upside down.

---

## REJECTED — O16, by finding 268: "texture × vertex colour × 2" is wrong for the PC

**The hypothesis.** O16: the vertex colour is neutral at 128 and the product is doubled (the PlayStation
convention). Every renderer built on the documents did so.

**Why it looked convincing.** It is what the PlayStation does, and the data are the PlayStation's.

**The falsifying evidence (`GAME`, then `CODE`).** Against screenshots of the PC game in the same
framing, everything was about twice as bright: a salmon hull for a brown one, a bright blue sea for
a navy one. The clean sample is the sea, a solid-colour face: game (0, 0, 63), × 2 (0, 0, 125), × 1
**(0, 0, 63)**. 306 then read it: a colour byte is `byte / 255` (table `0x467B50`) times the texel,
**255 is neutral**.

**Corrected reading.** × 1 on the PC; "255 doubles" holds for the PlayStation only. The hull had
stayed 1.1 to 1.3 times brighter in the game than the × 1 rule gives: that residue was a second
thing, the gamma of 1 / 1.2 the PC applies to every texture at upload (306), measured afterwards on
the same screenshots (310: k = 0.856 against 0.833 read, 1.011 on the control).

---

## CORRECTED — finding 261: "the pose is the animation with the most transform records"

**Old claim.** Among an object's animations, the one to show at rest is the one with the most TRS
records; choosing by role number had picked a degenerate stream.

**Why it looked convincing.** It turned a heap of parts into a recognisable pirate.

**The falsifying evidence.** The normal carrot: the stream with the most records opens with the body
squashed to scale 0 (the carrot popping out of the ground), so it came out short and stuck in the
planks, while in the game it floats tilted (the object and the record with scale 0 were not noted
at the time; "floats tilted" is an observation from play, `GAME` without a measurement).

**Corrected reading.** 272: the game's own chain. A type 14 object starts in state 2, else 1; the
first slot of that state names a step, and the step carries the animation's role. The chain reaches
an animation the object owns for 73 of 77 rigged objects of *Hey... What's up, Dock? 1* (`L03A`), and Bugs starts with his hands
on his hips, as in the game.

---

## REJECTED — O15 and finding 282: "terrain sectors of mode `0x1000` are invisible walls"

**The hypothesis.** Ombelll's documents describe the 20-byte records of mode `0x1000` as invisible
boundary and collision faces. 282 measured them: 1,527 records in 26 levels, all vertical quads,
"the curtains". An option of the level viewer, "fake walls" (vertical faces one walks through), was built on the same idea.

**Why it looked convincing.** Every one of the 1,527 quads is vertical and stands in an opening.

**The falsifying evidence.** `GAME` (288): in *Follow the Red Pirate Road* (`L03D1`) one such quad
stands where the real invisible wall is an object's box (284), and the other stops nothing at all.
The "fake walls" option was withdrawn at the same time: it marked walls that clearly stop Bugs. `CODE`
(293): the terrain drawer's case for mode `0x10` (`0x410B06`) projects the quad, clips it against
the window and records the `u32` at `+16` as **an area that is visible this frame**.

**What stands.** They are **portals**: the last 4 bytes, which 282 had left unread, are the area
seen through the opening: the area of *another* chunk of the same level in 1,527 of 1,527 records
(null 133). The collision code never reads them. What stops Bugs is in 298–300 and 309.

---

## CORRECTED — finding 278 (and a guess of 275): "one animation block is one tick"

**Old claim.** Animation blocks are numbered 0, 1, 2… with no gaps, so a block is a tick; the
spinning carrot measured on the PlayStation with frame advance gives 15 "ticks" a second. 275 had
guessed 25 a second for the animated textures, labelled as a guess.

**Why it looked convincing.** The measurement was careful: 203 frames for three turns of a 17-block
animation.

**The falsifying evidence (`CODE`, 316).** The PC game runs **30 logic ticks a second**, every
handler once a tick. An animation's frame lasts **(word `+2` of the stream header) / 2 ticks**: on
the disc 8,309 streams at one tick a frame, 1,582 at two (15 frames a second, the rate measured on
the carrot), 50 at three.

**Corrected reading.** "One block per tick" holds for the first kind only; the measured 15 a second
was the carrot's frame rate, not the game's clock. 343 adds that one pass of the loop moves
everything once: there is no second clock.

---

## CORRECTED — finding 298: "no ceiling", and the prediction that followed it, "the flight ignores the ceiling"

**Old claim.** In the functions read for 298 (the wall sweep, the ground query, the landing) nothing
stops a mover from above: over the top of the highest collision slab Bugs is in no block, and
movement is free. It was marked open until the player handler was read; the test named was a jump
under the top of *Follow the Red Pirate Road* (`L03D1`) (y −6400).

**Why it looked convincing.** The three functions were read whole, and none has a ceiling.

**The falsifying evidence (`GAME`, N5).** A community tester jumped there with a Cheat Engine table:
Bugs was pushed back between y −5900 and −6100. Placed directly at y −7000 he fell back with no
jolt: no ceiling from outside. The code read afterwards (`0x437360`, 299): in the rising half of a
jump, when the top of Bugs's box would pass the top of the slab he is in, his height is clamped to
`top − box top`: about y −5990 there.

**The second mistake (N5 → N6; 302 corrects 299 on this point).** 299 wrote "only a jump is
clamped", and from it N5 predicted: "with the flight cheat the ceiling does not hold you". **Wrong,
and the tester had already seen it fail**: flying, he was held in the same band. The byte the flight
cheat holds is not an animation, it is Bugs's current *series* (`player+0x17C`); series 9 (and 7)
opens with a step whose bit `0x10` starts a jump, in 54 of the 54 level files whose player has series 9 (bar 95%,
null 12.8%: held). Holding the series restarts the jump as soon as it ends: **the flight is a
jump**, and the clamp applies.

**Corrected reading.** 299 and 302: the top of a slab stack is a ceiling for a jumping or flying
Bugs, about 410 units below it; put above it by other means he is simply in no block.

---

## CORRECTED — the list of death zones with a hole in them (299): "3 on the disc"

**Old claim.** An earlier count of death zones that hold a collision block of another area (where
Bugs would not die) found 3 on the whole disc.

**Why it looked convincing.** It applied the zone test of 290 as written, skipping the rotated zones
as too awkward to sample.

**The falsifying evidence.** A community tester pointed out that the zones "skipped as rotated"
nearly all carry the rotation (4096, 4096, 4096). In the code (`0x434D19`, N6) the zone test builds
its matrix with a routine that masks each angle with `0xFFF`: **a rotation of 4096 is no rotation**.
60 of the 61 death zones without flag bit 0 are of that kind, and the old list had also tested only
at the zone's own Y.

**Corrected reading (302).** Redone over the zone's whole Y span: **165 zones examined, 18 with
columns of another area, 13 of them whole columns**. The one death zone with a real angle, zone 4 of
*Train your Brain! 4* (`L05A4`), has an X extent of −31787 in a signed 16-bit test `0 <= x <=
extent`: it never holds anyone.

---

## CORRECTED — finding 303: "effect `0x1000` makes the object face its target at once", so "the orange `?` stands at the world origin"

**Old claim.** The orange `?` (five objects in three levels) is placed at (0, 0, 0), nothing in its
rules moves it, and its effect `0x1000` had been read as "face the target at once". Prediction: in
the game it stands at the world origin, 450 to 600 units up.

**Why it looked convincing.** A test of bit `0x1000` really is at `0x441B6F`, in code that turns an
object towards its target.

**The falsifying evidence (`CODE`, N7; `GAME`).** That test is in the turning code, where the word
being tested is **the step's control word**, not the rule's effect word: two structures, one offset.
In the rule's effect handling (`0x443DBB`) bit `0x1000` zeroes the object's position, rotation and
scale and makes it **a child of the player**. The `?` has the effect in the unconditional rule of
its starting step: on its first tick it attaches itself to Bugs, and its box (447 to 602 above its
origin, Bugs is about 410 tall) puts it just above his head. **Seen in the game: the `?` appears
above Bugs's head.**

**Corrected reading.** 304; the prediction of 303 is withdrawn. The blind check written for it
missed outright, for a reason worth keeping: most rules with the effect belong to *later* steps of
objects placed normally (things that go to Bugs when taken); see the table.

**The reusable shape.** The same offset in two records is two fields. Ombelll's file has this lesson
already (their "an offset is not a field"); it was walked into again.

---

## CORRECTED — finding 304 / N7: "the sky domes are authored in world coordinates: the origin is their right place"

**Old claim.** Counting every object placed at (0, 0, 0), 304 set aside the pieces with flags
`0x22000000` and no rules (the dome of *Hey... What's up, Dock? 1* (`L03A`) among them) as models authored in world coordinates.

**Why it looked convincing.** Type 9 objects had just been read as camera followers, these were type
14, and a dome of ±23,465 units centred on the origin does cover a whole level.

**The falsifying evidence.** The blind check P2 of 339 ("the type 14 objects at the origin that no
class explains have a world-coordinate pose") missed, 40 of 76 against 80%, and 21 of the 36 misses
were exactly these domes and backdrops. That miss is what sent the reading back to the end of the
type 14 handler (`0x4445AD`): with **bit `0x20000000`** of the second dword of opcode `0x16`, every
tick the object's position becomes the camera's (`0x4B38C0`…), the Y plus the word of opcode `0x1E`:
the same as type 9.

**Corrected reading (339).** The domes **follow the camera**; their placed position is overwritten
on the first tick. 36 placed type 14 objects have the bit and 11 of them are *not* at the origin in
the file (in *Era selector* (`LS01`), *When Sam met Bunny* (`L03B`), *Follow the Red Pirate Road* (`L03D1`), *The Conquest for
Planet X!*, *The Carrot-henge Mystery 1* (`L02C1`), the credits): a tool that trusts the file draws those
skies in the wrong place. How two such shells share the sky is 346.

---

## REJECTED — "a sky can be told by its size", and "by bit `0x02000000`"

**The hypothesis.** For a tool that must decide which objects travel with the camera: a sky is an
object larger than the level; or, second try, an object with bit `0x02000000` of the second dword of
opcode `0x16`, which every sky of *The Carrot-henge Mystery* has.

**Why it looked convincing.** Both rules gave the right sky in the first levels looked at.

**The falsifying evidence (`DATA`).** Size: *When Sam met Bunny* (`L03B`) has three objects larger
than the level itself and *Mine or mine? 3* (`L03C2`) two, and none is a sky (one is Yosemite Sam's
ship); dragged with the camera they cut across the scene as flat slabs. In *The Carrot-henge Mystery
3* the sky arrives as a **cloned template**, which a rule about placed objects never sees. Bit
`0x02000000` is carried by hundreds of small objects too; in the code (`0x44CD31`) it only caps the
draw distance ("backdrop").

**What stands.** The game's own rule (339, N39): **type 9, or type 14 with bit `0x20000000`**,
clones included: 97 object blocks on the disc. When two followers are on screen together the depth
buffer decides between them per pixel, not their order in the list (346).

---

## REJECTED — "the fireballs at (0, 0, 0)" of *What's cookin', Doc? 6*

**The hypothesis.** Seven objects of *What's cookin', Doc? 6* (`L02A6`) stand at the origin with
effect-like resources: misplaced fireballs, to be put where the game shows them.

**Why it looked convincing.** Objects piled on the origin with no sensible place there are usually
effects the game moves somewhere else, and an earlier survey of things out of place had these down as
fireballs.

**The falsifying evidence (`CODE` + `DATA`, 339).** They carry bit `0x40000000` of the second dword
of opcode `0x16`: with the pause flag set, only objects with that bit are run (`0x4488C8`). They are
**the pages of the pause menu**: each waits for level byte 42 to be its page number, shows texts,
reads the pad; one changes level (the quit entry). The same seven, with the same rule counts (56,
280, 37, 122, 124, 293, 33), are in every playable level (at the origin in 57 of the files; in *The Carrot-henge Mystery 1* (`L02C1`), *Train your Brain! 4* (`L05A4`), *The Planet X File! 3* (`L05A5`) and *The Conquest for Planet X!* (`L05B1`) the pages stand elsewhere). Their model is an empty model of 12 bytes:
they draw nothing. Check P1 held: 427 of 427 placed objects with the bit have no drawable model
(null 6.6%).

**What stands.** There are no fireballs at the origin. 399 of the objects placed at the origin on
the disc are these pages.

---

## REJECTED — "the crate pirate is misplaced: the game sets objects down on the ground when they are born"

**The hypothesis.** The pirate of *Hey... What's up, Dock? 1* (`L03A`) (object 108) is placed *inside* the pile of crates he
stands on in the game. So the game must put objects on the ground, or on the box below them, at
birth, and a level viewer must do the same.

**Why it looked convincing.** In the game he is on top of the pile, 591 units above his file
position; and many objects do stand exactly on the ground.

**The falsifying evidence (`CODE` for the handler `0x440290`, `DATA` for the pirate; 340).** Nothing
is done at birth. A step whose control word has bit `0x1` never moves its object at all (3,184 of
the 3,446 placed type 14 objects with a position start in such a step, in the air too); a step
without `0x1` and without `0x80000000` puts the object on the heightmap's ground **every tick**;
object boxes are never ground for another object. The pirate's steps are free in three dimensions.
He has **two animations**: he starts crouched at the file's height, hidden inside the crates, and
when Bugs comes near (bit 1 of level byte 122, set by a zone) he plays the same figure authored 591
units higher; he ducks back when Bugs leaves.

**What stands.** The file position is right; what changes is the state. Not tested in the game.

---

## CORRECTED — N10 / N7: "an object with bit `0x100` of `object+0x14` exists, runs, and is not drawn"

**Old claim.** A template without a model record gives an object with that bit and no model
instance: a full object that is simply invisible. N10 also took the children of the type 6 manager
for particles "by the look".

**Why it looked convincing.** The bit is set exactly where a model is found missing.

**The falsifying evidence (`CODE`, 311).** The live-list tick (`0x447EB0`) tests the bit twice: an
object with it **is not run** and is deleted right after its turn. **Bit `0x100` means "delete
me".** And the type 6 manager only calls the HUD's update: its 47 children are the HUD's slots
(314), with the template ids in the executable.

**Corrected reading.** 311, then 314 and 315 for the rest of how a clone ends. Two more corrections
belong here. 311 had missed that **a step can end its object** (control bit 4 when the animation
ends): the instruction takes `0x100` from a register and the scan for writers of the bit did not see
it. And 314 said a free clone is born at its spawner's position; 315: the attachment marker holds
for its frame only, and **a free clone is born at the marker's world position**. (314 also first
said that a spawning row fires "when an animation starts", corrected in the row itself: the
rows with bit 8 of their second flag byte fire at every frame, as O80 already had it.)

---

## CORRECTED — 319 / N18: the level taken for *Nowhere* (`MERLIN`) was another level

**Old claim.** 319 read "Nowhere's green crate", "the strength test of Nowhere", "level byte 59 of
Nowhere" and the apple in the file *Wabbit on the run! 1* (`L01A`).

**Why it looked convincing.** The file does hold a green crate, an apple and a tall thing with
health 10 that only Bugs's own blow moves: everything the glitches of Nowhere were said to involve.
The list that says which file is which level had not been read.

**The falsifying evidence.** The level table: **Nowhere is the file `MERLIN`**; `L01A` is *Wabbit on
the run! 1*. Pointed out at once by a community tester.

**Corrected reading (N19, 320).** Everything 319 says of those objects holds *for `L01A`*, and
object 130 of `L01A` is not the strength test. The code readings (the crate moves first and Bugs is
put back by what it could not do; two kinds of hit; type 8 is a swaying chain) are not touched.
Nowhere's real crate is `MERLIN` object 129 (Z only, one way), and its strength test is read piece
by piece in 320: **there is no force**; the indicator has two animations and the target asks *who*
hit it (the hammer: full rise, gong, golden carrot; anything else: the half rise). One more slip of
the same note: the crate's one-way bits were given the wrong way round; `0x100` lets it go only the
positive way, `0x200` only the negative way (`0x445E91`…).

**The reusable shape.** A level's name was written from memory, with the list that settles it (the
official names in the game's own texts, 291) at hand.

---

## CORRECTED — 329 and 331: condition `0x34` "tests a flag of Bugs", and the level exits are "gates"

**Old claim.** Condition `0x34` = Bugs's flag `+0x18 & 0x4`, "what sets it was not found"; the
objects that change level on it were called the levels' gates.

**Why it looked convincing.** The flag reading came straight off the condition's code: the handler for
`0x34` (`0x42B470`) does test bit `0x4` of Bugs's second flag word (`+0x18`), and no effect of the rules
writes that word, so its writer was outside the rules and was left open. "Gates" was the natural name
for a type 14 object standing at the end of a level that changes level when Bugs reaches it: the other
ways out of a level (the end of a cutscene, the pause menu's exit) are objects of the same kind, and a
closed exit that opens on a condition is what a gate is. What the condition waited for could not be
told without the player handler, which is where the bit is written every tick from Bugs's current step.

**The falsifying evidence (`CODE`, 334).** Every tick the player handler (`0x43AECC`…) copies bit
`0x2000` of Bugs's current step into that flag. Exactly one series of Bugs has the bit: series 30,
entered by the action button in the air, in 45 files: **the dive into a rabbit hole**.

**Corrected reading.** Condition `0x34` is "Bugs is diving", and the level exits of 329 are **rabbit
holes**. (Condition `0x33`, the twin on step bit `0x1000`, has 63 uses and can never pass: no step
of Bugs has that bit.)

---

## REJECTED — "each ability is a move of Bugs"

**The hypothesis.** The four abilities Merlin teaches (fan, music, super jump, open sesame) are four
moves of Bugs, so each must put him in series of its own. Written down as blind check B of N36.

**Why it looked convincing.** The super jump *is* a move.

**The falsifying evidence.** `CODE`: save byte 8 (`config+0x10048`; PlayStation `MainRAM 0x010048`)
is touched in two places only, the cheat function (`0x447DC3`, `0x447E12`) and the HUD (`0x44B413`); no handler and no rule of Bugs reads
bits 1, 4, 8, 16. `DATA`: the check missed: series 314 is common to all four and series 121 is
shared by the fan, the super jump and open sesame; only the super jump (116, 100) and the music
(118, 130, 149, 154) have series of their own.

**What stands (336).** An ability is **a gate in the rules of the 23 magic devices** of the levels;
the fan and open sesame are things the device does, not Bugs. Bit 2 of the byte is set by the "all
abilities" cheat only and read by nothing.

---

## CORRECTED — 335 / N35: the cheat function's gate, and cases 0 and 4 the other way round

**Old claim.** The cheat function (`0x447CF0`) "runs only while a byte of the state equals `0x30`";
its case 4 (save byte 8 `|= 0x80`) is the level select and case 0, which only marks save byte 36, is
the extra key.

**Why it looked convincing.** Case 0 "does nothing", which suits a key nobody could find, and a
first run on the PlayStation agreed: the published level-select sequence opened the levels. But the
two bytes were not written down before and after, so that run fitted both readings.

**The falsifying evidence.** `CODE` (N36, N50): the `0x30` is not a state, it is **two buttons of
the pad word `config+0x10010` held down** (the keys bound to triangle and circle); the function is
called every tick, in any level, paused or not. (The PlayStation's function, `0x80035244`, masks L2
and R1 out of the word it compares: holding them, as the published instructions say, is allowed and
not needed.) The seven entries of the table (`0x4AE130`) count from 0 to 6 in the binary order of
the published endings: **case 0 = level select = bit 1 of save byte 36**, read by 79 rules, all in
*Era selector* (`LS01`); **case 4 = extra key = bit `0x80` of save byte 8**, the key Bugs carries
(the HUD shows its icon, the locks of eight levels test it). `GAME`, on the PlayStation: **setting
bit 1 of `MainRAM 0x010064` by hand opens the rabbit holes of the Era selector, clearing it closes
them**: the level select, without the pad. They open at once, with their opening sound (heard
standing next to one while the value was changed).

**Corrected reading.** 336. The PC's sequences are eight presses of four buttons, not the
PlayStation's nine. Still a `HYPOTHESIS`, read in the PlayStation executable (table `0x8005E768`, N50)
and not run: after the six presses common to all seven sequences (cross, square, R2, L1, circle,
cross) the last three are square and **triangle** (`0x80` and `0x10` in the pad's layout, where the
circle of the opening itself is `0x20`), not square and circle as the cheat sites print them. Read as
a binary count with square = 0 and triangle = 1, first press most significant, the seven endings are:
level select square, square, square; 99 carrots square, square, triangle; all abilities square,
triangle, square; full health square, triangle, triangle; **extra key triangle, square, square**;
complete ending triangle, square, triangle; incomplete ending triangle, triangle, square. The level
select, all squares, cannot tell the two readings apart; the extra key is the first that can.

---

## CORRECTED — "save byte 0 is the lives"

**Old claim.** 318, 335 and the earlier lists of bytes called save byte 0 "lives" (3 in a new
game).

**Why it looked convincing, and where the name came from.** Not from the game: **the game has no
lives and no game over**, and players have never spoken of any. O102 named index **1** "lives" and
index 2 "their maximum": they are the health and its maximum (six half carrots). O102 left index 0,
3 at startup, unnamed. An early note of this reading (N17), writing about the damage routine, said
in passing "byte 0 is 3: the lives": a guess from the number 3 with nothing read behind it. Later
notes and lists repeated it.

**The falsifying evidence (`CODE`, 348).** `bugs.exe` accesses `config+0x10040` directly four times
and every one is "write 3": the startup (`0x406400`), two places of the save service (`0x40734C`,
`0x407430`), and a branch of the game loop (`0x406E8B`) that is dead, because no instruction writes
the value that selects it. No instruction reads the byte, adds to it or takes from it. The action
O102 quotes as "if (lives != max) lives++" is health. The PlayStation executable has one direct
access, the same startup (`0x80015EF0`). On the disc no rule writes index 0 and two files read it,
both in a rule whose neighbours name another index: there it works as **a constant 3**.

**Corrected reading.** "Always 3: set by the code, read by nothing in the code." A label in a
document is not a fact of the game. By hand (`MainRAM 0x010040`, `config+0x10040`): with 0, 1 or 2
in the byte the finishing line of *Downhill Duck!* (`LB02`) stops setting two bytes and one step of the cars
of *Objects in the mirror are closer than they appear! 1* stops hurting Bugs; dying never moves it.
Not run.

---

## REJECTED — four published GameShark codes that never did what they say

**The hypothesis.** The code lists online: `30010040 0006` and `30010042 0006` "infinite health",
`80010044 9A00` "infinite time", `30010047 00FF` "max golden carrots".

**Why it looked convincing.** They are published, and a code that writes 6 next to the health looks
like a health code.

**The falsifying evidence (`CODE`: the save-byte table is at `MainRAM 0x010040`, same indices as the
PC's; 335, 348).**

- `30010040 0006` writes 6 into **save byte 0**, which no code reads (the two level rules that test
  it, see the entry above, pass with 6 as they do with 3): it does nothing. Health is
  `MainRAM 0x010041`.
- `30010042 0006` writes 6 into save byte 2, the **largest** health, which is 6 already: nothing.
- `80010044 9A00` is a 16-bit write that puts `0x9A` into `MainRAM 0x010045`, save byte 5, and `0x00` into `0x010044`, save byte 4: **the
  clocks**, 154 of them. There is no timer there.
- `30010047 00FF` is **half a code**: the golden carrots are one 16-bit number, low byte in save
  byte 7 and high byte in save byte 252 (`MainRAM 0x01013C`). It gives 255 and leaves the high byte
  as it was; the game's own cheat writes `0x014D` = 333.

**What stands.** Health `0x010041`, carrots `0x010043` (capped at 99 by the HUD's code), clocks
`0x010045` (the game's cheat writes 124), golden carrots `0x010047` + `0x01013C`, abilities and key
`0x010048`. Read from the addresses; the four codes were not run one by one.

---

## REJECTED — "the `_8` files are the demos that start by themselves"

**The hypothesis.** Nine files of the PC disc end in `_8` and are not in the level table (281, which
took them for another revision that the game never loads by LevID). A player's theory: they are the
demos the game plays by itself at the title.

**Why it looked convincing.** The level table does name five `DEMO` files that the PC disc does not
have, and the `_8` files are almost, but not quite, their twins.

**The falsifying evidence (`CODE` + `DATA`, 344).** In the level loader (`0x42EAD3`), when the
renderer word `0x4AC094` is 0 (**the 8-bit software renderer**) and the LevID is one of nine, the
file name gets `_8` before its extension. The nine LevIDs, with `_8`, are exactly the nine files: 9
of 9, none left over (held; the level table had not been read before the bar was set). The engine
does have a demo player, a recorded input stream in block `0x47`, but no file of the PC disc carries
such a block and the title file has one level change only, the menu's own.

**What stands.** They are **the levels of "software, 256 colours"**: same collision byte for byte in
all nine pairs, small differences in objects, textures and one terrain. **The twist (N50):** on the
PlayStation disc the idle demo *does* exist. The main loop (`0x80016870`…) counts passes at the
title and at 901 (30 seconds) starts a level with the demo switch on; the disc has five `DEMO`
files. So the theory was right about the game and wrong about these files.

---

## REJECTED — "the missing golden carrot of *Magic Hare Blower 1* (`L01D1`) is at (27285, −3987, 9482)"

**The hypothesis.** Players know that with the low graphics settings one golden carrot of *Magic
Hare Blower 1* (`L01D1`) is not there, and 100% cannot be done. A position was given for it.

**Why it looked convincing.** The players' account is sound: with the 256-colour setting one golden
carrot of the level is not there and 100% cannot be reached, so a carrot missing from one rendering of
the same level was to be expected somewhere. The position came from a picture of the game taken
where the carrot should be, and it lies in the level's playable space, a plausible spot for a
collectible. With the `_8` files still read as demos and the level believed to be one file for every
renderer, a missing pick-up could only be an object hidden or removed by a rule, which sends the
search to the objects placed near a given point rather than to a block-by-block comparison of two
files.

**The falsifying evidence.** No object of either file stands there: that position was **the
camera's**, read off a photograph of the game (judged from the photograph's viewpoint against the
level's geometry; no camera log was taken). Comparing the two files block by block (344), the
8-bit file lacks exactly one object block: **object 26, a golden carrot at (25910, −1800, 8190)**,
next to Bugs's start (25600, −1600, 8120).

**What stands (`CODE` + `DATA`; the carrot's identity confirmed by a community tester).** No rule
removes it and nothing hides it: the 8-bit renderer loads another file, and that file does not have
the block.

---

## REJECTED — the torch's glow: four leads, a falling blue, and a driver profile

Three hypotheses about one sprite, the glow of the torches of *Hey... What's up, Dock? 1* (`L03A`), which a renderer built on the
files drew as a faint haze while a frame of a speedrun video showed it far stronger.

**The four leads (338), each followed in the code: all four are a no.** *Drawn more than once*: one
record per sprite instance per frame, one quad; in the data each torch has one glow (the "two
sprites" are two templates for two families of torches). *A second, larger copy*: one record of the
level uses the glow's texture and no face does (5,515 faces). *A vertex colour other than neutral*:
the OpenGL callback writes 1.0 into all sixteen colour floats. *Another conversion for blend mode
1*: read again, alpha = m − m / 4 with every channel pushed up by 255 − m, the gamma on the colour
only. What the reading did find is elsewhere: the glow is **centred** on its point by bit
`0x8000000` of the second dword of the object's opcode `0x16` (not in the sprite record, which is
why it had not been found); drawn resting on its point it sits 256 units too high.

**"In the video the blue falls 30 pixels from the middle" (the blue channel from 91 to 80).** No blend the code has can
lower the blue there. 338 supposed the video's compression. `GAME` (345): on a lossless screenshot
of the game on OpenGL the samples along a line through the flame are (84, 75, 94), (66, 57, 93),
(34, 31, 83), (6, 6, 67), (0, 0, 63): **the blue never falls.** The little extra blue at middle
distance is the mode 1 conversion itself, which whitens the dim texels of the rim.

**"The video was recorded on the `Direct3D` driver profile."** A community tester's hypothesis, and
a fair one: it is the one profile with another blend, `glBlendFunc(GL_ONE, GL_SRC_COLOR)`, screen =
texel + screen × texel. **Examined against the numbers, it does not fit:** with the texels pushed to
full brightness the result can never be under the texel, so 120 pixels out the prediction is about
(255, 227, 240) where the frame shows (73, 78, 119), and the blue can only rise. Dropped; and after
the lossless screenshot it has nothing left to explain.

**What stands.** One layer, white, the mode 1 conversion, centred. The check of 345 missed by little
(largest channel error 12 against 8) on the quad's middle row and comes within 2 on all twelve
values with the row the samples were really taken on, an observation made after the run.

---

## REJECTED — the type 8 record with flag `0xB`: "back to the rest pose", and "left where the last animation put it"

**The hypothesis.** 280 had read on the data that this animation record removes a part until a TRS
record puts it back. Two cases did not seem to fit one reading: the time machine of *Era selector* (`LS01`),
where the removal looks right, and **the knight of** *What's cookin', Doc? 6* (`L02A6`), whose walk "removes
the five helmet parts and never puts them back", while players never see him without his helmet.
Four readings were on the table: (a) hidden until a TRS record, (b) back to the rest pose, (c) left
where the animation before put it, (d) something else.

**The falsifying evidence (`CODE`, 347; the frame applier `0x44E5A0`, case at `0x44E86E`).** Flag
nibble `0xB` zeroes the part's scale and sets bit `0x8000` of its flag word; the renderer's front
end (`0x40A250`) gives such a part a visible byte of 0; a TRS record begins by clearing the bit,
whatever its flags. **Not (b)**: nothing is taken from a rig or a rest pose; a rig has no TRS
records at all. **Not (c) for the part named**, though (c) is what happens to a part an animation
does *not* name: parts outlive the animation and keep their last state.

**Why the knight seemed a counter-example.** His helmet is in the rig **twice**: the helmet and
visor meshes are linked by rig parts 7 and 8 (on the head) and again by parts 25 and 26 (a loose
helmet, for when he is knocked into it). The walk hides the loose one and drives the head's all
along. **The game draws parts, not meshes**; a tool that keeps one transform per mesh lets the part
linked last win, the hidden one, and the helmet goes.

**What stands.** Reading (a), read in the code. Both blind checks held: the first TRS after a
`0xB` record carries the scale in 2,371 of 2,371; where a stream leaves hidden a part of a mesh
linked twice, another part of that mesh is shown in 56 of 56. Only 4 rigs of 2,426 link a mesh more
than once.

---

## REJECTED — the magic door skip: "the rudder leaves the physics while the camera turns"

The skip is in *Follow the Red Pirate Road* (`L03D1`), in the corridor that runs east to the chest
room between two walls. The door is a boulder, object 37 at (20440, −1499, 4745): a hard box that
stands across the corridor's whole width and stops Bugs; nothing else closes the way. The two rudders
are things Bugs carries and puts down in front of it (one at (20140, −1488, 4201), the other put down
at (20156, −1488, 4079), against the south wall), and the skip is the run through the boulder with
the rudders on the ground beside him.

**The hypothesis.** A player's explanation of the skip of *Follow the Red Pirate Road*
, built on 294 and 297: an object culled off screen is dropped from the chain of solid
objects, so turning the camera away from a carried rudder makes it stop colliding for a moment.

**Why it looked convincing.** 294 and 297 are right: objects culled without the keep-running bits
are neither run nor solid, and the skip does involve camera work.

**The falsifying evidence (`CODE` + `DATA`, 322).** The rudders' culling word is `0xA2`: with bit
`0x80` they never leave the chain; and an object is immune to the screen test until another test has
culled it once, which a fresh clone has not been. The boulder is not terrain but a **hard** object
box (flags `0x20380`).

**What stands (`CODE` + `GAME`, 357).** The boulder is a hard box thin in x (±70) and long in z
(±850) whose push-out throws a mover along its LONG axis when the mover is in that sector (from the
box's centre, |Δz| > 4.5 · |Δx|): a Bugs near the corridor's south wall is thrown south, by whatever
it takes, never west; the Z-only re-sweep meets the `0x7F` wall at its first sample and returns 0,
the X-only re-sweep is the run itself, and X is never corrected: x advances 40 a tick and z stays at
4079. The line is x ≥ 20294 at the south wall (20306 at the north), 74 and 86 past the grown face;
a recording of the skip replayed tick by tick from x 20294 on comes out to the unit, and four failed
recordings never reached the line. The rudders' only job is to carry Bugs past the boulder's west
face: they are soft boxes averaged with the hard one's delta (322), and one swing that ends with him
east of the line at the wall hands him to the −Z sector for good. The "half throw added through the
door" reading of 322 is corrected by the same finding: a soft box does not add through the door, it
moves Bugs into the region where the hard box no longer pushes back. The camera has no part (no
camera word in the push-out or the sweeps; its swing in the recording is the effect of the run).
Seven of the eleven long thin hard boxes of the disc are open to the same run by the game's own routines,
untested in the game (357, N62).

---

## REJECTED — "the zone without a condition where Bugs starts in *Hey... What's up, Dock? 1* is a teleport"

**The hypothesis.** Zone 12 of `L03A` has a teleport rule with condition 0 (always), to (30370,
−1200, 6346). Read from the 32-byte record as the documents describe it, it fires the moment Bugs is
in the zone: and Bugs starts in it.

**The falsifying evidence (`CODE`, 337).** The record's first dword, which no reader of the files
took, is **the id of the object the rule is for** (1 = Bugs), compared with the object's 16-bit id
before the condition is even looked at. This rule is for id **100000**: no 16-bit id can equal it.
*Hey... What's up, Dock? 2* (`L03A2`) has the twin. On the disc: 1,847 zone rules for Bugs, 548 for another
object, 2 for an id over 65535.

**What stands.** The two rules never fire, which is what is seen in the game: by their look a
designer's shortcut across the level, switched off by an id out of reach.

---

## CORRECTED — 299: "the zip rises 70 units a tick"

**Old claim.** 299 read the vertical resolve for a mover inside a sub-cell whose ground is
above him: he is lifted toward it at 70 units a tick, and the zips of the palisade and of the trunk
of *Nowhere* were taken to be that lift, spread over several ticks.

**Why it looked convincing.** 70 a tick is what the code does for a mover in the air (the fall is
capped at 70, `0x4370B1`), and the videos of the zips show Bugs rising fast rather than appearing.

**The falsifying evidence (`CODE` + `GAME`, 355, 356).** A recording of the palisade of the *Era
selector* (`LS01`) shows Bugs at (2783, −17240, 18991), then at (2800, −17240, 19000), then at
Y −18300: 1060 up in one tick, X and Z still; the six recordings of the trunk clip of *Nowhere*
(`MERLIN`) show 1,000 in one tick the same way. In the code the vertical resolve on the ground
(`0x437472`…: mode 2, no jump) is `dy = ground − y` whole: a Bugs standing in a sub-cell whose
ground is above him is put on it at once.

**What stands.** 70 a tick is the rate in the air (a jump into such a sub-cell); on the ground the
snap is the whole step in one tick. What 299 said of the hole itself stands: the entry into the
high sub-cell is the thing to explain, and 354, 355, 356 and 357 explain four ways in, each
reproduced to the unit by the game's own code run on a recorded tick.

---

## CORRECTED — 319 / N18: "the trunk clip is the pushed crate's carry correction"

**Old claim.** 319 (a) and N18 (a): the trunk clip of *Nowhere* (`MERLIN`) with the green
crate is the push code's correction: after the crate's own tick Bugs is moved by (what the crate
did − what was asked) with no wall test (`0x4460B5`), so a crate held against the trunk pushes him
back through anything, one refused move a tick, until he is inside the trunk's sub-cell.

**Why it looked convincing.** The correction exists and is unswept; the clip needs a crate and, by
what players said, "a certain speed"; the crate of *Nowhere* is pushable along Z, toward the trunk.

**The falsifying evidence (`CODE` + `GAME`, 356).** Six recordings of the clip: in the tick of the
entry the crate is not pushed at all, Bugs stands inside its box (a crate he had pushed is his child
while he pushes and the box test skips it; once released, the test finds him deep inside) and the
pending push is 0. The entry is the box push-out (`0x433670`) throwing him out through the face
nearest to him, by whatever it takes in one tick (dx = −202), followed by the two per-axis re-sweeps
from the same start (`0x43D061`: X only; `0x43D0B8`: Z only), whose sum (14119, 12890) lands in a
corner cell neither sweep entered. The game's own push-out and sweeps, run on the recorded tick,
give that landing for all six; and the clip of the recordings is made with the small carried cube put
down beside the trunk, not with the pushed crate.

**What stands.** The push handler of 319 (the crate moves first, Bugs is put back by what it could
not do; the PC / PlayStation difference) is type 5's and is not touched. The crate in the clip is a
wall Bugs is inside, not a thing he pushes; any solid box works, in any of the four orientations of a
convex corner, and the box push-out plus the per-axis re-sweeps are the mechanism (356).

---

## REJECTED — "the camera and the speed cause the zips"

**The hypothesis.** The community's, and for a while this work's: the zips of the palisade and of
the drawbridges need a camera flip, or a certain speed (a run, a roll), or both; a camera turned the
right way at the right moment "lets Bugs through".

**Why it looked convincing.** The recordings of the zips show camera flips and fast moves; a camera
flip does change what is solid (322: an object culled from the camera's view leaves the ordering
chain); and a cheat that let Bugs through walls "almost everywhere" seemed to say the walls are soft.

**The falsifying evidence (`CODE` + `GAME`, 354, 355).** Neither the wall sweep (`0x434E40`) nor the
detach reads a camera word: the camera enters only as the direction the stick maps to, so a flip
changes where Bugs walks, not what stops him. A move of any length is sampled at every sub-cell line:
a longer move meets more samples, never fewer; the hole of the palisade is the endpoint on the corner
(648 of the 1,600 corner-ending moves of any length), the hole of the drawbridge is the direction of
the vector from the parent's origin (a band of a few units of cross offset), and the trunk clip needs
a walk of one tick's length, 46 or 58 alike.

**What stands.** Speed and camera decide where the endpoint falls and where the stick points; the
holes are geometric (354, 355, 356, 357), and each is reproduced to the unit by the game's own code
run with no camera in it.

---

## CORRECTED — N50: "the PlayStation executable uses no `$gp`"

**Old claim.** N50: `SLUS_008.38` has no `gp`: every global is reached with `lui` + offset, so
the readers and writers of an address can all be listed by a search for absolute loads and stores.

**Why it looked convincing.** The functions read first (the level change, the cheat function, the
startup) reach their globals that way, and the list of writers found for the level-change word was
exactly the PC's four.

**The falsifying evidence (`CODE`, 351, 361).** The entry sets `$gp` at `0x80014A88` to
`0x80069320`; the camera's hold word is `$gp + 0xD6` = MainRAM `0x0693F6` and the held angle
`$gp + 0xE2` = `0x069402`: globals reached through `$gp`, which a search for absolute stores does not
see.

**What stands.** The mapping of N50 stands function by function, and every address it gives was read
with its instructions; but a list of "every reader of an address" on the PlayStation is complete only
when the `$gp`-relative accesses are counted too.

---

## REJECTED — "the stopping distance at a wall differs between the X and Z axes"

**The hypothesis.** From a first look at three recordings against `0x7F` walls: along X Bugs's origin
parks 40 short of the wall's line, along Z 16…28: two rules, one per axis, or two copies of the sweep
sampling differently.

**Why it looked convincing.** The sweep does have three copies chosen by the move (X-only, Z-only,
two-axis: `0x434F8F`), and the recorded numbers were what they were: 6679 against a line at 6640,
8440 against 8480, and 6732, 6744 against 6760.

**The falsifying evidence (`CODE` + `GAME`, 358).** The one-axis copies sample the same way on both
axes (the line going +, the line − 1 going −, then one lookahead sample beyond the endpoint): a walk
along either axis parks the origin at the line − 40 going + and the line + 39 going −, whatever the
speed, and from there every further move along the axis is cut to 0. The game's own sweep run on
the three levels gives the recorded 6679 and 8440 to the unit and 6720 for the Z wall; the recorded
6732 and 6744 came on ticks whose move had an x component (the position had just been rewritten from outside: Bugs's height was being held fixed with Cheat Engine while measuring),
where the two-axis copy walks into the last free sub-cell:
(29, 12) → 6732, then 6744, and (0, 20) → 0.

**What stands.** One rule, 40 short along any axis (39 going −); with any cross component the origin
can reach the line itself (going −) or the line − 1 (going +), never the wall's sub-cell.

---

## REJECTED — the gate of the King's Square: "a zero-thick collision sheet, and a sampling hole through it"

**The hypothesis.** The closed gate 133 of *What's cookin', Doc? 3* (`L02A2`) has a script box of
449 × 667 × 0: a sheet on the line z = 7955 with no thickness. The PlayStation clip through the
gate (a weight put down against it, a second one carried through a jump and put down, then a
walk) was read as a sampling hole in that sheet, of the exact-corner family of 355: a move that
ends on the sheet's own line is never tested against it, and Bugs at z 7945 is already a hundred
units past where the closed gate stops a walk (the parking at z 7845, his front against the gate's
near face at 7885), the "wall" a player meets.

**Why it looked convincing.** The script's box is what the level file states for the object, and
a zero-thick box is exactly the kind of geometry the sweep's sampling can miss; the recording does
show Bugs standing at 7945, a hundred past the parking and ten units short of the sheet's line
z = 7955, and walking through from there.

**The falsifying evidence (`DATA` + `CODE`, 363).** The closed state's animation carries a type 9
record (O123), and the box the game puts in force is that one: (−177, 125, −70)…(595, −855, 132)
in the gate's frame, 772 wide, 980 tall, 202 deep, hard (`0x20100`), world z 7885…8087. A Bugs at
7945 is *inside* it, not past a sheet. With the `0x7F` post column beside the gate (x 5200…5239)
the gate is a magic door of 357's family: the box push-out throws along the box's long axis from
z ≥ 7960 at x 5279, the post wall cuts that throw to nothing in the X-only re-sweep, and the Z-only
re-sweep carries the walk through. The game's own routines, run on the level's data with the
recorded positions, give the recorded parking (7843…7845) and the hover against the first weight
(7885…7889) to the unit.

**What stands.** The clip is the long-hard-box mechanism, not a sampling hole; its line on the PC
is z ≥ 7960 at x 5279, 115 past where the gate parks Bugs; the two forward pushes the recording
shows (+39 in the air, +58 on the release of the second weight) are not produced by the PC's code
with the weights where the PC's put-down leaves them, so the PC's route as read stops 30 short of
the line, and the PlayStation's put-down, not read, is where the difference must lie (363).

---

## CORRECTED — 344: "the 8-bit files lack 12 and 14 texture animators"

**Old claim.** 344 read the `_8` files of *Hey... What's up, Dock? 1* and *3* (`L03A`, `L03ACOM`) as
lacking 12 and 14 texture animators (type 2): those textures would stand still under the 8-bit
renderer.

**Why it looked convincing.** The first comparison matched the objects of a pair by their resource
counts, and the `_8` files carry other resource numbers for nearly every object (their texture tables
are re-made and renumbered); the animators, which carry a texture slot instead of a position, matched
worst of all, so twelve and fourteen of them came out "missing".

**The falsifying evidence (`DATA`, 371).** Matched by their whole script (states, steps, rules, flags,
position, culling: everything but the resource numbers), then by kind, type, id and position, then by
kind and type in file order, the pairs come out 282 / 281 and 122 / 121: **one animator is missing in
each file**; the other eleven and thirteen are there with other resource numbers.

**What stands.** The rest of 344: the swap by the renderer, the nine files, the missing golden carrot
of *Magic Hare Blower 1* (`L01D1`), the moved pair of *Mine or mine? 1* (`L03C`), the collision identical.

---

## CORRECTED — 331: "action `0x30` adds one to health, up to save byte 2"

**Old claim.** 331's table of actions read `0x30` as "health += 1 up to save byte 2": a heal.

**Why it looked convincing.** The handler's first branch is exactly that: with save byte 1 (health)
under save byte 2 (its maximum) it adds one to health, and the carrots that use the action do heal a
hurt Bugs.

**The falsifying evidence (`CODE`, 370).** The handler `0x42D1A0` has a second branch: **when health
is already at its maximum, it adds one to the save byte named by the action's index** — index 3, the
carrot counter. That is how the carrots are counted: no rule of the disc adds to byte 3 directly (335
looked for one and, finding none, wrote that the code counts them without the rules), and the writers
of byte 3 in the code are only the cheat and the HUD's cap.

**What stands.** The heal, when Bugs is hurt; 335's five counters and their bytes.

---

## CORRECTED — small slips of names and addresses, each put right by a later note

- **`0x431A60` is the camera's update, not the player's** (N2 → N4); so `0x431510` is the camera's
  collision with object boxes. The player's handler is `0x43A9F0`.
- **The fourth argument of `0x4334F0` is a mask of flags to exclude**, not an object (298 → 300).
- **`0x411BA0` is the drawer of one model part**, not the frame flush with the blend function, which
  is `0x412540` (305 → 306, N9).
- **The type 6 handler is `0x447600`**; `0x4473F0` is type 10's (N10 → N12).
- **Type 5**, taken for a carrier with a path, **is the pushable crate** (317, 319); the carriers
  are in 324.
- The HUD's icon-id table is at `0x4AE27C` (N36); a first run of its check had read it two words
  too early and had no power, since those templates are in every level.
- **The PlayStation's camera flags word was first written `0x0699A8`**: an arithmetic slip on the
  block's base (`0x06D928` + `0x80`); it is MainRAM `0x06D9A8` (350, 361).
- **The rabbit-hole gates 19 and 21 of the *Era selector* (`LS01`) were first judged closed to the long-box
  skip**, from a walk of the collision data beside the boxes; the sweep from the box's centre, as the
  game makes it, finds their walls inside the boxes' reach: open, 0…7 past the grown face (357,
  N62). The judgement on the walked data was a reading, never a finding.
- **Save byte 159 on the PlayStation was first placed at `0x069592`** (save byte N = `0x0694F3` + N,
  derived from a word of 349 that is a countdown's ticks word, not a save byte); the save bytes are
  at MainRAM `0x010040` + N (348, N50), so byte 159 is `0x0100DF`, where the save service's twin reads and
  writes it: the "no reader" of the first reading was the search at the wrong address (361).

---

## Blind checks that missed

**The method.** Wherever a reading of the code predicts something countable on the level files, the
prediction and its pass mark were written down *before* the count was run on the whole disc, with a
null (the same count on a population the reading says nothing about) where one could be had; in the
later checks the levels the reading had been made on are left out of the count. A miss is reported
as a miss, **and the bar is never moved afterwards**: a second form written after a run is labelled
as such and is not a blind test any more. Four outcomes: *held*; *missed*; *missed by little*; *held
without power* (the bar was reached, but the null reaches it too, so the check says nothing).

A reading of the code can stand when the check built on it missed. The instructions say what they
say; what a check tests is a *consequence* the reader expected to see in the data, and the
expectation can be the weak part: a premise about how the designers or their exporter worked ("a
camera follower is placed at the origin"), or a bar set on the wrong population (the cutscenes have
a player and no HUD). Where that is the case the last column says so. It does not turn the miss into
a pass.

| finding | the prediction | pass mark | result | what was learnt |
|---|---|---|---|---|
| 293 (N1) | the centre of a portal A→B falls inside a quad of the portal B→A | 80% | **missed**: 963 of 1,527 = 63.1% (null 223). A first form, twin centres within 64 units, failed outright: 257 of 1,527 | an opening is tiled with several quads, differently on its two sides; 91 portals have no portal back. The claim rests on the first prediction (the number is another chunk's area: 1,527 of 1,527) and on the code |
| 293 (N1) | among the other chunks, the one nearest to the quad is the area it names | 80% | **missed by little**: 1,123 of 1,424 = 78.9% (null 313; the 103 portals of levels with a single other chunk are left out, that chunk being the nearest by force) | the same |
| 298 (N4) | the wall rule stops a mover walking through the drawn walls, and far more often than one walking along them | 70%, and 3 × the null | **both missed**: 6,258 of 23,655 = 26.5%, null 10.2%. Second form, written after the run (only faces that cross the body of whoever stands beside them): 2,971 of 3,970 = 74.8%, null 25.4%, ratio 2.9: the 3 × bar **missed by little** | the rule is associated with the drawn walls and no more can be said from the files: it rests on the code, which agrees with O116. The 999 faces that stop nobody are candidates for walls one walks through, not verified |
| 301 (N6) | sub-cells with a positive height byte (slides) have a neighbour 20 units higher or lower at least 3 times as often as the others | 3 × | **missed**: 18.6% against 9.4%, twice | a poor premise: a slide is slippery, not necessarily steep. Unplanned: positive bytes exist in 8 level files only, the same eight that carry cell flag `0x2000` (O114) |
| 304 (N7) | placed type 9 objects are at the origin; the others are not | 80%; under 10% | **both missed by little**: 39 of 50 = 78.0% against 540 of 4,838 = 11.2% | the 11 others stand on round points (section origins); the null is raised by the pause menu's pages. A premise on the exporter, not on the code |
| 304 (N7) | placed type 14 objects with an effect `0x1000` rule are at the origin | 80%; under 15% for the others | **missed outright**: 18 of 421 = 4.3% against 508 of 4,165 = 12.2% | wrong premise: the effect acts when its rule fires, and most such rules belong to later steps of objects placed normally (things that go to Bugs when taken) |
| 304 (N7) | second form, after the run: the same for an unconditional rule of the starting step | (not blind) | 15 of 29 at the origin (null 11.2%). Seen afterwards: 28 of 29 on the origin or on a point whose coordinates are multiples of 320 (null 12.3%) | the placed position of such an object is a don't-care. The reading itself was then seen in the game (the `?` above Bugs's head) |
| 307 (N9) | double-sided faces (flag bit `0x02`) are wound at least 10 points less consistently than single-sided ones | 10 points under 78.9% | **missed**: 74.9% (19,559 of 26,122), 4 points lower | double-sided faces are wound as carefully as the others; the reading of the bit rests on the code |
| 308 (N10) | the box of the load script (opcodes `0x1B`, `0x1C`) equals the first box of the starting pose | 70%, and 3 × the null | **missed**: 2,050 of 4,735 = 43.3%, null 27.5% | the script's box is data of its own (in 184 of 238 disagreeing objects looked at, it is in no animation of the object): valid until an animation's box record plays |
| 308 (N10) | the fourth word of `0x1C` is within the box's horizontal radius | 80%, and 2 × the null | 3,995 of 4,035 = 99.0%: first bar held, **the ratio missed** (null 66.1%) | boxes of one level are alike: little power |
| 309 (N11) | no placed player starts with the ground more than 100 units above him | none written: the row is here because the check had no power, not because it missed a bar | 60 of 60 (the 79 placed players, one per file, less the 19 whose start sub-cell has no ground, `0x7E`), **held without power**: the null is 96% | it says nothing (the twin check, no start inside a `0x7F` sub-cell, held: see below) |
| 311 (N11) | clone templates carry their own death (actions `0x01`, `0x02`, `0x18`, effect `0x2000000`) far more often than placed objects | 50%, and 2 × the placed share | first form **missed outright**: 0 of 1,501 and 0 of 4,124. Second form, after the run (effect `0x10000` with third parameter not 1): 1,030 of 1,501 = 68.6% held; ratio 1.78 against 2 **missed by little** | the premise was wrong, not the reading of the bit: those actions exist in the engine and no level uses them. Placed objects die this way too (what is collected) |
| 313 (N12) | a type 2 animator's starting step plays one of its own frame sequences | 95%, and 2 × the null | 391 of 391, but the null is 93.1%: **second bar missed, no power** | roles are numbers shared by a level's objects |
| 314 (N12) | templates have a death of their own (a step with control bit 4, or effect `0x10000`) | 85% | **missed**: 1,574 of 2,221 = 70.9% (78.1% counting those another rule deletes by id, seen after the run) | the rest die with their parent, by a handler, or last until the level ends |
| 314 (N12) | every file that places a player holds the ten HUD digit templates | 90% | **missed**: 61 of 79 = 77.2% (null 0) | the wrong population: the 18 without are the cutscenes, the credits and the title, which place a player and have no HUD; among the levels 61 of 61 |
| 315 (N13) | a known end for the type 14 and type 2 templates | 90% | **missed**: 1,759 of 2,221 = 79.2% | the wrong population again: 294 of the 462 left are the HUD's digits, re-made in place and never pool clones. Without the HUD ids 91.3%, an observation made after the run; the bar stays missed |
| 315 (N13) | the two new ways of dying explain at least half of what 314 left out | 50% | **missed outright**: 25 of 487 = 5.1% | — |
| 317 (N16) | a walking step's target word names an id placed in the same file | 90% | **missed by little**: 131 of 150 = 87.3% (null 3.1%) | which steps name an id that is not placed (clones?) was not looked into |
| 317 (N16) | a patrol is a loop | 90% | **missed by one object**: 17 of 19 = 89.5% | — |
| 317 (N16) | a walker is placed within 2,000 units of its first waypoint | 80% | **missed**: 12 of 19 = 63.2% (null 7.7%) | — |
| 326 (N27) | a teleport's destination is near a terrain vertex of its file | 90% | **missed**: 24 of 33 = 72.7% (null 6.1%) | the bar asked for the wrong thing: the nine misses are three places where Bugs is put on objects, not on terrain |
| 336 (N36) | check B: the four abilities put Bugs in four different sets of series | all four | **missed**: series 314 is common to all, 121 is shared by three | "each ability is a move of Bugs" was wrong: see the entry above |
| 337 (N37) | every zone rule id other than Bugs's (and the two dead ones) is the id of an object or template of its file | 90% | **missed by little**: 359 of 401 = 89.5% | ids 6, 59, 195, 999 belong to nobody in their files; 6 is treated apart by the zone runner |
| 338 (N38) | centred sprites (bit `0x8000000`) are blended at least twice as often as the others | 2 ×, and 60% | **missed**: 58 of 108 = 53.7% against 446 of 1,185 = 37.6% | a poor premise (many centred sprites are opaque cut-outs: sparkles); the reading of the instruction does not rest on it. Seen after the run: 108 of 1,293 sprite objects have the bit against 22 of 8,922 others |
| 339 (N39) | P2: the type 14 objects at the origin that no class explains have a starting pose in world coordinates | 80%, null under 10% | **missed**: 40 of 76 = 52.6% (null 2.5%) | the useful miss: 21 of the 36 were the sky domes, and looking at why led to bit `0x20000000` |
| 339 (N39) | P3: a camera follower is placed at the origin | 80%, null under 10% | **missed**: 25 of 36 = 69.4% (null 2.5%) | a poor premise on the exporter, which left 11 of them elsewhere; the reading is the code's |
| 339 (N39) | P4: the origin is inside a camera follower's model | 80% | 26 of 28 = 92.9%, **held without power**: the others give 98.6% | it says nothing |
| 340 (N40) | a type 14 object whose starting step stands on the ground has collision ground below it | 95% | **missed by little**: 150 of 158 = 94.9% | the 8 without were not looked at. The rest of that survey cannot fail and says so; the test that can is in the game |
| 341 (N41) | with the clamp, under 10% of the textured faces still mix more than 10% of the opposite edge at a corner | under 10% (null 51.8% without the clamp) | **missed**: 100,613 of 323,157 = 31.1% | a third of the disc's faces sit on 32-texel textures, where the corner still mixes 18% over 0.18 of a texel: a fraction of a screen pixel. For sides of 16 and under (8.6% of the faces) the reading predicts a faint mix players do not report: too small to see, or something still unread |
| 345 (N45) | one quad, the glow's texture converted as the code does, blended over the sea colour, matches a lossless screenshot along the quad's middle row | largest channel error 8 | **missed by little**: 12 (red 78 against 66, green 65 against 57 at 40 pixels); blue within 2 at all four distances | after the run, with two numbers fitted against twelve: with the row 10 to 13% of the quad's height off its middle all twelve values come within 2; the samples had been taken through the flame, which rises from the glow's centre |
| 346 (N46) | by the depth rule, the camera followers wider than 10,000 units that can be on screen together each show over at least 5% of the sphere | 90% | **missed**: 13 of 16 = 81.2%. The contrast, painted in list order: 9 of 16 = 56.2% (the prediction had power) | a poor premise on size: the three misses are 14 to 20 blended triangles each, a planet and a sun, not shells, shown wherever they are met (0.4 to 0.7% of the sphere). The bar stays missed |

### The checks that held

| finding | the prediction | the numbers |
|---|---|---|
| 293 | the last 4 bytes of a mode `0x1000` record are the area of another chunk of the same level | 1,527 of 1,527 (null 133) |
| 294 | culling bit 1 goes with a draw distance above 0; an object with the area bit lies inside the plan of its area's chunk | 5,218 of 5,227 (null 1,906 of 6,270); 1,897 of 1,961 (null 409) |
| 300 | an object of solid class 2 has a box; the solid mask `0x9A690A` meets the first word of opcode `0x16` | 4,814 of 4,888 (bar 90%, others 0 of 520); 3,642 of 4,888 (bar 50%, others 0 of 520) |
| 302 | the flight's series (9) opens with a step that starts a jump | 54 of 54 level files (bar 95%, null 12.8%) |
| 307 | walking a model part as the code does lands on its record count; one double-sided bit per batch; single-sided faces wound one way; no `0x4A` / `0x4E` texture is blended by a UV face | 12,899 of 12,899; 24,466 of 24,466; 78.9% outwards (bar 75%, null 50%); held |
| 309 | no placed player starts inside a `0x7F` sub-cell | 0 of 79 (every placed player of the disc, one per file, all inside a collision block; bar at most 1%; `0x7F` is 9.9% of the sub-cells of those blocks) |
| 310 | the PC's texture gamma shows in the channel ratios of the hull of *Hey... What's up, Dock? 1* (`L03A`) on screenshots of the game | k = 0.856 (window 0.77…0.90; 0.833 read); control without gamma 1.011 |
| 312 | the terrain walks as the code says: entry counts, one mode per batch, headers, one double-sided bit per batch | 205 of 205; 168,963 of 168,963; 8,268 of 8,268; 15,911 of 15,911 |
| 314 | a step with control bit 4 is far more common in templates than in placed objects; an object with a HUD id is a template | 30.9% against 4.8%, ratio 6.39 (bar 2); 1,378 of 1,378 |
| 326 | the arrival picked through the save byte is within 3,000 units of a zone that leads back to the old level | 18 of 21 = 85.7% (bar 80%; null under half) |
| 336 | a level-byte test on a gate's way has a writer in its own file; files that use a carried bit hold its HUD icon's template | 265 of 266 = 99.6% (bar 90%, null 13.9%); 10 of 10 (bar 90%, nulls 5.0%, 2.5%, 0.4%) |
| 337 | no object of the disc has an id over 65535 (every object block of every file: the id is a 16-bit word, opcode `0x27`); at most 5 zone rules for such an id, each next to a rule for Bugs | held; 2, both so (little power for the second half) |
| 339 | P1: a placed object with the pause bit has no drawable model | 427 of 427 (bar 95%, null 6.6%) |
| 342 | a role is one kind of animation across the disc: its steps share one play mode | 84.9% on average over 60 roles and 10,981 steps (bars 75% and null + 15; null 43.5%) |
| 344 | the nine LevIDs the loader renames for the 8-bit renderer are the nine `_8` files | 9 of 9, none left over |
| 347 | the first TRS record after a `0xB` record carries the scale; a mesh linked by two rig parts stays shown through the other | 2,371 of 2,371 (bar 99%, null 8.48%); 56 of 56 (bar 80%) |
| 354 | the game's own detach sweep, run on the recorded tick from the drawbridge's pivot (6500, 11000) with the recorded vector (−1558, +6.6), lands where the recording put Bugs | (6439, 11000), to the unit; the three normal detaches of the same recording come out free |
| 355 | the game's sweep from the recorded (2783, 18991) with the recorded move (17, 9) reaches the corner (2800, 19000) unsampled, and it is the only move of that tick's size that does; the ground snap is the whole step | (2800, 19000), to the unit; the one move of its size; a lift of 1060 in one tick; 648 of 1,600 corner-ending moves pass, 0 of 150 in each other orientation |
| 356 | the push-out and the two per-axis re-sweeps, run on the recorded tick with the cube's box at the logged place, give the landing of the recordings | (14119, 12890) within the sector's margin (the logged cube 3 units off the diagonal); six recordings of six, x = 14119, z 12884…12902 |
| 357 | the recorded run through the boulder, replayed tick by tick from x 20294 on, gives the recorded positions; the failed recordings stay short of the line | to the unit from 20294 on (x + 40 a tick, z 4079); the four failures 21…125 short of the line and never across it |
| 358 | the game's sweep run with the recorded moves gives the three recorded stops; the `0x7F` wall reaches the top of its block and is tested with Bugs's Y | 6679, 8440 and 6720, to the unit (6732 and 6744 of the third recording only with the x component its log shows); the height rule 4 of 4 in the game (stops at 1 and 5 above the top, passes at the top and 5 below) |
