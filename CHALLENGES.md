# Technical Challenges

Problems I hit building this game solo to a client specification, and how I handled them.

> **On evidence.** Where a claim can be checked against the code or commit history, the file or commit is cited. Where it can't, it's marked as judgment rather than measurement.

---

## 1. Three different genres, one progression system

**Context**
The specification called for three stages that teach three different concepts — password composition, firewall versus antivirus, and information privacy. Each concept needed a different interaction to land: you can't teach "mix your character types" with a shooter, and you can't teach "block versus remove" with drag-and-drop.

So the game is a drag-and-drop puzzle, then a shooter, then a sorting game. All three still needed to share lives, timing, stars, coins, tutorials, dialogue and scene flow.

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
Adding stage four — the quiz — meant writing the quiz and wiring it to the same manager. No changes to stages one through three.

**Accepted trade-off**
The raycast-and-compare-tag block is copy-pasted across all three rules classes. Twelve lines, three times. It was the right call under a fixed delivery window — a shared input abstraction would have cost more than the duplication does — but it is duplication, and a change to input handling means three edits rather than one.

**Lesson**
Inheritance models "is a kind of." These stages aren't kinds of anything; they're unrelated games that happen to sit inside the same shell. Composition was the honest shape, and reaching for a base class would have produced an abstraction that existed only to look organized.

---

## 2. Arabic in Unity doesn't work out of the box

**Symptom**
Arabic text rendered as disconnected letters in the wrong order — every word reversed, every letter in its isolated form. The engine treats a string as a left-to-right sequence of independent glyphs, which is exactly wrong for Arabic.

**Diagnosis**
Arabic is cursive: a letter's shape depends on its neighbours, so each letter has up to four contextual forms, and the script runs right-to-left. Correct rendering needs shaping — choosing the right form per letter based on position — and bidirectional ordering, before anything is laid out. Unity's text system does neither.

This is not a font problem. A correct Arabic font renders incorrectly shaped text just as wrongly, only prettier.

**Solution**
Every user-facing string goes through `RTLTextMeshPro`, backed by an `ArabicSupport` assembly that shapes and reorders before layout.

The part that needed attention was dynamic content. Static UI is easy to convert once and forget; generated text is where the wrong component slips in. Stage 3 builds its information cards at runtime, and each one has to be populated through the RTL component rather than a plain text field:

```csharp
card.transform.GetChild(0).GetComponent<RTLTextMeshPro>().text = cardsText_privet[rand_2];
```

The same applies to the certificate, which assembles the player's name, times and attempt counts into one multi-line Arabic string at the end of the game.

**Lesson**
Right-to-left support isn't a translation task, it's a rendering one, and it has to be decided before the UI is built. Retrofitting it means touching every text field in the project. Choosing the RTL component as the default early cost nothing; discovering the need late would have cost days.

---

## 3. Adding tests to a project already in motion

**Context**
By March 2025 the game was largely built and the loss system was becoming the thing I was most afraid to touch — hearts are decremented from every stage, and a mistake there is invisible until a player reaches a fail state.

**What I did**
Added the Unity Test Framework with separate EditMode and PlayMode assemblies, each with its own `asmdef`, and covered the heart-loss path in both.

EditMode constructs the manager in isolation, with no scene:

```csharp
var go = new GameObject("GameManager");
var gameManager = go.AddComponent<GameManager>();
gameManager.LoseHeart(3);
Assert.IsTrue(gameManager.itsLose);
```

PlayMode runs the same assertion against a real loaded scene, which catches anything that depends on scene wiring rather than on the class alone.

**The part that took longest**
Not the tests — the assembly definitions. Test assemblies need explicit references to the code they test and to the test framework, and until they are right the test files compile into nothing and the Test Runner shows an empty list with no error explaining why. That is a setup problem, not a testing problem, and it is most of the friction in adding tests to an existing Unity project.

**What it didn't cover**
Two tests on one system. Password validation and card categorization are both effectively pure functions and would be straightforward to cover; I didn't get to them.

**Lesson**
The valuable part of adding a test harness mid-project is the harness, not the coverage. Once `asmdef` files exist and the Test Runner lists something, writing the next test is minutes. Before that, it's an afternoon.

---

## 4. Learning what Unity should never commit

**Symptom**
The repository was carrying generated content — `Library/`, `Logs/`, `UserSettings/`, the `.sln` and every `.csproj`. Those directories are rebuilt by Unity on open and regenerated per machine, so they change constantly, conflict on merge, and add nothing.

**Fix**
A cleanup pass in March 2025 removing all of it and correcting `.gitignore`, plus `.gitattributes` for Git LFS.

**Why it matters more than it sounds**
`Library/` alone can be gigabytes. Beyond size, generated project files differ per machine and per Unity version, so they conflict on every collaboration and every version bump — and the conflicts are in files nobody wrote.

**Lesson**
Unity generates a lot of files that look like project files and aren't. Getting `.gitignore` right at project creation is a five-minute task; doing it later means rewriting history or living with the noise.

---

## Known limitations

| Area | Current state | Better approach |
|---|---|---|
| Stage triggers | Polled every frame against counters in `Update` | Event callbacks from the dialogue system |
| Input handling | Raycast-and-tag block duplicated per stage | One input service dispatching to stage handlers |
| Object identity | String tags, no compile-time checking | Typed components or enums |
| Progression storage | `PlayerPrefs` with string keys across several scripts | A serialized save model with keys in one place |
| Test coverage | One system, two tests | Password validation and card categorization next |

## What building to a client spec taught me

The specification was detailed — four stages, named mechanics per stage, a certificate, a report, accounts, a coin store. That detail was the most useful thing about the project, because it made scope a question of fact rather than of memory: a feature was either in the document or it wasn't.

The dependency I would manage differently is assets. Art and audio came from the client, which meant parts of the build could only be finished when they arrived. Next time I would build every screen against placeholders from day one, so that no system's completion is blocked on someone else's schedule.
