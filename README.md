<div align="center">

# Cyber Security Game

**An Arabic educational game that teaches children the basics of staying safe online.**

[![Play on itch.io](https://img.shields.io/badge/Play-itch.io-fa5c5c?logo=itchdotio&logoColor=white)](https://wedad.itch.io/cyber-security-game)
![Unity](https://img.shields.io/badge/Unity-2D-black?logo=unity)
![C#](https://img.shields.io/badge/C%23-gameplay-239120)
![Language](https://img.shields.io/badge/language-Arabic-lightgrey)

**[▶ Play it here](https://wedad.itch.io/cyber-security-game)**

</div>

---

## About

A commissioned game built solo over roughly six weeks, to a written client specification. The brief was to teach three specific cyber security concepts to children — not to explain them, but to make players *do* them, so that the lesson comes out of the mechanic rather than out of a text box.

Every level is a different genre, chosen so that the interaction itself carries the idea:

| Stage | What the player does | What they learn |
|---|---|---|
| **1** | Collects boxes of letters, numbers and symbols, then drags characters into password slots while a strength bar responds | Why mixing character types makes a password strong |
| **2** | Shoots incoming viruses, and picks up a fresh shield each time the current one expires | The difference between a **firewall** (the shield that blocks) and an **antivirus** (the weapon that removes) |
| **3** | Sorts information cards into "safe to share" and "never share" — a monster attacks on a wrong sort | Which personal information should stay private |
| **4** | Answers three questions per completed stage to unlock the way forward | Consolidates all three lessons |

The whole game is in Arabic, including voiced dialogue and a robot guide who introduces each stage.

**Role:** Solo developer — all programming, systems and integration. Art and audio were supplied by the client.

## Features

### Progression and feedback

Each stage runs on a shared layer: a tutorial before entry, a robot introduction, a countdown timer, a heart-based life system, and coins and stars awarded on performance. Losing all hearts replays the stage rather than ending the run, since the audience is children and a hard fail state teaches nothing.

### End-of-game certificate

Finishing the game issues a personalized certificate with the player's name, and a report covering per-stage time and attempt count, total playtime, stars earned, and quiz accuracy. The same figures are sent to analytics, so the client can see where players actually struggle rather than guessing.

### Accounts and persistence

Players register and log in, with profile data stored server-side through **PlayFab**. Progress, coins, stars and per-stage timings persist locally, and a purchasable tag store lets players spend earned coins on cosmetic name tags.

### Full Arabic support

Unity's text rendering does not shape or order Arabic correctly on its own. The entire UI runs through RTL-aware text components, with an Arabic support layer handling letter joining and right-to-left ordering.

## Under the hood

The three gameplay stages are genuinely different games — drag-and-drop, a shooter, and a sorting puzzle — that share one progression system. Each stage owns a `StageNRules` component holding only its own logic, while lives, timing, scoring, dialogue and scene flow live in a persistent `GameManager` that knows nothing about how any individual stage plays.

Full write-up: **[ARCHITECTURE.md](ARCHITECTURE.md)**

## Problems worth reading about

Delivering three distinct genres under one progression system, rendering Arabic correctly in Unity, and adding an automated test suite to a project already in flight:

**[CHALLENGES.md](CHALLENGES.md)**

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
