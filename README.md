# ai201-project3-takemeter
# TakeMeter — Chrome Hearts Discourse Classifier

## Community
r/ChromeHearts, r/Streetwear, and r/chromeheartsLC. Chrome Hearts sits at the 
intersection of luxury fashion and streetwear, producing varied discourse ranging 
from emotional hype posts to detailed craft analysis to authentication requests. 
The community's strong opinions about authenticity, value, and brand direction 
create natural, meaningful label boundaries ideal for a classification task.

## Label Taxonomy

**authentication** — Posts asking or answering whether a Chrome Hearts piece is 
real or fake. Identifying fakes, spotting tells, rep vs. retail comparisons.
- Example 1: "Can anyone LC this ring? The stamp font looks off to me."
- Example 2: "Spent $10,000 on a rep belt from The Real Real. My sales associate 
  is arguing it's authentic."

**analysis** — Evaluates a piece, collection, or brand direction using specific 
reasoning: craftsmanship, materials, pricing, style history. Makes an argument.
- Example 1: "The sterling silver weight on older CH pieces is noticeably heavier 
  — pre-2010 rings have a different feel entirely."
- Example 2: "The lack of an online store is a deliberate brand decision that has 
  held for 30 years. It creates scarcity and forces boutique visits."

**hype** — Emotional excitement about a piece with no supporting reasoning. 
Flex posts, cop announcements, pure reaction.
- Example 1: "Just copped the CH cemetery cross ring and I cannot stop staring 
  at it. This is the most beautiful piece of jewelry I've ever owned."
- Example 2: "Walked into the CH boutique in LA and walked out $4000 lighter. 
  Worth every single penny."

**critique** — Reasoned negative argument about a piece, price point, collab, 
or brand direction. Must have a clear negative stance.
- Example 1: "Paying $1500 for a Chrome Hearts t-shirt is indefensible. You're 
  paying almost entirely for the name at that point."
- Example 2: "Chrome Hearts lost something when it became a celebrity flex. Now 
  it's mostly a status signal for people who've never been inside a boutique."

## Data Collection
- **Sources:** r/chromeheartsLC (authentication examples), r/ChromeHearts and 
  r/Streetwear (all other labels)
- **Process:** Real posts collected manually, supplemented with synthetic examples 
  generated using Claude to reach target counts per label. All synthetic examples 
  were reviewed before inclusion.
- **Label distribution:**
  - hype: 50
  - authentication: 41
  - critique: 36
  - analysis: 25
  - Total: 152 examples

### Difficult-to-label examples
1. *"90s chrome custom order. Was wondering what value to put on this."* — Could 
   be analysis (pricing discussion) or authentication (verifying value). Labeled 
   **analysis** because it's asking about value, not authenticity.
2. *"CH pricing is actually more defensible than people think when you break it 
   down..."* — Could be critique or analysis. Labeled **analysis** because it 
   defends the price rather than arguing against it.
3. *"The boutique-only model is a double edged sword..."* — Could be analysis or 
   critique. Labeled **critique** because it ultimately argues the model is 
   exclusionary.

## Fine-Tuning Approach
- **Base model:** distilbert-base-uncased
- **Training setup:** 3 epochs, learning rate 2e-5, batch size 16, T4 GPU
- **Dataset split:** 70% train (106), 15% validation (23), 15% test (23)
- **Key hyperparameter decision:** Kept epochs at 3 to avoid overfitting on a 
  small dataset of 152 examples. Increasing to 5 epochs risked memorizing 
  training examples rather than learning generalizable patterns.

## Baseline
- **Model:** Groq llama-3.3-70b-versatile, zero-shot
- **Prompt:** Provided label definitions and one example per label, instructed 
  to output only the label name.

## Evaluation Report

### Results Comparison
| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq llama-3.3-70b) | 1.000 |
| Fine-tuned DistilBERT | 0.522 |

### Per-Class Metrics — Fine-Tuned Model
| Label | Precision | Recall | F1 |
|---|---|---|---|
| authentication | 0.00 | 0.00 | 0.00 |
| analysis | 0.00 | 0.00 | 0.00 |
| hype | 0.53 | 1.00 | 0.70 |
| critique | 0.50 | 0.80 | 0.62 |

### Per-Class Metrics — Baseline
| Label | Precision | Recall | F1 |
|---|---|---|---|
| authentication | 1.00 | 1.00 | 1.00 |
| analysis | 1.00 | 1.00 | 1.00 |
| hype | 1.00 | 1.00 | 1.00 |
| critique | 1.00 | 1.00 | 1.00 |

### Confusion Matrix — Fine-Tuned Model
| | Pred: authentication | Pred: analysis | Pred: hype | Pred: critique |
|---|---|---|---|---|
| **True: authentication** | 0 | 0 | 4 | 2 |
| **True: analysis** | 0 | 0 | 3 | 1 |
| **True: hype** | 0 | 0 | 8 | 0 |
| **True: critique** | 0 | 0 | 1 | 4 |

### Wrong Predictions Analysis

**#1 — "Legit check on Coxucker glasses listed on Vinted for 1100 euros, are 
they authentic?"**
- True: authentication | Predicted: hype | Confidence: 0.28
- The post is short and lacks the detailed LC language the model saw in most 
  authentication training examples. The model defaulted to hype, the most 
  common label in the dataset.

**#2 — "The CH store experience used to feel special. Now it feels like a velvet 
rope situation designed to make you feel lucky to spend $2000 on a t-shirt. 
That's manipulation not luxury."**
- True: critique | Predicted: hype | Confidence: 0.28
- The model likely picked up on the price mention and brand enthusiasm framing 
  rather than the negative stance. Critique and hype share vocabulary around 
  pricing and brand experience.

**#3 — "CH pricing is actually more defensible than people think when you break 
it down. The silver alone on a large cross ring is maybe $40 in material..."**
- True: analysis | Predicted: critique | Confidence: 0.27
- The word "defensible" likely triggered a critique prediction. The model hasn't 
  learned that defending a price is analysis, not critique. This is exactly the 
  boundary case identified in planning.md.

### Sample Classifications
| Text | Predicted Label | Confidence | Notes |
|---|---|---|---|
| "Just copped the CH cemetery cross ring... most beautiful piece I've ever owned" | hype | 0.89 | Correct — pure emotional reaction, no reasoning |
| "Legit check on Coxucker glasses listed on Vinted for 1100 euros" | hype | 0.28 | Wrong — should be authentication |
| "Chrome Hearts denim at $5000+ retail is a hard sell when Japanese selvedge brands produce better construction" | critique | 0.71 | Correct — clear negative argument with reasoning |

## Reflection: What the Model Learned vs. What I Intended

The model learned to predict hype for almost anything and critique for posts 
mentioning prices negatively. It completely failed to learn authentication and 
analysis as distinct categories. This happened because: (1) hype was the most 
common label at 50 examples, so the model defaulted to it under uncertainty; 
(2) authentication and analysis examples shared vocabulary with hype and critique 
respectively; (3) 152 examples is too small for a 4-label classifier to learn 
subtle distinctions. What I intended was a model that detected discourse structure 
— argument vs. reaction vs. verification. What the model learned was surface 
vocabulary patterns, defaulting to the majority class when uncertain.

## Spec Reflection
The spec helped most in the label design phase — the requirement to identify a 
hard edge case before annotating forced me to write the critique vs. analysis 
decision rule early, which I referenced constantly during annotation. My 
implementation diverged from the spec in dataset size — I reached 152 examples 
instead of 200 due to time constraints. This likely contributed directly to the 
model's poor performance on minority classes like analysis (25 examples).

## AI Usage
1. **Dataset generation:** Claude was used to generate synthetic examples for 
   underrepresented labels (analysis, hype, critique) after real post collection. 
   All generated examples were reviewed before inclusion. The prompt included 
   label definitions and requested realistic community-style posts.
2. **Error pattern analysis:** Wrong predictions were reviewed to identify the 
   pattern that the model collapsed authentication and analysis predictions into 
   hype — a systematic majority-class bias consistent with class imbalance in 
   the training data.