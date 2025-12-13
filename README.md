# l10n-expansion-data

![License: CC0-1.0](https://img.shields.io/badge/License-CC0%201.0-lightgrey.svg)
![Data Source: OPUS-100](https://img.shields.io/badge/Data%20Source-OPUS--100-blue)
![Format: JSON/CSV/YAML](https://img.shields.io/badge/Format-JSON%20%7C%20CSV%20%7C%20YAML-orange)

**Data-driven text expansion ratios for robust software localization and UI planning.**

Stop guessing how much space to leave for German or how much Vietnamese text will expand. This repository provides highly granular statistical expansion ratios derived from massive parallel corpora (currently **OPUS-100**), helping developers and designers prevent UI overflows before translation begins.

All data is generated using the [Expansion Rate Generator (ERG)](https://github.com/TheSkyC/expansion-ratio-generator) tool.

## 📊 Why Use This Data?

When translating software, text length changes unpredictably.
*   **English → Chinese:** Text usually shrinks (~0.6x).
*   **English → German:** Text often expands (~1.3x).
*   **Short Strings:** UI labels (e.g., "OK", "New") behave very differently from long paragraphs.

This dataset offers:
1.  **Granularity:** Ratios are grouped by source string length (0-20 chars, 20-50 chars, etc.).
2.  **Reliability:** Based on millions of sentence pairs, not just heuristics.
3.  **Flexibility:** Provides multiple statistical metrics (mean, median, percentiles) to fit different risk tolerances.

## 📂 Directory Structure

The data is organized by **Source Corpus** -> **Export Strategy** -> **File**.

```text
data/
└── opus-100/                  # Derived from the Helsinki-NLP OPUS-100 corpus
    ├── detailed/              # Bucketed data with full statistics
    │   ├── opus-100_detailed_mean.json
    │   ├── opus-100_detailed_med.json
    │   └── ... (and CSV/YAML variants)
    │
    └── simple/                # Single global value per language pair
        ├── opus-100_simple_wgt_mean.json
        ├── opus-100_simple_wgt_p75.json
        └── ... (and CSV/YAML variants)
```

## 📝 Data Formats & Strategies

We provide two main export strategies: **Simple** and **Detailed**.

### 1. Simple Strategy (`/simple`)

Provides a **single value** per language pair. Ideal for quick estimates or runtime checks.

*   **File Naming:** `corpus_simple_<strategy>_<metric>.json`
    *   `<strategy>`: `wgt` (weighted average), `max` (worst-case), `bkt0-20` (specific bucket).
    *   `<metric>`: `mean`, `med` (median), `p25`, `p75`, `rng25-75` (range).

**Example (`opus-100_simple_bkt0-20_mean.json`):**
```json
{
  "en-sh": 1.1321,
  "en-se": 1.9319,
  "en-ro": 1.1275,
  "en-zh": 0.5633
}
```

**Example (`opus-100_simple_wgt_rng25-75.json`):**
```json
{
  "en-sh": "0.88-1.09",
  "en-se": "0.97-1.98",
  "en-ro": "0.83-1.15",
  "en-zh": "0.26-0.40"
}
```

### 2. Detailed Strategy (`/detailed`)

Provides **bucketed data** with rich statistics. Ideal for dynamic UI layout engines or in-depth analysis.

*   **File Naming:** `corpus_detailed_<metric>.json`
    *   `<metric>`: The primary metric used for the `val` key (e.g., `mean`, `med`).

**Example (`opus-100_detailed_mean.json`):**
```json
{
  "en-zh": {
    "0.0-20.0": {
      "val": 0,
      "count": 176354,
      "std": 0.5426569950687576,
      "min": 0.1,
      "max": 10.0,
      "mean": 0.563295776173526,
      "median": 0.42857142857142855,
      "p25": 0.3333333333333333,
      "p75": 0.6
    },
    "20.0-50.0": {
      "val": 0,
      "count": 286113,
      "std": 0.2895320290537651,
      "min": 0.1,
      "max": 9.954545454545455,
      "mean": 0.38877682122354107,
      "median": 0.32558139534883723,
      "p25": 0.25925925925925924,
      "p75": 0.41379310344827586
    }
  }
}
```

## 🤔 Which File Should I Use?

With so many files, here’s a quick guide:

*   **For general UI development (buttons, labels):**
    *   Use `simple/opus-100_simple_bkt0-20.json`. This gives you a based on short strings (0-20 characters), which is the most common scenario for UI text.

*   **For a single, balanced ratio for your entire app:**
    *   Use `simple/opus-100_simple_wgt_mean.json`. This provides a weighted average across all text lengths.

*   **For dynamic layout engines that adapt to text length:**
    *   Use `detailed/opus-100_detailed_mean.json`. This allows you to look up the appropriate ratio based on the source text's character count.

*   **For data analysis or academic research:**
    *   Use the `.csv` files in the `detailed/` directory, which can be easily imported into Excel or Pandas.

## 🚀 Usage Example

### Python

```python
import json

# For simple, quick checks, use the 'simple' weighted mean data.
with open('data/opus-100/simple/opus-100_simple_wgt_mean.json', 'r', encoding='utf-8') as f:
    ratios = json.load(f)

def get_estimated_length(text, target_lang_code):
    """Get a simple estimated length."""
    ratio = ratios.get(f"en-{target_lang_code}", 1.0) # Fallback to 1.0
    return len(text) * ratio

print(f"Estimated length for 'Save File' in German: {get_estimated_length('Save File', 'de'):.2f}")
```

## 🔬 Methodology

1.  **Source:** We utilize the `train` split from the [OPUS-100](https://huggingface.co/datasets/Helsinki-NLP/opus-100) corpus, which contains 1 million sentence pairs per language.
2.  **Filtering:**
    *   Pairs with extreme expansion ratios (<0.1 or >10.0) are discarded as likely alignment errors.
    *   Source strings with zero length are excluded.
3.  **Calculation:**
    *   `Ratio = CharacterLength(Target) / CharacterLength(Source)`
    *   All calculations are based on character counts.

## ⚖️ License & Attribution

### Dataset License
The statistical data in this repository is released under the **CC0 1.0 Universal (Public Domain Dedication)**. You are free to use, modify, and distribute it in any commercial or open-source software without restriction.

### Disclaimer
This repository contains **statistical metadata only**. It does **not** contain any original sentence pairs or text content from the source corpora.

### Source Attribution
This data is derived from the **OPUS-100** corpus. If you use this data in academic work, please cite the original paper:

> **OPUS-100**  
> *Zhang, Biao et al. "Improving Massively Multilingual Neural Machine Translation in Zero-Shot Scenarios." ACL (2020).*

```bibtex
@inproceedings{zhang-etal-2020-improving,
    title = "Improving Massively Multilingual Neural Machine Translation and Zero-Shot Translation",
    author = "Zhang, Biao  and
      Williams, Philip  and
      Titov, Ivan  and
      Sennrich, Rico",
    editor = "Jurafsky, Dan  and
      Chai, Joyce  and
      Schluter, Natalie  and
      Tetreault, Joel",
    booktitle = "Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics",
    month = jul,
    year = "2020",
    address = "Online",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2020.acl-main.148",
    doi = "10.18653/v1/2020.acl-main.148",
    pages = "1628--1639",
}
```
