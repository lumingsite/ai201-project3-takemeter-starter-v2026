# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

My posts come from r/movies, the general film discussion subreddit, with a
smaller number from r/TrueFilm for additional volume. Both are public film
discussion communities with no login required. Reading the feed, I noticed it
splits cleanly into link-only shares (a headline plus an external article,
trailer, or image — no original writing) and original written posts (reviews,
analysis, personal takes, questions). My classifier sorts the original
written posts into three labels: `analysis` (a claim backed by a specific,
checkable detail from the film), `hot_take` (a verdict or opinion with no
such detail offered), and `ask` (a genuine question or recommendation
request with no stated position of its own).

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** Makes a claim about the film backed by a specific, checkable
detail — a scene, a line, a shot, a fact.

**Example 1:**
> In "The Departed", I just noticed this one moment where Matt Damon's
> character lets his mask slip... Billy asks what he's going to do about
> "Costello's rat"... Sullivan responds by asking whether "Frank" ever
> mentioned a name.

**Example 2:**
> The deer scene, the scene where Welby hands the orange juice to Dixon and
> POINTS THE STRAW, all the letters of Willoughby... so many iconic and deeply
> humanizing scenes in this movie.

### `hot_take`

**Definition:** States a verdict or opinion on a film with no specific detail
offered to back it.

**Example 1:**
> The film is a classic with good performances from our 4 lead characters...
> it's worth a watch. (The Haunting (1963) review — no scene cited.)

**Example 2:**
> I can't get into The Departed because how good Infernal Affairs was... just
> can't shake how exact the ripoff was.

### `ask`

**Definition:** A genuine question or recommendation request, with no stated
position of its own.

**Example 1:**
> Any good werewolf or chupacabra movie recommendations?

**Example 2:**
> Movie depicting the Appalachian Rust Belt... I'd be interested in watching
> movies from around this same time period (60s-70s) that depict what life
> was like in the area.

### The hardest boundary

**Which two labels:** `analysis` vs. `hot_take`

**The decision rule I used every time:**
If the post points to a specific scene, line, or detail in the film itself to
justify its claim, it's `analysis` — even if the reasoning is personal or the
tone is emotional. If the claim rests only on the poster's say-so with
nothing specific to point at, it's `hot_take`.

Borderline case: "19 years later, Superbad hits in a way I never expected" —
personal and emotional (friend group drifted apart, like the movie), but it
cites the actual ending scene (Seth and Evan, about to go their separate
ways) as its evidence. Under the rule, that's `analysis`.

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** r/movies, with a smaller number from r/TrueFilm
(both public, general film-discussion communities, no login required).

**Why 172, not 200:** Collecting was slower than expected because most posts
in either subreddit are link-only shares (news, posters, trailers) with no
original text to classify — on several passes fewer than half the posts in a
scroll were usable. Rather than pad the count with weak examples just to hit
200, I stopped at 172 once I had enough depth across all three labels and
the hard cases to make real decisions on the boundary, per the milestone's
stop rule.

**How I labelled them:** I collected posts in batches, filtered out link-only
posts (news links, posters, trailers with no original text), then had Claude
pre-label each post against my `analysis`/`hot_take`/`ask` definitions as I
went, which I reviewed and corrected (150 rows, marked `pre-labeled, checked`
in `labels.csv`). Separately, I collected 22 more posts and labeled all of
them myself first, with no AI help, using only my written definitions — those
rows are marked `cold`.

One thing worth noting: my cold-labeled batch skews noticeably more toward
`hot_take` (6 of 22, ~27%) than the AI-assisted batch did (20 of 150, ~13%).
Labeling alone, I was apparently more willing to call a post a bare opinion
than I was while reviewing Claude's suggestions — worth keeping in mind when
reading the agreement report next unit.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `analysis` | 71 | 41.3% |
| `ask` | 75 | 43.6% |
| `hot_take` | 26 | 15.1% |
| **Total** | 172 | 100% |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Ive been randomly watching zombie & world apocalypse movies
> lately. Ive been searching for good ones & came across The Mist.. this was
> really a good movie! The ending was sad as hell but a 10/10. This movie
> reminds me of Stranger Things somewhat.. anyone know any similar movies?"
>
> *Could have been:* `hot_take` (the rating "10/10" is asserted with no
> specific evidence) or `ask` (it closes by asking for similar recommendations).
>
> *I chose, because:* the bulk of the post is an unevidenced verdict on the
> film itself, not a request for information about something else — the
> recommendation ask reads as a tacked-on afterthought, not the point of the
> post. I labeled it `hot_take`.

**2.**
> *The post:* "Today/ tonight I watched 'of an age' ... the way the story is
> and how every character, scene, picture and music is built up in every
> moment really makes it up ... I honestly love this movie so much and it
> might be one of my top movies this year."
>
> *Could have been:* `analysis` (it does gesture at specific elements —
> "character, scene, picture and music") or `hot_take` (none of those
> elements are actually described or tied to a concrete moment in the film).
>
> *I chose, because:* my decision rule asks whether a post points to a
> *specific* detail, not just names a *category* of detail. Naming "the
> music" without describing a moment doesn't meet that bar. I labeled it
> `hot_take`.

**3.**
> *The post:* "An utter gem of satire with a cast that could be described as
> 'too good'. Jack Nicholson plays at least two major characters. Glenn
> Close. Michael J Fox. Pierce Brosnan. ... I can not find a fault."
>
> *Could have been:* `hot_take` (the overall verdict — "I can not find a
> fault" — is a pure, unargued claim) or `analysis` (it backs that verdict
> with a specific, checkable cast list).
>
> *I chose, because:* the cast list is a concrete, checkable detail about the
> film that supports the claim about casting quality, even though the
> broader "no fault" verdict isn't separately argued. Per my rule, a specific
> detail anywhere in the post is enough to qualify it as `analysis`.
>
> *Could have been:*
>
> *I chose, because:*

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, 128 max tokens,
seed 42.

**Anything I changed from the defaults, and why:** Nothing — I ran the
notebook's defaults as-is.

**Device:** CPU (Intel Mac, no CUDA/MPS available), torch 2.2.2. Recorded in
`results.json` per the milestone's note that hardware is part of what you
found, not just a detail about the laptop.

**Split sizes:** train 120 · val 26 · test 26 (from 172 total rows, 70/15/15).

Per-label counts in the test split:

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 49 | 11 | 11 |
| `hot_take` | 18 | 4 | 4 |
| `ask` | 53 | 11 | 11 |

`hot_take` has only 4 examples in the test split — the notebook flagged this
directly. A metric on 4 examples will move a lot between seeds, which is
expected given `hot_take` is also my thinnest label overall (26/172).

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* Filter raw pasted Reddit feed text down to posts with
  original writing (vs. link-only news/trailers/posters) and pre-label each
  against my `analysis`/`hot_take`/`ask` definitions.
- *What came back:* A labeled batch with a short reason per post (e.g. "cites
  a specific scene" → `analysis`; "vague praise, no evidence" → `hot_take`).
- *What I changed:* I kept most of the suggested labels but used them as a
  starting point I was responsible for reviewing, not a final answer — this
  is the `pre-labeled, checked` workflow disclosed below.

**Moment 2**

- *What I asked for:* In Milestone 2, generate 8 synthetic posts designed to
  sit on the `analysis`/`hot_take` boundary, to stress-test my definitions
  before collecting real data.
- *What came back:* 8 short posts plus a classification of each under my
  rule. All 8 classified cleanly, which told me the decision rule (does the
  post cite a specific, checkable detail from the film?) didn't need
  tightening before I started labeling for real.
- *What I changed:* Nothing in the definitions — the stress test passed as
  written.

**Pre-labelling disclosure:** 150 of the 172 rows in `labels.csv` were
pre-labeled by Claude against my written definitions, which I read and
corrected as I went (marked `pre-labeled, checked`). The remaining 22 rows I
labeled myself with no AI involvement, as the milestone's honesty check
(marked `cold`).

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

*(Run on the 222-row dataset — see the Dataset Expansion note above. train
154 · val 34 · test 34.)*

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.353 | 0.559 | +0.206 |
| Macro F1 | 0.330 | 0.345 | +0.015 |
| F1 — `analysis` | 0.467 | 0.681 | +0.214 |
| F1 — `ask` | 0.250 | 0.353 | +0.103 |
| F1 — `hot_take` | 0.273 | 0.000 | -0.273 |

Both runs scored on the same 34-post `test_split.csv` written by the
notebook, so this is a like-for-like comparison.

**What I predicted before I looked:** I predicted the trained model would
beat the baseline on `ask` and `analysis` (my two largest, best-represented
labels) but might lose ground on `hot_take`, my thinnest label with the
fewest training examples to learn from.

**What the gap actually means:** This prediction held up well on direction,
if not on the overall picture I expected. The trained model beats the
baseline on accuracy, macro F1, `analysis`, and `ask` — fine-tuning did add
something real on the labels with enough examples. But `hot_take` collapses
to F1 0.000, exactly as predicted, while the baseline (with zero training)
still manages 0.273 on it. The confusion matrix confirms this isn't a
borderline miss: the model predicts `analysis` for `hot_take` posts far more
than the reverse. With `hot_take` sitting at only 28 of 222 rows (20 train,
4 test), the model doesn't have enough examples to learn that boundary at
all, even while it learns the other two labels reasonably well. The honest
summary: fine-tuning helped overall, but it didn't touch my thinnest label —
that's a dataset-size problem, not a hyperparameter problem.

**Definitions vs. bare label names:** I also ran the baseline using no
definitions (bare label names only) to measure what my definitions are
worth. Here the result ran counter to what I'd expect: accuracy went from
0.353 to 0.529, and macro F1 from 0.330 to 0.482 — the bare label names
*outscored* my actual written definitions on this particular 34-post split.
`hot_take` in particular does much better bare (0.571) than with my
definition (0.273). I don't read this as "my definitions are bad" so much as
evidence of how volatile a 34-post, 3-label zero-shot evaluation is — a
handful of posts going either way swings these numbers a lot, which is the
same instability the three-seed run below also surfaces independently.

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Trained on the current `labels.csv` (222 rows, same dataset as the Baseline
vs. Trained section above — see the Dataset Expansion note). Device: CPU,
torch 2.2.2.

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.559 | 0.471 | 0.647 | 0.176 |
| Macro F1 | 0.345 | 0.213 | 0.439 | 0.225 |
| F1 — `analysis` | 0.681 | 0.640 | 0.744 | 0.104 |
| F1 — `hot_take` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `ask` | 0.353 | 0.000 | 0.571 | 0.571 |

Accuracy moved 0.176 across seeds — more than 10 points. My dataset is too
small or too lopsided for a stable measure at this label split. That's a
genuine diagnosis, not a mistake: `hot_take` has only 28 of 222 rows, and its
F1 is 0.000 on every single seed, not just unstable — the model never learns
to predict it at all.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy ≥ 0.65 | 0.65 | 0.559 | 0.471 | 0.647 | **MISSED** — closest at seed 2024 but still under on all three |
| 2. `analysis` F1 ≥ 0.55 | 0.55 | 0.681 | 0.640 | 0.744 | **MET** — clears target on all three seeds |
| 3. `hot_take` and `ask` F1 ≥ 0.65 each | 0.65 | hot_take 0.000 / ask 0.353 | hot_take 0.000 / ask 0.000 | hot_take 0.000 / ask 0.571 | **MISSED** — badly, especially `hot_take` |
| 4. Largest label ≤ 2× smallest | 2× | — | — | — | **MISSED** — 106 `analysis` vs 28 `hot_take` = 3.8× in the current 222-row set |
| 5. ≥80% agreement with a second labeler | 0.80 | — | — | — | Pending — Milestone 3's staff agreement check |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Confusion matrix from seed 42:

| true \ predicted | analysis | hot_take | ask |
|---|---|---|---|
| **analysis** | 16 | 0 | 0 |
| **hot_take** | 4 | 0 | 0 |
| **ask** | 11 | 0 | 3 |

**My biggest off-diagonal number, and what it means:** 11 real `ask` posts
were called `analysis`, and 0 real `analysis` posts were called `ask` — a
one-directional confusion, not a hard boundary the model is split on. The
model never predicts `hot_take` or `ask` as often as it should; it collapses
toward predicting `analysis` for almost everything. Combined with `hot_take`'s
F1 of 0.000 on every seed, this points at my smallest label (`hot_take`, 28
rows) not having enough examples for the model to learn it at all, and the
model defaulting to my largest, best-represented label instead.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 | **MISSED** | Seeds 0.559, 0.471, 0.647 — none reach 0.65, and the target has to hold on all three, not just the best draw |
| 2 | `analysis` F1 | ≥ 0.55 | **MET** | Seeds 0.681, 0.640, 0.744 — clears the target on all three seeds with room to spare |
| 3 | `hot_take` and `ask` F1 each | ≥ 0.65 | **MISSED** | `hot_take` is 0.000 on all three seeds; `ask` is 0.353, 0.000, 0.571 — neither label clears 0.65 on any seed |
| 4 | Balance (largest ≤ 2× smallest) | ≤ 2× | **MISSED** | 106 `analysis` vs 28 `hot_take` in the 222-row set = 3.8×, nearly double the cap |
| 5 | ≥80% agreement with a second labeler | ≥ 0.80 | **MISSED** | Milestone 3's staff-set check: 21/30 = 70% |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**Criterion 5 (agreement, 70%) — a labeling-consistency problem, and the root
cause of the others below.** Of my 9 disagreements with staff, 6 shared one
exact pattern: on posts where a specific, checkable fact (a stat, a count, a
named comparison) was followed by a confidently-worded closing sentence, I
labeled the post by the *tone* of the closing sentence (`hot_take`) instead of
checking whether the fact in front of it was load-bearing (`analysis`, per
staff's Rule 1). That's not noise spread across many boundaries — it's one
boundary, applied the same wrong way six times in a row. Cause: **a boundary I
applied inconsistently**, and the agreement report is direct evidence of it,
not an inference from the model's behavior.

**Criterion 3 (`hot_take`/`ask` F1) — `hot_take` is two compounding causes,
`ask` is one.** For `hot_take`: it has only 28 of 222 rows (20 train, 4 test),
which is cause #1 (too few examples) on its own. But the Milestone 3 finding
adds a second layer: if I apply the exact same analysis/hot_take boundary
inconsistently on a blind staff-defined taxonomy, I almost certainly applied
it inconsistently while building my own 222-row set too — meaning some
fraction of my 28 `hot_take` rows may actually belong to `analysis` (or vice
versa), making an already-small training signal noisier on top of being
small. The confusion matrix supports this: 4 of 4 real `hot_take` test posts
were predicted `analysis`, zero correct, zero predicted as anything else —
the model never saw a clean enough signal to learn the label at all.

For `ask`: the confusion matrix's single biggest off-diagonal is 11 real
`ask` posts called `analysis` (0 going the other way) — a one-directional
collapse toward `analysis`, not a two-way confusion. `ask` has a decent
training count (88 rows), so this isn't primarily a too-few-examples problem.
I read this as cause #4: **a task the model can't fully reach from 200
examples** — distilbert-base-uncased fine-tuned for 3 epochs on ~154 training
rows across 3 classes doesn't have enough signal to learn a confident,
well-separated decision boundary for every pair, and it defaults toward my
largest class (`analysis`, 106 rows) whenever it's unsure, which shows up as
`ask` recall loss specifically.

**Criterion 1 (overall accuracy) — a downstream consequence of the two
diagnoses above, not a separate problem.** With `hot_take` contributing
nothing (F1 0.000 on every seed) and `ask` losing a chunk of its recall to
the same `analysis`-leaning collapse, overall accuracy is being pulled down
by the same two causes already diagnosed, not a third independent issue.

**Criterion 4 (balance, 3.8×) — a collection problem, not a labeling or model
problem.** This one isn't about how I applied the boundary; it's that
confident-opinion-with-zero-evidence posts (true `hot_take`) are genuinely
rarer in r/movies and r/TrueFilm discussion than posts that either argue from
a detail (`analysis`) or ask something (`ask`). I noted this as a real,
measured finding back in Milestone 4 of the previous unit, and it held
through the dataset expansion to 222 rows. The fix here is collection effort
targeted at `hot_take`, not a labeling-consistency fix.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 21 / 30 = 70%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

**1.**
> *The post:* "Pulled the splits from the last five editions. The winning
> time has moved by 40 seconds total in ten years, but the 100th-place time
> is four minutes faster. The depth changed, the front didn't."
>
> *Staff said `analysis` / I said `hot_take`:*
>
> *My call, and why:* Staff is right. The contrast between the two numbers
> (40 seconds vs. four minutes) *is* the claim — remove the numbers and there
> is no argument left, just "the depth changed." Rule 1 asks whether the
> fact is load-bearing, and here it's the entire structure.
>
> *Which pile:* (a) — the rule already covers this, I applied it loosely by
> reacting to the confident tone of the conclusion sentence instead of
> checking whether the numbers in front of it were doing the work.

**2.**
> *The post:* "The club championship scoring changed in 2019 from top-6 to
> top-10. Since then our B team has finished above the A team twice. That's
> not depth improving, it's the scoring rewarding turnout."
>
> *Staff said `analysis` / I said `hot_take`:*
>
> *My call, and why:* Staff is right. Named date, named rule change, named
> count ("twice") — all three are used specifically to rule out an
> alternative explanation (depth improving). That's exactly what Rule 1
> describes as a load-bearing fact.
>
> *Which pile:* (a) — same pattern as #1: I read the closing editorial line
> ("that's not depth improving, it's...") as the whole post and under-weighted
> the specific facts that preceded it.

**3.**
> *The post:* "Everyone blames the hill at 18 miles. I split the field into
> people who ran the first half under 1:45 and over, and the slowdown after
> 18 was almost identical for both groups. If the hill were the problem the
> fast starters would fall apart harder."
>
> *Staff said `analysis` / I said `reaction`:*
>
> *My call, and why:* Staff is right. Nothing here is about the writer's own
> immediate experience or feeling — it's a structured comparison between two
> groups, reporting a result, and drawing an inference from it. I mistook the
> first-person "I split the field" for personal narration instead of noticing
> it was describing an analysis, not a reaction.
>
> *Which pile:* (a) — Rule 2 (reaction vs. hot_take) and the `reaction`
> definition itself are both specifically about the writer's own recent
> experience; this post has none of that.

**4.**
> *The post:* "Carbon plates aren't the whole story. My times dropped 2% in
> the shoes and another 2% the following year in the same pair, when the
> only thing that changed was going from 35 to 50 miles a week."
>
> *Staff said `analysis` / I said `reaction`:*
>
> *My call, and why:* Staff is right, and staff flagged this one `HARD`
> themselves. The opening clause reads like an opinion, but the two
> comparable percentage figures tied to two different variables (shoes vs.
> mileage) do the actual isolating work the claim depends on. Rule 1: the
> fact carries it, even though the first sentence sounds like a take.
>
> *Which pile:* (a) — I stopped reading at the opinion-flavored opener and
> didn't weigh the numbers that followed it.

**5.**
> *The post:* "Ran the same loop in 4C and in 24C within a fortnight, same
> effort by heart rate. 90 seconds slower in the heat over six miles. People
> keep saying heat costs you 'a bit' and it's more than that."
>
> *Staff said `analysis` / I said `hot_take`:*
>
> *My call, and why:* Staff is right. Same loop, same effort, controlled-ish
> comparison, with a specific number (90 seconds over six miles) directly
> supporting the conclusion. I over-weighted the closing editorial sentence
> ("it's more than that") and under-weighted the measurement in front of it.
>
> *Which pile:* (a) — same pattern as #1/#2/#4: I'm consistently letting a
> confident-sounding closing line override a load-bearing fact that precedes
> it. That's a real pattern, not four unrelated misses.

**6.**
> *The post:* "Went back through the club results going back to 2012. Every
> single year the number of people who enter the spring marathon and don't
> start it sits between 18 and 23 percent. It's not a bad-luck year, it's the
> base rate."
>
> *Staff said `analysis` / I said `hot_take`:*
>
> *My call, and why:* Staff is right. Named span (2012–present), specific
> percentage range, used explicitly to argue against a specific alternative
> explanation ("a bad-luck year"). That is precisely what Rule 1 means by a
> load-bearing fact.
>
> *Which pile:* (a) — the fifth instance of the same pattern.

**7.**
> *The post:* "Three of us ran the same session on the same day, different
> shoes, within 10 seconds of each other over 5 miles. I'm not saying shoes
> do nothing, but whatever they do was smaller than the noise between three
> people."
>
> *Staff said `analysis` / I said `reaction`:*
>
> *My call, and why:* I'm keeping `reaction`. Staff flagged this one `HARD`
> too, and I think my reading is defensible rather than wrong: this is a
> hedged, casual, first-person account of something three people did
> together ("I'm not saying... but"), with an n of 3 and no real
> measurement design — it reads much closer to an informal personal account
> than to the more deliberate test-and-infer structure in posts like #3. The
> number (10 seconds) is doing less argumentative work here than in the
> clearer `analysis` cases above, because the hedge undercuts the conclusion
> rather than the number supporting it.
>
> *Which pile:* (c) — the rule is genuinely ambiguous on how much hedging and
> how small a sample can still count as a fact "carrying" an argument, and my
> reading is defensible even if staff called it the other way.

**8.**
> *The post:* "If you need a gel before 90 minutes you have a fuelling
> problem, not an energy problem."
>
> *Staff said `hot_take` / I said `reaction`:*
>
> *My call, and why:* Staff is right. There is nothing in this post that
> ties it to something that "just happened" to the writer — no first-person
> recent event, no feeling about an experience. It's a flat, confident,
> general prescription with zero support. That's the textbook `hot_take`.
>
> *Which pile:* (a) — I think I pattern-matched on the imperative, slightly
> personal-sounding phrasing ("you have a problem") rather than checking
> whether the `reaction` definition's actual requirement (recent, first-person
> experience) was present. It wasn't.

**9.**
> *The post:* "Every injury thread on this forum is the same injury and the
> same three replies."
>
> *Staff said `hot_take` / I said `analysis`:*
>
> *My call, and why:* I'm keeping `analysis`. Staff flagged this one `HARD`
> and their reasoning is that no count is actually given — the writer asserts
> "every" without showing it. But the definitions don't require the *post*
> to show its work with a number; `analysis` requires "a specific, checkable
> fact" to exist and carry the claim, and "every injury thread" is itself a
> specific, falsifiable claim about the content of a forum — it's checkable
> in principle by anyone who goes and reads the threads, which is different
> from an unfalsifiable opinion like "road racing peaked in 2015." The rule
> doesn't explicitly say whether the checkable fact has to be *quantified in
> the post itself* or merely *exist and be checkable*, and I don't think
> that's spelled out anywhere in `staff_taxonomy.md`.
>
> *Which pile:* (b) — the rule genuinely doesn't say whether a specific,
> verifiable-in-principle claim with no attached number counts as the kind
> of "checkable fact" Rule 1 means, or whether it needs a number/date/quote
> attached to qualify. That's a gap, not a misapplication on my part.

**What the pattern in my disagreements tells me:** Six of my seven losses
(#1, #2, #4, #5, #6, #8) share one pattern: I kept weighting the post's final,
most confidently-worded sentence over the specific fact that preceded it,
even when the fact was clearly what the conclusion depended on. That's a
consistent labeling habit, not six unrelated mistakes — worth watching for
in my own 222-row set, where the same thing could be happening between my
`analysis` and `hot_take` labels. The two I'm keeping (#7, #9) are both posts
staff themselves flagged `HARD`, and both involve a genuine gap in how
explicit a definition has to get before a fact "counts" — a hedge-and-small-
sample case, and a quantifier-with-no-number case. Those feel like legitimate
defensible reads, not me being stubborn.

---

## The Improvement

**What I changed:** Added 30 more `hot_take` examples (28 → 58, roughly doubling
the label), collected and labeled with the same care as the original batch
(short, confident, no-evidence reactions — e.g. "Just watched X, loved it"
with no specific detail). `labels.csv` went from 222 to 282 rows; `hot_take`'s
share rose from 12.6% to 20.6%.

**Which diagnosis pointed at it:** Criterion 3 — `hot_take` F1 was 0.000 on
every one of the three "before" seeds, and the confusion matrix showed all
real `hot_take` test posts being predicted `analysis`. The most direct
candidate cause from the four standard ones is **too few examples for a
label**: 20 training rows for one of three classes is very little for a
transformer fine-tune to learn from, independent of any labeling-consistency
issue.

### Run Log — After

*(Also run on the device recorded in `results_three_seeds_after.json`: CPU,
torch 2.2.2. Same 282-row `labels.csv`, same three seeds.)*

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.512 | 0.512 | 0.535 | 0.023 |
| Macro F1 | 0.324 | 0.326 | 0.356 | 0.032 |
| F1 — `analysis` | 0.655 | 0.644 | 0.667 | 0.023 |
| F1 — `hot_take` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `ask` | 0.316 | 0.333 | 0.400 | 0.084 |

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy ≥ 0.65 | 0.65 | 0.512 | 0.512 | 0.535 | **MISSED** — further under target than before |
| 2. `analysis` F1 ≥ 0.55 | 0.55 | 0.655 | 0.644 | 0.667 | **MET** — still clears on all three seeds |
| 3. `hot_take` and `ask` F1 ≥ 0.65 each | 0.65 | hot_take 0.000 / ask 0.316 | hot_take 0.000 / ask 0.333 | hot_take 0.000 / ask 0.400 | **MISSED** — `hot_take` unchanged at 0.000 |
| 4. Largest label ≤ 2× smallest | 2× | — | — | — | **MISSED, but closer** — 125 `analysis` vs 58 `hot_take` = 2.16× (was 3.8×) |
| 5. ≥80% agreement with a second labeler | 0.80 | — | — | — | **MISSED** — 70% (unchanged; no new agreement check run this milestone) |

Confusion matrix (seed 42, after):

| true \ predicted | analysis | hot_take | ask |
|---|---|---|---|
| **analysis** | 19 | 0 | 0 |
| **hot_take** | 8 | 0 | 1 |
| **ask** | 12 | 0 | 3 |

**Did it help, and how do I know:** Mostly no, on the thing I specifically
targeted — and that's a real result, not a non-finding. `hot_take` F1 is
exactly 0.000 on all three "after" seeds, identical to "before," even though
I nearly doubled its training examples (20 → ~46 train rows) and its test
split grew from 4 to 9. The confusion matrix shows the same one-directional
collapse: `hot_take` posts are still predicted `analysis` almost every time.
Doubling the example count did not touch the sample-size cause of the
`hot_take` failure, which tells me the deeper problem is more likely the
labeling-inconsistency cause Milestone 4 flagged (the analysis/hot_take
boundary I apply inconsistently), or a genuinely hard boundary for this
model at this scale — not just "needs more rows."

The change did help on something I wasn't specifically targeting: **stability**.
The accuracy spread across seeds dropped from 0.176 (unstable, "too small or
lopsided for a stable measure") to 0.023 (the notebook's own threshold for
"stable enough to trust"). Overall accuracy and macro F1 both came in
slightly *lower* on average than before (0.512–0.535 vs 0.471–0.647), but far
more consistently — a smaller, more reliable number beat a larger, noisier
one. Balance also improved substantially (3.8× → 2.16×), even though it
still misses the 2× cap. So: the diagnosed criterion stayed missed, but two
other real problems (instability, lopsidedness) measurably improved from the
same single change.

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

## Update — Dataset Expansion (after submission)

This section is additive — nothing above it was changed or overwritten.

After the Unit 5 submission (172 rows: 150 pre-labeled + 22 cold), I collected
50 more posts from r/movies and r/TrueFilm and had them pre-labeled and
reviewed the same way as the original batch, bringing the pre-labeled count
to exactly 200. Combined with the 22 cold-labeled rows, `labels.csv` now has
**222 total rows**.

**Updated counts per label (all 222 rows):**

| Label | Count | Share |
|---|---|---|
| `analysis` | 106 | 47.7% |
| `ask` | 88 | 39.6% |
| `hot_take` | 28 | 12.6% |
| **Total** | 222 | 100% |

No label exceeds 70%. `hot_take` remains the thinnest label, consistent with
the original dataset.

**Update, Unit 6:** starting with this unit's Milestone 1, `results.json`,
`test_split.csv`, and every milestone from here on were re-run against the
full 222-row `labels.csv` (train 154 · val 34 · test 34), replacing the
172-row numbers the Unit 5 sections above were originally written against.
The **Baseline vs. Trained**, **Run Log — Before**, and confusion matrix
sections below reflect the 222-row run. The Unit 5 sections above this one
(Label Taxonomy, The Dataset, The Training Run counts, etc.) are left as
originally submitted and are not retroactively edited — this note is the
record of the change.

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
