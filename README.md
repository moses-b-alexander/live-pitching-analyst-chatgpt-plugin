# Live Starting Pitching Analyst

A live MLB starting-pitching companion for ChatGPT and other skill-capable AI agents.

The goal is simple: **make watching starting pitching more interesting without making you stare at another dashboard**.

The analyst tracks a game's starting-pitching matchup, keeps the factual bookkeeping straight, maintains an evolving interpretation of each start, and turns casual observations from the viewer into testable statistical questions.

> **Track the matchup; analyze the pitcher.**

A session can track one or two starters, but live analysis is always pitcher-specific. Pregame can cover both starters, each live checkpoint analyzes one starter at a time, starter-exit summaries are individual, and direct pitcher-to-pitcher comparisons are explicit.

---

## What it does

The workflow is built around three products:

### 1. Stat Line — ground truth

A compact, source-checked account of what actually happened.

```text
Line: 17 P · 11S/6B | 1 R · 3 H | 2 K · 1 BB
Mix: 4S 8 (47%) · SI 3 (18%) · FC 2 (12%) · CU 4 (24%)
Hits: S8 · S7 · D9
```

The Stat Line is deliberately boring.

It should be:

* factual rather than interpretive,
* compact enough to glance at between innings,
* based on current reliable sources,
* explicit when live data are incomplete,
* deterministic whenever possible.

Pitch counts and pitch-type percentages stay together. Hits use compact baseball scoring notation such as `S8` or `D9` when the underlying source supports it.

The analyst should never invent a pitch classification, location, hit destination, or other missing fact merely to complete the format.

Because the current plugin does not bundle its own MLB data feed, the strength of source verification depends on the live-data and browsing tools available to the host.

---

### 2. Summary — an evolving baseball model

The Summary answers a different question:

> **What does this start look like now?**

Instead of generating a new scouting story every inning, the analyst maintains a small number of persistent ideas and updates them as evidence accumulates.

For example:

```text
• Command — weakening:
  He is still throwing strikes, but more fastballs are leaking over
  the middle of the plate.

• Mistakes — still survivable:
  A 99 mph middle-middle four-seamer can still produce a foul ball
  even though the location itself was poor.

• Curveball role — strengthened:
  Repeated early-count usage suggests the curve is shaping what hitters
  have to protect against, not merely finishing plate appearances.

• Location — reframed:
  The earlier high/low description was too narrow; working the outside
  part of the plate has become equally important.
```

Some recurring distinctions matter here:

**Throwing strikes is not the same as locating well.**

A pitcher can continue filling the zone while missing his best spots.

**A good result does not mean it was a good pitch.**

A poorly located pitch can survive because of velocity, movement, deception, or sequencing.

**Pitch movement is not the same as pitch location.**

Velocity, vertical break, horizontal movement, release point, and extension describe the pitch itself. High/low, inside/outside, edge/middle, and backdoor/frontdoor describe how that pitch is being used.

**Stuff and sequencing are competing explanations.**

Sometimes movement, velocity, and location explain an outcome well enough. Sometimes the count, previous pitches, and repeat exposure materially change the hitter's decision. The analyst should not invent a complicated sequencing story when the pitch itself already explains what happened.

The Summary is always **data-led**. Viewer observations may sharpen it, but they do not become ground truth merely because they were noticeable on television.

---

### 3. Analyst — observation → statistical question

This is the most interactive part of the skill.

The viewer does **not** need to speak statistically.

You can say:

> “His curve looks way lower tonight.”

The analyst can translate that into a measurable question.

**Illustrative example:**

```text
Class: location / proportion

Question:
Is the pitcher throwing a larger share of curveballs below the zone
than he normally does?

Tonight: 11/13 below zone (84.6%)
Season baseline: 61.2%
Difference: +23.4 percentage points

Test: exact binomial, one-sided
p = .041
```

And then return to baseball:

> The curve has been unusually low in this sample. That does not, by itself, mean the pitch is breaking more than usual.

Or you might simply say:

> “His velo looks cooked.”

The analyst may turn that into:

* four-seam velocity tonight versus the pitcher's usual velocity drop over the course of a start,
* an early-innings versus later-innings comparison,
* comparison with the pitcher's own historical pitch data,
* or another baseline appropriate to the observation.

The user does not need to choose the null hypothesis, statistical test, comparison group, or conditioning variables.

The desired loop is:

```text
watch
  ↓
notice something
  ↓
analyst turns it into a measurable question
  ↓
compare or test
  ↓
update the read on the outing
  ↓
keep watching
```

---

## Matchup vs. pitcher scope

This distinction is central to the skill.

### Matchup state

A game session can contain:

* the game and teams,
* one or two tracked starting pitchers,
* regular-season and postseason baselines,
* shared game state,
* the ability to compare the starters when requested.

### Pitcher state

Each starter gets a completely independent state containing:

* factual inning summaries,
* pitch mix,
* viewer observations,
* evolving takeaways,
* statistical questions,
* source-completeness state,
* final starter-exit summary.

Normal live output is **never a blended two-pitcher summary**.

If both starters are being tracked, the analyst simply alternates with the game:

```text
Starter A pitches
      ↓
Starter A checkpoint

Starter B pitches
      ↓
Starter B checkpoint
```

The other pitcher's state remains intact in the background.

Direct comparison is available when useful:

```text
compare their curve usage through three innings

compare their four-seam velocity drop

did these two pitchers actually attack hitters similarly?

compare how often mistakes over the plate were punished
```

---

## Game lifecycle

### Pregame

Pregame analysis establishes a baseline for the tracked starters.

By default, it uses:

* the current regular season,
* the current postseason, when applicable.

It does not automatically reach into arbitrary career windows, previous seasons, opponent history, or hand-picked recent starts unless requested.

Pregame focuses on a few things worth watching:

* pitch mix,
* velocity and movement,
* release point,
* likely locations and usage,
* the decisions hitters may have to make,
* a few questions worth following during the start.

These are **starting ideas, not predictions**.

The user can discuss them normally.

A phrase such as:

```text
game starting
```

ends pregame mode and freezes those ideas as the starting read for the live outing.

---

### Live

The live workflow prioritizes the first several innings, when the starter's approach and hitter adjustments are becoming clearer.

The skill deliberately allows some delay rather than racing an unstable live feed.

A typical checkpoint contains:

```text
STAT LINE

EXECUTIVE SUMMARY

optional statistical question / hypothesis update
```

The viewer can interrupt at any time with normal language.

There is no required syntax.

---

### Starter exit

A starter leaving the game does **not** immediately trigger an unverified final line.

The analyst first reconciles the start against reliable sources.

The exit process is conceptually:

```text
ACTIVE
  ↓
STARTER REMOVED
  ↓
SOURCE CHECK
  ↓
FINALIZED
```

The analyst checks:

* innings / outs,
* pitches,
* strikes and balls,
* hits,
* runs and earned runs,
* walks,
* strikeouts,
* pitch mix,
* official pitching change.

If the starter leaves runners on base, run attribution can remain unresolved until those runners are accounted for.

The final start then gets:

1. the complete factual Stat Line,
2. a whole-outing Summary,
3. a short set of the most useful statistical questions raised by the start.

---

### Postgame

Postgame analysis can become substantially richer.

Possible questions include:

* pitch mix by inning, count, or batter handedness,
* pitch-location patterns,
* how often pitches were left over the middle of the plate,
* changes in pitch movement,
* velocity drop,
* release-point changes,
* pitch sequencing,
* how hitters changed their approach the second or third time through,
* launch angle and quality-of-contact changes,
* how long different pitches stayed on similar paths,
* formal tests of ideas generated during the game.

The live notes can then be revisited as:

```text
supported
corrected
misleading visual impression
unresolved
```

The objective is calibration, not proving that either the viewer or the analyst was right.

---

## Viewer and analyst annotations

Executive-summary points can include compact superscript annotations.

There are two independent axes.

### Viewer axis — numbers

| Marker | Meaning                                                          |
| ------ | ---------------------------------------------------------------- |
| ¹      | Viewer observation supported / consistent with current data      |
| ²      | Viewer observation partially supported / mixed                   |
| ³      | Viewer observation not supported / evidence points the other way |
| ⁴      | Viewer observation unresolved / insufficient data                |

### Analyst axis — letters

| Marker | Meaning                                                |
| ------ | ------------------------------------------------------ |
| ᵃ      | Analyst read strengthened / supported                  |
| ᵇ      | Analyst read partially supported / needs qualification |
| ᶜ      | Analyst read weakened                                  |
| ᵈ      | Analyst read corrected / reframed                      |
| ᵉ      | Analyst read unresolved / insufficient data            |

They can be combined:

```text
¹ᵃ  viewer observation supported; analyst read strengthened

²ᵇ  both are partly supported

³ᶜ  viewer observation unsupported; analyst read weakened

ᵈ   analyst-only correction

⁴ᵉ  insufficient evidence on both axes
```

These are deliberately approximate signals, not numerical scores.

---

## Statistical philosophy

The skill is designed to take live baseball observations seriously without pretending that tiny in-game samples are larger than they are.

### Use pitcher-specific baselines

Prefer:

> “Is this unusual for this pitcher under comparable conditions?”

over:

> “Is this unusual relative to some arbitrary league threshold?”

Useful comparisons may account for:

* pitch type,
* batter handedness,
* count,
* previous pitch,
* times through the order,
* inning,
* pitch number.

Only add those conditions when they improve the comparison without making the sample uselessly small.

---

### Preserve inning-defined questions

Baseball innings tell stories.

If the user asks:

```text
compare inning 2 with innings 3–5
```

the analyst should not silently replace that with equal-pitch samples.

A three-inning span containing 20 pitches and one containing 60 pitches still represent the baseball windows the user asked about.

The statistical interpretation should simply acknowledge that one sample is much smaller and therefore less certain.

> The innings can be comparable baseball stories without being equally strong statistical samples.

---

### Separate exploration from confirmation

Suppose a viewer notices something in the second inning:

```text
I2:
"his curve seems much lower tonight"
```

That inning helped generate the idea.

Testing the same inning against a historical baseline is therefore exploratory.

But the idea can now be frozen:

```text
OBSERVE I2
     ↓
freeze the question
     ↓
collect I3–I5
     ↓
test the later sample
```

The later innings can provide prospective evidence if the pitcher remains in the game.

If the observation is only articulated after watching I1–I3, then I1–I3 are exploratory. Future pitches may still provide a cleaner test.

The important distinction is **whether the data helped generate the idea or were observed afterward**, not the inning number itself.

---

### Avoid significance theater

The skill emphasizes:

* sample size,
* historical baseline,
* size of the difference,
* uncertainty,
* baseball importance.

A p-value alone is not an analysis.

When appropriate, the analyst can use methods such as:

* exact binomial tests,
* Fisher exact tests,
* comparisons of rates,
* resampling,
* permutation tests,
* bootstrap intervals,
* simple trend models,
* pitch-sequence probabilities.

More sophisticated methods are welcome when the question warrants them, but complexity is not the goal.

---

## Natural-language interface

Structured commands are optional.

The skill understands shorthand such as:

```text
OBS:
TEST:
COMPARE:
WHY:
POST:
```

but normal conversation is preferred.

All of these are valid inputs:

```text
his slider looks flatter

they're sitting high fastball now

why is he surviving so many pitches over the plate?

is he actually using the sinker more after four-seamers?

his velo looks cooked

those curves seem way lower than normal

compare the two starters' velo drop
```

The formal structure belongs inside the analyst, not inside the user's prompt.

---

## Source discipline

Live feeds can lag the broadcast and pitch classifications can be revised.

The skill therefore prefers correctness over false precision.

When information has not stabilized, it should say so:

```text
Mix: exact counts pending
```

rather than guess.

Starter exits receive additional checking because:

* inherited runners can change R/ER,
* live pitch classifications can be revised,
* different public scoreboards may update at different speeds.

The skill is designed to distinguish:

```text
known
provisional
unresolved
```

rather than flatten those into a single confident answer.

---

## What this is not

This project is intentionally narrow.

It is **not**:

* a betting model,
* a game-outcome predictor,
* a next-pitch prediction engine,
* a full scouting platform,
* a bullpen-management system,
* an automated broadcast replacement,
* a generic baseball statistics dashboard.

It is specifically a **starting-pitching companion for people actively watching a baseball game**.

Bullpen games, openers, and bulk-reliever structures are outside the default workflow rather than silently forcing the model to reinterpret them as conventional starts.

---

## Current implementation

The current release is a **skills-only plugin**.

The core behavior is defined in:

```text
skills/
└── live-starting-pitching-analyst/
    └── SKILL.md
```

There is no bundled MLB data server or proprietary data feed.

The skill relies on the host agent's available browsing, sports, or data-retrieval capabilities for live information.

A separate agent implementation is planned around the same specification, with deterministic data ingestion/stat lines and interchangeable model backends for:

* executive interpretation,
* statistical reasoning,
* optional local/open-weight inference.

---

## Repository layout

A typical package looks like:

```text
.
├── plugin.json
├── LICENSE
├── README.md
├── assets/
│   └── baseball.svg
└── skills/
    └── live-starting-pitching-analyst/
        └── SKILL.md
```

`SKILL.md` is the canonical behavioral specification.

The packaged plugin is a release snapshot of that specification.

---

## Development workflow

The project intentionally separates **working specification** from **published releases**.

```text
SKILL.md
   │
   ├──→ GitHub commit
   │
   └──→ validated plugin package
             ↓
        plugin release
```

Changes to the skill do not need to become plugin releases immediately.

This keeps everyday iteration separate from intentional publishing.

---

## Design principle

The central product constraint is attention.

A baseball fan should not need to choose between:

```text
watching the pitch
```

and:

```text
reading the analytics
```

The system should carry the bookkeeping and statistical machinery in the background, then surface only what is useful at natural pauses.

The intended experience is closer to having a technically rigorous pitching analyst sitting next to you than opening another baseball dashboard.

---

## Status

Early development / live-game testing.

The specification is being refined through actual live-game use, with particular attention to:

* information density,
* source reliability,
* statistical rigor,
* continuity between innings,
* useful corrections,
* viewer attention cost.

Feedback and issues are welcome.

---

## Author

**Moses Alexander**

---

## License

MIT License.

See [`LICENSE`](LICENSE) for details.
