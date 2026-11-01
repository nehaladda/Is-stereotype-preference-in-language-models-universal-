# Is stereotype preference in language models universal, or does it depend on the cultural context of the test items?

This project compares how masked language models score more biased and less biased sentences on two benchmarks. One benchmark has US-centric content (CrowS-Pairs) and the other has Indian-context content (IndiBias).

**Author:** Neha Ladda

## Summary

- Both models, BERT and XLM-RoBERTa, preferred the more biased sentence significantly more often than chance on both benchmarks.
- The preference rate did not differ significantly between CrowS-Pairs and IndiBias, either overall or in any of the seven shared categories.
- The per-category samples are small, so these results do not show that the two benchmarks are equal. They show only that this experiment could not detect a difference. The Limitations section explains this in more detail.

## Question and hypothesis

**Question:** Is stereotype preference in language models universal, or does it depend on the cultural context of the test items?

**Hypothesis:** I wrote this hypothesis before running the experiments. If stereotype preference is universal, the models should show similar preference rates on both benchmarks. If it is culture-specific, the rates should differ, especially in categories where the cultural content differs most, such as caste and race.

## Method

- **Models:** I used two models from Hugging Face Transformers, `bert-base-uncased` and `xlm-roberta-base`.
- **Scoring:** I scored each sentence with pseudo-log-likelihood (PLL). Each token is masked in turn, and the log-probabilities of the original tokens are summed. A pair counts as "prefers the more biased sentence" when the more biased sentence has the higher PLL.
- **CrowS-Pairs:** I used the full file `crows_pairs_anonymized.csv` (Nangia et al., 2020).
- **IndiBias:** I used the publicly available sample of 502 pairs and its English columns (Sahoo et al., 2024). The full dataset was not available to me.
- **Categories:** The two benchmarks share seven categories: age, disability, gender, physical-appearance, race/caste, religion and socioeconomic. I treated race (CrowS-Pairs) and caste (IndiBias) as an analogy, not an exact match. I dropped nationality and sexual-orientation because only CrowS-Pairs has them.
- **Sampling:** Within each category, I randomly sampled both datasets (seed 42) to the same size, with a cap of 100 pairs. This gives 456 pairs per dataset.
- **Statistics:** I used a two-sided binomial test against 50% for each model and dataset. I used Fisher's exact test to compare CrowS-Pairs with IndiBias, both overall and per category.

## Results

The tables show the percentage of pairs where the model preferred the more biased sentence. A value of 50% means no preference.

### BERT (`bert-base-uncased`)

| Category | Pairs per dataset | CrowS-Pairs (%) | IndiBias (%) |
|---|---|---|---|
| age | 62 | 58.1 | 53.2 |
| disability | 24 | 75.0 | 83.3 |
| gender | 100 | 53.0 | 65.0 |
| physical-appearance | 41 | 70.7 | 58.5 |
| race/caste | 48 | 66.7 | 66.7 |
| religion | 81 | 76.5 | 76.5 |
| socioeconomic | 100 | 59.0 | 63.0 |
| **Overall** | **456** | **63.4** | **65.6** |

### XLM-RoBERTa (`xlm-roberta-base`)

| Category | Pairs per dataset | CrowS-Pairs (%) | IndiBias (%) |
|---|---|---|---|
| age | 62 | 38.7 | 50.0 |
| disability | 24 | 70.8 | 87.5 |
| gender | 100 | 57.0 | 52.0 |
| physical-appearance | 41 | 70.7 | 58.5 |
| race/caste | 48 | 52.1 | 50.0 |
| religion | 81 | 70.4 | 79.0 |
| socioeconomic | 100 | 54.0 | 63.0 |
| **Overall** | **456** | **57.7** | **61.2** |

### Statistical tests

All four combinations of model and dataset are significantly above 50% (binomial test, all p < 0.01).

The overall difference between the two benchmarks is not significant for either model:

| Model | CrowS-Pairs (%) | IndiBias (%) | Fisher's exact p-value |
|---|---|---|---|
| BERT | 63.4 | 65.6 | 0.534 |
| XLM-RoBERTa | 57.7 | 61.2 | 0.312 |

None of the 14 per-category comparisons (seven categories for each of the two models) was significant. Their p-values ranged from 0.114 to 1.000.

### Observations

These observations are descriptive and were not tested statistically. Religion and disability have the highest preference rates in both datasets. XLM-RoBERTa has a lower overall rate than BERT. XLM-RoBERTa is below 50% on the CrowS-Pairs age category.

The charts are in `bias_bert-base-uncased.png` and `bias_xlm-roberta-base.png`. The raw per-pair scores are in the `results_*.csv` files.

## Limitations

- Not finding a significant difference is not the same as showing that the benchmarks are equal. With 24 to 100 pairs per category, only large differences can be detected.
- IndiBias is a sample of 502 rows, not the full dataset, and the disability category has only 24 pairs per dataset.
- My scoring sums the log-probability of all tokens in each sentence. The official CrowS-Pairs metric scores only the tokens that both sentences share. The swapped identity words, such as "brahmin" and "OBC", differ in frequency and tokenization, so my scores may partly reflect word frequency and not bias. Re-running the analysis with the shared-token metric is my first planned follow-up.
- The race and caste categories are an analogy and not a like-for-like comparison.
- The two benchmarks were built differently, and IndiBias is partly LLM-generated. Any differences or similarities may therefore be artifacts of how the benchmarks were made.
- PLL measures a preference in a masked language model. It does not measure harm in real applications.
- I used English text only and tested only two models.

## Planned follow-ups

1. Re-score the pairs with the shared-token metric and check whether the conclusions still hold.
2. Add more models, including an India-focused multilingual model.
3. Score the Hindi versions of the IndiBias pairs to separate the effect of language from the effect of culture.
4. Report confidence intervals for the gap between the benchmarks, in addition to p-values.

## How to reproduce

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USERNAME/REPO/blob/main/NOTEBOOK.ipynb)

1. Install the required packages. In a terminal, run the command below. In a Colab cell, put `!` before `pip`.

   ```bash
   pip install transformers torch pandas matplotlib scipy openpyxl
   ```

2. Clone the CrowS-Pairs repository:

   ```bash
   git clone https://github.com/nyu-mll/crows-pairs.git
   ```

3. Download the IndiBias sample from https://github.com/sahoonihar/IndiBias and place it next to the notebook. I do not redistribute it here.
4. Open the notebook, set the runtime to a T4 GPU, and run all cells in order. Scoring takes roughly 10 to 20 minutes per model on a T4 GPU.

## References

- Nangia, N., Vania, C., Bhalerao, R., and Bowman, S. (2020). *CrowS-Pairs: A Challenge Dataset for Measuring Social Biases in Masked Language Models.* EMNLP 2020.
- Sahoo, N., et al. (2024). *IndiBias: A Benchmark Dataset to Measure Social Biases in Language Models for Indian Context.* NAACL 2024. arXiv:2403.20147.
