# Persona Phishing Vulnerability Analysis Pipeline

A Google Colab–based experimental pipeline for generating personas with multiple LLM providers, identifying which persona is judged more vulnerable to phishing, and analysing the trustworthiness of those explanations.

## What this project does

This project runs a six-step workflow:

1. create the project template and configuration files
2. generate persona triplets with Prompt 1
3. parse the raw persona outputs into a structured dataset
4. run Prompt 2 to ask which persona is more phishing-vulnerable
5. label, score, and annotate trustworthiness-related metrics
6. perform statistical analysis and produce result files

The notebooks are designed to be run **sequentially** in **Google Colab** with files stored in **Google Drive**.

---

## Pipeline overview

```text
01_setup_and_template.ipynb
        ↓
02_run_prompt1_api.ipynb
        ↓
03_parse_prompt1_to_dataset.ipynb
        ↓
04_run_prompt2_api.ipynb
        ↓
05_label_and_score_metrics.ipynb
        ↓
06_analysis(1).ipynb
```

---

## Repository structure

```text
.
├── 01_setup_and_template.ipynb
├── 02_run_prompt1_api.ipynb
├── 03_parse_prompt1_to_dataset.ipynb
├── 04_run_prompt2_api.ipynb
├── 05_label_and_score_metrics.ipynb
├── 06_analysis(1).ipynb
└── README.md
```

During execution, the notebooks create and use files inside the following Google Drive folder:

```python
BASE_DIR = "/content/drive/MyDrive/Colab Notebooks/a2"
```

---

## Recommended environment

This project is built for **Google Colab**.

Why Colab is recommended:
- the notebooks already mount Google Drive
- the file paths are hard-coded for the Colab + Drive workflow
- common libraries are already available or easy to install

### Required Python packages

If needed, install the dependencies with:

```python
!pip install pandas openpyxl requests numpy matplotlib scipy
```

Main packages used:
- `pandas`
- `openpyxl`
- `requests`
- `numpy`
- `matplotlib`
- `scipy`

---

## Model providers used

The notebooks are set up to call open-source or openly accessible models through multiple providers, including:

- OpenRouter
- Together
- Groq
- SambaNova
- DeepInfra

API keys are requested interactively in the generation notebooks via `getpass()`.

Expected key names include:
- `OPENROUTER_API_KEY`
- `TOGETHER_API_KEY`
- `GROQ_API_KEY`
- `SAMBANOVA_API_KEY`
- `DEEPINFRA_API_KEY`

---

## Before you run

### 1. Upload the notebooks to Google Drive or Colab

Open all six notebooks in Google Colab.

### 2. Keep the same project folder

The notebooks expect this folder to exist:

```text
/content/drive/MyDrive/Colab Notebooks/a2
```

### 3. Decide which models to enable

In `01_setup_and_template.ipynb`, a model configuration file is created.
Each model has an `enabled` field.

Use:
- `"yes"` to run the model
- `"no"` to skip the model

### 4. Prepare API keys

When running `02_run_prompt1_api.ipynb` and `04_run_prompt2_api.ipynb`, paste the provider keys when prompted.

If a key is left blank, the corresponding provider will be skipped.

---

## How to run

## Step 1 — Run `01_setup_and_template.ipynb`

This notebook initializes the project.

### What it creates

- `persona_level_dataset.xlsx`
- `models_config.xlsx`
- `prompt1.txt`
- `prompt2.txt`

### Main purpose

- mounts Google Drive
- creates the working directory
- creates the empty dataset template
- defines the model/provider configuration
- writes Prompt 1 and Prompt 2 into text files

### Important defaults

The current setup uses:

- `PROMPT1_RUNS_PER_MODEL = 2`
- `PROMPT2_RUNS_PER_GROUP = 10`

This means the total number of persona-level records can be estimated as:

```text
number_of_enabled_models × 2 × 10 × 3
```

For example, if 15 models are enabled:

```text
15 × 2 × 10 × 3 = 900 persona-level samples
```

---

## Step 2 — Run `02_run_prompt1_api.ipynb`

This notebook sends Prompt 1 to the enabled models and stores the raw outputs.

### Input

- `models_config.xlsx`
- `prompt1.txt`

### Output

- `raw_prompt1_outputs.xlsx`
- `raw_prompt1_outputs_clean.xlsx`

### What it does

- reads enabled models from `models_config.xlsx`
- requests provider API keys using `getpass()`
- runs Prompt 1 for each enabled model
- saves all raw responses
- keeps a cleaned version containing successful usable responses

### Default generation parameters

- `PROMPT1_RUNS_PER_MODEL = 2`
- `TEMPERATURE = 0.7`
- `MAX_TOKENS = 1200`
- `WAIT_SECONDS_BETWEEN_CALLS = 3`

### Notes

- completed rows can be skipped automatically
- only successful, non-empty outputs should remain in the clean file

---

## Step 3 — Run `03_parse_prompt1_to_dataset.ipynb`

This notebook converts the Prompt 1 text outputs into a structured persona dataset.

### Input

- `raw_prompt1_outputs_clean.xlsx`
- `persona_level_dataset.xlsx`

### Output

- updated `persona_level_dataset.xlsx`

### What it extracts

Examples of extracted fields include:

- `provider`
- `model`
- `group_id`
- `prompt1_run_id`
- `persona_id`
- `persona_name`
- `profile_details`
- `name`
- `age`
- `age_group`
- `gender`
- `personality_traits`
- `domain_of_work`
- `years_of_experience`
- `years_experience_group`
- `location`
- `location_region`
- `education_level`
- `devices_technologies_use`

### Notes

- each Prompt 1 result is expected to contain **exactly 3 personas**
- malformed generations may be skipped during parsing

---

## Step 4 — Run `04_run_prompt2_api.ipynb`

This notebook uses each persona triplet as the input for Prompt 2.
The model is asked to choose which persona appears more vulnerable to phishing and explain why.

### Input

- `persona_level_dataset.xlsx`
- `prompt2.txt`

### Output

- `raw_prompt2_outputs.xlsx`
- `raw_prompt2_outputs_clean.xlsx`

### What it does

- groups personas using `group_id`
- reconstructs each 3-persona set
- sends Prompt 2 to the same provider/model setup
- stores raw and cleaned outputs

### Default generation parameters

- `PROMPT2_RUNS_PER_GROUP = 10`
- `TEMPERATURE = 0.7`
- `MAX_TOKENS = 1400`
- `WAIT_SECONDS_BETWEEN_CALLS = 3`

---

## Step 5 — Run `05_label_and_score_metrics.ipynb`

This notebook maps Prompt 2 decisions back onto the personas and adds scoring / trustworthiness annotations.

### Input

- `persona_level_dataset.xlsx`
- `raw_prompt2_outputs_clean.xlsx`

### Output

- `persona_level_scored_dataset.xlsx`

### What it adds

Examples of derived fields include:

- `phishing_susceptible`
- `reasons`
- `reason_category`
- `bias_flag`
- `bias_type`
- `factuality_flag`
- `factuality_notes`
- `privacy_security_flag`
- `privacy_security_notes`
- `ethical_reasoning_flag`
- `ethical_reasoning_notes`
- `toxicity_flag`
- `interpretation_notes`

### Notes

- the notebook attempts to identify the chosen persona from the Prompt 2 explanation
- labels are then applied back to all personas in the relevant group

---

## Step 6 — Run `06_analysis(1).ipynb`

This notebook reads the scored dataset and performs the final analysis.

### Input

- `persona_level_scored_dataset.xlsx`

### Output

- `qualitative_sample_25pct.xlsx`
- `statistical_test_results.xlsx`
- plots and summary tables generated in the notebook

### What it analyses

The notebook includes analysis of variables such as:

- provider
- model
- gender
- age and age group
- region
- education level
- domain of work
- trustworthiness-related flags

### Statistical methods used

The notebook imports and uses methods such as:

- chi-square tests
- independent two-sample t-tests
- Fisher’s exact test
- descriptive summaries and visualisations

---

## Main generated files

After the full pipeline finishes, the key generated files are:

```text
persona_level_dataset.xlsx
models_config.xlsx
prompt1.txt
prompt2.txt
raw_prompt1_outputs.xlsx
raw_prompt1_outputs_clean.xlsx
raw_prompt2_outputs.xlsx
raw_prompt2_outputs_clean.xlsx
persona_level_scored_dataset.xlsx
qualitative_sample_25pct.xlsx
statistical_test_results.xlsx
```

---

## Minimal run order

If you only need the shortest instruction set, run the notebooks in this exact order:

```text
01_setup_and_template.ipynb
02_run_prompt1_api.ipynb
03_parse_prompt1_to_dataset.ipynb
04_run_prompt2_api.ipynb
05_label_and_score_metrics.ipynb
06_analysis(1).ipynb
```

---

## Troubleshooting

### File not found

Make sure all notebooks use the same `BASE_DIR`:

```python
BASE_DIR = "/content/drive/MyDrive/Colab Notebooks/a2"
```

If one notebook writes to a different folder, later notebooks will not find the expected files.

### API calls fail

Check the following:
- the model is marked `enabled = "yes"`
- the correct provider key was entered
- the provider actually supports that model name
- the provider account still has valid quota / credit

### Empty or unusable outputs

This usually means one of the following:
- the model returned no content
- the provider response format was different from expected
- the request timed out or was partially blocked

Use the raw output files first when debugging:
- `raw_prompt1_outputs.xlsx`
- `raw_prompt2_outputs.xlsx`

### Parsing issues in Step 3

If a Prompt 1 result does not clearly contain three personas, that record may be skipped.
Review `raw_prompt1_outputs_clean.xlsx` and inspect the formatting of the problematic outputs.

### Label-matching issues in Step 5

If the explanation does not clearly mention `P1`, `P2`, or `P3`, the notebook may need to infer the target persona by name or context. Ambiguous outputs can reduce matching accuracy.

---

## Reproducibility notes

Because this project depends on external LLM APIs, results may vary across runs due to:

- model updates by providers
- stochastic generation
- temporary API instability
- rate limits or quota restrictions
- differences in provider-side model routing

For this reason, exact replication of outputs is not guaranteed even when the same notebooks are reused.

---

## Suggested README title for GitHub

If you want this file to be your repository homepage, a clean repository name could be:

```text
persona-phishing-vulnerability-pipeline
```

or

```text
llm-phishing-persona-analysis
```

---

## License / academic use note

This repository appears to be structured as coursework or an academic experiment pipeline. Before publishing it publicly, make sure doing so does not violate any course, provider, or institutional policy.
