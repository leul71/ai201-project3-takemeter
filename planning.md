# TakeMeter — Planning Document

## Community
r/Streetwear and r/ChromeHearts (with r/chromeheartsLC for authentication examples).
Chrome Hearts sits at the intersection of luxury fashion and streetwear — discourse ranges
from pure hype to detailed craft analysis, making it a strong fit for a classification task.
The community has strong opinions about authenticity, value, and brand direction, which
creates natural, meaningful label boundaries.

## Label Taxonomy

**hype** — Emotional excitement about a piece with no supporting reasoning. Flex posts,
cop announcements, "this goes crazy" reactions.
- Example 1: "Just copped the CH cemetery cross ring, I'm done 🔥🔥"
- Example 2: "This hoodie is the most fire thing I've ever seen no cap"

**analysis** — Evaluates a piece, collection, or brand direction using specific reasoning:
craftsmanship, materials, pricing, style history. Makes an argument or observation.
- Example 1: "The sterling silver weight on older CH pieces is noticeably heavier —
  pre-2010 rings have a different feel entirely"
- Example 2: "The Matty Boy collab works because it stays true to the gothic motifs
  instead of chasing hype like the recent celebrity collabs"

**authentication** — Posts asking or answering whether a piece is genuine.
Identifying fakes, spotting tells, rep vs. retail comparisons.
- Example 1: "Can anyone LC this ring? Font on the stamp looks off to me"
- Example 2: "Legit check — bought this off Grailed, seller had good feedback
  but the cross looks too symmetrical"

**critique** — Reasoned negative argument about a piece, price point, collab,
or brand direction. Must have a stance *against* something, not just neutral evaluation.
- Example 1: "CH has completely sold out — every collab is just a cash grab
  for people who want the name without understanding the brand"
- Example 2: "$1200 for a t-shirt with a cross on it is insane when the
  construction is identical to a $40 Gildan"

## Hard Edge Cases

**Critique vs. Analysis:** If the post is primarily evaluating something neutrally
(even if flaws are mentioned) → analysis. If the post has a clear negative stance
and is arguing against something → critique.
Decision rule: "Does this post have a thesis I could summarize as 'X is bad/wrong/overpriced'?"
If yes → critique. If no → analysis.

**Hype vs. Analysis:** A post can be enthusiastic AND analytical. If it gives specific
reasons (materials, history, construction) → analysis. If the enthusiasm is the whole
point with no reasoning → hype.

## Data Collection Plan
- 50 examples per label (200 total)
- hype + analysis + critique: collected from r/Streetwear and r/ChromeHearts
- authentication: collected from r/chromeheartsLC
- If a label is underrepresented after initial collection, return to the subreddit
  and filter by "top" or "controversial" to find more examples

## Evaluation Metrics
- Overall accuracy (both models)
- Per-class F1 for each label (precision and recall individually)
- Confusion matrix to identify which label pairs are hardest to distinguish
- F1 is prioritized over accuracy because the task has 4 balanced classes —
  a model predicting one class always would still hit 25% accuracy

## Definition of Success
Fine-tuned model achieves >70% overall accuracy and no single class falls
below F1 of 0.60. Must meaningfully outperform the zero-shot baseline.

## AI Tool Plan
- **Label stress-testing:** Ask Claude to generate 10 boundary posts between
  critique/analysis and hype/analysis to pressure-test definitions before annotating
- **Annotation assistance:** May use LLM to pre-label batches of 20–30 posts,
  then review and correct every label manually
- **Failure analysis:** After fine-tuning, paste misclassified examples into Claude
  and ask for pattern identification, then verify patterns by re-reading examples