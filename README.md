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

TakeMeter sorts Hacker News comments by what kind of contribution they make. The community is public Hacker News front-page threads, and the unit is a single top-level comment. It has four labels: `evidence` (something a stranger can check), `opinion` (a judgment with nothing to check), `experience` (a first-person account), and `question` (a genuine request for information). I used a fourth label as a stretch feature because first-person accounts and real questions were both common enough in the text that three labels would have forced them into `opinion` or `evidence`.

**Stretch features:** a fourth label (`question`), plus extra `question` examples. I added 14 extra `question` rows beyond the first 200 rather than the 50 the stretch asks for, because I ran out of questions in the 359 comments I collected. The second-run stretch was not done in unit 5; the unit 6 improvement (epochs 3 to 10) is a hyperparameter change, measured in "The Improvement" below.

**Result of the first run (details in the training section):** it did poorly. It reached 0.455 accuracy and never predicted `experience` or `question`.

---

## Label Taxonomy

### `evidence`

**Definition:** Makes a point backed by something a stranger could check without trusting the writer: a link, a quote, a number, a date, or a command.

**Example 1:**
> On x86_64 and x32, the syscall number goes in eax, not rax. The upper 32 bits of rax are ignored and can be anything.

**Example 2:**
> This piece was published in mid-October based on rumored numbers. Anthropic released their Q3 numbers today which show: - $16.96B in revenue Q3 - Up 47% quarter over quarter (Q2 was $11.6B) - Up >1000% year over year I wouldn't say their growth "flattened"? [...]

### `opinion`

**Definition:** A judgment, prediction, or argument with no checkable support offered. Rhetorical questions count as opinion.

**Example 1:**
> Intel, like Boeing cannot fail. If you believe they can, you better start searching for a bunker. Advanced chip manufacturing is so incredibly hard. The finance bozos are clueless, again.

**Example 2:**
> Everyone is really tightening usage now because of all the slopmaxers who burned millions of dollars of tokens and achieved nothing.

### `experience`

**Definition:** A first-person account of what the writer did, used, or saw, where the only support is that the writer says so.

**Example 1:**
> My old T420 still works and I still always smile when using the keyboard on it. Pure bliss.

**Example 2:**
> Well, I did it. This was the straw that broke the camel's back: I have moved over to Rocky Linux. Took me maybe four hours to do the full transition. [...]

### `question`

**Definition:** A genuine request for information, an explanation, or a recommendation that someone could answer.

**Example 1:**
> I'm not a security expert. Requests for comments from career security experts: is this any good?

**Example 2:**
> What is keeping people on Ubuntu instead of just using Debian? I'm genuinely curious. What is Ubuntu offering that Debian doesn't provide?

### The hardest boundary

**Which two labels:** `evidence` and `experience`.

Comments like "I tried this, it took four hours, here's my benchmark" are both a first-person account and full of specifics. They are the biggest source of pauses.

**The decision rule I used every time:** Ask whether a stranger could check the support without trusting the writer. If the comment contains a link, a quote with its source, a named document or version, or a command someone can run, it is `evidence`, even if it is written in the first person. If the only support is "I did this" or "I saw this", with or without a number that only the writer could know ("took four hours", "I have 20 servers"), it is `experience`.

A second rule I hit often: a question mark does not make a `question`. If the comment is asking in order to make a point ("Why would anyone use this?"), it is `opinion`. If someone could reasonably answer it, it is `question`.

---

## The Dataset

**Where the posts came from:** 359 top-level comments from the first dozen comments of 60 front-page Hacker News stories, pulled in 2026 from the public Hacker News Firebase API (no login needed), shuffled, restricted to 60-600 characters. I labeled the first 200 in shuffled order and then added 18 more from the remaining 159 to thicken `question`, so the file is 218 posts. The 141 comments I did not use are not in the file.

**How I labelled them:** I based them on my understanding of how the extracted text was formatted. For example, opinions and questions picked out.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| opinion | 91 | 41.7% |
| experience | 48 | 22.0% |
| question | 40 | 18.3% |
| evidence | 39 | 17.9% |
| **Total** | 218 | 100% |

**Three hard cases**

**1.**
> *The post:* "> the number of Americans using AI at work fell significantly between the summers of 2025 (46%) and 2026 (33%). Surprising. I couldn't find the study they're citing, and they didn't include a link. Anecdotally, my friends across various industries (tech, finance, medicine, scientific research) are all using AI more heavily as time goes on."
>
> *Could have been:* `evidence` or `opinion`.
>
> *I chose* `evidence`, because it quotes specific figures that can be checked. This is the weakest call of the three: the writer's own argument rests on anecdote and says they couldn't verify the quote, so `opinion` is defensible.

**2.**
> *The post:* "Have companies been compromised because of this vulnerability? Ubuntu has an absolutely massive install base. The lack of notable incidents suggests that 9.8 might not reflect real-world well. Maybe it's time for StealMyBitcoinV2…"
>
> *Could have been:* `question` or `opinion`.
>
> *I chose* `question`, because it opens with a genuine request that someone could answer with a yes or a link. But the rest argues a position, so it is nearly a toss-up.

**3.**
> *The post:* "I tried to reproduce this using Astra for a natural language to bash command small model[1][2], but got no net improvement. Editing 1,920 weights fixed some grep mistakes but broke other commands. [...] [1] https://news.ycombinator.com/item?id=49966238 [2] https://news.ycombinator.com/item?id=49869735"
>
> *Could have been:* `experience` or `evidence`.
>
> *I chose* `evidence`, because it links the two things it is working from and gives a specific count. Without the links I would have said `experience`. This is the case that produced the rule above.

---

## The Training Run

**Base model:** distilbert-base-uncased

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128 tokens, seed 42. Trained on `mps` (Apple Silicon GPU), torch 2.15.0.

**Anything I changed from the defaults, and why:** Nothing. The run used every default.

**Split sizes:** 152 train / 33 val / 33 test. Test split per label: evidence 6, opinion 14, experience 7, question 6. Three of the four labels have fewer than about 8 test examples, so any per-label F1 from this run will move a lot between seeds.

**What came out:** accuracy 0.455, macro F1 0.259. It never predicted `experience` or `question` (F1 0.000 for both) and sent most `experience` posts to `opinion`. This is below my own targets for accuracy (0.60) and per-label F1 (0.45).

---

## How I Used AI

**Moment 1**

- *What I asked for:* Claude wrote eight comments that sit on the boundaries between my four labels, for the Milestone 2 stress test, and then classified them against my definitions.
- *What came back:* Eight short synthetic comments. Because the same assistant wrote and classified them, this was a check on whether the rules read clearly, not an independent test.
- *What I changed:* Nothing. The definitions stayed as first written. The test did not change them.

**Moment 2**

- *What I asked for:* Collect real public comments for the dataset and label them against my definitions. Reddit refused unauthenticated requests, so the assistant switched the source to the Hacker News API.
- *What came back:* 359 real comments, 218 of which went into `labels.csv` with labels.
- *What I changed:* The labels.

**Moment 3 (unit 6)**

- *What I asked for:* The assistant ran the baseline, the three-seed training, and the agreement check, and read the confusion matrix.
- *What came back:* It spotted that the model was predicting `opinion` for nearly every post in all three seeds, and that training loss had barely moved off chance. It proposed more epochs, which became my one improvement.
- *What I changed:* The number. I picked 10 and ran it.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

**Stretch features I added:** a confidence check (criterion 5 tests it), and a short per-label error write-up for `question` (under "What's Still Broken").

## Baseline vs. Trained

Both scored on the same 33 held-out posts (`test_split.csv`, seed 42). The baseline is `facebook/bart-large-mnli` zero-shot, given my label definitions from `label_definitions.txt`. "Trained" is the unit 5 run in `results.json` (3 epochs).

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.364 | 0.455 | +0.091 |
| Macro F1 | 0.297 | 0.259 | -0.038 |
| F1 — `evidence` | 0.429 | 0.444 | +0.016 |
| F1 — `experience` | 0.000 | 0.000 | +0.000 |
| F1 — `opinion` | 0.133 | 0.591 | +0.458 |
| F1 — `question` | 0.625 | 0.000 | -0.625 |

The same baseline against the improved model (10 epochs, seed 42 from `results_three_seeds_after.json`):

| Measure | Baseline | Trained (10 epochs) | Difference |
|---|---|---|---|
| Overall accuracy | 0.364 | 0.667 | +0.303 |
| Macro F1 | 0.297 | 0.683 | +0.386 |
| F1 — `evidence` | 0.429 | 0.727 | +0.298 |
| F1 — `experience` | 0.000 | 0.857 | +0.857 |
| F1 — `opinion` | 0.133 | 0.615 | +0.482 |
| F1 — `question` | 0.625 | 0.533 | -0.092 |

**The definitions made the baseline worse.** I also ran the baseline with bare label names (no definitions): accuracy 0.455 and macro F1 0.380 (`baseline_bare_names_results.json`), against 0.364 and 0.297 with my definitions. My definitions were worth -0.091 accuracy to this model. I report the with-definitions run as the baseline because that is the one the assignment specifies. Against the stronger bare-names run, the 3-epoch trained model's accuracy advantage is zero (0.455 vs 0.455).

**What I predicted before I looked:** Written to `prediction.md` before running `baseline.py`: the baseline would beat my trained model on `question` and probably `evidence`, my trained model would beat it on `opinion` and `experience`, and overall the baseline would be close to or better than the trained model.

I was right about `question` (baseline 0.625 vs 0.000) and about the baseline being competitive overall (macro F1 0.297 vs 0.259). I was wrong about `experience`: both scored 0.000. I was wrong about `evidence`: they tied (0.429 vs 0.444). I was right that the trained model wins on `opinion`, but for a bad reason, covered below.

**What the gap actually means:** For the 3-epoch model, the +0.091 accuracy gap is almost entirely the `opinion` label, and it is not a sign of learning. That model predicts `opinion` for almost every post, and `opinion` is 42% of the data, so it gets 42-45% accuracy by always guessing the biggest label. The baseline is not good either (0.364, below always guessing `opinion`), so the honest summary is that neither model had learned this task. With 10 epochs the trained model clearly beats the baseline on accuracy and on three of four labels, and loses on `question`. Fine-tuning only added something once it was trained long enough to leave the majority-label guess.

---

## Run Log — Before

3 epochs, learning rate 2e-5, batch 16, max length 128. Seeds 42, 7, 2024. Trained on `mps` (Apple Silicon GPU), torch 2.15.0. Source: `results_three_seeds_before.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 | 0.455 | 0.424 | 0.424 | MISSED |
| 2. Every label F1 | ≥ 0.45 | lowest 0.000 (`experience`, `question`) | lowest 0.000 (all but `opinion`) | lowest 0.000 (all but `opinion`) | MISSED |
| 3. Balance | no label > 50%, none < 12% | 41.7% / 17.9% | 41.7% / 17.9% | 41.7% / 17.9% | MET |
| 4. Agreement with staff | ≥ 75% (22 of 30) | 100% (30/30) | same | same | MET |
| 5. Confidence | top third ≥ 0.75 and above bottom third | 0.545 vs 0.273 | 0.818 vs 0.091 | 0.818 vs 0.000 | MISSED |

Rows 3 and 4 do not depend on the seed, so they repeat.

Full spread table:

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.455 | 0.424 | 0.424 | 0.030 |
| Macro F1 | 0.259 | 0.149 | 0.152 | 0.110 |
| F1 — `evidence` | 0.444 | 0.000 | 0.000 | 0.444 |
| F1 — `opinion` | 0.591 | 0.596 | 0.609 | 0.018 |
| F1 — `experience` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `question` | 0.000 | 0.000 | 0.000 | 0.000 |

### Confusion matrix

Seed 42, before (3 epochs). Rows are true labels, columns are guesses.

| true \ predicted | evidence | opinion | experience | question |
|---|---|---|---|---|
| **evidence** | 2 | 4 | 0 | 0 |
| **opinion** | 1 | 13 | 0 | 0 |
| **experience** | 0 | 7 | 0 | 0 |
| **question** | 0 | 6 | 0 | 0 |

**My biggest off-diagonal number, and what it means:** 7 real `experience` posts were called `opinion`, and 0 went the other way. But that is not a boundary the model failed to learn. The whole `opinion` column is full (4 + 7 + 6 off-diagonal), and the `experience`, `question`, and (on seeds 7 and 2024) `evidence` rows send everything to `opinion`. The model did not learn any boundary. It guessed the biggest label.

---

## Verdicts and Diagnoses

Against the targets in `criteria.md`, "before" run. A target has to hold on all three seeds.

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.60 | MISSED | 0.455, 0.424, 0.424. Not one seed reached 0.60. |
| 2 | Every label F1 | ≥ 0.45 | MISSED | `experience` and `question` scored 0.000 on all three seeds, and `evidence` scored 0.000 on two. |
| 3 | Balance | no label > 50%, none < 12% | MET | Largest `opinion` 41.7%, smallest `evidence` 17.9%. The file does not change between seeds. |
| 4 | Agreement with staff | ≥ 75% | MET | 30/30 on the staff set. I have caveats on what that proves; see the Agreement Report. |
| 5 | Confidence | top third ≥ 0.75 and above bottom third | MISSED | Seeds 7 and 2024 pass (0.818 vs 0.091 and 0.818 vs 0.000). Seed 42 fails at 0.545. A target has to hold on every seed. |

**Diagnoses**

**Criteria 1 and 2: the model underfit and collapsed to the biggest label.** Evidence:
- On seed 7 all 33 test posts were predicted `opinion`, and on seed 2024 it was 32 of 33.
- Training loss barely moved: 1.35 to 1.19 on seed 42, where random guessing across four labels scores about 1.386. With 152 training posts and batch size 16, three epochs is only about 30 optimizer steps.
- The cause is not too few examples per label, because the same 152 posts with the same labels produced `experience` F1 of 0.857 and `question` F1 of 0.533 on seed 42 when I trained longer (see "The Improvement"). A labeling problem would not have been fixed by more epochs.

I used the agreement report as evidence here and it is weak evidence. My agreement was 30/30, but on the staff's three-label taxonomy, not mine, and the same AI assistant did both sets of labels. So it rules out nothing about how consistently I applied `evidence` vs `experience`. What ruled out a labeling cause was that the fix was training length.

**Criterion 5: confidence meant little when the model was nearly constant.** On seed 42 the model predicted `opinion` on almost everything with a top probability of about 0.4 (mean confidence when right 0.400, when wrong 0.384), so its "most confident third" was just whichever posts it was least unsure about. On seeds 7 and 2024 the pattern did exist (0.818 vs 0.091 and 0.0), so the criterion is sensitive to seed. I count it as a miss because the target said it had to hold.

---

## Agreement Report

I labeled the 30 staff posts under the staff taxonomy in `data/staff_taxonomy.md` (rules 1 to 4), and saved them to `my_staff_labels.csv` before opening `data/staff_labels.csv`.

**Agreement rate:** 30 / 30 = 100%

**Disagreements:**

Posts where I could see a staff disagreement being defensible, and what decided it:
- **sp_014** ("Times are down about 3% since the shoe rules changed, so obviously the shoes are the whole story"): `hot_take`. It has a statistic, but rule 1's own example ("times are down 3%... ruined racing") shows the number is decoration on a conclusion already reached.
- **sp_018** ("Track sessions are wasted on anyone running slower than 8 minute miles"): `hot_take`. "8 minute miles" is a threshold in the claim, not a checkable fact that carries the argument.
- **sp_007** (4C vs 24C loop, "90 seconds slower"): `analysis`. The measurement is the reason for the conclusion, so rule 1 is met.
- **sp_010** (three runners, different shoes, within 10 seconds): `analysis`. The comparison is the argument, even though it hedges.
- **sp_029** ("Just realised the shoes I set every PB in this year are the ones I bought by accident"): `reaction`. Rule 2: it reports something about the writer, not a general claim about the world.

**What the pattern in my disagreements tells me:** There is no pattern because there were no disagreements. What it does tell me is limited: I can apply someone else's written rule to obvious cases and to the posts the rules were written for (014 and 018 are the two the rules explicitly cover).

---

## The Improvement

**What I changed:** Epochs, from 3 to 10. Nothing else: same labels, same taxonomy, same criteria, same learning rate (2e-5), batch size (16), max length (128), base model, and the same three seeds (42, 7, 2024). In section 6 of the notebook I set `TRAIN_EPOCHS = 10` and `RUN_LABEL = "after"`, which writes `results_three_seeds_after.json`. `EPOCHS` in section 2 stays at 3, so `results.json` is still the 3-epoch run.

**Which diagnosis pointed at it:** Criteria 1 and 2 failed because the model collapsed to `opinion`, and training loss had barely moved off chance after about 30 steps. That is a training problem, so I changed a training setting.

### Run Log — After

10 epochs. Source: `results_three_seeds_after.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 | 0.667 | 0.758 | 0.727 | MET |
| 2. Every label F1 | ≥ 0.45 | lowest 0.533 (`question`) | lowest 0.714 (`experience`) | lowest 0.667 (`evidence`) | MET |
| 3. Balance | no label > 50%, none < 12% | 41.7% / 17.9% | 41.7% / 17.9% | 41.7% / 17.9% | MET |
| 4. Agreement with staff | ≥ 75% (22 of 30) | 100% | same | same | MET |
| 5. Confidence | top third ≥ 0.75 and above bottom third | 0.818 vs 0.545 | 1.000 vs 0.455 | 0.818 vs 0.545 | MET |

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.667 | 0.758 | 0.727 | 0.091 |
| Macro F1 | 0.683 | 0.770 | 0.717 | 0.086 |
| F1 — `evidence` | 0.727 | 0.923 | 0.667 | 0.256 |
| F1 — `opinion` | 0.615 | 0.714 | 0.783 | 0.167 |
| F1 — `experience` | 0.857 | 0.714 | 0.706 | 0.151 |
| F1 — `question` | 0.533 | 0.727 | 0.714 | 0.194 |

Seed 42, after (10 epochs):

| true \ predicted | evidence | opinion | experience | question |
|---|---|---|---|---|
| **evidence** | 4 | 1 | 1 | 0 |
| **opinion** | 1 | 8 | 0 | 5 |
| **experience** | 0 | 1 | 6 | 0 |
| **question** | 0 | 2 | 0 | 4 |

**Did it help, and how do I know:** Yes, a lot. Mean accuracy across the three seeds went from 0.434 to 0.717, and mean macro F1 from 0.187 to 0.723. The model now predicts all four labels on every seed, and every criterion passes. How I know it is real and not the split: the three seeds use the same splits as the before run, and all three improved on accuracy by at least 0.21.

Caveats:
- The measure is 33 test posts, so accuracy has a spread of 0.091 across seeds, and the notebook's own note says a criterion landing within that band "can't really be called either way". Criterion 1 passes by 0.067 on its worst seed, which is inside that band.
- I tried one number (10 epochs) and judged it on the same test splits I report on. I did not use the validation split to choose it, and I don't know if 10 is better than 6 or 20.
- Criterion 2 passes on seed 42 only by 0.083 on `question`, which had 6 test posts.

---

## What's Still Broken

No criterion is missed in the after run, but several pass narrowly, so I would not claim the model is reliable.

- **Criterion 2 (every label F1 ≥ 0.45), `question`.** It passed with the weakest margin (0.533 on seed 42). On that seed 5 of 14 real `opinion` posts were called `question`, while only 2 `question` posts went the other way. I defined rhetorical questions as `opinion`, and 6 of the 14 `opinion` posts in the test split contain a question mark. That is consistent with the model using the question mark as a shortcut, but I did not save per-post predictions, so I have not confirmed it. If I continued I would add rhetorical-question `opinion` examples and re-check. I stopped because the assignment limits me to one change.
- **Criterion 1 (accuracy ≥ 0.60).** Passes, but inside the seed-to-seed spread, so I'd call it met on the evidence I have and not safe.
- **Evidence F1 spread of 0.256 across seeds** (0.667 to 0.923) with 6 test posts: this label is too thin to measure with a 33-post test split.

**The gap between what I meant my labels to capture and what the model learned:**
I meant `question` to be genuine requests for information, so that rhetorical questions would belong to `opinion`; the matrix shows the model sends 5 of 14 `opinion` posts to `question`, so it learned more of "has a question mark" than of "asks for something". In the 3-epoch run it did not learn any of the four distinctions at all, and 7 of 7 `experience` posts were called `opinion`.

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
