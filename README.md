# PROJECT: Emotional Drift Analyzer in Long Text

This program analyzes how emotions change over time in long text like movie scripts, diaries, or stories. 
A detailed conceptual explanation is provided in `Explanation.md`.

For CSA4028: Natural Language Processing.

### Group Members

| REGISTRATION NO.    | NAME              |
|---------------------|-------------------|
| 23BAI10001          | ABHAY TYAGI       |
| 23BAI10082          | RISHAB TOMAR      |
| 23BAI10424          | SHANSIT SUMAN     |
| 23BAI10620          | SHREYA GOYAL      |
| 23BAI10847          | ASHISH VISHWAKARMA|
---

## Demo
Demo notebook is `emotional_drift_demo.ipnyb` and walks through the process.

## Setup

1. create venv
   
   ```
   py -m venv venv
   ```
   
2. activate venv
   
   ```
   venv/scripts/activate
   ```

3. install reqs
   
   ```
   pip install -r requirements.txt
   ```
   if spaCy english model not installed:
   ```
   python -m spacy download en_core_web_sm
   ```

4. run `emotional_drift.py`
