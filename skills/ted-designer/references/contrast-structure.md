# Contrast structure

Nancy Duarte's research on hundreds of speeches found a single recurring pattern: the most effective talks oscillate between **what is** (current reality) and **what could be** (the new world the speaker proposes). The audience is pulled across that gap by tension and release.

This reference describes how the ted-designer skill detects, validates, and (when missing) flags the contrast structure in a script before generating slides.

## Why this matters

A script without contrast structure produces flat slides. The audience has nothing to lean toward. Even visually beautiful decks fall flat if every slide reinforces a single steady-state message.

Slides amplify the script's structure. If the script is flat, slides will be flat. Fix the script first.

## The two poles

**What is** — the current state. Pain, status quo, problem, broken assumption, market reality, audience's existing belief.

**What could be** — the proposed state. Solution, new way of thinking, future market, alternative belief, the speaker's call to action.

A talk with strong contrast moves repeatedly between these poles, ending firmly on a what-could-be call to action.

## Detection workflow

### Step 1: Segment the script

Break the script into beats. A beat is one continuous thought, typically 1–3 sentences. Each beat will eventually map to one slide.

### Step 2: Tag each beat

For each beat, assign one tag:

- `IS` — describes current reality, problem, status quo, pain, friction
- `COULD` — describes proposed reality, solution, future state, call to action
- `BRIDGE` — neutral connective tissue (anecdote, data, expert quote that supports either pole)
- `META` — talk meta (intro, agenda, "today I'll cover..."). These should be cut or minimized.

### Step 3: Sparkline the script

Plot the tags in order. A healthy talk shows oscillation:

```
IS, IS, BRIDGE, COULD, IS, BRIDGE, COULD, COULD, COULD (close)
```

A flat talk shows clustering:

```
IS, IS, IS, IS, IS, IS, IS, IS, IS, IS  ← all problem, no solution
COULD, COULD, COULD, COULD, COULD       ← all solution, no problem context
META, META, IS, COULD, META             ← drowning in meta
```

### Step 4: Validate

Apply these gates. If any fail, flag the script before generating slides.

| Gate | Pass criteria |
|------|---------------|
| **What-is present** | At least 20% of beats tagged `IS` |
| **What-could-be present** | At least 20% of beats tagged `COULD` |
| **Ends on COULD** | Last 1–3 beats are `COULD` (call to action) |
| **Oscillates** | At least 3 transitions between `IS` and `COULD` across the talk |
| **Meta minimal** | `META` beats less than 10% of total |
| **STAR present** | At least one beat marked as candidate STAR (see below) |

### Step 5: Surface gaps to user

When a gate fails, surface the specific issue with concrete revision suggestions:

- "Script lacks what-could-be beats. The argument describes the problem but never paints the proposed alternative. Suggest adding 2–3 beats describing the new state after beat #14."
- "Script does not end on a call to action. Final beat is descriptive. Suggest replacing or appending a what-could-be close."
- "No oscillation detected — first half is all problem, second half is all solution. Talks land harder when they alternate. Suggest weaving a brief solution preview into the problem section, or a brief problem callback into the solution section."

Do not generate slides until the script passes or the user explicitly accepts the gaps.

## STAR moment identification

A STAR moment (Duarte: Something They'll Always Remember) is engineered to be unforgettable. Every TED-style talk should have at least one.

### Categories

| Type | Example |
|------|---------|
| Dramatization | Bill Gates releasing mosquitos at TED |
| Scale shock | A number 10x larger than expected |
| Vulnerable disclosure | Personal story that shifts the room's energy |
| Demonstration | Live performance of the concept (Jill Bolte Taylor) |
| Memorable line | Phrase that gets quoted afterward |

### Detection heuristic

Scan the script for beats that match any of these signals:

- Speaker uses second person directly ("imagine you...", "look at your...")
- Surprising statistic introduced for the first time
- Personal narrative paragraph (often signaled by "I remember when...")
- Imperative verb that asks the audience to do something physical
- Sentence that could be screenshotted and shared without context

If at least one beat hits 2+ signals, mark it as a STAR candidate. If zero beats qualify, flag: "No STAR moment detected. Recommend engineering one. Suggested location: between beats X and Y, where the argument peaks emotionally."

### STAR placement

Most effective STAR moments land at:

- ~70–80% through the talk (just after the climax, before the resolution)
- Immediately after a `BRIDGE` beat (so the bridge sets the table)
- Followed by a `blank` archetype slide (let the moment breathe)

## Output of contrast analysis

The skill embeds the analysis in the spec output:

```json
{
  "story": {
    "what_is": "summary of status-quo argument",
    "what_could_be": "summary of proposed reality",
    "sparkline": ["IS", "IS", "BRIDGE", "COULD", ...],
    "star_moment_slide": 14,
    "star_type": "vulnerable disclosure",
    "structural_warnings": []
  }
}
```

If `structural_warnings` is non-empty, the skill blocks emission of `slides[]` until the user resolves or overrides.

## Override

User may pass `--accept-structural-gaps` to skip the gates. Spec is emitted with `structural_warnings` populated and renderer skills must surface them in their output (e.g. log line, frontmatter comment in HTML).

## When to skip enforcement

For non-narrative formats (status updates, pure data debriefs, training material), the contrast structure is the wrong frame. The user should choose `pptx-deck` or `html-deck` directly, not route through `ted-designer`.

The skill should ask early: "Is this a narrative talk or an information-transfer deck?" If the latter, redirect.
