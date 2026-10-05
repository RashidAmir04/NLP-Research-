# NLP-Research-
NLP research Paper on lemmatization and stemming on Indic Language
# Lemmatization and Stemming Model for Indic Languages

![Python](https://img.shields.io/badge/python-3.8%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)

An **unsupervised stemmer and lemmatizer for Hindi** (Devanagari script). It
learns from raw text alone: it splits every word into all possible
stem + suffix pairs, prunes them by frequency, clusters words under the shortest
valid stem, and derives lemmas. No annotated data or language-specific rules are
needed.

**Authors:** Rashid Amir, Gurshan Singh, Enjula Uchoi
School of Computer Science and Engineering, Lovely Professional University, Phagwara, India

---

## Table of Contents

- [Motivation](#motivation)
- [Features](#features)
- [How It Works](#how-it-works)
- [Worked Example](#worked-example)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Usage](#usage)
- [Parameters](#parameters)
- [Evaluation](#evaluation)
- [Results](#results)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [References](#references)
- [Citation](#citation)
- [License](#license)

---

## Motivation

Stemming and lemmatization reduce the inflected (and sometimes derived) forms of
a word to a common form. They are widely used in information retrieval for query
processing and in machine translation to reduce data sparsity.

Indic languages are morphologically rich. A single root appears in many variants
that express case, honorific status, part of speech, person, tense and more.
This makes downstream tasks such as machine translation, parsing and word sense
disambiguation harder. Word-form normalization is therefore an important
preprocessing step.

Stemming and lemmatization differ in an important way:

- A **stem** is the common part of a set of variants. It is not guaranteed to be
  a valid word, and words are considered in isolation.
- A **lemma** is the canonical, dictionary form of a word and must be a valid
  form in the language.

Mature stemmers and lemmatizers exist for English and other European languages,
but few are available for Hindi and other South Asian languages. Existing
approaches are either probabilistic (they need large amounts of data) or
rule-based (language-dependent, and costly to build for each language). This
project takes an unsupervised, corpus-driven approach that needs only raw text.

## Features

- Unsupervised: trains on plain UTF-8 text, with no labels or rule sets
- Produces both **stems** and **lemmas** in a single pipeline
- Splits every word into `stem + suffix`, where `@` marks an empty suffix
- Frequency-based pruning of unreliable stems and suffixes
- Stem packs and suffix packs: stems that share the same suffix set are grouped
- Pure Python standard library, no dependencies
- Command-line interface, Python API and unit tests

## How It Works

The algorithm has nine steps.

| Step | Description |
|------|-------------|
| 1 | **Preprocessing.** Read the corpus as UTF-8. Keep only Devanagari characters, which removes punctuation, digits, English letters and special characters. |
| 2 | **Unique word list.** Tokenize the text and collect the distinct words. |
| 3 | **Split every word** into all possible `stem + suffix` pairs, with a maximum suffix length of 8 and a minimum stem length of 2. `@` marks an empty suffix, meaning the stem also occurs as a word. |
| 4 | **Collect suffixes** for each stem into a suffix bag. |
| 5 | **Frequency-based stem elimination.** Drop stems that occur with no more than 2 suffixes. |
| 6 | **Frequency-based suffix elimination.** Drop suffixes that occur with no more than 2 stems. |
| 7 | **Clustering.** Among competing decompositions, choose the **shortest stem**. For lemmatization, use the decomposition with the `@` suffix, meaning the stem occurs as a word in the corpus. |
| 8 | **Stem packs and suffix packs.** Group stems that share the same suffix set. |
| 9 | **Output.** Produce `stem + suffixes` and `lemma`. |

### Extension beyond the paper

When a stem is not itself a word in the corpus (for example `लड़क`, from
`लड़का` and `लड़कों`), the `@` rule cannot select a lemma. In that case the lemma
falls back to the **most frequent word in the stem's cluster**, with ties broken
in favor of the shorter word. On very small corpora where all frequencies tie,
this choice is arbitrary. On realistic corpora, word frequencies decide it.

## Worked Example

Given a corpus containing inflected forms of "boy", "child" and "horse":

```
लड़का  लड़के  लड़कों  लड़की  लड़कियाँ  लड़कियों
बच्चा  बच्चे  बच्चों  बच्ची  बच्चियाँ  बच्चियों
घोड़ा  घोड़े  घोड़ों  घोड़ी  घोड़ियाँ  घोड़ियों
```

the model produces:

| Word | Stem | Suffix | Lemma |
|------|------|--------|-------|
| लड़कों | लड़क | ों | लड़का |
| बच्चियाँ | बच्च | ियाँ | बच्चा |
| घोड़े | घोड़ | े | घोड़ा |
| लड़का | लड़क | ा | लड़का |

## Project Structure

```
indic-lemmatization-stemming/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── src/
│   ├── stemmer.py          # IndicStemmer class and command-line interface
│   └── demo.py             # minimal example
├── data/
│   ├── sample_corpus.txt   # small illustrative Hindi corpus
│   └── gold_lemmas.tsv     # small illustrative gold set (word<TAB>lemma)
├── tests/
│   └── test_stemmer.py     # unit tests
├── results/                # generated outputs and figures
└── paper/                  # the research paper
```

## Installation

Requires **Python 3.8 or newer**. There are no third-party dependencies.

```bash
git clone https://github.com/rashidamir04/indic-lemmatization-stemming.git
cd indic-lemmatization-stemming
```

## Usage

### Quick demo

```bash
python src/demo.py
```

### Command line

```bash
python src/stemmer.py data/sample_corpus.txt --t-suffix 1 --gold data/gold_lemmas.tsv --out results/output.tsv
```

This writes a tab-separated file with the columns `word`, `stem`, `suffix` and
`lemma`. If `--gold` is given, it also prints the lemma accuracy.

### Python API

```python
import sys
sys.path.insert(0, "src")
from stemmer import IndicStemmer

with open("data/sample_corpus.txt", encoding="utf-8") as f:
    model = IndicStemmer(min_stems_per_suffix=1).fit(f.read())

print(model.stem("लड़कों"))       # लड़क
print(model.lemmatize("लड़कों"))  # लड़का
print(model.analyze("लड़कों"))
# {'word': 'लड़कों', 'stem': 'लड़क', 'suffix': 'ों', 'lemma': 'लड़का'}
```

### Run the tests

```bash
python -m unittest discover -s tests -v
```

## Parameters

| CLI option | Python argument | Default | Meaning |
|------------|-----------------|---------|---------|
| `--max-suffix` | `max_suffix_len` | 8 | Maximum suffix length considered when splitting |
| `--min-stem` | `min_stem_len` | 2 | Minimum stem length |
| `--t-stem` | `min_suffixes_per_stem` | 2 | Drop stems seen with at most this many suffixes |
| `--t-suffix` | `min_stems_per_suffix` | 2 | Drop suffixes seen with at most this many stems |

The default thresholds are intended for large corpora. On small files they can
remove too much, so lower them (for example `--t-suffix 1`).

## Evaluation

To measure lemma accuracy, provide a gold file with one `word<TAB>lemma` pair per
line:

```
लड़कों	लड़का
बच्चे	बच्चा
```

Then run:

```bash
python src/stemmer.py your_corpus.txt --gold your_gold.tsv
```

Lemma accuracy is the fraction of gold words for which `lemmatize(word)` equals
the gold lemma.

> **Note:** the files in `data/` are tiny and illustrative. They exist to show
> the file formats and to support the unit tests. They are not a benchmark, and
> accuracy on them says nothing about performance on real text.

## Results

> Add the results from the paper here: the confusion matrix, model accuracy,
> accuracy distribution, and the comparison table. Save the figures in
> `results/` and reference them like this:
>
> ```markdown
> ![Confusion matrix](results/confusion_matrix.png)
> ![Model accuracy](results/model_accuracy.png)
> ![Accuracy distribution](results/accuracy_distribution.png)
> ```

## Limitations

- Choosing the **shortest** stem can over-stem. The two frequency filters
  (steps 5 and 6) reduce this but do not eliminate it.
- The preprocessing step keeps Devanagari characters only. Other Indic scripts
  need the token regex in `src/stemmer.py` adapted.
- The default thresholds assume a large corpus. Small corpora need lower
  thresholds, and lemma selection can then be arbitrary.
- The method is purely orthographic. It does not use part-of-speech information
  or context, so it cannot disambiguate surface forms that come from different
  lemmas.

## Future Work

- Extend the method to other Indic languages and scripts that share the same
  script family
- Tune the parameters that control accuracy (suffix length, thresholds)
- Evaluate on larger corpora and against existing Hindi stemmers
- Investigate scalability on large corpora

## References

1. Goldsmith, J. (2001). Unsupervised learning of the morphology of a natural language. *Computational Linguistics*, 27(2), 153-198.
2. Pandey, A. K., and Siddiqui, T. J. (2008). An unsupervised Hindi stemmer with heuristic improvements. *Proceedings of the Second Workshop on Analytics for Noisy Unstructured Text Data*.
3. Ramanathan, A., and Rao, D. D. (2003). A lightweight stemmer for Hindi. *Workshop on Computational Linguistics for South-Asian Languages, EACL*.
4. Manning, C. D., Raghavan, P., and Schütze, H. (2008). *Introduction to Information Retrieval*. Cambridge University Press.
5. Banerjee, S., and Pedersen, T. (2002). An adapted Lesk algorithm for word sense disambiguation using WordNet. *CICLing*.
6. Koskenniemi, K. (1984). A general computational model for word-form recognition and production. *COLING*.
7. Plisson, J., Lavrac, N., and Mladenic, D. (2004). A rule based approach to word lemmatization. *Proceedings of IS*.
8. Chrupała, G., Dinu, G., and van Genabith, J. (2008). Learning morphology with Morfette.
9. Poon, H., Cherry, C., and Toutanova, K. (2009). Unsupervised morphological segmentation with log-linear models. *NAACL-HLT*.
10. Gesmundo, A., and Samardzic, T. (2012). Lemmatisation as a tagging task. *ACL*.
11. Nicolai, G., and Kondrak, G. (2016). Leveraging inflection tables for stemming and lemmatization. *ACL*.
12. Paik, J. H., et al. (2013). Effective and robust query-based stemming. *ACM Transactions on Information Systems*, 31(4).

## Citation

If you use this work, please cite it:

```bibtex
@misc{amir2026indic,
  title  = {Lemmatization and Stemming Model for Indic Languages},
  author = {Amir, Rashid and Singh, Gurshan and Uchoi, Enjula},
  year   = {2026},
  note   = {School of Computer Science and Engineering, Lovely Professional University},
  url    = {https://github.com/rashidamir04/indic-lemmatization-stemming}
}
```

## License

Released under the [MIT License](LICENSE).

## Contact

Rashid Amir: [GitHub](https://github.com/rashidamir04) · [LinkedIn](https://linkedin.com/in/rashid-amir-a191b5253)
