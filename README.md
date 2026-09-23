# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This is a retrieval-augmented Q&A system built on the `city_guides` corpus: fourteen long, sectioned travel guides covering nine fictional towns plus five cross-cutting guides (eating, walking, seasons, accessibility, and regional transport). It answers specific, factual questions about those towns — opening hours, the best time of year to visit, whether a place is walkable, where to eat on a given day — by embedding the question, retrieving the closest chunks from a Chroma vector index, and asking a language model to answer using only what was retrieved. A relevance gate checks the best retrieved distance before generating anything, so questions clearly outside the corpus (unrelated trivia, other domains entirely) get refused instead of answered with a guess.

## Chunking Strategy

**Chunk size:**
**Overlap:**

<!-- What about YOUR documents made you pick these numbers? Short posts and
     long sectioned guides don't want the same chunking, and "800 seemed
     reasonable" earns nothing. Point at something you noticed when you read
     the documents in Milestone 1.

     If you changed your mind partway through, say so and say why. That's worth
     more than pretending you got it right first time.

     Milestone 3. -->

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `` — produced by: chunker.py::fallback_split``

```# Getting around the region with limited mobility

An honest assessment rather than a promotional one. Some of these places are
difficult and it is better to know in advance.

## Straightforward

**Thornby Wells** is the easiest town in the region. It is flat, compact, and
everything is within three minutes of everything else. Parking is free for two
hours anywhere in town and the station is centr
```

**Chunk 2** — source: `` — produced by: chunker.py::fallback_split``

```cheese and little else, and it closes at 4pm. Bring supplies; this is not a place with options.

## What to see

The valley itself is the attraction. The footpath network is dense and well marked, and a circuit taking in three of the four villages is about nine miles with 500 metres of ascent. The chapel in the second village is 12th century and always unlocked.

## Where to stay
```

**Chunk 3** — source: `` — produced by: chunker.py::fallback_split``

```attached to the mill, open 10 to 4 daily except Tuesdays, which sells bread made from the flour ground twenty metres away and is the reason most people come. One pub, food served lunchtimes and Thursday to Saturday evenings.

## What to see

The mill runs tours on the hour from 11 to 3 and the machinery is operating during them, which is loud and much more impressive than a static exhibit. The chu
```

**Chunk 4** — source: `` — produced by: chunker.py::fallback_split``

```# Marchwood

Marchwood is the regional hub — 180,000 people, the junction everyone changes trains at, and a city most visitors pass through rather than stop in. That is a mistake, though an understandable one, since almost nothing of interest is near the station.

## Getting there

Every railway line in the region meets here, which is the city's defining feature. Trains to Brightwater run every 40
```

**Chunk 5** — source: `` — produced by: chunker.py::fallback_split``

```lk up.

## Walking and cycling

The river path from Brightwater runs four miles upstream on a good surface. The
old railway trackbed from Kestrelford runs six miles on an easy gradient and is
the best walking in the region for the effort involved. The coastal path from
Halden Bay is more serious — exposed, and closed in high wind.

Cycling is pleasant on the river path and the trackbed, and unplea
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** Where to go eat at Elder Ness on Monday?

**Answer:**

```
Based on the provided documents, Elder Ness has one pub, but it is closed on Mondays, and there is no mention of anywhere else to eat there on that day (*guide_eating.md*).

Sources retrieved: guide_accessibility.md, guide_eating.md, guide_elder_ness.md, guide_kestrelford.md
```

**My relevance cutoff:**

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

| Question | In corpus? | Best distance |
|---|---|---|
|  |  |  |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** I asked Claude to run three of my five test questions through `app.py retrieve` and read the *full* chunk text (not just the truncated preview) to judge whether the top-5 results were genuinely on-topic or just sharing a few words with the question. For "What's the most accessible city in town?" it came back showing that only the #1 result (distance 0.412, naming Thornby Wells) actually answered the question — ranks 2–5 (distance 0.49–0.54) were near-identical "Practical notes" boilerplate about hospitals and mobile coverage that appears in every town's guide, pulled in by word overlap with "accessible" rather than actual topical relevance. That changed how I'm reading distance scores for Milestone 4: I stopped assuming the whole top-5 is relevant just because it passed the gate, and I'm now looking at the ~0.46–0.49 gap between "answers the question" and "shares vocabulary" as a candidate place for the cutoff, rather than trusting rank order alone.

**2.** I asked Claude to run `app.py ask` on the same three questions to see the actual generated answers with sources. For "When is the best time to visit Halden Bay," I expected "June and September" (that's what I wrote in `questions.py`), but the model's answer only said "June," citing `guide_seasons.md`. That told me the September mention either isn't in the chunks that got retrieved for this phrasing, or it got separated from the June sentence by chunking — something I need to check directly in Milestone 3's diagnosis step rather than assume the retrieval is simply wrong.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

> No `scorer.py` exists yet, so I judged every run myself by reading the
> "Real output" sections of the three files below. Retrieval is deterministic
> (same top-k sources and best distance on every run, every file), so
> criterion 1 and criterion 3 don't actually vary between Run 1/2/3 — only
> the generated wording does (criterion 2). Run 1 = `results/run_2026-09-23_1838.md`,
> Run 2 = `results/run_2026-09-23_1906_before.md`, Run 3 = `results/run_2026-09-23_1908_before.md`
> (three separate full executions of `run_eval.py`, each already averaging
> 3 internal regenerations per question).

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. | | | | | |
| 5. | | | | | |

**Criterion 1 evidence** — produced by `store.py::search` (sources/distance) and judged against `questions.py::QUESTIONS[*]["expects"]`. The one miss is the same question in all three files:

> **When is the best time to visit Halden Bay** — expects `"June and September"`.
> Retrieved sources every run: `guide_halden_bay.md, guide_regional_transport.md, guide_seasons.md, guide_walking.md` (best distance 0.2926).
> Across all 9 generations (3 files × 3 runs), the answer mentions June (and often the July/August parking problem) but **never September** — e.g. from `run_2026-09-23_1908_before.md`, run 3:
> ```
> June is excellent everywhere. (Source: `guide_seasons.md`)
> ```
> `guide_halden_bay.md` itself has a `## When to go` section reading "June and September are the sweet spot," so the fact exists in the corpus — the chunk that got retrieved from that file evidently wasn't the one carrying that sentence.

**Criterion 2 evidence** — every one of the 45 generated answers (5 questions × 3 runs × 3 files) names at least one source. Example from `run_2026-09-23_1838.md`, "Should I walk to Corry Vale" — run 2:

> ```
> The **Corry Vale circuit** provides a moderate walk that takes in three of the four villages, covering about nine miles with 500 metres of ascent (*guide_walking.md*).
>
> Additionally, the documents note that Corry Vale features footpaths rather than pavements, with villages located two to four miles apart, and lacks public transport (*guide_accessibility.md*).
> ```

**Criterion 3 evidence** — produced by `run_eval.py::check_out_of_scope`, cutoff 0.6. Identical in all three files:

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.799 | refused |
| How do I change the oil in a diesel engine? | 0.892 | refused |
| Who won the 1994 World Cup? | 0.811 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.839 | refused |
| How do I write a for loop in Rust? | 0.839 | refused |

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer (4 of 5) | MET | 4/5 held in all three separate runs (`run_2026-09-23_1838.md`, `1906_before.md`, `1908_before.md`). The one recurring miss is the same question every time — "When is the best time to visit Halden Bay" expects "June and September," but across all 9 generations (3 runs × 3 internal regenerations) the answer only ever surfaces June, never September, even though `guide_halden_bay.md`'s own `## When to go` section literally says "June and September are the sweet spot." That sentence's chunk evidently isn't among the ones retrieval hands back, so it missed consistently rather than occasionally — the other four questions held 3/3 every time. |
| 2 | Every answer names a source (5 of 5) | MET | 5/5 in every one of the three runs — all 45 generated answers (5 questions × 3 regenerations × 3 runs) named at least one source file with no exceptions. Not close either way. |
| 3 | Gate stops out-of-corpus questions (4 of 5) | MET | The gate refused 5 of 5 out-of-scope questions, identically in all three runs (this check is deterministic, not resampled). Best distances for the out-of-scope set (0.799–0.892) sit well clear of the 0.6 cutoff, so there's no borderline case pulling this down in a future run. |
| 4 | At least 4 of 5 sample chunks read as a complete section | MISSED | Of the 5 sample chunks pasted above (from `chunker.py::fallback_split`), 2 start cleanly on a heading (Chunk 1, Chunk 4) but every single one — including those two — ends mid-word or mid-sentence: Chunk 1 cuts off at "...station is centr", Chunk 3 starts at "attached to the mill..." (already mid-sentence) and ends at "...The chu", Chunk 4 ends at "...run every 40". That's 0 of 5 chunks intact start-to-finish, well under the 4-of-5 target — not a close call. |
| 5 | Every corpus source is retrieved by at least one test question | MISSED | Pooling the "Sources retrieved" lists across all 5 test questions (from any of the three run files, since retrieval is deterministic) gives 10 distinct files. The corpus has 14: `guide_brightwater.md`, `guide_givens_mill.md`, `guide_marchwood.md`, and `guide_thornby_wells.md` never appear for any of my five questions, so 4 of 14 sources are never exercised by this test set. |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
