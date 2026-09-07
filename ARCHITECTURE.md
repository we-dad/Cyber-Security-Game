# Architecture

A Unity 2D project where three structurally different games share one progression system. The organizing idea is that stage-specific logic and cross-stage state never touch: `GameManager` owns lives, timing, scoring and flow, and knows nothing about how any given stage is played.

---

## Script layout

```
Assets/Scripts/
├── General/       Cross-stage systems — GameManager, SoundManager, tutorial,
│                  dialogue, drag-drop, settings, certificate, tag store
├── Main Menu/     Registration, gender selection, menu navigation
├── Player Data/   PlayerData singleton, PlayFab persistence, total-time tracking
├── Gates/         Door progression and star display between stages
├── Questions/     ScriptableObject-driven quiz system
├── Level1/        Password building — Stage1Rules, PasswordComputerPanel, RandomSprites
├── Level2/        Virus shooter — Stage2Rules, Spawner, GunShots, Bullet, EnergyBar
└── Level3/        Card sorting — Stage3Rules, CardsGenerator
```

## The stage pattern

Each stage exposes one `StageNRules` component. It owns exactly two things: what a click on the world means in this stage, and when this stage's objects appear. Everything shared is reached through singletons.

```csharp
if (Input.GetMouseButtonDown(0))
{
    Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
    RaycastHit2D hit = Physics2D.Raycast(ray.origin, ray.direction);
    if (hit.collider != null)
    {
        if (hit.collider.tag == "LettersBox") { /* stage-specific */ }
    }
}
```

**Why this held up.** Adding a fourth stage means writing one new `StageNRules` and touching nothing else. Lives, timer, stars, dialogue and scene transitions come along for free. With three genres in one product and a fixed delivery window, a shared base class that tried to abstract "what a stage is" would have cost more than it saved — the stages have almost nothing in common mechanically.

**The cost.** The raycast-and-tag-compare block is duplicated in all three rules classes. It is the same twelve lines three times, and a change to input handling means three edits. String tags also give no compile-time safety: a typo in `"LettersBox"` fails silently at runtime.

## Stage mechanics

**Stage 1 — password building.** Characters are collected from world boxes into the player's bag, then dragged into slots. `PasswordComputerPanel` walks the filled slots and counts four categories:

```csharp
if (capetalLetter >= 1 && smallLetter >= 1 && number >= 1
    && symbol >= 1 && childNum == 6 && GameManager.Instance.heartNum > 0)
```

The strength bar is driven by the same four counts, so the visual and the win condition can never disagree:

```csharp
filled.fillAmount = currentValue / 4;
```

`RandomSprites` reshuffles which characters appear on retry, so a failed attempt cannot be solved by repeating a memorized sequence.

**Stage 2 — firewall and antivirus.** `Spawner` generates viruses that attack the player; `GunShots` and `Bullet` handle removal; `EnergyBar` drives a slider showing how much shield time remains. When it runs out, the player must collect another. The two systems are deliberately separate mechanics because they represent two separate concepts — a shield that blocks and a weapon that removes.

**Stage 3 — information sorting.** `CardsGenerator` holds two string lists, private and public, draws from them without replacement, tags each spawned card accordingly, and populates it through an RTL text component:

```csharp
int rand_2 = Random.Range(0, cardsText_privet.Count);
card.transform.GetChild(0).GetComponent<RTLTextMeshPro>().text = cardsText_privet[rand_2];
card.tag = "Privet";
cardsText_privet.RemoveAt(rand_2);
```

Removing on draw guarantees no repeated card within a round.

**Stage 4 — quiz.** `QuestionsSystem` loads `QuestionData` ScriptableObjects from resources, filters by category so each completed stage contributes three questions, and unlocks progression on a majority-correct result. Questions are authored as assets rather than in code, so the client can revise wording without a rebuild.

## Cross-stage systems

| System | Notes |
|---|---|
| `GameManager` | Hearts, per-stage timer, dialogue counter, loss state. Persistent singleton. |
| `PlayerData` | Name, gender, stars, coins, quiz percentage; analytics dispatch. |
| `SaveToTheServier` | PlayFab writes for registration and player profile. |
| `DialogueTriggers` | Voiced dialogue sequencing, gated on a click counter. |
| `GameTutorial` | Pre-stage instruction panels. |
| `DoorsSystem` / `DoorsStars` | Gate progression and star display between stages. |
| `TagsStore` | Coin-based cosmetic purchases, priced from `TagData` assets. |
| `Certificate` | End-of-game certificate and report, rendered and captured from a dedicated camera. |

Progression values (`Level1Time`, `Level1Atempt`, `PlayerCoins`, `PlayerTotalTimeFloat`) are stored in `PlayerPrefs`.

**Trade-off.** `PlayerPrefs` is a plain key–value store meant for settings, not for game state. It is unstructured, untyped, and trivially editable by the player. For a children's educational game with no competitive element it is a reasonable choice, and it removed an entire serialization layer from a six-week build — but the keys are string literals scattered across several scripts, and a typo in one silently reads a default instead of failing.

## Testing

`Assets/EditModeTests` and `Assets/PlayModeTests`, each with its own assembly definition. The heart-loss system is covered in both modes — in isolation, and against a loaded scene:

```csharp
[UnityTest]
public IEnumerator GameManger_LoseHeart()
{
    yield return SceneManager.LoadSceneAsync("1");
    GameManager gameManager = Object.FindObjectOfType<GameManager>();
    gameManager.LoseHeart(3);
    Assert.IsTrue(gameManager.itsLose);
}
```

Coverage is narrow — one system, two tests — but the harness is in place, which is the part that is awkward to add later.

## Arabic text

Unity's built-in text rendering neither joins Arabic letters nor orders them right-to-left. Every user-facing string goes through `RTLTextMeshPro`, backed by an `ArabicSupport` assembly that performs letter shaping before layout. This applies to dynamically generated content too — the information cards in stage 3 are populated through the same component rather than a plain `TMP_Text`.

## Known limitations

| Area | Current state | Better approach |
|---|---|---|
| Stage triggers | Polled every frame in `Update` against counters | Event-driven callbacks from the dialogue system |
| Input handling | Raycast-and-tag block duplicated per stage | One shared input service dispatching to stage handlers |
| Object identity | String tags | Typed components or enums, checked at compile time |
| Progression storage | `PlayerPrefs` with scattered string keys | A serialized save model with keys defined in one place |
| Test coverage | One system | Password validation and card categorization are pure and directly testable |
