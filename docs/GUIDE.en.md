# Princess Maker: Children of Revelation (預言之子) — Ending Conditions (from the game's data tables)

Data sources: `TableData` (EndingStatus, Ending, Reputation, Quest, Event) and the `C_Flow_*` event scripts, cross-checked against the Bahamut guide's "ending parameters".
Last updated: 2026-11-21

## How to read this

- The "ending conditions" are `EndingStatus`. **Different condition groups are AND; within one group, conditions are OR** (confirmed from the game's evaluation code; only the Queen's Warrior/Magic Rating is placed in the same group).
- Ratings are written in two ways: "Rating ≥ N" is the rating value itself; "Rating grade ≥ N" is the grade (1: 0, 2: 50, 3: 150, 4: 250, 5: 350, 6: 500).
- Lesson/job grades: 3 = B, 4 = A, 5 = S (B and A are matched by the forum and by `Quest_RAAPresident` requiring "Art, Music and Dance at A = grade 4"; S is deduced from the sequence).
- "Alignment" is the good/evil value: ≤ -150 is Chaos, ≥ 225 is Lawful (Queen).
- A "prerequisite event chain" is the occurrence condition of the `Flow_*` scripts. Only an `event completed Flow_xxx` listed in the ending conditions is required. **Event chains not listed in the ending conditions (for example Alchemist of Chaos) are not required for the ending**; they are just story.
- Trigger: 1010 start of month, 1017 random in the latter part of the schedule, 2012 meeting someone in the Royal Capital, 5001 after the festival ends, 5002 festival starts.

## How the ending is decided (from the GameAssembly disassembly: `lxc::mko` / `mau` / `sai`)

**1. Candidate eligibility.** Only the 55 regular endings with `EndingType = 2` are considered (`Couple_*` is `EndingType = 3` and is not part of this round). For an ending to become a candidate, **every condition group must have at least one condition that holds** (OR within a group, AND between groups; confirmed in both `Cond_CheckEndingCondition` and `mko`). Whether a single condition holds: `CompareType = 0` (event, no condition) means "current value ≥ 1"; the others compare with CompareValue using ≥ / ≤.

**2. Scoring.** Each candidate's score = **Weight + bonus**. Only the **highest score** is kept, and **all ties are kept**.

**3. Drawing.** If 1 is kept, it is used; if several, **1 is drawn at random** (`sai`). So tied endings are random.

**4. Bonus (`mau`).**
- Endings with Weight ≥ 30000: no bonus.
- Endings with Weight < 30000: bonus = `min(2 × value, 8000)`. The value is the accumulated value, in the player's record (record field `lcq`), of the lesson referenced by each "lesson/part-time job grade" condition (Type 8) in that ending's conditions; the **largest one** is taken.
- **Bard is a special case**: bonus = `min(2 × (accumulated Edu_Music + Edu_Literature), 8000)`.
- The field name `lcq` is obfuscated, but its purpose was confirmed from the disassembly of `ld::fno(lesson Key, accumulated value)`: among the `ScheduleStat` grade rows of that lesson, the function takes the row with the smallest `NeedCount_GradeUp` that is **greater than** the accumulated value as the current grade. That is, `NeedCount_GradeUp` is a **cumulative threshold**, and `lcq` is the cumulative count compared against it (for example for a lesson: cumulative ≥ 42 times is grade 2, ≥ 96 is B, ≥ 160 is A, ≥ 234 is S). So **in practice the bonus is 2 × cumulative count, roughly a few hundred and at most a little over one thousand**; the 8000 cap is almost never reached.

**5. What this means for you.**
- High-difficulty endings (Queen 52000, Demon King 51000, Princess 50000, Great General, etc.) simply override everything through Weight; if all conditions pass, it is that ending.
- The bonus is too small to **cross a Weight tier** (for example the 20000 series cannot catch up with the 27000 series). Its role is to **reorder endings with similar Weight**: for example 20000, 20080, 20160, 20200, 20240, 20495, 20700, 20850, 20900, which differ by tens to hundreds; whoever trained more takes priority.
- Exact ties (for example Rich, Magic Knight and Diplomat are all 40000, with no bonus) are drawn at random.

| Ending | Weight | Lesson/job bonus |
|---|---|---|
| Queen (Queen) | 52000 | None |
| Demon King (DemonKing) | 51000 | None |
| Princess (Princess) | 50000 | None |
| Great General (General) | 43000 | None |
| Royal Wizard (RoyalWizard) | 42000 | None |
| President of the Royal Academy of Arts (RAAPresident) | 41000 | None |
| Diplomat (Diplomat) | 40000 | None |
| Magic Knight (MagicKnight) | 40000 | None |
| Rich (Rich) | 40000 | None |
| Salon Lobbyist (Sting) | 34000 | None |
| Femme Fatale (Femme) | 33000 | None |
| Alchemist of Chaos (ChaosAlchemist) | 32000 | None |
| Mysterious Thief (MysteriousThief) | 32000 | None |
| Fortune Teller (FortuneTeller) | 31000 | None |
| King of the Back Alleys (TheGodfather) | 31000 | None |
| Bard (Bard) | 27000 | Yes (about 2 × cumulative count) |
| Jeweler (Jeweler) | 27000 | Yes (about 2 × cumulative count) |
| Magic Engineer (MagicEngineer) | 27000 | Yes (about 2 × cumulative count) |
| Perfumer (Perfumer) | 27000 | Yes (about 2 × cumulative count) |
| Sigil Drafter (SigilDrafter) | 27000 | Yes (about 2 × cumulative count) |
| Sword Dancer (Sworddancer) | 27000 | Yes (about 2 × cumulative count) |
| Academy Dean (AcademyDean) | 23000 | Yes (about 2 × cumulative count) |
| Prima Ballerina (Dancer) | 23000 | Yes (about 2 × cumulative count) |
| Prima Donna (PrimaDonna) | 23000 | Yes (about 2 × cumulative count) |
| Cook (Cook) | 22500 | Yes (about 2 × cumulative count) |
| Magic Clerk (MagicClerk) | 20900 | Yes (about 2 × cumulative count) |
| Capital Elite Corps (CapitalEliteTroop) | 20850 | None |
| Adventurer (Adventurer) | 20700 | None |
| Alchemist (Alchemist) | 20495 | Yes (about 2 × cumulative count) |
| Dance Tutor (DanceTutor) | 20495 | Yes (about 2 × cumulative count) |
| Etiquette Teacher (Governess) | 20495 | Yes (about 2 × cumulative count) |
| Historian (Historian) | 20495 | Yes (about 2 × cumulative count) |
| Magic Instructor (MagicTutor) | 20495 | Yes (about 2 × cumulative count) |
| Martial Arts Instructor (MartialArtsInstructor) | 20495 | Yes (about 2 × cumulative count) |
| Music Tutor (MusicTutor) | 20495 | Yes (about 2 × cumulative count) |
| Painter (Painter) | 20495 | Yes (about 2 × cumulative count) |
| Swordsmanship Dojo Instructor (SwordInstructor) | 20495 | Yes (about 2 × cumulative count) |
| Writer (Writer) | 20495 | Yes (about 2 × cumulative count) |
| Bartender (Bartender) | 20240 | Yes (about 2 × cumulative count) |
| Logistics Manager (UnloadingManager) | 20240 | Yes (about 2 × cumulative count) |
| Research Assistant (Assistant) | 20200 | Yes (about 2 × cumulative count) |
| Private Tutor (PrivateTutor) | 20200 | Yes (about 2 × cumulative count) |
| Maid Cafe Superstar (Superstar) | 20200 | Yes (about 2 × cumulative count) |
| Blacksmith (Blacksmith) | 20160 | Yes (about 2 × cumulative count) |
| Clothing Designer (Designer) | 20160 | Yes (about 2 × cumulative count) |
| Logistics Handler (Logistics) | 20080 | Yes (about 2 × cumulative count) |
| Soldier (Soldier) | 20080 | Yes (about 2 × cumulative count) |
| Carpenter (Carpenter) | 20000 | Yes (about 2 × cumulative count) |
| Village Healer (Doctor) | 20000 | Yes (about 2 × cumulative count) |
| Housekeeping Agency Owner (Housekeeper) | 20000 | Yes (about 2 × cumulative count) |
| Ranch Manager (RanchManager) | 20000 | Yes (about 2 × cumulative count) |
| Horticultural Therapist (Therapist) | 20000 | Yes (about 2 × cumulative count) |
| Restaurant Server (Waitress) | 20000 | Yes (about 2 × cumulative count) |
| Villager (Villager) | 2 | None |
| Jobless (Jobless) | 1 | None |

## How the Fortune Teller gives hints (`lxc::mca` / `lzj` / `lyx`)

- Each ending's progress = Σ (highest Score of each condition group × the progress of the **best condition** in that group) ÷ Σ the highest Score of each group.
- Progress of a single condition: for an ordinary ≥ condition it is "current value ÷ target value", clamped to 0 to 1; for grade-type conditions (rating, lesson grade) it is (current grade − 1) ÷ (target grade − 1); for an Alignment ≤ condition it is (upper limit − current) ÷ (upper limit − target); for yes/no conditions (event, no condition) it is 0 or 1.
- Only endings with progress ≥ a threshold are considered; threshold = 0.2 + 0.6 × elapsed days ÷ 2920 (8 years), that is, rising gradually from 0.2 to 0.8.
- `Ending_Jobless` (matched by Key string) and endings with Weight < 1000 (Jobless and Villager) are excluded.
- Sorting: progress from high to low; ties are broken by Weight + bonus, high to low. At most **3** are shown.

## Conditions for the Dancer and the Prima Donna

The last condition group of both is "Art Festival victory" (Type 15 Dance, 16 Music). How `lxc::hch` works: it walks through the Art Festival history records; for each record, the tags separated by `&` must **contain both `Rank=1` and `Category=<that category>`** to count once, and the condition is ≥ 1, that is, "took first place in the Art Festival of that category". Also, the event node `Cond_CheckEndingCondition` does not handle 15 and 16, but in the entire data set only Princess Episode 6 uses it, so it does not affect these two endings.

## How the couple ending is decided (from `pk::OnTheEnd` / `ShowCard` / `go::glc`)

**The couple ending has nothing at all to do with the EndingStatus conditions**; it only looks at one number, "the wedding partner in the game state". It is **a separate ending card from the career ending**: the career ending is decided by `lxc::sai` (previous section), the couple ending is decided by the wedding partner, and the ending screen shows them in sequence.

**Mapping rule** (wedding partner code = `EPartnerNPC`, set by `Act_SetWeddingNPC`):

| Wedding partner code | Partner | Couple ending | Wedding dialogue graph |
|---|---|---|---|
| 7 | Dragon_Female | `Ending_Couple_DragonFemale` | `D_DragonFemale_Wedding` |
| 6 | Dragon_Male | `Ending_Couple_DragonMale` | `D_DragonMale_Wedding` |
| 5 | Forest_Female | `Ending_Couple_ForestFemale` | `D_ForestFemale_Wedding` |
| 4 | Forest_Male | `Ending_Couple_ForestMale` | `D_ForestMale_Wedding` |
| 3 | Demon_Female | `Ending_Couple_DemonFemale` | `D_DemonFemale_Wedding` |
| 2 | Demon_Male | `Ending_Couple_DemonMale` | `D_DemonMale_Wedding` |
| 1 | Furry_Female | `Ending_Couple_FurryFemale` | `D_FurryFemale_Wedding` |
| 0 | Furry_Male | `Ending_Couple_FurryMale` | `D_FurryMale_Wedding` |
| -4 | Npc_Cube | `Ending_Couple_Cube` | `D_Cube_Wedding` |
| -5 | Npc_Caim | `Ending_Couple_Caim` | `D_Caim_Wedding` |
| -6 | Npc_Icosa | `Ending_Couple_Icosa` | `D_Icosa_Wedding` |
| -7 | Npc_Kimaris | `Ending_Couple_Kimaris` | `D_Kimaris_Wedding` |
| -8 | Npc_Muzai | `Ending_Couple_Muzai` | `D_Muzai_Wedding` |
| -1 (villager) / -3 (work partner) | Ordinary marriage | `Ending_Couple_Normal` | — |
| -2 (single) | Not married | No couple ending | — |

**The only special case**: when the wedding partner is Agares (DemonMale, 2) and the **career ending is `Ending_Princess`** (Princess), the couple ending becomes **`Ending_Couple_DemonMale_King`** instead of `Ending_Couple_DemonMale`. The code matches the string `Ending_Princess` (`a::gx`), so only by completing the Princess route do you get the "Demon King" version. Other combinations (for example a career ending that is not Princess) still give `Ending_Couple_DemonMale`.

**Other confirmed points**
- The only call site of the career ending is `pk::chs` (the ending screen) calling `lxc::sai`; there is no other entry point.
- A wedding partner of -1 or -3 is an ordinary marriage and the same `Couple_Normal` card; -2 is single, which only gives the "unmarried" achievement and no couple ending.
- The achievement "All marriage endings" counts the distinct partners, with wedding partner code ≥ -4, across all past clears (-1 and -3 are merged into one); it is achieved when the total reaches **11**; `go::glb` and `go::glc` are responsible for putting the endings of past clears into the collection list.
- The ending screen also adds one more card, `Card_Second_EA` (`a::ha`), only under a certain condition (`hl::ljk()` is true and that card is already unlocked); I did not trace its purpose.

**Final determination at the ending (`pk::gde`, called at the start of the ending screen `pk::chs`, with the result written back to the wedding partner)**

The wedding partner has two sources: it is written by `Act_SetWeddingNPC` when a proposal event is accepted (code ≥ 0, the 8 tribe companions), and it is filled in at the moment of the ending by `pk::gde` from the completed events and the career ending. The order in `gde` is as follows (address `0x182638610`):

1. **Wedding partner is already ≥ 0** (a proposal was accepted): use it directly and skip everything after.
2. Otherwise check in order; the first match is used:

| Order | Condition (all must hold) | Result |
|---|---|---|
| 1 | Event `Caim_Episode1` completed, and the career ending is `Ending_Waitress` (Restaurant Server) | -5 Caim |
| 2 | Event `Icosa_Episode1` completed, and the career ending is `Ending_MagicTutor` (Magic Instructor) | -6 Icosa |
| 3 | Event `Kimaris_Episode1` completed, and the career ending is `Ending_Soldier` (Soldier) | -7 Kimaris |
| 4 | Event `Muzai_Episode1` completed, and the career ending is `Ending_Writer` (Writer) | -8 Muzai |
| 5 | Event `Cube_Episode2` completed (**the career ending is not checked**) | -4 Cube |
| 6 | The sum of the 12 base stats (Stamina, Strength, Agility, Intelligence, Magic, Spirit, Elegance, Manners, Speech, Sensitivity, Sense, Charm) **≥ 3600** | -2 Single (no couple ending) |
| 7 | The career ending is `Ending_Jobless` or `Ending_Villager` | -1 Ordinary marriage (`Couple_Normal`) |
| 8 | None of the above | Keep the current value. The default for a new game is **-3 (work partner, also `Couple_Normal`)**: the constructor `do::do()` of the save-data class `do` initializes this field to -3, confirmed |

In addition, each of 1 to 4 and 5 has a shortcut "if the current value is already that code, keep it", which does not affect the results above.

**What this means**
- To get the couple ending of Caim, Icosa, Kimaris or Muzai, **the career ending must be exactly the matching one** (Restaurant Server, Magic Instructor, Soldier, Writer), plus completing the respective `Episode1`. With the wrong career ending, it falls straight to the next rule.
- Cube holds as soon as `Cube_Episode2` is completed, regardless of career.
- These 5 have higher priority than Single (sum ≥ 3600), so a daughter with very high stats still gets their couple ending as long as the matching event is completed.
- Single only happens when the stat sum is ≥ 3600 (average 300) and there is no companion; with stats short of that and a Jobless/Villager career it is an ordinary marriage. This 3600 is a constant read from the assembly, and the 12 stats are `EStatus` 1 to 12; I did not test it in the game.
- The 8 tribe companions (codes 0 to 7) must be accepted in the March 1218 proposal event for the code to become ≥ 0; the conditions are in "Each companion's proposal event" below.

## Comparison with the Bahamut guide: broadly matches, but many numbers differ

**Fully matching**: Ranch Manager, Research Assistant, Music Tutor, Dance Tutor, Magic Instructor, Historian, Magic Clerk (Intelligence+Magic+Spirit 1500, Magic Rating grade 4 = 250), Prima Ballerina (Dance A plus winning the Dance contest), Magic Knight (Strength and Magic 600, both ratings ≥ 250), Godfather (Strength 800), Adventurer (exploring the Dragon, Forest and Beastkin regions), the Queen's four sums (1500 / 1500 / 2400 / 1500), the Demon King's two-stat sum 2400, the Princess's two-stat sum 2400, the President of the Royal Academy of Arts' Sensitivity+Sense+Charm 2400 and Artistic Rating 350, and the Royal Wizard's Intelligence+Magic+Spirit 2400 and Magic Rating 350.

**Numbers that differ (the data is authoritative)**

| Ending | Bahamut guide says | Actual data |
|---|---|---|
| Princess | Social Rating ≥ 450 | **≥ 400** |
| Queen | Warrior and Social Rating both 500 | Social ≥ 500, **Warrior or Magic (either one) ≥ 400** |
| Demon King | Warrior and Magic Rating 500, Alignment -300 | The ending conditions have **no rating**, only the stat sum 2400; Alignment **≤ -225** (the event chain only needs ≤ -150) |
| President of the Royal Academy of Arts | Elegance+Manners+Speech ≥ 1500 | **No such condition**; but requires **winning the Art Festival** plus Art, Dance and Music at A |
| Royal Wizard | Magic and Alchemy at A | Both Magic Studies and Alchemy must be **S (grade 5)** |
| Great General | Stamina+Agility+Strength ≥ 2100, Elegance 200, Manners 300 | Stamina+Strength+Agility **≥ 2400**, Elegance+Manners **≥ 600**, and additionally **Swordsmanship S, Martial Arts S, winning the Martial Arts Tournament** |
| Diplomat | Intelligence > 750, three stats ≥ 2100, Social 500 | Intelligence **≥ 500**, Elegance+Manners+Speech **≥ 1800**, Social **≥ 350** |
| Salon Lobbyist | Intelligence > 750, each 700, Social 500 | Intelligence **≥ 500**, three-stat sum **≥ 1600**, Social **≥ 350**, Alignment ≤ -150 |
| Femme Fatale | Bar A, Stamina and Speech > 300 | Tavern at **S**, Charm 600, **no Stamina or Speech condition** |
| King of the Back Alleys | Warrior Rating ≥ 250 | **≥ 350**, Stamina 400, Strength 800 |
| Mysterious Thief | Intelligence 600 | Intelligence **≥ 300**, Agility 600, Sense 600, event Flow_MysteriousThief_Ep3 |
| Alchemist of Chaos | Magic > 600, Alchemy A, three events | Alchemy A, Intelligence+Magic+Spirit ≥ **1500**, Sense 300; **events not required** |
| Fortune Teller | Intelligence, Magic, Spirit, Speech 500 each, Magic A | Spirit, Speech, Sensitivity **400** each, Magic Rating grade ≥ 4, Alignment ≤ -150; **Magic A not required** |
| Sword Dancer | Charm and Strength > 600, Dance A, Swordsmanship A | Swordsmanship A, **Dance B**, Charm **≥ 500** |
| Etiquette Teacher | Etiquette B, Manners and Elegance > 300 | Etiquette **A**, Manners 400, Elegance 400, Sensitivity 200 |
| Magic Engineer | Alchemy A | Magic A, Alchemy A, **Intelligence+Magic+Spirit sum ≥ 2000** |
| Martial Arts Instructor | Strength 300, Spirit 300 | **Stamina** 300, Spirit 300 (Martial Arts B) |
| Bartender | Stamina and Speech > 300 | **Speech and Elegance** 300 each (Tavern A) |
| Maid Cafe Superstar | Sensitivity and Charm > 400, Social up two grades | Maid Cafe A, Charm 300, Sensitivity 300, Speech 200, **no Social Rating condition** |
| Rich | Money > 50,000 | Money ≥ 50000 **and** event Flow_Rich_Ep4 completed (both are required, see the correction below) |

**Endings that the Bahamut guide lacks but the data has**: Cook, Private Tutor, Clothing Designer, Soldier, Restaurant Server, Blacksmith, Logistics Handler (Fruit Warehouse), Logistics Manager (the forum's "Logistics Manager" is actually the Waterfront, corresponding to Logistics Manager), Capital Elite Corps, Academy Dean, Perfumer, Jeweler, Bard, Sigil Drafter, Prima Donna, Writer, Painter, Swordsmanship Dojo Instructor, Martial Arts Instructor, Housekeeping Agency Owner, Villager.

## Correction

I previously said Rich was "Money 50,000 **or** the event"; **that was wrong**. The two are in different condition groups, and under the consistent rule **both are required**: first complete the Flow_Rich event chain (Ep1 starting from Money ≥ 20000, Ep2 requires `Event_Rich_Invest` first and a 3-month gap, Ep3 a 4-month gap, Ep4 an 8-month gap), and at the ending Money must also be ≥ 50000.

## Quest conditions (Quest table)

- `Quest_RoyalWizard1`: defeat 20 monsters using magic. `Quest_RoyalWizard2`: win the Martial Arts Tournament using magic.
- `Quest_RAAPresident`: reach A (grade 4) in all three of Art, Music and Dance.
- `Quest_Adventurer1`: defeat the Dark Mage. `Quest_Adventurer2`: defeat the Desert Raiders.
- `Quest_Queen1`: visit the Acting Regent at the banquet. `Quest_Queen2`: visit the Acting Regent at the Demon King's Castle.

## Key events that require a choice (from the D_Flow dialogue graphs)

| Event | Choice to pick | Consequence of the wrong choice |
|---|---|---|
| Flow_DemonKing_Ep5 (February 1218, only once, MaxCount 1) | "I understand. Let's go together." → sets `Event_DemonKing_Allow` | Choosing "...I don't know yet." means the door never opens again and the Demon King ending is cut off |
| Flow_Rich_Ep1 | "Invest." (costs 5000 gold) → sets `Event_Rich_Invest` | Choosing "Don't invest" makes the Ep2 condition fail, cutting off the Rich chain |
| Flow_MysteriousThief_Ep1 | "...How interesting. I'll hide you." (Alignment -10) | Choosing "Hand over to the soldiers" gives 200 gold but sets `Event_MysteriousThief_Prison`; the Ep2 condition requires that this flag is **not** set, so the Mysterious Thief chain is cut off |
| Flow_Adventurer_Ep1 | "Accept." → sets `Quest_Adventurer1` | Choosing "Decline" means the quest needed by Ep2 is missing |
| Flow_TheGodfather_Ep1 / Ep2 | Must go toward Chaos: in Ep1 choose "Keep taking it out on them" (-20); in Ep2 choose "Accept the proposal" (-30) | Choosing "Hand over to the guards" gives +20 each; it does not affect the event chain but pushes Alignment toward Lawful, and the Godfather requires Alignment ≤ -150 |

## Princess route: Agares (DemonMale) proposal conditions (from the Partner event graph)

Fondness grades (`Cond_CompareFavorGrade`): 3 Fondness (20–29), 4 Affection (30–39), 5 Special (40–49), 6 Soulmate (50).

| Event | Condition |
|---|---|
| Episode1 | DemonMale_FirstEpisode completed, Ep1 not yet completed; game year ≤ 1213. Can trigger directly within 1213; before that, it requires a random 0–2 months after FirstEpisode |
| Episode2 | DemonMale_InterEpi2 completed, Ep2 not yet completed; year ≤ 1213, likewise can trigger directly within 1213, and before that a random 0–2 months |
| Episode3 | Ep2 completed; **lesson cumulative ≥ 2 times**; year ≤ 1213; afterwards same as above |
| Episode4 | Ep3 completed; Karen's age ≥ 14 |
| Episode5 | Ep4 completed; age ≥ 15; Fondness grade ≥ 3 (Fondness) |
| Episode6 | Ep5 completed; age ≥ 16; Fondness grade ≥ 3 |
| Episode7 | Ep6 completed; age ≥ 17; Fondness grade ≥ **4 (Affection)** |
| Episode8 | Ep7 completed (no other conditions) |
| **Proposal** | **Start of month in March 1218** (Trigger 1010); Ep8 completed; and **`Event_DemonKing_Allow` not set** |

In the proposal event, only choosing "Accept the proposal" executes `Act_SetWeddingNPC(DemonMale)` and sets **`DemonMale_Proposal_Allow`**; choosing "Decline the proposal" sets no flag, and the proposal event happens only once (MaxCount 1). If there is **already a wedding partner** when entering the event (`Cond_HasWeddingPartner`), the dialogue is skipped directly and no flag is set.

So:
- **The flag required for the Princess ending comes from accepting Agares's proposal in March 1218.**
- **Demon King and Princess are mutually exclusive**: if you chose "Let's go together" in Demon King Episode 5 (`Event_DemonKing_Allow`), the proposal will not happen.
- Fondness grade is only required at Ep5 to Ep7 as 3, 3, 4; **the data does not require reaching Soulmate (grade 6)**; the Bahamut guide's "raise to Soulmate" is only to be safe. Ep8 and the proposal itself have no Fondness threshold, but I did not check the content of Ep8, so it may have implicit restrictions.
- The proposal events of the other companions have been parsed (see the next section): likewise the start of month in March 1218; if another companion's proposal was accepted first, Agares's proposal is skipped because of "already has a wedding partner"; the only exception is the Forest Tribe Female, whose proposal does not check for an existing partner.

## Each companion's proposal event and story chain (from the Partner event graph)

**Rules common to proposal events** (8 companions in total: FurryMale / FurryFemale / ForestMale / ForestFemale / DragonMale / DragonFemale / DemonFemale / DemonMale):
- The timing is exactly the same: **start of month in March 1218** (Trigger 1010), requiring **`Event_DemonKing_Allow` not set**, and **that companion's last story event completed** (see the last row of the table below).
- All 8 proposal events have Priority 1 and MaxCount 1: each proposes only once, and choosing to decline uses it up.
- The dialogue flow is the same: upon entering the event it first checks `Cond_HasWeddingPartner`, and **if there is already a wedding partner it is skipped directly**; otherwise two options, "Accept / Decline", appear, and only choosing accept executes `Act_SetWeddingNPC(that companion)`.
- **The ForestFemale (Forest Tribe Female) dialogue graph has no `Cond_HasWeddingPartner` check**, so even if another's proposal was accepted first, her proposal still appears, and accepting **overwrites** the wedding partner (judging from the structure of the graph; not tested).
- **Only Agares's (DemonMale) proposal sets the flag `DemonMale_Proposal_Allow`**, so only he leads to the Princess ending; for the other 7, accepting the proposal only gives their own couple ending.
- The 7 non-Agares companions all **require no specific Fondness grade to propose**; Fondness only appears in the story chain (see the table below), and the proposal itself only looks at "the last episode of the story chain completed".

**When several companions qualify in the same month**: the 8 events all have Trigger 1010, Priority 1, MaxCount 1. According to the logic of `EventManager::sch`, at the same trigger point only the group with the "smallest positive priority" is put in the queue, and everything in the same group is queued, so the 8 proposal events queue together (in March there are also other Priority 1 events at 1010, such as Flow_Queen_Ep6, Flow_Rich_Ep1~4 and Flow_DemonKing_Ep2; if a Priority 0 event qualifies that month it is picked first and takes them away). The queue order is **inferred** to follow the row order of the Event table: ForestFemale → ForestMale → DragonFemale → DragonMale → FurryMale → FurryFemale → DemonFemale → DemonMale (this is an inference; the sorting of `jq::fca` was not verified). Each dialogue first checks `Cond_HasWeddingPartner`, and only "accept" sets the partner, so normally the one earlier in the queue whom you accept becomes the partner and the later ones are skipped (note that the ForestFemale dialogue lacks this check). The safe approach is still to develop only one companion in the same year.

### Beastkin Male

| Event | Trigger / Location | Condition |
|---|---|---|
| Episode1 | Schedule start / 208 | Daughter's age ≤ 13 and FurryMale Fondness grade ≥ 2 |
| Episode2 | 3301 / 1205 | Dialogue FurryMale_Episode1 completed |
| Episode3 | Schedule start / 204 | Dialogue FurryMale_Episode2 completed and FurryMale Fondness grade ≥ 3 |
| Episode4 | 3304 / 1204 | Dialogue FurryMale_InterEpi completed |
| Episode5 | 3301 / 1205 | Dialogue FurryMale_Ready completed |
| Episode6 | Start of month / 0 | Dialogue FurryMale_Episode5 completed and ≥ 0 years 2 months 0 days since the start of the month in which FurryMale_Episode5 was completed |
| **Proposal** | Start of month / — | NOT event Event_DemonKing_Allow completed and year = 1218 and month = 3 and companion schedule FurryMale_Episode6 completed |

### Beastkin Female

| Event | Trigger / Location | Condition |
|---|---|---|
| Episode1 | Before schedule end / 304 | FurryFemale Fondness grade ≥ 2 |
| Episode2 | Before schedule end / 304 | Dialogue FurryFemale_Episode1 completed |
| Episode3 | Before schedule end / 304 | FurryFemale Fondness grade ≥ 3 |
| Episode4 | 3301 / 1205 | Dialogue FurryFemale_Episode3 completed and ≥ 0 years 2 months 0 days since the start of the month in which FurryFemale_Episode3 was completed |
| Episode5 | Schedule start / 304 | Dialogue FurryFemale_Bag completed and FurryFemale Fondness grade ≥ 4 |
| Episode6 | 3301 / 1205 | Event FurryFemale_InterEpi completed |
| **Proposal** | Start of month / — | NOT event Event_DemonKing_Allow completed and year = 1218 and month = 3 and companion schedule FurryFemale_Episode6 completed |

### Forest Tribe Male

| Event | Trigger / Location | Condition |
|---|---|---|
| Episode1 | Schedule start / 203 | Event ForestMale_FirstEpisode completed and ForestMale Fondness grade ≥ 2 |
| Episode2 | Before schedule end / 304 | Dialogue ForestMale_Episode1 completed and ForestMale Fondness grade ≥ 2 |
| Episode3 | Schedule start / 306 | Dialogue ForestMale_Episode2 completed and ForestMale Fondness grade ≥ 3 |
| Episode4 | Visit (2005) / 503 | Dialogue ForestMale_Episode3 completed and daughter's age ≥ 14 and ForestMale Fondness grade ≥ 3 |
| Episode5 | 3202 / 1303 | Dialogue ForestMale_Episode4 completed and ForestMale Fondness grade ≥ 4 |
| Episode6 | 3202 / 1303 | Companion schedule ForestMale_InterEpi completed |
| **Proposal** | Start of month / — | NOT event Event_DemonKing_Allow completed and year = 1218 and month = 3 and companion schedule ForestMale_Episode6 completed |

### Forest Tribe Female

| Event | Trigger / Location | Condition |
|---|---|---|
| Episode1 | Schedule start / 206 | Daughter's age < 14 and dialogue ForestFemale_FirstEpisode completed and ForestFemale Fondness grade ≥ 2 |
| Episode2 | Schedule start / 206 | Dialogue ForestFemale_Episode1 completed |
| Episode3 | Before schedule end / 308 | Dialogue ForestFemale_Episode2 completed and ForestFemale Fondness grade ≥ 3 |
| Episode4 | Schedule start / 205 | Dialogue ForestFemale_Episode3 completed and daughter's age ≥ 14 |
| Episode5 | Schedule start / 206 | Dialogue ForestFemale_Episode4 completed and ForestFemale Fondness grade ≥ 4 |
| Episode6 | 3005 / 1301 | Dialogue ForestFemale_InterEpi completed |
| **Proposal** | Start of month / — | NOT event Event_DemonKing_Allow completed and year = 1218 and month = 3 and companion schedule ForestFemale_Episode6 completed |

### Dragon Tribe Male

| Event | Trigger / Location | Condition |
|---|---|---|
| Episode1 | 3103 / 1105 | Dialogue DragonMale_FirstEpisode completed and DragonMale Fondness grade ≥ 2 |
| Episode2 | 3101 / 1103 | Dialogue DragonMale_Episode1 completed |
| Episode3 | Schedule start / 302 | Dialogue DragonMale_Episode2 completed and > 0 years 3 months 0 days since the start of the month in which DragonMale_Episode2 was completed |
| Episode4 | 3103 / 1105 | Dialogue DragonMale_Episode3 completed and daughter's age ≥ 14 |
| Episode5 | Schedule start / 213 | Dialogue DragonMale_Episode4 completed and DragonMale Fondness grade ≥ 4 |
| Episode6 | 3101 / 1103 | Dialogue DragonMale_InterEpi completed |
| **Proposal** | Start of month / — | NOT event Event_DemonKing_Allow completed and year = 1218 and month = 3 and companion schedule DragonMale_Episode6 completed |

### Dragon Tribe Female

| Event | Trigger / Location | Condition |
|---|---|---|
| Episode1 | Schedule start / 305 | Dialogue DragonFemale_FirstEpisode completed and DragonFemale Fondness grade ≥ 2 |
| Episode2 | Schedule start / 305 | Dialogue DragonFemale_Episode1 completed |
| Episode3 | Schedule start / 307 | Dialogue DragonFemale_Episode2 completed and, since the start of the month after Episode2 was completed, the combined lesson count of Magic Studies and Etiquette ≥ 2 (the date offset parameter is 1 year 0 months 0 days; the exact interval is unconfirmed) |
| Episode4 | 3103 / 1105 | Dialogue DragonFemale_Episode3 completed |
| Episode5 | Schedule start / 305 | Dialogue DragonFemale_Episode4 completed and DragonFemale Fondness grade ≥ 4 |
| Episode6 | 3101 / 1103 | Dialogue DragonFemale_InterEpi completed |
| **Proposal** | Start of month / — | NOT event Event_DemonKing_Allow completed and year = 1218 and month = 3 and companion schedule DragonFemale_Episode6 completed |

### Demon Tribe Female (DemonFemale)

| Event | Trigger / Location | Condition |
|---|---|---|
| Episode1 | Schedule start / 303 | Dialogue DemonFemale_FirstEpisode completed and DemonFemale Fondness grade ≥ 2 |
| Episode2 | Schedule start / 212 | Maid Cafe cumulative count ≥ 1 and dialogue DemonFemale_Episode1 completed and event DemonFemale_Encounter_Dance completed |
| Episode3 | Meeting in the royal castle / 710 | Dialogue DemonFemale_Episode2 completed and DemonFemale Fondness grade ≥ 3 |
| Episode4 | Schedule start / 309 | Dialogue DemonFemale_Episode3 completed and DemonFemale Fondness grade ≥ 3 |
| Episode5 | Meeting in the royal castle / 710 | Dialogue DemonFemale_Episode4 completed and DemonFemale Fondness grade ≥ 4 and event DemonFemale_DemonEvent completed |
| Episode6 | Meeting in the royal castle / 710 | Dialogue DemonFemale_InterEpi completed |
| **Proposal** | Start of month / — | NOT event Event_DemonKing_Allow completed and year = 1218 and month = 3 and companion schedule DemonFemale_Episode6 completed |

### Cube and other NPCs (Cube, Caim, Icosa, Kimaris, Muzai)

These 5 have no proposal event and no "accept" dialogue graph. The graphs contain only story (`Episode1`; Cube has 1 and 2) and the wedding dialogue; becoming the marriage partner is decided at the ending by `pk::gde` from "which event was completed" plus the "career ending" (see "Final determination at the ending" above). Their story event conditions are as follows:

| Event | Trigger | Condition |
|---|---|---|
| Cube_Episode1 | 1002 (location 201 home) | Relationship with Cube ≥ 70 |
| Cube_Episode2 | 1002 (location 201 home) | Event Cube_Episode1 completed and Relationship with Cube ≥ 120 |
| Caim_Episode1 (Caim, restaurant manager) | 1017 latter part of a schedule, priority 30 | Event Job_Restaurant_Ep3 completed and this month's schedule is Restaurant |
| Icosa_Episode1 (Icosa, Magic Studies teacher) | 1017, priority 30 | Event Edu_Magic_Ep3 completed and this month's schedule is Magic Studies |
| Kimaris_Episode1 (Kimaris, Checkpoint) | 1017, priority 30 | Event Job_Checkpoint_Ep3 completed and this month's schedule is Checkpoint |
| Muzai_Episode1 (Muzai, Literature teacher) | 1017, priority 30 | Event Edu_Literature_Ep3 completed and this month's schedule is Literature |

The Episode1 of the four bosses/teachers can each trigger only once (MaxCount 1), and only **in the month in which that schedule is being carried out**, so you must first finish the corresponding `_Ep3` and then schedule the same activity again afterwards. "Relationship with Cube" is Status 21 (range -100 to 500). In the entire database and all flow graphs, only `E_Job_House_Ep1/2/3` (the three Housework job events) add to it: each stage gives +1 regardless of the option chosen, for a total of only 3, which cannot reach 70 / 120, so the main source of increase is not in the data or graphs and should be in the C# code (for example settled each time a housework-type schedule runs). Confirmed on the code side: `hl::sim(EStatus,int)` is a generic status add/subtract (limits taken from the Status table's Min/Max, -100 to 500 for Cube) with no special handling for Cube; `hl::dfs` is "set to a specified value", and its callers are mostly debugging or UI. On the data side: the `Schedule` table's `Job_House` uses StatType 21 (Cube, range 10 to 80) in the `NeedStatCritical` column, which is probably "the stat used for the great-success check" (inferred), not a source of Fondness increase; no other table references Cube by code 21. **The source has been found (confirmed in code)**: `ue::cez::MoveNext` (the schedule execution flow) executes `hl::sim(RelationshipCube, end day - start day + 1)` at the start of a schedule if that schedule's location is 201 (0xC9). The only schedule in the data tables with location 201 is `Job_House` (the Housework job), so **each time you do the Housework job, Relationship with Cube increases by "the number of days in that schedule"**, settled at the start of the schedule and independent of success, failure or great success. (The number of days in one schedule is decided by `mbg::sfv`: each month has 3 schedule slots, the first and second ten days each being 10 days, and the last being "days in the month − 20" days, i.e. 10 days for a 30-day month, 11 days for a 31-day month, and 8 to 9 days for February. So one Housework job usually gives +10: reaching 70 cumulatively takes 7 runs, and reaching 120 takes 12 runs; the +1 from each of the three events totals only +3 and does not save a single run. In other words, scheduling Housework in all three slots every month reaches 70 in a bit over 3 months (7 slots) and 120 in about 4 months (12 slots).) In addition, each of the three events `E_Job_House_Ep1/2/3` gives another +1. There are no other routes: `ue::ulx` (schedule settlement applies only the Stats column, with Job_House at 3 / 11 / 6, doubled on great success), gifts (`Partner_Npc_Cube`'s FavorStatus is 0), items / tea / meals, and the monthly settlement `uk::cdv` (which only touches Stamina, mind, fatigue and money) add nothing.

**"Ordinary marriage" and "single"**: `D_Village_Wedding`, `D_Job_Wedding` and `D_None_Wedding` correspond to -1, -3 and -2. No event sets them; `pk::gde` assigns the value at the ending according to rules 6 to 8 above.

## What is still missing

1. The flags `Event_DemonKing_Allow` and `DemonMale_Proposal_Allow` are set by dialogue choices, confirmed above.
2. The `Score` column is the weighting for Fortune Teller progress and `Weight` is the priority for the final decision, confirmed from the code.
3. The lesson/job cumulative value `lcq` is confirmed to be the cumulative count compared against `NeedCount_GradeUp`; the meaning of the `yaw` and `lcr` fields (`yaw` is used for Type 9 lesson counts) was not examined closely.
4. The order when several companions qualify for a proposal in the same month is unconfirmed; the source and number of days for "Relationship with Cube" are both confirmed (+10 per Housework job, and in the last slot +8 to 11 depending on the days in the month). The default marriage partner of -3 is confirmed to be written by the `do::do()` constructor; in addition `ei::hbo` (which shows a Korean message meaning "proposal-related data initialized", apparently a debug or reset tool) and `du::cga` / `du::lzk` (debug functions that apply ending conditions directly to the save) also write it, but normal play never reaches them.
5. Flow_Vacance (the Vacation events) was not parsed.
6. The scene names corresponding to the Place values of the royal castle meeting events (703 to 708) are in the code.

---

# Full conditions for each ending (by Weight, high to low)

## Queen (Queen)  Priority Weight 52000
**Ending conditions (all must be met)**
- Social Rating ≥ 500
- Any one of: Warrior Rating ≥ 400 or Magic Rating ≥ 400
- Stamina + Strength + Agility ≥ 1500
- Intelligence + Magic + Spirit ≥ 1500
- Elegance + Manners + Speech ≥ 2400
- Sensitivity + Sense + Charm ≥ 1500
- Alignment ≥ 225

**Prerequisite event chain**
- `Flow_Queen_Ep1`: times met Castle_RegentDeputy ≥ 3; Speech ≥ 200
- `Flow_Queen_Ep2`: event Flow_Queen_Ep1 completed; ≥ 0 years 2 months 0 days since the start of the month in which Flow_Queen_Ep1 was completed
- `Flow_Queen_Ep3`: event Flow_Queen_Ep2 completed; warrior-three (Stamina+Strength+Agility) sum ≥ 840; Warrior Rating ≥ 245
- `Flow_Queen_Ep3`: event Flow_Queen_Ep2 completed; warrior-three (Stamina+Strength+Agility) sum ≥ 840; [NOT] Warrior Rating ≥ 245; magic-three (Intelligence+Magic+Spirit) sum ≥ 840; Warrior Rating ≥ 245
- `Flow_Queen_Ep3`: event Flow_Queen_Ep2 completed; [NOT] warrior-three (Stamina+Strength+Agility) sum ≥ 840; magic-three (Intelligence+Magic+Spirit) sum ≥ 840; Warrior Rating ≥ 245
- `Flow_Queen_Ep4`: event Flow_Queen_Ep3 completed; ≥ 0 years 2 months 0 days since the start of the month in which Flow_Queen_Ep3 was completed
- `Flow_Queen_Ep5`: event Flow_Queen_Ep4 completed
- `Flow_Queen_Ep6`: event Flow_Queen_Ep5 completed; Warrior Rating ≥ 400; Social Rating ≥ 500; warrior-three (Stamina+Strength+Agility) sum ≥ 1500; magic-three (Intelligence+Magic+Spirit) sum ≥ 1500; social-three (Elegance+Manners+Speech) sum ≥ 2400; art-three (Sensitivity+Sense+Charm) sum ≥ 1500; Alignment ≥ 225
- `Flow_Queen_Ep6`: event Flow_Queen_Ep5 completed; [NOT] Warrior Rating ≥ 400; Magic Rating ≥ 400; Social Rating ≥ 500; warrior-three (Stamina+Strength+Agility) sum ≥ 1500; magic-three (Intelligence+Magic+Spirit) sum ≥ 1500; social-three (Elegance+Manners+Speech) sum ≥ 2400; art-three (Sensitivity+Sense+Charm) sum ≥ 1500; Alignment ≥ 225
- `Flow_Queen_Ep7`: event Flow_Queen_Ep6 completed

## Demon King (DemonKing)  Priority Weight 51000
**Ending conditions (all must be met)**
- Event Event_DemonKing_Allow completed
- Alignment ≤ -225
- Stamina + Strength + Agility ≥ 2400
- Intelligence + Magic + Spirit ≥ 2400

**Prerequisite event chain**
- `Flow_DemonKing_Ep1`: Alignment ≤ -150; warrior-three (Stamina+Strength+Agility) sum ≥ 800; magic-three (Intelligence+Magic+Spirit) sum ≥ 800
- `Flow_DemonKing_Ep2`: event Flow_DemonKing_Ep1 completed; Alignment ≤ -150; warrior-three (Stamina+Strength+Agility) sum ≥ 1200; magic-three (Intelligence+Magic+Spirit) sum ≥ 1200
- `Flow_DemonKing_Ep3`: event Flow_DemonKing_Ep2 completed
- `Flow_DemonKing_Ep4`: event Flow_DemonKing_Ep3 completed; Alignment ≤ -150; warrior-three (Stamina+Strength+Agility) sum ≥ 2000; magic-three (Intelligence+Magic+Spirit) sum ≥ 2000
- `Flow_DemonKing_Ep5`: event Flow_DemonKing_Ep4 completed; Alignment ≤ -225; warrior-three (Stamina+Strength+Agility) sum ≥ 2400; magic-three (Intelligence+Magic+Spirit) sum ≥ 2400; year = 1218; month = 2

## Princess (Princess)  Priority Weight 50000
**Ending conditions (all must be met)**
- Event DemonMale_Proposal_Allow completed
- Social Rating ≥ 400
- Artistic Rating ≥ 400
- Elegance + Manners + Speech ≥ 2400
- Sensitivity + Sense + Charm ≥ 2400
- Alignment ≥ -149

**Prerequisite event chain**
- `Flow_Princess_Ep1`: times met Castle_Consort ≥ 3; graph variable KarenRank = 1; dialogue DemonMale_Episode3 completed
- `Flow_Princess_Ep2`: dialogue Flow_Princess_Ep1 completed; Charm ≥ 400
- `Flow_Princess_Ep3`: dialogue Flow_Princess_Ep2 completed; social-three (Elegance+Manners+Speech) sum ≥ 2000
- `Flow_Princess_Ep4`: dialogue Flow_Princess_Ep3 completed
- `Flow_Princess_Ep5`: dialogue Flow_Princess_Ep4 completed
- `Flow_Princess_Ep6`: dialogue Flow_Princess_Ep5 completed; currently satisfies ending condition Ending_Princess
- `Flow_Princess_Ep7`: dialogue Flow_Princess_Ep6 completed
## Great General (General)  Priority Weight 43000
**Ending conditions (all must be met)**
- Win the Martial Arts Tournament
- Swordsmanship reaches S (level 5)
- Martial Arts reaches S (level 5)
- Warrior Rating ≥ 350
- Stamina + Strength + Agility ≥ 2400
- Elegance + Manners ≥ 600
- Event Flow_General_Ep3 completed
- Alignment ≥ -149

**Prerequisite event chain**
- `Flow_General_Ep0`: times met Castle_RegentDeputy > 0; Festival_Battle history matches [Rank=1&Weapon=Physical]; event Flow_CapitalEliteTroop_Ep2 completed; Alignment ≥ -149
- `Flow_General_Ep1`: event Flow_General_Ep0 completed
- `Flow_General_Ep2`: event Flow_General_Ep1 completed; Warrior Rating grade ≥ 3; Martial Arts level ≥ 5; Swordsmanship level ≥ 5; Alignment ≥ -149
- `Flow_General_Ep3`: event Flow_General_Ep2 completed

## Royal Wizard (RoyalWizard)  Priority Weight 42000
**Ending conditions (all must be met)**
- Alchemy reaches S (level 5)
- Magic Studies reaches S (level 5)
- Event Flow_RoyalWizard_Ep3 completed
- Magic Rating ≥ 350
- Intelligence + Magic + Spirit ≥ 2400
- Alignment ≥ -149

**Prerequisite event chain**
- `Flow_RoyalWizard_Ep1`: times met Castle_Archmage ≥ 1; Magic Rating grade > 4; Alchemy level ≥ 3; Magic Studies level ≥ 4
- `Flow_RoyalWizard_Ep2`: event Flow_RoyalWizard_Ep1 completed; quest condition met for Quest_RoyalWizard1
- `Flow_RoyalWizard_Ep3`: event Flow_RoyalWizard_Ep2 completed; quest condition met for Quest_RoyalWizard2

## President of the Royal Academy of Arts (RAAPresident)  Priority Weight 41000
**Ending conditions (all must be met)**
- Art reaches A (level 4)
- Dance reaches A (level 4)
- Music reaches A (level 4)
- Artistic Rating ≥ 350
- Sensitivity + Sense + Charm ≥ 2400
- Win the Art Festival
- Alignment ≥ -149

**Prerequisite event chain**
- `Flow_RAAPresident_Ep1`: times met Castle_Consort > 2; [NOT] quest Quest_RAAPresident completed; Festival_Art history matches [Rank=1]
- `Flow_RAAPresident_Ep2`: quest Quest_RAAPresident completed

## Rich (Rich)  Priority Weight 40000
**Ending conditions (all must be met)**
- Money held ≥ 50000
- Event Flow_Rich_Ep4 completed

**Prerequisite event chain**
- `Flow_Rich_Ep1`: money held ≥ 20000
- `Flow_Rich_Ep2`: event Event_Rich_Invest completed; ≥ 0 years 3 months 0 days since the start of the month in which Flow_Rich_Ep1 was completed
- `Flow_Rich_Ep3`: event Flow_Rich_Ep2 completed; ≥ 0 years 4 months 0 days since the start of the month in which Flow_Rich_Ep2 was completed
- `Flow_Rich_Ep4`: event Flow_Rich_Ep3 completed; ≥ 0 years 8 months 0 days since the start of the month in which Flow_Rich_Ep3 was completed

## Magic Knight (MagicKnight)  Priority Weight 40000
**Ending conditions (all must be met)**
- Warrior Rating grade ≥ 4 (rating ≥ 250)
- Magic Rating grade ≥ 4 (rating ≥ 250)
- Event Flow_MagicKnight_Ep1 completed
- Magic ≥ 600
- Strength ≥ 600

**Prerequisite event chain**
- `Flow_MagicKnight_Ep1`: Warrior Rating ≥ 2; Magic Studies level ≥ 3; matches [Rank!=1&Weapon=Physical]
- `Flow_MagicKnight_Ep2`: event Flow_MagicKnight_Ep1 completed; matches [Rank=1]

## Diplomat (Diplomat)  Priority Weight 40000
**Ending conditions (all must be met)**
- Event Flow_Diplomat_Ep3 completed
- Social Rating ≥ 350
- Elegance + Manners + Speech ≥ 1800
- Intelligence ≥ 500
- Alignment ≥ -149

**Prerequisite event chain**
- `Flow_Diplomat_Ep1`: social-three (Elegance+Manners+Speech) sum ≥ 1200; Social Rating grade ≥ 3; Intelligence ≥ 100; Alignment ≥ -149
- `Flow_Diplomat_Ep2`: event Flow_Diplomat_Ep1 completed
- `Flow_Diplomat_Ep3`: event Flow_Diplomat_Ep2 completed; social-three (Elegance+Manners+Speech) sum ≥ 1500; Intelligence ≥ 300

## Salon Lobbyist (Sting)  Priority Weight 34000
**Ending conditions (all must be met)**
- Alignment ≤ -150
- Social Rating ≥ 350
- Elegance + Manners + Speech ≥ 1600
- Intelligence ≥ 500

**Prerequisite event chain**
- `Flow_Sting_Ep1`: Social Rating grade ≥ 3
- `Flow_Sting_Ep2`: event Flow_Sting_Ep1 completed; Alignment ≤ -150
- `Flow_Sting_Ep3`: event Flow_Sting_Ep2 completed; Intelligence > 400; Alignment ≤ -150

## Femme Fatale (Femme)  Priority Weight 33000
**Ending conditions (all must be met)**
- Alignment ≤ -150
- Tavern reaches S (level 5)
- Charm ≥ 600
- Charm ≥ 400
- Charm ≥ 600

**Prerequisite event chain**
- `Flow_Femme_Ep1`: Tavern cumulative count ≥ 10; Alignment ≤ -90; Charm > 100
- `Flow_Femme_Ep2`: event Flow_Femme_Ep1 completed; Tavern level ≥ 4; Charm > 300; Alignment ≤ -120
- `Flow_Femme_Ep3`: event Flow_Femme_Ep2 completed; Tavern cumulative count ≥ 25; Charm > 500; Alignment ≤ -150

## Alchemist of Chaos (ChaosAlchemist)  Priority Weight 32000
**Ending conditions (all must be met)**
- Alignment ≤ -150
- Alchemy reaches A (level 4)
- Intelligence + Magic + Spirit ≥ 1500
- Sense ≥ 300

**Prerequisite event chain**
- `Flow_ChaosAlchemist_Ep1`: Alchemy level ≥ 3
- `Flow_ChaosAlchemist_Ep2`: event Flow_ChaosAlchemist_Ep1 completed; Alchemy level ≥ 4; Alignment ≤ -10
- `Flow_ChaosAlchemist_Ep3`: event Flow_ChaosAlchemist_Ep2 completed; Alignment ≤ -90

## Mysterious Thief (MysteriousThief)  Priority Weight 32000
**Ending conditions (all must be met)**
- Alignment ≤ -150
- Intelligence ≥ 300
- Agility ≥ 600
- Sense ≥ 600
- Event Flow_MysteriousThief_Ep3 completed

**Prerequisite event chain**
- `Flow_MysteriousThief_Ep1`: Agility ≥ 400
- `Flow_MysteriousThief_Ep2`: [NOT] event Event_MysteriousThief_Prison completed; event Flow_MysteriousThief_Ep1 completed; Sense ≥ 400; Alignment ≤ -150
- `Flow_MysteriousThief_Ep3`: event Flow_MysteriousThief_Ep2 completed; Intelligence ≥ 200; Alignment ≤ -150

## Fortune Teller (FortuneTeller)  Priority Weight 31000
**Ending conditions (all must be met)**
- Magic Rating grade ≥ 4 (rating ≥ 250)
- Alignment ≤ -150
- Spirit ≥ 400
- Speech ≥ 400
- Sensitivity ≥ 400

**Prerequisite event chain**
- `Flow_FortuneTeller_Ep1`: month ≠ 4; Speech > 200; Sensitivity > 200; Spirit > 200; Alignment ≤ -150
- `Flow_FortuneTeller_Ep2`: month ≠ 4; Speech > 300; Spirit > 300; Sensitivity > 300; Alignment ≤ -150

## King of the Back Alleys (TheGodfather)  Priority Weight 31000
**Ending conditions (all must be met)**
- Alignment ≤ -150
- Strength ≥ 800
- Stamina ≥ 400
- Warrior Rating ≥ 350

**Prerequisite event chain**
- `Flow_TheGodfather_Ep1`: event Flow_UnloadingManager_Ep1 completed; Warrior Rating grade ≥ 2
- `Flow_TheGodfather_Ep2`: event Flow_TheGodfather_Ep1 completed; [NOT] event Flow_TheGodfather2_Ep2 completed; Warrior Rating grade ≥ 3

## Magic Engineer (MagicEngineer)  Priority Weight 27000
**Ending conditions (all must be met)**
- Magic Studies reaches A (level 4)
- Alchemy reaches A (level 4)
- Intelligence + Magic + Spirit ≥ 2000

**Prerequisite event chain**
- `Flow_MagicEngineer_Ep1`: this month's schedule is Alchemy; Magic Studies cumulative count ≥ 13; Alchemy cumulative count ≥ 13; magic-three (Intelligence+Magic+Spirit) sum ≥ 1500
- `Flow_MagicEngineer_Ep2`: this month's schedule is Alchemy; Magic Studies cumulative count ≥ 18; Alchemy cumulative count ≥ 18; magic-three (Intelligence+Magic+Spirit) sum ≥ 1800

## Sword Dancer (Sworddancer)  Priority Weight 27000
**Ending conditions (all must be met)**
- Swordsmanship reaches A (level 4)
- Dance reaches B (level 3)
- Charm ≥ 500

**Prerequisite event chain**
- `Flow_Sworddancer_Ep1`: this month's schedule is Swordsmanship; Swordsmanship cumulative count ≥ 12; Dance cumulative count ≥ 8; Agility ≥ 150; event StatCheck_Sword_Ep1 completed
- `Flow_Sworddancer_Ep2`: this month's schedule is Dance; Swordsmanship cumulative count ≥ 16; Dance cumulative count ≥ 10; Charm ≥ 450; event Flow_Sworddancer_Ep1 completed

## Perfumer (Perfumer)  Priority Weight 27000
**Ending conditions (all must be met)**
- Botanical Garden reaches A (level 4)
- Alchemy reaches B (level 3)
- Elegance ≥ 500

**Prerequisite event chain**
- `Flow_Perfumer_Ep1`: this month's schedule is Botanical Garden; Botanical Garden cumulative count ≥ 15; Alchemy cumulative count ≥ 8; Sensitivity ≥ 150; event StatCheck_Garden_Ep1 completed
- `Flow_Perfumer_Ep2`: this month's schedule is Alchemy; Botanical Garden cumulative count ≥ 25; Alchemy cumulative count ≥ 10; Elegance ≥ 450; event Flow_Perfumer_Ep1 completed

## Jeweler (Jeweler)  Priority Weight 27000
**Ending conditions (all must be met)**
- Blacksmith reaches A (level 4)
- Art reaches B (level 3)
- Artistic Rating ≥ 100
- Elegance ≥ 500

**Prerequisite event chain**
- `Flow_Jeweler_Ep1`: this month's schedule is Blacksmith; Blacksmith cumulative count ≥ 12; Art cumulative count ≥ 8; Stamina ≥ 150
- `Flow_Jeweler_Ep2`: this month's schedule is Blacksmith; Blacksmith cumulative count ≥ 16; Art cumulative count ≥ 10; Artistic Rating ≥ 100; Elegance ≥ 450; event Flow_Jeweler_Ep1 completed

## Bard (Bard)  Priority Weight 27000
**Ending conditions (all must be met)**
- Music reaches A (level 4)
- Literature reaches B (level 3)
- Event Flow_Bard_Ep3 completed
- Speech ≥ 500

**Prerequisite event chain**
- `Flow_Bard_Ep1`: this month's schedule is Literature; Literature cumulative count ≥ 8; Music cumulative count ≥ 10; Sensitivity ≥ 150
- `Flow_Bard_Ep2`: this month's schedule is Music; Music cumulative count ≥ 16; Literature cumulative count ≥ 10; Speech ≥ 450; event Flow_Bard_Ep1 completed
- `Flow_Bard_Ep3`: Speech ≥ 450; event Flow_Bard_Ep2 completed

## Sigil Drafter (SigilDrafter)  Priority Weight 27000
**Ending conditions (all must be met)**
- Magic Studies reaches A (level 4)
- Art reaches B (level 3)
- Artistic Rating ≥ 100
- Sensitivity ≥ 500

**Prerequisite event chain**
- `Flow_SigilDrafter_Ep1`: this month's schedule is Magic Studies; Magic Studies cumulative count ≥ 12; Art cumulative count ≥ 8; Magic ≥ 150
- `Flow_SigilDrafter_Ep2`: this month's schedule is Art; Magic Studies cumulative count ≥ 16; Art cumulative count ≥ 10; Artistic Rating ≥ 100; Sensitivity ≥ 450; event Flow_SigilDrafter_Ep1 completed

## Prima Ballerina (Dancer)  Priority Weight 23000
**Ending conditions (all must be met)**
- Dance reaches A (level 4)
- Charm ≥ 300
- Sense ≥ 300
- Elegance ≥ 300
- Win the Art Festival dance competition

**Prerequisite event chain**
- `Flow_Dancer_Ep1`: this month's schedule is Dance; Dance cumulative count ≥ 22; Elegance ≥ 100
- `Flow_Dancer_Ep2`: this month's schedule is Dance; Dance cumulative count ≥ 27; Elegance ≥ 200

## Prima Donna (PrimaDonna)  Priority Weight 23000
**Ending conditions (all must be met)**
- Music reaches A (level 4)
- Sensitivity ≥ 300
- Charm ≥ 300
- Speech ≥ 300
- Win the Art Festival music competition

**Prerequisite event chain**
- `Flow_PrimaDonna_Ep1`: this month's schedule is Music; Music cumulative count ≥ 22; Speech ≥ 100
- `Flow_PrimaDonna_Ep2`: this month's schedule is Music; Music cumulative count ≥ 27; Speech ≥ 200
## Academy Dean (AcademyDean)  Priority Weight 23000
**Ending conditions (all must be met)**
- Academy reaches A (level 4)
- Intelligence ≥ 500
- Speech ≥ 300

**Prerequisite event chain**
- `Flow_AcademyDean_Ep1`: this month's schedule is Academy; Academy cumulative count ≥ 23; Intelligence ≥ 200; Sense ≥ 150
- `Flow_AcademyDean_Ep2`: this month's schedule is Academy; Academy cumulative count ≥ 28; Intelligence ≥ 300; event Flow_AcademyDean_Ep1 completed
- `Flow_AcademyDean_Ep3`: this month's schedule is Academy; Intelligence ≥ 450; Sense ≥ 200; Speech ≥ 270; event Flow_AcademyDean_Ep2 completed

## Cook (Cook)  Priority Weight 22500
**Ending conditions (all must be met)**
- Restaurant reaches S (level 5)
- Won first place at the Cooking Festival
- Sense ≥ 500
- Magic ≥ 200

**Prerequisite event chain**
- `Flow_Cook_Ep1`: this month's schedule is Restaurant; Restaurant cumulative count ≥ 38
- `Flow_Cook_Ep2`: this month's schedule is Restaurant; Restaurant cumulative count ≥ 45; Festival_Cook win count ≥ 1

## Magic Clerk (MagicClerk)  Priority Weight 20900
**Ending conditions (all must be met)**
- Magic Rating rank ≥ 4 (rating ≥ 250)
- Magic Studies reaches A (level 4)
- Intelligence + Magic + Spirit ≥ 1500
- Event Flow_MagicClerk_Ep2 completed

**Prerequisite event chain**
- `Flow_MagicClerk_Ep1`: this month's schedule is Magic Studies; Magic Studies cumulative count ≥ 10; times met Castle_MagicClerk ≥ 3
- `Flow_MagicClerk_Ep2`: event Flow_MagicClerk_Ep1 completed; Magic Studies cumulative count ≥ 10; times met Castle_MagicClerk ≥ 3

## Capital Elite Corps (CapitalEliteTroop)  Priority Weight 20850
**Ending conditions (all must be met)**
- Warrior Rating rank ≥ 4 (rating ≥ 250)
- Alignment ≥ 30
- Stamina + Strength + Agility ≥ 1500

**Prerequisite event chain**
- `Flow_CapitalEliteTroop_Ep1`: this month's schedule is Free Time; Swordsmanship level ≥ 3
- `Flow_CapitalEliteTroop_Ep2`: event Flow_CapitalEliteTroop_Ep1 completed; Strength > 400

## Adventurer (Adventurer)  Priority Weight 20700
**Ending conditions (all must be met)**
- Warrior Rating rank ≥ 4 (rating ≥ 250)
- Stamina ≥ 400
- Strength ≥ 400
- Has explored Dragon Tribe territory
- Has explored Forest Tribe territory
- Has explored Beastkin territory

**Prerequisite event chain**
- `Flow_Adventurer_Ep1`: monsters killed (total) > 10
- `Flow_Adventurer_Ep2`: event Flow_Adventurer_Ep1 completed; quest condition met Quest_Adventurer1
- `Flow_Adventurer_Ep3`: event Flow_Adventurer_Ep2 completed; Warrior Rating > 3
- `Flow_Adventurer_Ep4`: event Flow_Adventurer_Ep3 completed; quest condition met Quest_Adventurer2

## Swordsmanship Dojo Instructor (SwordInstructor)  Priority Weight 20495
**Ending conditions (all must be met)**
- Swordsmanship reaches B (level 3)
- Agility ≥ 300
- Stamina ≥ 300

**Prerequisite event chain**
- `Flow_SwordInstructor_Ep1`: this month's schedule is Swordsmanship; Swordsmanship cumulative count ≥ 12
- `Flow_SwordInstructor_Ep2`: this month's schedule is Swordsmanship; Swordsmanship cumulative count ≥ 20; Agility ≥ 270; Stamina ≥ 270

## Writer (Writer)  Priority Weight 20495
**Ending conditions (all must be met)**
- Literature reaches B (level 3)
- Speech ≥ 300
- Sensitivity ≥ 300

**Prerequisite event chain**
- `Flow_Writer_Ep1`: this month's schedule is Literature; Literature cumulative count ≥ 12
- `Flow_Writer_Ep2`: this month's schedule is Literature; Literature cumulative count ≥ 20; Speech ≥ 270; Sensitivity ≥ 270

## Painter (Painter)  Priority Weight 20495
**Ending conditions (all must be met)**
- Art reaches B (level 3)
- Sense ≥ 300
- Sensitivity ≥ 300

**Prerequisite event chain**
- `Flow_Painter_Ep1`: this month's schedule is Art; Art cumulative count ≥ 12
- `Flow_Painter_Ep2`: this month's schedule is Art; Art cumulative count ≥ 20; Sense ≥ 270; Sensitivity ≥ 270

## Dance Tutor (DanceTutor)  Priority Weight 20495
**Ending conditions (all must be met)**
- Dance reaches B (level 3)
- Charm ≥ 300
- Sense ≥ 300

**Prerequisite event chain**
- `Flow_DanceTutor_Ep1`: this month's schedule is Dance; Dance cumulative count ≥ 12
- `Flow_DanceTutor_Ep2`: this month's schedule is Dance; Dance cumulative count ≥ 20; Charm ≥ 270; Sense ≥ 270

## Music Tutor (MusicTutor)  Priority Weight 20495
**Ending conditions (all must be met)**
- Music reaches B (level 3)
- Sensitivity ≥ 300
- Charm ≥ 300

**Prerequisite event chain**
- `Flow_MusicTutor_Ep1`: this month's schedule is Music; Music cumulative count ≥ 12
- `Flow_MusicTutor_Ep2`: this month's schedule is Music; Music cumulative count ≥ 20; Sensitivity ≥ 270; Charm ≥ 270

## Historian (Historian)  Priority Weight 20495
**Ending conditions (all must be met)**
- History reaches B (level 3)
- Intelligence ≥ 300
- Elegance ≥ 300

**Prerequisite event chain**
- `Flow_Historian_Ep1`: this month's schedule is History; History cumulative count ≥ 12
- `Flow_Historian_Ep2`: this month's schedule is History; History cumulative count ≥ 20; Intelligence ≥ 270; Elegance ≥ 270

## Magic Instructor (MagicTutor)  Priority Weight 20495
**Ending conditions (all must be met)**
- Magic Studies reaches B (level 3)
- Magic ≥ 300
- Intelligence ≥ 300

**Prerequisite event chain**
- `Flow_MagicTutor_Ep1`: this month's schedule is Magic Studies; Magic Studies cumulative count ≥ 12
- `Flow_MagicTutor_Ep2`: this month's schedule is Magic Studies; Magic Studies cumulative count ≥ 20; Intelligence ≥ 270; Magic ≥ 270

## Etiquette Teacher (Governess)  Priority Weight 20495
**Ending conditions (all must be met)**
- Etiquette reaches A (level 4)
- Manners ≥ 400
- Elegance ≥ 400
- Sensitivity ≥ 200

**Prerequisite event chain**
- `Flow_Governess_Ep1`: this month's schedule is Etiquette; Etiquette cumulative count ≥ 12
- `Flow_Governess_Ep2`: this month's schedule is Etiquette; Etiquette cumulative count ≥ 20; Manners ≥ 350; Elegance ≥ 350

## Martial Arts Instructor (MartialArtsInstructor)  Priority Weight 20495
**Ending conditions (all must be met)**
- Martial Arts reaches B (level 3)
- Spirit ≥ 300
- Stamina ≥ 300

**Prerequisite event chain**
- `Flow_MartialArtsInstructor_Ep1`: this month's schedule is Martial Arts; Martial Arts cumulative count ≥ 12; Spirit ≥ 100; event StatCheck_MartialArts_Ep1 completed
- `Flow_MartialArtsInstructor_Ep2`: this month's schedule is Martial Arts; Martial Arts cumulative count ≥ 20; Spirit ≥ 270; Stamina ≥ 270; event Flow_MartialArtsInstructor_Ep1 completed

## Alchemist (Alchemist)  Priority Weight 20495
**Ending conditions (all must be met)**
- Alchemy reaches B (level 3)
- Intelligence ≥ 300
- Magic ≥ 300

**Prerequisite event chain**
- `Flow_Alchemist_Ep1`: this month's schedule is Alchemy; Alchemy cumulative count ≥ 12; Intelligence ≥ 100
- `Flow_Alchemist_Ep2`: this month's schedule is Alchemy; Alchemy cumulative count ≥ 20; Intelligence ≥ 270; Magic ≥ 270; event Flow_Alchemist_Ep1 completed

## Logistics Manager (UnloadingManager)  Priority Weight 20240
**Ending conditions (all must be met)**
- Waterfront reaches A (level 4)
- Stamina ≥ 300
- Spirit ≥ 300

**Prerequisite event chain**
- `Flow_UnloadingManager_Ep1`: this month's schedule is Waterfront; Waterfront cumulative count ≥ 10
- `Flow_UnloadingManager_Ep2`: this month's schedule is Waterfront; Waterfront level ≥ 4; Stamina ≥ 270; Spirit ≥ 270

## Bartender (Bartender)  Priority Weight 20240
**Ending conditions (all must be met)**
- Tavern reaches A (level 4)
- Speech ≥ 300
- Elegance ≥ 300

**Prerequisite event chain**
- `Flow_Bartender_Ep1`: this month's schedule is Tavern; Tavern cumulative count ≥ 10
- `Flow_Bartender_Ep2`: this month's schedule is Tavern; Tavern level ≥ 4; Speech ≥ 270; Elegance ≥ 270

## Private Tutor (PrivateTutor)  Priority Weight 20200
**Ending conditions (all must be met)**
- Tutor reaches A (level 4)
- Intelligence ≥ 300
- Manners ≥ 300

**Prerequisite event chain**
- `Flow_PrivateTutor_Ep1`: this month's schedule is Tutor; Tutor cumulative count ≥ 10
- `Flow_PrivateTutor_Ep2`: this month's schedule is Tutor; Tutor level ≥ 4; Intelligence ≥ 270; Manners ≥ 270

## Maid Cafe Superstar (Superstar)  Priority Weight 20200
**Ending conditions (all must be met)**
- Maid Cafe reaches A (level 4)
- Charm ≥ 300
- Sensitivity ≥ 300
- Speech ≥ 200

**Prerequisite event chain**
- `Flow_Superstar_Ep1`: this month's schedule is Maid Cafe; Maid Cafe cumulative count ≥ 10; Charm ≥ 300; Sensitivity ≥ 150
- `Flow_Superstar_Ep2`: this month's schedule is Maid Cafe; Maid Cafe level ≥ 4; Charm ≥ 500; Sensitivity ≥ 300; Speech ≥ 150

## Research Assistant (Assistant)  Priority Weight 20200
**Ending conditions (all must be met)**
- Academy reaches B (level 3)
- Intelligence ≥ 300
- Sense ≥ 300

**Prerequisite event chain**
- `Flow_Assistant_Ep1`: this month's schedule is Academy; Academy cumulative count ≥ 10
- `Flow_Assistant_Ep2`: this month's schedule is Academy; Academy cumulative count ≥ 19; Intelligence ≥ 270; Sense ≥ 270

## Clothing Designer (Designer)  Priority Weight 20160
**Ending conditions (all must be met)**
- Clothing Store reaches A (level 4)
- Elegance ≥ 300
- Charm ≥ 300

**Prerequisite event chain**
- `Flow_Designer_Ep1`: this month's schedule is Clothing Store; Clothing Store cumulative count ≥ 11
- `Flow_Designer_Ep2`: this month's schedule is Clothing Store; Clothing Store cumulative count ≥ 24; Elegance ≥ 280; Charm ≥ 280

## Blacksmith (Blacksmith)  Priority Weight 20160
**Ending conditions (all must be met)**
- Blacksmith reaches A (level 4)
- Stamina ≥ 300
- Magic ≥ 300

**Prerequisite event chain**
- `Flow_Blacksmith_Ep1`: this month's schedule is Blacksmith; Blacksmith cumulative count ≥ 12
- `Flow_Blacksmith_Ep2`: this month's schedule is Blacksmith; Blacksmith cumulative count ≥ 24; Stamina ≥ 270; Magic ≥ 270

## Soldier (Soldier)  Priority Weight 20080
**Ending conditions (all must be met)**
- Checkpoint reaches A (level 4)
- Manners ≥ 300
- Stamina ≥ 300

**Prerequisite event chain**
- `Flow_Soldier_Ep1`: this month's schedule is Checkpoint; Checkpoint cumulative count ≥ 11
- `Flow_Soldier_Ep2`: this month's schedule is Checkpoint; Checkpoint level ≥ 4; Manners ≥ 270; Stamina ≥ 270

## Logistics Handler (Logistics)  Priority Weight 20080
**Ending conditions (all must be met)**
- Fruit Warehouse reaches A (level 4)
- Strength ≥ 300
- Spirit ≥ 300

**Prerequisite event chain**
- `Flow_Logistics_Ep1`: this month's schedule is Fruit Warehouse; Fruit Warehouse cumulative count ≥ 13
- `Flow_Logistics_Ep2`: this month's schedule is Fruit Warehouse; Fruit Warehouse level ≥ 4; Strength ≥ 270; Spirit ≥ 270

## Housekeeping Agency Owner (Housekeeper)  Priority Weight 20000
**Ending conditions (all must be met)**
- Housework reaches A (level 4)
- Intelligence ≥ 250

**Prerequisite event chain**
- `Flow_Housekeeper_Ep1`: this month's schedule is Housework; Housework cumulative count ≥ 18
- `Flow_Housekeeper_Ep2`: this month's schedule is Housework; Housework cumulative count ≥ 38

## Village Healer (Doctor)  Priority Weight 20000
**Ending conditions (all must be met)**
- Infirmary reaches B (level 3)
- Spirit ≥ 300
- Stamina ≥ 300

**Prerequisite event chain**
- `Flow_Doctor_Ep1`: this month's schedule is Infirmary; Infirmary cumulative count ≥ 18
- `Flow_Doctor_Ep2`: this month's schedule is Infirmary; Infirmary cumulative count ≥ 35; Stamina ≥ 270; Spirit ≥ 270

## Carpenter (Carpenter)  Priority Weight 20000
**Ending conditions (all must be met)**
- Woodworking Workshop reaches B (level 3)
- Strength ≥ 300
- Sense ≥ 300

**Prerequisite event chain**
- `Flow_Carpenter_Ep1`: this month's schedule is Woodworking Workshop; Woodworking Workshop cumulative count ≥ 18
- `Flow_Carpenter_Ep2`: this month's schedule is Woodworking Workshop; Woodworking Workshop cumulative count ≥ 35; Strength ≥ 270; Sense ≥ 270

## Restaurant Server (Waitress)  Priority Weight 20000
**Ending conditions (all must be met)**
- Restaurant reaches B (level 3)
- Sense ≥ 300
- Speech ≥ 300

**Prerequisite event chain**
- `Flow_Waitress_Ep1`: this month's schedule is Restaurant; Restaurant cumulative count ≥ 18
- `Flow_Waitress_Ep2`: this month's schedule is Restaurant; Restaurant cumulative count ≥ 30

## Ranch Manager (RanchManager)  Priority Weight 20000
**Ending conditions (all must be met)**
- Ranch reaches B (level 3)
- Stamina ≥ 300
- Sensitivity ≥ 300

**Prerequisite event chain**
- `Flow_RanchManager_Ep1`: this month's schedule is Ranch; Ranch cumulative count ≥ 18
- `Flow_RanchManager_Ep2`: this month's schedule is Ranch; Ranch cumulative count ≥ 35

## Horticultural Therapist (Therapist)  Priority Weight 20000
**Ending conditions (all must be met)**
- Botanical Garden reaches B (level 3)
- Sensitivity ≥ 300
- Magic ≥ 300

**Prerequisite event chain**
- `Flow_Therapist_Ep1`: this month's schedule is Botanical Garden; Botanical Garden cumulative count ≥ 18
- `Flow_Therapist_Ep2`: this month's schedule is Botanical Garden; Botanical Garden cumulative count ≥ 35; Sensitivity ≥ 270; Magic ≥ 270

## Villager (Villager)  Priority Weight 2
**Ending conditions (all must be met)**
- Manners ≥ 100

**Prerequisite event chain**
- `Flow_Villager_Ep1`: this month's schedule is Housework; Housework cumulative count ≥ 15
- `Flow_Villager_Ep2`: this month's schedule is Housework; Housework cumulative count ≥ 29

## Jobless (Jobless)  Priority Weight 1
**Ending conditions (all must be met)**
- (No conditions)

**Prerequisite event chain**
- `Flow_Jobless_Ep1`: this month's schedule is Rest at Home; year ≤ 1210; Rest count ≥ 15
- `Flow_Jobless_Ep2`: this month's schedule is Rest at Home; year ≤ 1211; Rest count ≥ 30
