# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1.

Overall accuracy on the held-out test split is at least 0.60, and holds on every one of the three seeds.

**Why this target:**

I have four labels, so a coin flip gets about 25%, and always guessing `opinion` (my biggest label, about 42% of the file) gets about 42%. 0.60 sits well above both, which is the minimum I'd call better than a lazy guesser. I did not go higher because the test split is only about 33 posts, so one seed can swing by 3-4 points on a couple of posts.

---

## 2.

Every one of the four labels has an F1 of at least 0.45 on the test split on all three seeds, and `question` and `evidence` are the ones I expect to be closest to the line.

**Why this target:**

`question` and `evidence` are my thinnest labels (roughly 6 test posts each), so their F1 will be jumpy and I set the floor looser than the overall accuracy. 0.45 is still well above chance for a four-way split. `question` has surface cues (a question mark, "how", "why"), but my definition excludes rhetorical questions, which will pull it down. `evidence` is hard because it only differs from `experience` by whether a stranger could check the claim.

---

## 3.

No label makes up more than 50% of `labels.csv`, and no label is under 12%.

**Why this target:**

My file is 218 posts and `opinion` is the biggest at about 42%. The assignment cap is 70%, but at 70% a model that always says `opinion` already looks decent, so I want a tighter cap. The 12% floor keeps every label to at least 8 or so test posts across a bigger sample, which is the point where I said F1 stops being meaningful.

---

## 4.

When I label the 30 staff posts under the staff's written definitions, without seeing the staff labels, I agree with the staff on at least 75% of them (22 or more of 30), and I report raw agreement before kappa.

**Why this target:**

The staff set uses someone else's three labels, so this measures whether I can apply a rule I did not write, which is the skill my own 218 labels depended on. The set is only 30 posts, so a single post moves agreement by 3 points. 75% means no more than 7 disagreements, which is strict enough that I can't pass by being sloppy with the boundary between `analysis` and `hot_take`, and loose enough that one or two arguable posts won't sink it.

---

## 5.

On the test split, accuracy on the most-confident third of predictions is at least 0.75, and it is higher than accuracy on the least-confident third.

**Why this target:**

A confident prediction should earn the trust it asks for. The test split is about 33 posts, so a third is only 11 predictions, and one wrong answer moves that score by 9 points. 0.75 allows two or three misses among the most-confident predictions. If the most-confident third is no more accurate than the least-confident third, the model's confidence tells nobody anything and shouldn't be shown to a user.

---


<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
