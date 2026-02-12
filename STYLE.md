# Writing Style Guide

Guidelines for documentation in this repository.

## Tone

Write technically but directly. Prefer short sentences. Use fragments when they
work. Don't hedge unnecessarily—if something is true, say it's true.

Contractions are fine. "It's", "doesn't", "won't", "we've"—all acceptable.

## Words to Avoid

These words are overused in AI-generated text and sound hollow:

| Category    | Avoid                                                                                                                                                              |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Emphasis    | delve, leverage, robust, comprehensive, seamless, crucial, pivotal, paramount, transformative, intricate, multifaceted, nuanced, meticulous, cornerstone, hallmark |
| Business    | empower, unlock (potential), harness, foster, streamline, optimize, enhance, elevate, facilitate, utilize, synergy, cutting-edge                                   |
| Transitions | Moreover, Furthermore, Additionally (at sentence start), Notably, Importantly                                                                                      |
| Hedging     | Certainly, Absolutely, Indeed, Essentially, Fundamentally                                                                                                          |

Use instead: simple verbs, direct statements, fewer adverbs.

**"Utilize" → "use"** **"Leverage" → "use"** **"Enable" → "let", "allow", or
restructure the sentence**

## Phrases to Avoid

Empty openers:

- "In today's X, Y is more important than ever"
- "In the ever-evolving world of..."
- "Let's dive into..." / "Let's delve into..."
- "Whether you're a X or a Y..."
- "It's no secret that..."

Filler:

- "It's important to note that..."
- "It's worth mentioning that..."
- "Needless to say..."
- "At the end of the day..."
- "Moving forward..."
- "That being said..."
- "In light of this..."

Just say the thing.

## Structure

**Vary paragraph length.** Some paragraphs are one sentence. Some are four.
Don't make them all the same.

**Vary sentence length.** Mix long explanatory sentences with short punchy ones.
Fragments work too.

**Break parallel structure sometimes.** Lists don't need identical grammar in
every item. Real writing has rough edges.

**Avoid perfect symmetry.** Not every section needs three subsections. Not every
list needs five items.

## Lists

Lists work well for examples, enumerations, and comparisons. Use them freely.

Bullet points shouldn't all have the same word count. Some items are a phrase.
Some are a sentence. One might be two sentences if it needs explanation.

Numbered lists when order matters. Bullets when it doesn't.

## Voice

**Use active voice by default.** "The observer produces observations" not
"Observations are produced by the observer."

**First person plural is fine.** "We call this..." or "We need primitives
for..." sounds natural in technical docs.

**Take positions.** "This is constraining" is better than "This may be
considered constraining by some."

## Formatting

Use bold sparingly—for key terms on first use, not for emphasis. Don't use bold
as a substitute for headings. If content needs a heading, give it one.

Em dashes work but don't overuse them. One per paragraph maximum, usually fewer.

Headers should be descriptive but not SEO-optimized. "Branch and Leaf
Observations" not "What Are Branch and Leaf Observations and Why Do They
Matter?"

## Technical Writing Specifics

**Examples over abstractions.** Show the X.509 certificate case. Show the code
signing case. Then generalize.

**Diagrams earn their space.** ASCII diagrams are good when they clarify
structure. Don't add them just for visual interest.

**Acknowledge limits.** If something is an open question, say so. If the
terminology might be wrong, note it.

## What Human Writing Looks Like

Human writing has:

- Occasional opinion ("This is constraining")
- Varied rhythm
- Imperfect parallelism
- Specific examples from the real world
- Admitted uncertainty
- Personality—even in technical docs

Human writing doesn't have:

- Perfect grammatical symmetry everywhere
- Every possible objection pre-addressed
- Relentless positivity
- Generic examples that could apply to anything
- Uniform paragraph lengths
- Transitions announcing "Now we will discuss X"

## Quick Test

Before committing, read your text aloud. If it sounds like a press release or a
Wikipedia summary, rewrite it.
