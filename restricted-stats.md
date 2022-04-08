## Tea's Restricted Stats Research ##

Restricted Stages are stages where all players (and enemies) have their stats scaled to a certain common baseline.

![File Apr 06, 18 52 08](https://user-images.githubusercontent.com/10483639/162104407-0d49060e-12f5-479c-ba3f-bc0b90c34bca.jpeg)

While the JP Altema site defines a formula for how stats are calculated in restricted stages, people have found that it doesn't quite seem to line up with what they see in-game, and the nature of it makes it difficult to figure out optimal restricted stage sets. So here we have compiled a series of in-game testing that we've done to really examine the details of restricted stage stat scaling and its formula, and use that to outline some practical ways to approach building restricted content sets.

### tl;dr ###

- the formula:
```
restricted stat = floor(restricted power * sum of all lvl 1 item stats / total item power)

where total item power = floor(total HP * 2 / 3 + total PATK + total PDEF + total MATK + total MDEF)
```
- this means there's no easy way to use in-game stat summaries to estimate your restricted stats
- for the most part, the details of the formula aren't important - just equipping as many items as you have that have the stat you want to maximize as its highest stat will generally give you a decent set for restricted content
- for support gear, different items with similar stat distributions will generally not have much of a difference between them, regardless of rarity
- however, higher-star support gear is more resilient to changes in the main grid due to their higher base values

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
| HP | 32 | 32/90.33=0.354 | 1000\*0.354=354 |
| PATK | 19 | 19/90.33=0.21 | 1000\*0.21=210 |
| PDEF | 23 | 23/90.33=0.255 | 1000\*0.255=255 |
| MATK | 6 | 6/90.33=0.066 | 1000\*0.066=66 |
| MDEF | 21 | 21/90.33=0.232 | 1000\*0.232=232 |

You may notice that the percentages don't add up to 1 - this is because the item power was calculated using 2/3 of the HP, whereas the restricted stat uses the full HP value.

Let's check these numbers in-game, at least the easy one to check which is HP. First we remove all equipment except this one and make sure the stats reflected in the stat summary are the same as the item stats:

![File Apr 06, 20 39 18](https://user-images.githubusercontent.com/10483639/162117333-cf7ef441-3c2c-4dea-b817-23aee510db45.jpeg)

Then we jump into a 1000 power restricted stage:

![File Apr 06, 20 40 18](https://user-images.githubusercontent.com/10483639/162117392-aa6c87a4-68da-4ed8-bd04-c22411f112d1.jpeg)

We are pretty close but a little off. Let's try another item to verify these calculations:

![File Apr 06, 21 06 52](https://user-images.githubusercontent.com/10483639/162118153-0878c91a-951c-4b0b-a91b-97e8429c4643.jpeg)

We get an item power of 89.67, so we plug that in:

| stat | base value | percentage | restricted value |
|---|---|---|---|
| HP | 22 | 22/89.67=0.245 | 1000\*0.245=245 |
| PATK | 15 | 0.167 | 167 |
| PDEF | 20 | 0.223 | 223 |
| MATK | 21 | 0.234 | 234 |
| MDEF | 19 | 0.212 | 212 |

Let's check in-game:

![File Apr 06, 21 13 29](https://user-images.githubusercontent.com/10483639/162118766-cc42d211-b62f-4290-8d24-f14d652c611f.jpeg)

Dang, we're still off. We're rounding a bit in our calculations, so we can imagine that maybe the game is doing a different kind of rounding that account for the difference. Or maybe they're not rounding at all, and doing something entirely different. Trying various kinds of decimal-to-integer operations at various points in the formula until we get the right number would be quite a hassle, but luckily the game gives us some hints that narrow down the approach:

- all stat numbers in the game are displayed as integers, so we can assume that the game is likely storing them as integers as well, not decimals
- likewise, combat power is always an integer, though per-item power is never displayed explicitly
- when equipping items in the support slot, the calculation for the stat contribution seems to drop the decimals after scaling it by 20% - e.g. an item with 24 HP will give 4 HP in the support slot
- mitra is an old game and has not really changed in a while, so its code is also likely very old - for computers, the `floor` operation, or dropping decimals, is significantly simpler and faster than rounding, especially in older programming languages

These hints tell us that the game is likely using `floor` to drop decimal points whenever it calculates stats or power. Let's give that a try on our previous item:

Instead of using 89.67, we use 89 as the item power, and we also use `floor` on the final stats (for brevity we will also drop the percentage to 3 decimals, though the game likely uses the full decimal value):

| stat | base value | percentage | restricted value |
|---|---|---|---|
| HP | 22 | 22/89=0.247 | floor(1000\*0.247)=247 |
| PATK | 15 | 0.168 | 168 |
| PDEF | 20 | 0.224 | 224 |
| MATK | 21 | 0.235 | 235 |
| MDEF | 19 | 0.213 | 213 |

Now we have the correct value for HP as we see in-game. We can go back to Crystal Clod and use the modified formula to make sure we get the right HP as well:

| stat | base value | percentage | restricted value |
|---|---|---|---|
| HP | 32 | 32/90=0.355 | 1000\*0.354=355 |
| PATK | 19 | 19/90=0.211 | 1000\*0.211=211 |
| PDEF | 23 | 23/90=0.255 | 1000\*0.255=255 |
| MATK | 6 | 6/90=0.066 | 1000\*0.066=66 |
| MDEF | 21 | 21/90=0.233 | 1000\*0.233=233 |

### multiple items ###

One detail we need to clarify is whether the restricted stats are calculated for each item and then summed, or if the base stats are summed first and the restricted stats calculated from the sum. Altema's formula specifies that it is to be used per-item, but there is a major issue with that approach: how do we aggregate the per-item restricted values? Clearly we can't just sum them, because the stats are already scaled to the stage power for each item. Do we average the stats across all items? Do we average the percentages? If we do average, we would start to see nontrivial stat loss from repeatedly using `floor`, and even worse for items in the support grid, where many stats would just end up becoming 0.

It seems more sensible that the game sums up the base stats of the items first, then calculates the restricted stats from the sums. Let's verify by equipping both items we used previously in the main grid. The formula tells us that our total item power is `floor(32 * 2 / 3 + 19 + 23 + 6 + 21 + 22 * 2 / 3 + 15 + 20 + 21 + 19) = 180` and our restricted HP should be `floor((32 + 22) / 180 * 1000) = 300`. Let's check in-game:

![File Apr 06, 23 13 03](https://user-images.githubusercontent.com/10483639/162132729-772c6543-f08f-4c81-a605-d3a60589532f.jpeg)

Interestingly, if we instead calculate the item power separately and sum it up to get `89 + 90 = 179` as our total item power, we get `floor((32 + 22) / 179 * 1000) = 301`, which is incorrect, further supporting our theory that the game is just adding everything up first.

For completeness, let's equip a full grid of various (lvl 1) gear, including support gear, and see if our calculations are still correct:

![Untitled](https://user-images.githubusercontent.com/10483639/162135464-3df53ce4-1ba2-4a38-a9db-30f2f8ccedc9.png)

Plugging it into our formula (total item power 843):

| stat | base value | percentage | restricted value |
|---|---|---|---|
| HP | 256 | 0.303 | 303 |
| PATK | 164 | 0.194 | 194 |
| PDEF | 176 | 0.208 | 208 |
| MATK | 158 | 0.187 | 187 |
| MDEF | 175 | 0.207 | 207 |

Checking in-game:

![File Apr 06, 23 40 33](https://user-images.githubusercontent.com/10483639/162136062-550e853e-efa7-4014-b76e-7c962eeb3768.jpeg)

### the actual formula ###

Summarizing the above, we can re-articulate the formula as follows:

```
restricted stat = floor(restricted power * sum of all lvl 1 item stats / total item power)

where total item power = floor(total HP * 2 / 3 + total PATK + total PDEF + total MATK + total MDEF)
```

### verifying with PDEF ###

Using only the HP stat to verify our calculations doesn't feel quite rigorous enough - we should verify each stat independently to make sure they all conform to the formula. Unfortunately, we can't really do this for PATK and MATK without knowing what the defensive stats of an enemy are. We can, however, verify PDEF/MDEF using the damage taken formula and a bit of algebra. We will use the level 0 stage of Gaiamaton, since Gaiamaton always uses a "Ssss..." physical attack on the first or second turn.

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

Using our formula, we get an attack power of `1966 / 5000 * (3788 + 5000) = 3455`. But wait, what if the game is using `floor` on the damage taken formula as well? Then the actual damage taken could be anywhere from 1966 to 1966.99999... - to compensate for this, we need to calculate the upper bound of the attack power as well, which would be `1967 / 5000 * (3788 + 5000) = 3457`.

Now we equip a few more items to increase our PDEF by a good amount:

![File Apr 07, 00 11 35](https://user-images.githubusercontent.com/10483639/162141067-df7c3361-e3f3-47f5-bd16-9e5a08bcf29c.jpeg)

The formula tells us that we should take at least `3455 * 5000 / (5160 + 5000) = 1700` and at most `3457 * 5000 / (5160 + 5000) = 1701` damage - let's see:

![File Apr 07, 00 14 22](https://user-images.githubusercontent.com/10483639/162141539-5c81be83-56b6-4252-9f5d-a4893af7fbf0.jpeg)

Good, we are confident that the damage taken formula is accurate. Let's now use it to verify our restricted PDEF stats across various equipment. We will first calculate the attack power of "Ssss..." on level 0 using a derived PDEF, then use it to try and predict the damage taken with other equipment.

Using the previous chart for Metal Core that used the correct formula, we get a restricted PDEF of 224. Let's check the damage taken with this:

![File Apr 07, 00 27 35](https://user-images.githubusercontent.com/10483639/162144104-6f4493cc-5d1f-4105-abd3-1b5979b60875.jpeg)

From this, we get an attack power range of `206 / 5000 * (224 + 5000) = 215` to `207 / 5000 * (224 + 5000) = 216`. If we now swap to Crystal Clod, with a restricted PDEF of 255, we expect that the damage taken would be between `215 * 5000 / (255 + 5000) = 204` and `216 * 5000 / (255 + 5000) = 205`. Let's see in-game:

![IMG_7302](https://user-images.githubusercontent.com/10483639/162174900-a06bc66b-5239-4898-b9cb-0b0f6d647299.jpg)

That looks good for our restricted PDEF calculations. Assuming the attack power formula uses `floor`, this also tells us that the attack power of "Ssss..." is likely 216, not 215.

For good measure, let's try it with an item with a significantly lower PDEF distribution:

![File Apr 07, 03 05 11](https://user-images.githubusercontent.com/10483639/162175517-ea53fee2-77f2-4208-a3d7-4ceadd3cabf8.jpeg)

Our formula tells us that our restricted PDEF will be 179, so our expected damage is `216 * 5000 / (179 + 5000) = 208`:

![File Apr 07, 03 06 10](https://user-images.githubusercontent.com/10483639/162176223-db0a6e37-880b-4a4f-b9bd-74facd9cc29d.jpeg)

### now what? ###

So we have a verified and accurate formula for calculating restricted stats from lvl 1 stats of your gear. What can we do with this information? How can we use this to create optimal sets for restricted content? It's very apparent that the formula alone isn't going to get us there - it would be entirely impractical to look up the lvl 1 stats of all the gear you want to use and sum them together in various combinations to get the stat totals you need to calculate and compare the restricted stats.

Intuitively, the formula tells us that stat distribution is important - if you want to maximize a particular stat, you want to wear a set of gear that maximizes that stat while minimizing all other stats (this is why much of our existing recommendations on restricted content gear is based on per-item stat distributions taken from the now-defunct kotori site). We can also tell that the magnitude of the lvl 1 stat is important - high stat values mean they constitute a larger portion of the stat totals and have a bigger impact on the final stat distribution. And since the support grid gear has their stats scaled down to 20%, this means that the main grid gear has significantly more impact on the final stat distribution than the support grid.

Main grid gear is not exactly easy to optimize for stats - weapons have to be tailored to your rotations and effectiveness against each boss, and armor/accessories are typically selected based on their passive skills. Since there isn't much we can do to control the stat distribution of your main grid, we can really only optimize our restricted sets by setting up our support grid to push our stat distribution towards whichever stat we want to maximize. Let's take a look at some examples, using a set main grid and comparing the stat changes with different support grid options.

### optimizing PDEF ###

Here is a basic but usable main grid for a guardian:

![File Apr 07, 18 08 44](https://user-images.githubusercontent.com/10483639/162343619-7eeaddf3-06a7-4175-812b-060379a79c11.jpeg)

Some of these happens to be lvl 1, but most are not. Luckily, we can find most common 4-stars in the shops and older rainbows in the nekia medal exchange:

![File Apr 07, 17 51 26](https://user-images.githubusercontent.com/10483639/162342391-a90d83d8-ce44-44f2-b3e4-8da3f3b6816e.jpeg)

Using this, we come up with the following lvl 1 stat totals and restricted stats (we'll omit the restricted stats for now since it is easy enough to see from the percentage in a 1000 power stage):

| stat | base value | percentage |
|---|---|---|
| HP | 331 | 0.374 |
| PATK | 186 | 0.210 |
| PDEF | 240 | 0.271 |
| MATK | 53 | 0.06 |
| MDEF | 184 | 0.208 |

Let's double-check in-game:

![File Apr 07, 18 09 20](https://user-images.githubusercontent.com/10483639/162343688-a021d620-7599-4e78-9609-75d1fb546eca.jpeg)

Now let's add in support weapons - we'll use 1-star Wooden Stake, which has one of the highest PDEF stat percentage in the support slot (largely due to `floor` making its already low other stats even lower). Here is the stat gain for adding one of these in the support slot:

![File Apr 07, 18 14 05](https://user-images.githubusercontent.com/10483639/162344172-264cb0dc-f5dd-44ac-9236-4264e8019711.jpeg)

And here's the stat totals and distribution for 10 of these:

| stat | base value | percentage |
|---|---|---|
| HP | 30 | 0.5 |
| PATK | 10 | 0.166 |
| PDEF | 20 | 0.333 |
| MATK | 0 | 0 |
| MDEF | 10 | 0.166 |

Adding to our main grid, we get the following restricted stats:

| stat | base value | percentage |
|---|---|---|
| HP | 361 | 0.382 |
| PATK | 196 | 0.207 |
| PDEF | 260 | 0.275 |
| MATK | 53 | 0.056 |
| MDEF | 194 | 0.205 |

Let's verify the PDEF against Gaiamaton - our expected damage is `216 * 5000 / (275 + 5000) = 204`:

![IMG_7315](https://user-images.githubusercontent.com/10483639/162344940-c34009ca-206b-4541-a8e3-183a190d2fdb.jpg)

Looking at the stat changes, we can see the largest improvements in HP and PDEF, with 0.8% and 0.4% respectively. It's not a lot, and we're gaining more HP than PDEF which is not ideal, as a result of Wooden Stake's HP stat percentage being so high. Because we're using the 1-star version, the magnitude of the total stats is relatively low, so you can imagine that if we used 4-star support gear with a similar stat distribution, we would see the gap in gains between HP and PDEF widen.

Let's try another item: the 3-star concierge set. Here is the distribution of 10 of these in the support slot:

| stat | base value | percentage |
|---|---|---|
| HP | 70 | 0.421 |
| PATK | 30 | 0.18 |
| PDEF | 50 | 0.301 |
| MATK | 10 | 0.06 |
| MDEF | 30 | 0.18 |

And added to our main grid:

| stat | base value | percentage |
|---|---|---|
| HP | 401 | 0.381 |
| PATK | 216 | 0.205 |
| PDEF | 290 | 0.276 |
| MATK | 63 | 0.06 |
| MDEF | 214 | 0.203 |

Wow, we've managed to take an entire 0.1% away from HP towards PDEF compared to our 1-star support set, and now have a whopping 0.5% improvement in PDEF over our base main grid.

Maybe the problem is our stat value magnitudes - we're not adding enough additional stats from our support grids to make a big impact on our final stat totals. Let's see what we get if we pick a bunch of 4-star gear with very high PDEF - say, Mitra+ armor, Guardian's Cups, and Dark Fangs:

![File Apr 07, 18 56 30](https://user-images.githubusercontent.com/10483639/162348652-993a6ff8-5e5a-413e-bddb-87216b240984.jpeg)

Here's the stat totals and distributions for equipping 10 of each in all of our support grids:

| stat | base value | percentage |
|---|---|---|
| HP | floor(54/5)\*10 + floor(57/5)\*10 + floor(38/5)\*10=401 | 0.381 |
| PATK | 202 | 0.192 |
| PDEF | 303 | 0.288 |
| MATK | 50 | 0.047 |
| MDEF | 230 | 0.218 |

This looks a bit more promising - our HP percentage is much lower, but our PDEF percentage is also a little bit lower, and our MDEF percentage is much higher. Let's add it to our main grid:

| stat | base value | percentage |
|---|---|---|
| HP | 732 | 0.378 |
| PATK | 388 | 0.2 |
| PDEF | 543 | 0.28 |
| MATK | 103 | 0.053 |
| MDEF | 414 | 0.213 |

That's not too bad - we've reduced the HP gain to 0.4% and now have 0.9% PDEF gain, as well as a 0.5% MDEF gain, which doesn't hurt. But we are moving grains of sand across a couple of giant sand dunes at this point. There's no obvious way to shift our initial 27.1% PDEF even by single-digit percentages.

The issue is that items with the same primary stat tend to have very similar stat distributions. All of the gear we've been playing with have high HP, PDEF, medium MDEF and PATK, and low MATK. If we wanted to make significant improvements, we would need to find support gear that still has high PDEF, but much lower HP, PAT, and MDEF, which in general doesn't exist. So if you are trying to optimize a certain stat and you are already using mostly gear that has that stat as its highest stat, there's really not much room for improvement between different gear options.

In other words, as long as you are using support gear that has a similar stat distribution to your main grid, the specific item choices don't make a huge difference as far as restricted stats are concerned.

### support gear resilience ###

...or does it? In the previous section, we found that if we generally use gear with similar stat distributions, we only introduce very minor changes in our final stat distribution. But what if we can't? Here's a not-so-uncommon scenario: you're running an EX boss with a group and the boss's magic damage raidbuster is giving your party a rough time. You ask if the archer has Crystal Sphere; they've never heard of it. Okay, that's fine, you have a flex slot in your weapon grid and you have a 2-star Crystal Sphere lying around in your inventory that you haven't cleaned for 2 weeks:

![File Apr 07, 20 28 40](https://user-images.githubusercontent.com/10483639/162357886-fa6328cf-8ea7-483c-b06e-89e79fbf8458.jpeg)

Using our earlier main grid, we swap out Mythril Cutlass. Here is our new main grid stats:

| stat | base value | percentage |
|---|---|---|
| HP | 306 | 0.365 |
| PATK | 179 | 0.203 |
| PDEF | 221 | 0.264 |
| MATK | 65 | 0.077 |
| MDEF | 177 | 0.211 |

Let's now compare two extremes from our support grid options - the 4-star set we looked at earlier, and a 1-star set composed of Wooden Stakes and Knight armor/accessories:

![File Apr 07, 20 32 57](https://user-images.githubusercontent.com/10483639/162357903-e35a87a5-78a7-44e8-9a62-e34180442738.jpeg)

Here is our distribution using the 4-star set with our new main grid:

| stat | base value | percentage |
|---|---|---|
| HP | 707 | 0.374 |
| PATK | 372 | 0.196 |
| PDEF | 524 | 0.277 |
| MATK | 115 | 0.06 |
| MDEF | 407 | 0.215 |

And here it is with the 1-star set:

| stat | base value | percentage |
|---|---|---|
| HP | 396 | 0.381 |
| PATK | 200 | 0.192 |
| PDEF | 281 | 0.27 |
| MATK | 65 | 0.062 |
| MDEF | 227 | 0.218 |

Comparing the two, the 4-start support grid gives us 0.7% more PDEF than the 1-star set in this game, which is a lot bigger than the differences in support grid choices that we looked at earlier. Adding an item with a significantly different stat distribution to our main grid caused our final stat distributions to shift by quite a bit, but the larger base values of our 4-star support grid meant that we could maintain our stat distribution closer to our primary stat, whereas the 1-star support grid, despite having a similar stat distribution, has a harder time maintaining the ideal stat distribution due to its lower base values. You can imagine that the more suboptimal items you end up using in your main grid, the more prominent this effect becomes. 4-star support grids, with their larger base values, are more resilient to changes in the main grid than lower-star support grids.

Is it important in practice? 0.7% in a 20k restricted EX boss is a difference of 140 in the stat; for defensive stats with diminishing returns it's nothing to cry over, but for linear-scaling offensive stats it might be a good chunk of additional damage. And you will frequently need to use a couple of items with sub-optimal stat distributions as part of adaptations to boss mechanics or party dynamics. So there is a case to be made for using higher-star support grids when given the choice. Of course, higher-star gear is harder to collect, and you want to have a full support grid for each element, so the lower-star support grids are much easier to put together to give you that all-important 20% elemental resistance and 10% CS.

### wow did you really read all of that? ###

If you've managed to get here, you now have two options:

- disagree with this general take and try to math out more optimal restricted gear setups
- agree and forget about all this complicated restricted stats stuff and just use the same sets for both normal and restricted content

I highly recommend the latter.

![File Apr 07, 21 40 22](https://user-images.githubusercontent.com/10483639/162364795-19009a14-0c92-440a-a3cb-55deb38e1a05.jpeg)
