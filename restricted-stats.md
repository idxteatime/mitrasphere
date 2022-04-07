## Tea's Restricted Stats Research ##

**Restricted Stages** are stages where all players (and enemies) have their stats scaled to a certain common baseline.

![File Apr 06, 18 52 08](https://user-images.githubusercontent.com/10483639/162104407-0d49060e-12f5-479c-ba3f-bc0b90c34bca.jpeg)

While the JP Altema site defines a formula for how stats are calculated in restricted stages, people have found that it doesn't quite seem to line up with what they see in-game, and the nature of it makes it difficult to figure out optimal restricted stage sets. So here we have compiled a series of in-game testing that we've done to really examine the details of restricted stage stat scaling and its formula, and use that to outline some practical ways to approach building restricted content sets.

### tl;dr ###

- the formula:
- this means there's no easy way to use in-game stat summaries to estimate your restricted stats
- instead, you can look at common stat distribution patterns to estimate the impact of certain items on your final stats
- main grid items dominate the final stat distribution; support grid should have items with stat distributions that are focused on whatever primary stat you want to increase

### the formula ###

It's important to note first that in restricted content, the only numbers that matter are the numbers on your main and support equipment. That means none of the following have any impact on your restricted stats:

- job stat bonuses
- essence of battle
- fishing leaves
- rank-up bonuses
- advanced support items

This is one of the reasons why it's difficult to estimate your restricted stats from your stat summary, as all of the above impact your stats and combat power - fortunately, there is a column in the stat summaries that total up only the stats coming from your equipment, which will help us with our testing

![File Apr 06, 19 21 52](https://user-images.githubusercontent.com/10483639/162107647-dbd654da-c766-47b4-8609-ba11f8a8fb1f.jpeg)


[Altema JP](https://altema.jp/mitrasphere/sentoucap) tells us that the formula for each stat on a given item in a restricted stage is as follows:

```
restricted stat = restricted power * lvl 1 item stat / item power

where item power = HP * 2 / 3 + PATK + PDEF + MATK + MDEF
```

Let's take an example:

![File Apr 06, 20 38 35](https://user-images.githubusercontent.com/10483639/162115728-250a2018-9a83-4501-89d7-7c3ee0b3c6c2.jpeg)

Using the above formula, we first calculate the item power: `32 * 2 / 3 + 19 + 23 + 6 + 21 = 90.33`

We then calculate the respective stat percentage and restricted stats in a 1000 power stage:

| stat | base value | percentage | restricted value |
|---|---|---|---|
| HP | 32 | 32\*2/3/90.33=0.236 | 1000\*0.236=236 |
| PATK | 19 | 19/90.33=0.21 | 1000\*0.236=210 |
| PDEF | 23 | 23/90.33=0.255 | 1000\*0.236=255 |
| MATK | 6 | 6/90.33=0.066 | 1000\*0.236=66 |
| MDEF | 21 | 21/90.33=0.232 | 1000\*0.236=232 |

Let's check these numbers in-game, at least the easy one to check which is HP. First we remove all equipment except this one and make sure the stats reflected in the stat summary is the same as the item stats:

![File Apr 06, 20 39 18](https://user-images.githubusercontent.com/10483639/162117333-cf7ef441-3c2c-4dea-b817-23aee510db45.jpeg)

Then we jump into a 1000 power restricted stage:

![File Apr 06, 20 40 18](https://user-images.githubusercontent.com/10483639/162117392-aa6c87a4-68da-4ed8-bd04-c22411f112d1.jpeg)

Seems we are off by quite a bit - in fact, we are actually off just about 1.5 times: `355/236 = 1.504`

As it turns out, in particular for HP we have to reverse the 2/3 scaling and multiply by 3/2 during the final stat calculation (which I found out through a lot of trial and error). Let's try another item to verify these calculations:

![File Apr 06, 21 06 52](https://user-images.githubusercontent.com/10483639/162118153-0878c91a-951c-4b0b-a91b-97e8429c4643.jpeg)

We get an item power of 89.67, so we plug that in:

| stat | base value | percentage | restricted value |
|---|---|---|---|
| HP | 22 | 22\*2/3/89.67=0.163 | 1000\*0.163\*3/2=245 |
| PATK | 15 | 0.167 | 167 |
| PDEF | 20 | 0.223 | 223 |
| MATK | 21 | 0.234 | 234 |
| MDEF | 19 | 0.212 | 212 |

Let's check in-game:

![File Apr 06, 21 13 29](https://user-images.githubusercontent.com/10483639/162118766-cc42d211-b62f-4290-8d24-f14d652c611f.jpeg)

Dang, not quite, but close. We're rounding a bit in our calcuations, so we can imagine that maybe the game is doing a different kind of rounding that accounts for the difference. Or maybe they're not rounding at all, and doing something entirely different. Trying various kinds of decimal-to-integer operations at various points in the formula until we get the right number would be quite a hassle, but luckily the game gives us some hints that narrows down the approach:

- all stat numbers in the game are displayed as integers, so we can assume that the game is likely storing them as integers as well, not decimals
- likewise, combat power is always an integer, though per-item power is never displayed explicitly
- when equipping items in the support slot, the calculation for the stat contribution seems to drop the decimals after scaling it by 20% - e.g. an item with 24 HP will give 4 HP in the support slot
- mitra is an old game and has not really changed in a while, so its code is also likely very old - for computers, the `floor` operation, or dropping decimals, is significantly simpler and faster than rounding, especially in older programming languages

These hints tell us that the game is likely using `floor` to drop decimal points whenever it calculates stats or power. Let's give that a try on our previous item:

Instead of using 89.67, we use 89 as the item power, and we also use `floor` on the final stats (for brevity we will also drop the percentage to 3 decimals, though the game likely uses the full decimal value):

| stat | base value | percentage | restricted value |
|---|---|---|---|
| HP | 22 | 22\*2/3/89=0.165 | floor(1000\*0.165\*3/2)=247 |
| PATK | 15 | 0.168 | 168 |
| PDEF | 20 | 0.224 | 224 |
| MATK | 21 | 0.235 | 235 |
| MDEF | 19 | 0.213 | 213 |

Now we have the correct value for HP as we see in game.

### multiple items ###

One detail we need to clarify is whether the restricted stats are calculated for each item and then summed, or if the base stats are summed first and the restricted stats calculated from the sum. Altema's formula specifies that it is to be used per-item, but there is a major issue with that approach: how do we aggregate the per-item restricted values? Clearly we can't just sum them, because the stats are already scaled to the stage power for each item. Do we average the stats across all items? Do we average the percentages? If we do average, we would start to see nontrivial stat loss from repeatedly using `floor`, and even worse for items in the support grid, where many stats would just end up becoming 0.

It seems more sensible that the game sums up the base stats of the items first, then calculate the restricted stats from the sums. Let's verify by equipping both items we used previously in the main grid. The formula tells us that our total item power is `floor(32 * 2 / 3 + 19 + 23 + 6 + 21 + 22 * 2 / 3 + 15 + 20 + 21 + 19) = 180` and our restricted HP should be `floor((32 + 22) * 2 / 3 / 180 * 1000 * 3 / 2) = 300`. Let's check in-game:

![File Apr 06, 23 13 03](https://user-images.githubusercontent.com/10483639/162132729-772c6543-f08f-4c81-a605-d3a60589532f.jpeg)

Interestingly, if we instead calculate the item power separately and sum it up to get `89 + 90 = 179` as our total item power, we get `floor((32 + 22) * 2 / 3 / 179 * 1000 * 3 / 2) = 301`, which is incorrect, further supporting our theory that the game is just adding everything up first.

For completeness, let's equip a full grid of various (lvl 1) gear, including support gear, and see if our calculations are still correct:

![Untitled](https://user-images.githubusercontent.com/10483639/162135464-3df53ce4-1ba2-4a38-a9db-30f2f8ccedc9.png)

Plugging it into our formula (total item power 843):

| stat | base value | percentage | restricted value |
|---|---|---|---|
| HP | 54 | 0.202 | 303 |
| PATK | 34 | 0.194 | 194 |
| PDEF | 43 | 0.208 | 208 |
| MATK | 27 | 0.187 | 187 |
| MDEF | 40 | 0.207 | 207 |

Checking in-game:

![File Apr 06, 23 40 33](https://user-images.githubusercontent.com/10483639/162136062-550e853e-efa7-4014-b76e-7c962eeb3768.jpeg)

### the actual formula ###

Summarizing the above, we can rearticulate the formula as follows:

```
restricted stat = floor(restricted power * sum of all lvl 1 item stats / total item power)

except for HP = floor(restricted power * sum of all lvl 1 HP / total item power * 3 / 2)

where total item power = floor(total HP * 2 / 3 + total PATK + total PDEF + total MATK + total MDEF)
```

### verifying with PDEF ###

Using only the HP stat to verify our calculations doesn't feel quite rigorous enough - we should verify each stat independently to make sure they all conform to the formula. Unfortunately, we can't really do this for PATK and MATK without knowing what the defensive stats of an enemy are. We can, however, verify PDEF/MDEF using the damage taken formula and a bit of algebra. We will use the level 0 stage of Gaiamaton, since Gaiamaton always uses either a "Ssss..." physical attack or a no-text physical attack on the first turn.

[This site](https://wikiwiki.jp/mitrasphere/%E8%A2%AB%E3%83%80%E3%83%A1%E3%83%BC%E3%82%B8%E3%81%A8%E8%80%90%E4%B9%85%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) tells us that the damage taken formula is the following:

```
damage taken = attack power * 5000 / (DEF + 5000) * elemental resistance

where DEF = PDEF for physical attack or MDEF for magic attack
```

We have no particular reason to doubt this formula, but we can quickly verify it in a non-restricted stage, say, level 1 Gaiamaton. To make things easier, we will avoid wearing armor and accessories to avoid factoring in elemental resistance and passive skills.

First we need to figure out the "Ssss..." attack's attack power, which we can get with some algebra:
```
attack power = damage taken / 5000 * (DEF + 5000)
```

We remove all equipment except for Crystal Clod again and check our total PDEF (not the equip one):

![File Apr 06, 20 39 18](https://user-images.githubusercontent.com/10483639/162140036-ae3c4cc1-4587-4d59-8a9b-810f6fb93200.jpeg)

Then we see how much damage we take:

![IMG_7297](https://user-images.githubusercontent.com/10483639/162140216-54b9d2f0-47fb-49f5-956f-ac6aa3baaff7.jpg)

Using our formula, we get an attack power of `1966 / 5000 * (3788 + 5000) = 3455`

Now we equip a few more items to increase our PDEF by a good amount:

![File Apr 07, 00 11 35](https://user-images.githubusercontent.com/10483639/162141067-df7c3361-e3f3-47f5-bd16-9e5a08bcf29c.jpeg)

The formula tells us that we should take `3455 * 5000 / (5160 + 5000) = 1700` damage - let's see:

![File Apr 07, 00 14 22](https://user-images.githubusercontent.com/10483639/162141539-5c81be83-56b6-4252-9f5d-a4893af7fbf0.jpeg)

Good, we are confident that the damage taken formula is accurate. Let's now use it to verify our restricted PDEF stats across various equipment. We will first calculate the attack power of "Ssss..." on level 0 using a derived PDEF, then use to try and predict the damage taken with other equipment.

Using the previous chart for Metal Core that used the correct formula, we get a restricted PDEF of 224. Let's check the damage taken with this:

![File Apr 07, 00 27 35](https://user-images.githubusercontent.com/10483639/162144104-6f4493cc-5d1f-4105-abd3-1b5979b60875.jpeg)

From this, we get an attack power of `206 / 5000 * (224 + 5000) = 215`. Let's
