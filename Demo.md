
# Example Input

```
I absolutely love the sleek design and the vibrant screen of this new smartphone; it feels truly premium in my hand. However, the battery life is incredibly disappointing and barely lasts half a day. It’s a shame because the camera quality is stunning, but I can't recommend it to anyone who travels often.
```

This text has **mixed sentiment** — positive + negative — which makes it a good test case.

---

# 1) Cleaning Step (`clean_text`)

### Original text

```
I absolutely love the sleek design and the vibrant screen ...
```

### After cleaning

Your regex does:

```py
text = re.sub(r'\s+', ' ', text)
text = re.sub(r'[^a-zA-Z0-9.,!? ]', '', text)
```

### What changes:

* `;` removed
* apostrophes removed (`It’s` → `Its`, `can't` → `cant`)
* extra spaces normalized

### Result (approx)

```
I absolutely love the sleek design and the vibrant screen of this new smartphone it feels truly premium in my hand. However, the battery life is incredibly disappointing and barely lasts half a day. Its a shame because the camera quality is stunning, but I cant recommend it to anyone who travels often.
```

**Important:** meaning still survives, but punctuation nuance is lost.

---

# 2) Sentence Segmentation (`segment_sentences`)

spaCy splits into sentences.

### Output list:

```py
[
 "I absolutely love the sleek design and the vibrant screen of this new smartphone it feels truly premium in my hand.",
 "However, the battery life is incredibly disappointing and barely lasts half a day.",
 "Its a shame because the camera quality is stunning, but I cant recommend it to anyone who travels often."
]
```

Now the pipeline treats each sentence as a separate emotional moment.

---

# 3) Sentiment Polarity (TextBlob)

TextBlob analyzes each sentence.

Approximate values (illustrative):

| Sentence                              | Meaning         | Polarity  |
| ------------------------------------- | --------------- | --------- |
| Design & screen praise                | Strong positive | **+0.75** |
| Battery complaint                     | Strong negative | **−0.65** |
| Mixed (camera good but not recommend) | Slight negative | **−0.25** |

So:

```py
scores = [0.75, -0.65, -0.25]
```

### What just happened?

The emotional tone **drops sharply** after sentence 1.

---

# 4) Subjectivity Scores

Subjectivity = opinion intensity.

Approx values:

| Sentence                 | Subjectivity           |
| ------------------------ | ---------------------- |
| Design praise            | 0.9 (very opinionated) |
| Battery complaint        | 0.95                   |
| Recommendation statement | 0.85                   |

```py
subjectivity_scores = [0.9, 0.95, 0.85]
```

This tells you:

> The whole paragraph is opinion-heavy (review style).

---

# 5) Sliding Window Smoothing

Window size = 3.

### Raw scores:

```
[0.75, -0.65, -0.25]
```

### Calculation:

1. i=0 → [0.75] → 0.75
2. i=1 → [0.75, -0.65] → 0.05
3. i=2 → [0.75, -0.65, -0.25] → -0.05

### Smoothed result:

```py
smoothed = [0.75, 0.05, -0.05]
```

### Interpretation

Emotion trend:

```
Strong Positive → Neutral → Slight Negative
```

This smoothing prevents one sentence from dominating the graph.

---

# 6) Emotional Shift Detection

Threshold = 0.5.

Check differences:

| Transition   | Difference | Shift? |
| ------------ | ---------- | ------ |
| 0.75 → 0.05  | 0.70       | YES    |
| 0.05 → -0.05 | 0.10       | No     |

Output:

```py
shifts = [1]
```

Meaning:

> Emotional jump occurs at sentence index 1 (battery complaint).

Exactly what humans feel while reading.

---

# 7) Emotion Categories (NRCLex)

Let’s approximate NRC outputs.

---

### Sentence 1 (design praise)

Keywords: love, premium, vibrant

Likely emotions:

```
joy        → high
trust      → medium
anticipation → small
```

---

### Sentence 2 (battery complaint)

Keywords: disappointing, barely lasts

Likely emotions:

```
sadness → high
anger   → medium
disgust → small
```

---

### Sentence 3 (camera good but can't recommend)

Mixed emotions:

```
joy      → from "stunning"
sadness  → from "shame"
trust ↓  → recommendation rejected
```

---

### After normalization

Values become small fractions like:

```py
{
 "joy": [0.12, 0.0, 0.05],
 "sadness": [0.0, 0.10, 0.08],
 "trust": [0.07, 0.0, 0.02],
 ...
}
```

---

# 8) Final Graph Interpretation (What You’d See)

### Sentiment timeline

```
Sentence 1  ▲ positive peak
Sentence 2  ▼ sharp drop (red shift line)
Sentence 3  → remains slightly negative
```

This visually shows:

> The review starts excited but ends with disappointment.

---

# 9) Real Insight (Why this example is GOOD)

Your paragraph demonstrates:

* mixed sentiment (realistic review)
* emotional reversal
* contrast words:

  * “However”
  * “It’s a shame”
  * “but”

These trigger polarity changes strongly.

---

# 10) Brutally Honest Analysis (Mentor mode)

Your pipeline handles this case **decently**, but:

### What works

* shift detection catches the battery complaint
* smoothing captures trend direction
* NRC categories show joy → sadness transition

### What breaks

1. Cleaning destroys contraction meaning:

   * `can't` → `cant` (bad NLP signal loss)

2. TextBlob oversimplifies:

   * “camera is stunning but can’t recommend” should be **strong negative**, but may become weakly negative.

3. NRCLex ignores context:

   * “stunning” adds joy even though overall sentence is negative.

This is lexicon limitation — not your fault.

---


