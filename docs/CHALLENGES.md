# Technical Challenges

Problems encountered while building **Cyber Hero**, and how they were resolved.

---

## 1. Three different genres, one progression system

**Context**
The game teaches three different concepts — password composition, firewall versus antivirus, and information privacy. Each concept needs a different interaction to land: "mix your character types" cannot be taught with a shooter, and "block versus remove" cannot be taught with drag-and-drop.

The result is a drag-and-drop puzzle, then a shooter, then a sorting game. All three still had to share lives, timing, stars, coins, tutorials, dialogue and scene flow.

**Alternatives considered**

| Approach | Outcome |
|---|---|
| A `BaseStage` class the three stages inherit from | ❌ Rejected — the stages share no mechanics, only surrounding state. The base class would have been an empty shell with three unrelated overrides |
| Three self-contained mini-games, each with its own progression | ❌ Rejected — hearts, stars and the timer would be reimplemented three times, and the certificate needs one consistent source for all of it |
| Composition: a persistent `GameManager` for shared state, one `StageNRules` per stage for local logic | ✅ Chosen |

**Implementation**
`GameManager` owns hearts, the timer, the dialogue counter and loss state, and persists across scenes. Each stage exposes a `StageNRules` component containing only what is true of that stage — which tags are clickable, which objects appear when, what counts as a correct action.

Nothing in `GameManager` knows a shooter exists. Nothing in `Stage2Rules` knows how stars are awarded.

**Result**
Adding stage four — the quiz — meant writing the quiz and wiring it to the same manager. Stages one through three were untouched.

**Accepted trade-off**
The raycast-and-compare-tag block is duplicated across all three rules classes. Twelve lines, three times. Under a fixed delivery window a shared input abstraction would have cost more than the duplication does, but it remains duplication: a change to input handling means three edits rather than one.

**Lesson**
Inheritance models "is a kind of." These stages aren't kinds of anything; they're unrelated games sitting inside the same shell. Composition was the honest shape, and a base class would have produced an abstraction that existed only to look organized.

---

## 2. Arabic in Unity doesn't work out of the box

**Symptom**
Arabic text rendered as disconnected letters in the wrong order — every word reversed, every letter in its isolated form. The engine treats a string as a left-to-right sequence of independent glyphs, which is exactly wrong for Arabic.

**Diagnosis**
Arabic is cursive: a letter's shape depends on its neighbours, so each letter has up to four contextual forms, and the script runs right-to-left. Correct rendering requires shaping — choosing the right form per letter based on position — and bidirectional ordering, before anything is laid out. Unity's text system does neither.

This is not a font problem. A correct Arabic font renders incorrectly shaped text just as wrongly, only prettier.

**Solution**
Every user-facing string goes through `RTLTextMeshPro`, backed by an `ArabicSupport` assembly that shapes and reorders before layout.

Dynamic content is where this needs the most attention. Static UI is converted once and forgotten; generated text is where the wrong component slips in. Stage 3 builds its information cards at runtime, and each one is populated through the RTL component rather than a plain text field:

```csharp
card.transform.GetChild(0).GetComponent<RTLTextMeshPro>().text = cardsText_privet[rand_2];
```

The same applies to the certificate, which assembles the player's name, times and attempt counts into one multi-line Arabic string at the end of the game.

**Lesson**
Right-to-left support is a rendering concern, not a translation one, and it has to be settled before the UI is built. Retrofitting it means touching every text field in the project. Choosing the RTL component as the default early costs nothing; discovering the need late costs days.

---

## Known limitations

| Area | Current state | Better approach |
|---|---|---|
| Stage triggers | Polled every frame against counters in `Update` | Event callbacks from the dialogue system |
| Input handling | Raycast-and-tag block duplicated per stage | One input service dispatching to stage handlers |
| Object identity | String tags, no compile-time checking | Typed components or enums |
| Progression storage | `PlayerPrefs` with string keys across several scripts | A serialized save model with keys in one place |
| Test coverage | Heart-loss system only | Password validation and card categorization are pure functions and directly testable |

## Working to a written specification

The game was built against a detailed written brief — four stages, named mechanics per stage, a certificate, a report, accounts, a coin store. That detail turned out to be the most useful property of the project, because it made scope a question of fact rather than of memory: a feature was either in the document or it wasn't.

The dependency worth managing differently is assets. Art and audio arrived from outside the codebase, which meant parts of the build could only be finished once they landed. Building every screen against placeholders from day one would have kept no system's completion blocked on an external schedule.
