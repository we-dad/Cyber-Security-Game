<div align="center">

<img src="images/title-screen.png" width="620" alt="Cyber Hero title screen"/>

# Cyber Hero

**An Arabic educational game that teaches children the basics of staying safe online.**

[![Play on itch.io](https://img.shields.io/badge/Play-itch.io-fa5c5c?logo=itchdotio&logoColor=white)](https://wedad.itch.io/cyber-security-game)
![Unity](https://img.shields.io/badge/Unity-2D-black?logo=unity)
![C#](https://img.shields.io/badge/C%23-gameplay-239120)
![Language](https://img.shields.io/badge/language-Arabic-lightgrey)

**[▶ Play it here](https://wedad.itch.io/cyber-security-game)**

</div>

---

## About

Cyber Hero teaches children three online safety concepts — not by explaining them, but by making players *do* them, so the lesson comes out of the mechanic rather than out of a text box.

Each stage is a different genre, chosen so the interaction itself carries the idea. A robot guide introduces every challenge, the whole game is voiced and written in Arabic, and players pick a character before starting.

<div align="center">
  <img src="images/character-select.png" width="400" alt="Character selection"/>
  <img src="images/robot-intro.png" width="400" alt="The robot guide introducing a challenge"/>
</div>

Progress runs through a central hub — a locked door whose three icons unlock as each stage is cleared:

<div align="center">
  <img src="images/hub.png" width="500" alt="The hub door with stage icons"/>
</div>

## The stages

### 1 · Building a strong password

<img src="images/stage1-password.png" width="500" alt="Password building on a tablet"/>

Characters are collected from around the level and dragged into the slots of a tablet. A strength meter fills as the password gains variety, and the "create password" button only accepts a combination containing an uppercase letter, a lowercase letter, a number and a symbol.

Characters are reshuffled on every retry, so a failed attempt can't be solved by repeating a memorized sequence — the player has to apply the rule rather than the answer.

**The lesson:** why mixing character types is what makes a password strong.

### 2 · Firewall and antivirus

<img src="images/stage2-firewall.png" width="500" alt="Shooting viruses while shields expire"/>

Viruses advance on the player, who shoots them down while a shield holds them off. The shield expires, and a fresh one has to be collected before the next wave.

The two mechanics are deliberately separate — a shield that *blocks* and a weapon that *removes* — because they represent two separate tools. A single "defence" mechanic would have collapsed the distinction the stage exists to teach.

**The lesson:** the difference between a firewall and an antivirus.

### 3 · What not to share

<img src="images/stage3-cards.png" width="500" alt="Sorting information cards into red and green folders"/>

Cards of personal information are sorted into two folders: green for things that are fine to share, red for things that aren't. Favourite colour, hobbies and favourite animal go one way; home address, email addresses and passwords go the other. A monster attacks when something private lands in the green folder.

**The lesson:** which personal information should stay private.

### 4 · The quiz

Three questions per completed stage. Answering most of them correctly unlocks the way forward; failing sends the player back to revisit the material. Questions are authored as data assets rather than in code, so wording can be revised without a rebuild.

## Progression

<div align="center">
  <img src="images/tutorial.png" width="400" alt="Movement tutorial"/>
  <img src="images/stage-complete.png" width="400" alt="Stage complete with stars and coins"/>
</div>

Every stage shares one layer: a tutorial before entry, a countdown timer, three hearts, and stars and coins awarded on completion time and accuracy. Losing all hearts replays the stage rather than ending the run — the audience is children, and a hard fail state teaches nothing.

Coins can be spent in a store on cosmetic name tags.

## Certificate and report

Finishing the game issues a personalized certificate with the player's name, alongside a report covering per-stage time and attempt count, total playtime, stars earned and quiz accuracy. The same figures are sent to analytics, making it possible to see where players actually struggle rather than guessing.

## Under the hood

The three gameplay stages are genuinely different games — drag-and-drop, a shooter, and a sorting puzzle — sharing one progression system. Each stage owns a `StageNRules` component holding only its own logic, while lives, timing, scoring, dialogue and scene flow live in a persistent `GameManager` that knows nothing about how any individual stage plays.

Full write-up: **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)**

## Problems worth reading about

Three distinct genres under one progression system, rendering Arabic correctly in Unity, and adding an automated test suite to a project already in flight:

**[docs/CHALLENGES.md](docs/CHALLENGES.md)**

## Running it

```bash
git clone https://github.com/we-dad/Cyber-Security-Game.git
```

Open in Unity and load the main menu scene. PlayFab needs a title ID in its settings asset before accounts will connect; the game is otherwise playable offline.

Tests live in `Assets/EditModeTests` and `Assets/PlayModeTests` and run from **Window → General → Test Runner**.

## Built with

| | |
|---|---|
| **Unity 2D** | Engine |
| **PlayFab** | Accounts, player data, analytics |
| **RTLTMPro / ArabicSupport** | Arabic text shaping and RTL layout |
| **Unity Test Framework** | EditMode and PlayMode tests |
| **ScriptableObjects** | Quiz question and tag data |
