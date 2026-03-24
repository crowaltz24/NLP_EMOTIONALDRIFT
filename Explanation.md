# Project Writeup: Tracking Emotional Drift in Text

## 1. Introduction

This project focuses on analyzing the evolution of emotions within a narrative text. The core idea is to move beyond a static, overall sentiment score and instead track how emotional states change over time, sentence by sentence. We call this process "emotional drift." By modeling a text as a sequence of emotional states, we can identify key moments of transition, understand the emotional arc of a story, and gain deeper insights into the narrative's structure.

Here we outline the conceptual framework and the pipeline we developed to achieve this.

## 2. Core Concepts

To build our analytical pipeline, we rely on several fundamental concepts from sentiment analysis and affective computing.

- **Sentiment:** The overall attitude or emotional tone of a piece of text. It is often measured on a spectrum from positive to negative. In our project, sentiment serves as the primary indicator of the overall emotional direction.

- **Polarity:** This is the quantitative measure of sentiment, typically represented as a numerical value. A polarity score close to **+1** indicates a very positive sentiment, while a score near **-1** indicates a very negative sentiment. A score of **0** suggests a neutral tone.

- **Subjectivity:** This metric measures the degree to which text expresses personal opinions, feelings, or beliefs rather than objective facts. A highly subjective sentence (score close to 1) is likely to be emotional or opinionated, while an objective sentence (score close to 0) is more factual. Tracking subjectivity alongside polarity helps us distinguish between objective narration and subjective emotional expression.

- **Emotional Affect:** This refers to the experience of feeling or emotion. We operationalize this using a set of eight basic emotion categories, as defined by the NRC (National Research Council Canada) lexicon:
    - **Anger**
    - **Anticipation**
    - **Disgust**
    - **Fear**
    - **Joy**
    - **Sadness**
    - **Surprise**
    - **Trust**

By analyzing the presence of these emotions in the text, we can build a multi-dimensional profile of the narrative's emotional state at any given point.

## 3. The Analytical Pipeline

Our pipeline processes a text in a series of steps to transform raw sentences into a timeline of emotional data.

### 3.1. Preprocessing and Segmentation

The first step is to clean the raw text to remove noise (like excessive whitespace or unusual characters) and then segment it into individual sentences. We use `nltk` for basic cleanup and `spaCy` for accurate sentence boundary detection. Each sentence becomes a discrete "time step" in our analysis.

### 3.2. Sentiment and Subjectivity Scoring

For each sentence, we use `TextBlob` to generate its polarity and subjectivity scores.

`TextBlob` uses a lexicon-based approach where each adjective in the Pattern library has pre-assigned base values. The final score is computed as:

```
Polarity = Σ (p_i × m_i × n_i) / k
Subjectivity = Σ (s_i × m_i × n_i) / k
```

Where:
- `p_i` = base polarity of the i-th adjective (range -1.0 to 1.0)
- `s_i` = base subjectivity of the i-th adjective (range 0.0 to 1.0)
- `m_i` = modifier multiplier (intensifiers like "very" = 1.5, diminishers like "slightly" = 0.5)
- `n_i` = negation multiplier (-0.5 if preceded by a negation word like "not", otherwise 1.0)
- `k` = number of scored adjectives in the sentence

This provides us with two time series: one for sentiment polarity and one for subjectivity.

### 3.3. Emotion Category Tagging

To get a more granular view of the emotional state, we use `NRCLex`. This tool is built upon the NRC Emotion Lexicon, a dictionary of over 27,000 words, each associated with one or more of the eight core emotions.

For each sentence, `NRCLex` counts the occurrences of words that map to each emotion category. We then normalize these counts by the total number of words in the sentence. This gives us a normalized score for each of the eight emotions per sentence, creating eight separate time series.

### 3.4. Smoothing and Shift Detection

Raw scores from individual sentences can be noisy, with minor fluctuations that obscure the overall trend. To address this, we apply a **smoothing** function. We use a **simple sliding window average**, where the score for a given sentence becomes the average of its own score and the scores of the previous `N` sentences (e.g., a window of 3). This helps to reveal the underlying emotional trajectory.

Finally, we aim to automatically detect significant **emotional shifts**. We do this by applying a **thresholding** method to the smoothed sentiment data. If the absolute difference between the sentiment scores of two consecutive sentences exceeds a predefined threshold (eg., 0.5), we flag that point as a potential emotional turning point in the narrative.

## 4. Visualization and Output

The final stage of the pipeline is visualization. By plotting the resulting time series, we can intuitively observe the emotional drift.

- **Sentiment Timeline:** A line graph showing the smoothed polarity score over the sequence of sentences. Red dashed lines mark detected emotional shifts.
- **Subjectivity Timeline:** A line graph showing how the level of opinionated or personal language evolves.
- **Emotion Categories Timeline:** A multi-line graph showing the normalized scores for all eight emotion categories over time, allowing for a comparison of how different emotions rise and fall.

These visualizations provide a powerful, at-a-glance summary of the text's emotional journey, highlighting periods of stability, moments of crisis, and eventual resolutions.
