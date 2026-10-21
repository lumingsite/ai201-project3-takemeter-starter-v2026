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

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



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

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
