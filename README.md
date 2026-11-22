# Princess Maker: Children of Revelation (預言之子) — Ending Conditions

**English** | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

Ending conditions compiled from the game's own data tables, using data extracted from **game version v1.1.0**.
The data was fed to an AI for analysis, and the Bahamut forum guide ("per-ending parameters") was also given to the AI as a reference and cross-checked against.

Last updated: 2026-11-21

> Unofficial fan project. Not affiliated with the game's developer or publisher. No game assets are included; this repository only contains written analysis.

## Sources

- `TableData`: `EndingStatus`, `Ending`, `Reputation`, `Quest`, `Event`, `Schedule`, `Partner`, `Status`
- Event scripts (`C_Flow_*`, `D_*`, `E_*`, `Partner/*`) exported as Visual Scripting graphs
- Decompiled game code (`GameAssembly`) to confirm how the game actually evaluates the data

## What is in this repository

| File | Content |
|---|---|
| [`docs/GUIDE.en.md`](docs/GUIDE.en.md) | The full guide: every ending's complete conditions ordered by priority, partner proposal chains, quests, and corrections to the forum guide. Also available as [繁體中文](docs/GUIDE.zh-TW.md) (original), [简体中文](docs/GUIDE.zh-CN.md), [日本語](docs/GUIDE.ja.md), [한국어](docs/GUIDE.ko.md) |
| [`docs/GLOSSARY.md`](docs/GLOSSARY.md) | Official names of endings, partners and statuses in 5 languages (from the game's `Text` table) |

> Only the Traditional Chinese guide is hand-checked against the game data; the other language guides were translated from it with AI, using the official names from the game's `Text` table. If they disagree, trust the Traditional Chinese one.

## How endings are decided (confirmed from the game code)

1. **Candidates.** Only the 55 normal endings (`EndingType = 2`) are considered. An ending is a candidate only if **every condition group has at least one satisfied condition** (OR inside a group, AND between groups).
2. **Score.** Score = `Weight` + bonus. Only the highest score stays; **ties are broken randomly**.
3. **Bonus.** Endings with `Weight` ≥ 30000 get no bonus. Below that, the bonus is `min(2 × cumulative count of the related lesson/job, 8000)`, which in practice is a few hundred. It can reorder endings with close weights but never jumps a weight tier.
4. **Examples.** Queen 52000, Demon King 51000, Princess 50000 (official English names from the game's text table). Rich, Magic Knight and Diplomat are all 40000 with no bonus, so they are chosen at random when all qualify.

Reputation tiers (reputation grade): 1 = 0, 2 = 50, 3 = 150, 4 = 250, 5 = 350, 6 = 500. Lesson/job grade: 3 = B, 4 = A, 5 = S.

## Couple (marriage) endings

The partner is decided in this order:

1. A partner already set by accepting a proposal (the 8 race partners) is kept.
2. Otherwise the first matching rule applies: Caim (Episode 1 cleared and ending Restaurant Server), Icosa (Episode 1 and Magic Instructor), Kimaris (Episode 1 and Soldier), Muzai (Episode 1 and Writer), Cube (`Cube_Episode2` cleared), Solo, i.e. no couple ending (sum of the 12 base stats ≥ 3600), "Ordinary Happiness" (ending Jobless or Villager).
3. If none match, the default partner is kept (the default is also "Ordinary Happiness").

Proposals happen at the start of March of year 1218 once the partner's final story episode is done. Only choosing "accept" sets the partner. Demon King and Princess are mutually exclusive, and the Princess ending needs the Demon Male (Agares) proposal flag.

## Fortune teller hints

The fortune teller hints at up to 3 endings. Progress is the score-weighted average of the best progress in each condition group; the display threshold rises from 20% to 80% over 2920 days. `Ending_Jobless` and endings with `Weight` < 1000 are excluded.

## Relationship with Cube

The status is 21 (`RelationshipCube`, range −100 to 500). It increases **once per Housework (`Job_House`) schedule slot by the number of days of that slot**, applied when the schedule starts regardless of its result. A slot is 10 days for the first two per month and (days in month − 20) for the last. Three `E_Job_House` events add +1 each. In-game names such as "Relationship with Cube" come from the game's `Text` table. `Cube_Episode1` needs ≥ 70 and `Cube_Episode2` needs ≥ 120, so about 7 and 12 slots of housework are required.

## Known gaps

- When several partners qualify for a proposal in the same month, the actual order is inferred (table order), not verified.
- `Flow_Vacance` (vacation events) and the scene names of royal-castle encounter places 703–708 are not analysed.

## License

MIT. See [LICENSE](LICENSE).
