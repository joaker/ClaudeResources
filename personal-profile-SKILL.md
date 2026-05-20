---
name: personal-profile
description: Generates a longitudinal, evidence-based personal profile of the user by mining conversation history and memory. Trigger on any request to characterize the user as a person — "give me my personal profile," "profile me," "describe me," "what do you know about me," "update my profile," "give me a self-portrait," or any variant. Also trigger on phrasings like "personality breakdown," "self-assessment," or "how would you describe me," even when "profile" isn't said. The skill produces a three-tier profile (laconic / ELI5 / detailed with strengths, weaknesses, quirks, language patterns, and notable items) and — critically — an arcs section that compares the current profile against prior profile generations found in conversation history, so the profile sharpens with each pass instead of resetting.
---

# Personal Profile

A longitudinal self-portrait of the user, built from conversation history and memory, designed to sharpen with each pass.

## Purpose

The user wants to see themselves clearly — strengths *and* weaknesses, with evidence. They have explicitly asked Claude to avoid praise inflation and to favor direct, strategic, evidence-based feedback. This skill exists so that every profile-generation request produces a result that is both candid and grounded in observable behavior, and so that successive profiles compose into an arc rather than a series of resets.

## When to use

Trigger on any phrasing where the user asks Claude to characterize them as a person:

- "Please give me my personal profile"
- "Profile me" / "Do me a profile"
- "What do you know about me"
- "Describe me" / "How would you describe me"
- "Update my profile"
- "Self-portrait" / "Personality breakdown" / "Self-assessment"

Trigger even when the phrasing is casual or the word *profile* isn't used. The user has marked profile generation as a recurring practice, not a one-off, so err toward triggering when in doubt.

Do **not** trigger when the user asks for a profile of *someone else* (a contact, a candidate, a public figure). This skill is for self-profiling only.

## Workflow

### 1. Identify the subject

This skill is used inside a dual-user profile. Check `userMemories` for routing rules and recent context. Determine who's asking from:

- The sign-off in the current or recent messages (e.g., "JO3" → John, "S" or full-name → Susan)
- The topic context (engineering, LivingDocument, MoveDocs → John; The Puff Puff Party, Vaya Foods, Dino Detour, comms work → Susan)
- A direct possessive ("my profile" from a signed message)

If genuinely ambiguous, ask once: "Profile you or your partner?" Don't guess.

### 2. Gather evidence

Run `conversation_search` with at least four distinct queries to pull diverse signal. Vary the queries — repeating phrasings won't return different results.

Suggested query shape:
- One query around the subject's main current project or recurring topic
- One query around their professional role / domain vocabulary
- One query around any distinctive phrase or behavior already noted in memory
- One query explicitly for prior profiles: try "personal profile," then "profile me," then "describe me" if the first two return nothing

Also read `userMemories` carefully. But treat memory as **background**, not source of truth for the profile itself. Memory captures stated preferences and stable facts; the profile is about *observed* behavior, language, and patterns over time. A profile that mostly recites memory entries has failed.

If a `recent_chats` lookup would catch material that keyword search misses (recent activity in the last week or two that hasn't yet formed search-friendly keywords), use it.

### 3. Build the profile in three tiers

**Tier 1 — Laconic.** One or two sentences. Aim for a line that, if the user only read this and nothing else, would still feel sharply seen. No throat-clearing, no setup.

**Tier 2 — ELI5.** One short paragraph, no jargon, written as if explaining this person to a curious child. Captures what they do and what kind of person they are, without flattening them into a cartoon.

**Tier 3 — Detailed.** Five sub-sections, in this order:

- **Strengths** — 4–6 bullets. Each bullet names the strength *and* grounds it in a specific observed behavior or pattern. Not "analytical" — *what they actually did that was analytical*.
- **Weaknesses** — 3–5 bullets. Be direct. Name patterns of risk, exploration-to-execution gaps, blind spots, ways the strengths fail them. No softening. The user has explicitly asked for parity with strengths.
- **Quirks** — 4–6 bullets. Distinctive habits, idiosyncratic word usage, signature behaviors. Specific enough that someone who knows the user would nod in recognition.
- **Language patterns** — 4–6 bullets. Recurring vocabulary, sign-off conventions, sentence structure tendencies, voice tells. Quote actual phrases briefly (under 15 words) where useful — quotes are evidence, not ornament.
- **Notable** — 2–4 items. Things that don't fit the other categories but are diagnostic. Often the most interesting section. Meta-observations belong here (e.g., "you're asking the AI to profile you, which is itself a tell").

### 4. Add the arcs section if prior profiles exist

After the prior-profile search, if any earlier profile was found, append a sixth section:

**Arcs since last profile** — 2–5 bullets. Each bullet names a change. Examples of well-formed arc bullets:

- A strength that's deepened (with what reinforced it)
- A weakness named previously that is still unaddressed (and what would clear it)
- A new quirk or vocabulary shift
- A commitment stated in a prior profile vs. what's actually been done
- A regression — something that has gotten worse or quieter

If the user is generating a profile for the first time, say so explicitly: *"First profile — no arc data yet. This becomes the baseline."* Don't fabricate arcs.

### 5. Tone discipline

The user's stated preferences apply with full force in this skill:

- No excessive praise. No sycophancy. No softening of weaknesses.
- Direct, evidence-based, data-driven.
- Brief beats padded.
- Markdown formatting with concise headings and bullets.
- The voice is a senior colleague's candid read, not a horoscope or a LinkedIn endorsement.

End the profile with a clean sign-off appropriate to the user's register (matched, not mimicked). Don't append generic encouragement. The profile should land, not console.

## Anti-patterns

Avoid these failure modes — they are the most common ways a profile goes flat:

- **Memory-parroting.** Listing items from `userMemories` as if they were observations. Memory says "user prefers X"; the profile should observe *"user has X-shaped behavior in these contexts."* If the bullet could have been written from the memory alone with no conversation history, rewrite it.
- **Generic personality language.** "Analytical and detail-oriented," "creative and curious," "open to feedback" — these apply to everyone. Replace with specifics that reference real behavior.
- **Asymmetric softening.** Strengths in confident prose, weaknesses hedged with "sometimes" and "might want to consider." The user has asked for parity. Weaknesses get the same directness as strengths.
- **Quote inflation.** Quoting at length to fill space, or quoting things that don't add evidence. Keep quotes under 15 words and use them only where the exact phrasing matters.
- **Single-source profiling.** Drawing only from the most recent conversation. Search broadly enough that the profile reflects *pattern*, not last Tuesday.
- **Arc fabrication.** Inventing change where the search returned no prior profile. If there's no baseline, say so.
- **Closing with affirmations.** "You've got this!" / "Keep going!" — not the register the user has asked for.

## Output format

```markdown
## Laconic

[One or two sentences. Sharp.]

## ELI5

[One short paragraph in plain language.]

## Detailed

**Strengths**
- *Strength name.* Specific, evidence-grounded explanation.
- ...

**Weaknesses**
- *Pattern name.* Direct, evidence-grounded explanation.
- ...

**Quirks**
- ...

**Language patterns**
- ...

**Notable**
- ...

**Arcs since last profile** *(omit on first profile; replace with the baseline note)*
- ...
```

## Examples of well-formed observations

Good (specific, evidence-grounded):

> *Recruits adversaries on purpose.* Asked Claude to write a pitch *to ChatGPT* so a rival model would push back. The Plato-interlocutor preference is the same instinct.

Bad (generic, no evidence):

> *Open to feedback.* You like getting input from others.

Good (names a risk pattern, not just a trait):

> *Exploration-to-execution gap.* Memory flags it directly. Stress-testing can quietly become procrastination dressed as rigor.

Bad (softened, hedged):

> *Sometimes you might want to consider committing more to your ideas.*

Good (quote as evidence, under 15 words):

> Restaurant-kitchen *"86"* used as a verb. *"86 references to Susan."* Almost no one outside hospitality uses it that way in writing.

Bad (quote as ornament, padded):

> You once said, "I'd like to remove references to my partner from this document," which shows you are thoughtful about boundaries.

## A note on the recursive design

The arcs section is the reason this skill is worth having instead of a one-time prompt. Each profile run reads the prior ones and asks: *what's changed?* Over time, the profile becomes a record of pattern shifts — commitments kept, weaknesses ground down, quirks evolved or shed — rather than a snapshot that resets every time. Treat that recursion seriously. The most valuable bullets in a mature profile are the arc bullets.
