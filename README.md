README — Persona Phishing Vulnerability Pipeline
Project Overview
This project is an experimental pipeline built with Google Colab + multiple LLM APIs. It is designed to:
Generate three personas using Prompt 1
Ask a model to choose which persona is more vulnerable to phishing using Prompt 2, with an explanation
Parse raw outputs into a structured dataset
Automatically add labels, scores, and trustworthiness-related indicators
Perform statistical analysis and visualization
The full workflow consists of 6 notebooks and should be run in order.
---
File Overview
Notebook Order
01_setup_and_template.ipynb  
Initializes the project directory, dataset template, model configuration, and the two prompt files.
02_run_prompt1_api.ipynb  
Calls provider/model APIs and runs Prompt 1 in batch to generate persona triplets.
03_parse_prompt1_to_dataset.ipynb  
Parses the raw Prompt 1 outputs into a structured dataset.
04_run_prompt2_api.ipynb  
Uses each persona triplet as input for Prompt 2, asking the model to choose the more phishing-vulnerable persona and explain why.
05_label_and_score_metrics.ipynb  
Maps Prompt 2 outputs back to the personas, creates `Yes/No` labels, and adds trustworthiness-related indicators such as:
`reason_category`
`bias_flag`
`bias_type`
`factuality_flag`
`privacy_security_flag`
`ethical_reasoning_flag`
`toxicity_flag`
06_analysis(1).ipynb  
Reads the scored dataset and produces descriptive statistics, significance tests, and figures.
---
Recommended Environment
Recommended platform: Google Colab
The notebooks use a fixed Google Drive path:
```python
BASE_DIR = "/content/drive/MyDrive/Colab Notebooks/a2"
```
So the most reliable setup is:
Open the notebooks in Google Colab
Mount Google Drive
Use the same `BASE_DIR` across all notebooks
---
Required Libraries
Most dependencies are already available in Colab. If needed, install them with:
```python
!pip install pandas openpyxl requests numpy matplotlib scipy
```
Main libraries used in this project:
`pandas`
`openpyxl`
`requests`
`numpy`
`matplotlib`
`scipy`
---
API Key Requirements
This project calls models from the following providers:
OpenRouter
Together
Groq
SambaNova
DeepInfra
In 02_run_prompt1_api.ipynb and 04_run_prompt2_api.ipynb, the notebooks use `getpass()` to request API keys such as:
`OPENROUTER_API_KEY`
`TOGETHER_API_KEY`
`GROQ_API_KEY`
`SAMBANOVA_API_KEY`
`DEEPINFRA_API_KEY`
Notes
If you do not want to use a provider, set its model entry to `"enabled": "no"` in 01_setup_and_template.ipynb
If a key is left blank, the corresponding provider will be skipped automatically
If `"enabled": "yes"` is kept but no valid key is provided, that model will not run successfully
---
How to Run
Step 1: Run `01_setup_and_template.ipynb`
Purpose:
Creates the main dataset template `persona_level_dataset.xlsx`
Creates the model configuration file `models_config.xlsx`
Creates:
`prompt1.txt`
`prompt2.txt`
Output files
`persona_level_dataset.xlsx`
`models_config.xlsx`
`prompt1.txt`
`prompt2.txt`
Notes
The current setup uses:
Prompt 1 runs per model: `2`
Prompt 2 runs per persona group: `10`
If 15 models are enabled, the theoretical sample size is:
```text
15 × 2 × 10 × 3 = 900 samples
```
---
Step 2: Run `02_run_prompt1_api.ipynb`
Purpose:
Reads `models_config.xlsx`
Reads `prompt1.txt`
Calls APIs to generate persona triplets for each enabled model
Saves the raw outputs
Input
`models_config.xlsx`
`prompt1.txt`
Output
`raw_prompt1_outputs.xlsx`
`raw_prompt1_outputs_clean.xlsx`
Notes
This notebook will:
Automatically skip completed results
Keep only successful responses (`status == 200`) with valid text content in the clean output file
---
Step 3: Run `03_parse_prompt1_to_dataset.ipynb`
Purpose:
Reads Prompt 1 raw outputs
Splits each output into 3 personas
Extracts structured fields
Writes them into the main dataset
Input
`raw_prompt1_outputs_clean.xlsx`
`persona_level_dataset.xlsx`
Output
Updated `persona_level_dataset.xlsx`
Extracted fields include
`provider`
`model`
`group_id`
`prompt1_run_id`
`persona_id`
`persona_name`
`profile_details`
`name`
`age`
`age_group`
`gender`
`personality_traits`
`domain_of_work`
`years_of_experience`
`years_experience_group`
`location`
`location_region`
`education_level`
`devices_technologies_use`
Notes
If a Prompt 1 output cannot be parsed into exactly 3 personas, that record will be skipped.
---
Step 4: Run `04_run_prompt2_api.ipynb`
Purpose:
Reads each persona triplet by `group_id`
Builds the Prompt 2 input
Calls the corresponding provider/model
Saves the raw Prompt 2 outputs
Input
`persona_level_dataset.xlsx`
`prompt2.txt`
Output
`raw_prompt2_outputs.xlsx`
`raw_prompt2_outputs_clean.xlsx`
Notes
This notebook also supports:
Automatically skipping completed `group_id + prompt2_run_id` combinations
Cleaning invalid results
Checking for missing Prompt 2 runs
There is also an optional cell later in the notebook for rerunning a single missing result. It is only needed when a specific run is missing.
---
Step 5: Run `05_label_and_score_metrics.ipynb`
Purpose:
Maps each Prompt 2 response back to one of the 3 personas
Assigns final labels for each persona group:
Selected persona: `phishing_susceptible = Yes`
Other two personas: `phishing_susceptible = No`
Automatically generates trustworthiness-related indicators
Input
`persona_level_dataset.xlsx`
`raw_prompt2_outputs_clean.xlsx`
Output
`persona_level_scored_dataset.xlsx`
Notes
This notebook attempts to:
Identify the selected `P1 / P2 / P3`
Extract reason categories
Infer flags related to bias, privacy/security, ethical reasoning, toxicity, and factuality
It also provides checks for:
Whether each group/run has exactly 3 rows
Whether each group/run has exactly 1 `Yes`
If some outputs cannot be matched correctly, manual checking may be required.
---
Step 6: Run `06_analysis(1).ipynb`
Purpose:
Reads the scored dataset
Performs statistical analysis and visualization
Exports summary tables and test results
Input
`persona_level_scored_dataset.xlsx`
Output
On-screen charts
`qualitative_sample_25pct.xlsx`
`summary_provider.xlsx`
`summary_model.xlsx`
`summary_gender.xlsx`
`summary_agegroup.xlsx`
`summary_region.xlsx`
`summary_reason_categories.xlsx`
`summary_bias_types.xlsx`
`statistical_test_results.xlsx`
Analysis includes
Overall susceptible / non-susceptible distribution
Provider-level susceptibility rates
Model-level susceptibility rates
Gender vs susceptibility
Age / age-group vs susceptibility
Region vs susceptibility
Education vs susceptibility
Domain / reason-category / bias-type analysis
Random qualitative sample export
Statistical tests such as chi-square, t-test, and Fisher’s exact test
---
Recommended Execution Order
Run the notebooks in the following order:
```text
01_setup_and_template.ipynb
→ 02_run_prompt1_api.ipynb
→ 03_parse_prompt1_to_dataset.ipynb
→ 04_run_prompt2_api.ipynb
→ 05_label_and_score_metrics.ipynb
→ 06_analysis(1).ipynb
```
---
Key Intermediate Files
The main file flow is:
```text
01_setup_and_template
 ├─ persona_level_dataset.xlsx
 ├─ models_config.xlsx
 ├─ prompt1.txt
 └─ prompt2.txt

02_run_prompt1_api
 ├─ raw_prompt1_outputs.xlsx
 └─ raw_prompt1_outputs_clean.xlsx

03_parse_prompt1_to_dataset
 └─ persona_level_dataset.xlsx   (updated)

04_run_prompt2_api
 ├─ raw_prompt2_outputs.xlsx
 └─ raw_prompt2_outputs_clean.xlsx

05_label_and_score_metrics
 └─ persona_level_scored_dataset.xlsx

06_analysis(1)
 ├─ qualitative_sample_25pct.xlsx
 ├─ summary_provider.xlsx
 ├─ summary_model.xlsx
 ├─ summary_gender.xlsx
 ├─ summary_agegroup.xlsx
 ├─ summary_region.xlsx
 ├─ summary_reason_categories.xlsx
 ├─ summary_bias_types.xlsx
 └─ statistical_test_results.xlsx
```
---
Common Issues
1. `FileNotFoundError`
This usually means the previous notebook was not run, or `BASE_DIR` is inconsistent.
Check:
Whether all notebooks use exactly the same `BASE_DIR`
Whether the previous notebook successfully generated the required file
---
2. API returns 404 / empty output / non-200 status
Possible reasons:
Incorrect model name
The provider does not support that model
Invalid API key
Free-tier quota or balance exhausted
Rate limiting
Suggestions:
Test one model first
Verify model names in `models_config.xlsx`
Check the saved `status` column in the output file
---
3. Prompt 1 parsing failure
If model output format is unstable, `03_parse_prompt1_to_dataset.ipynb` may fail to split the result into 3 personas correctly.
Suggestions:
Inspect the raw text in `raw_prompt1_outputs_clean.xlsx`
If necessary, adjust the regex rules in `split_personas()` or `parse_persona_block()`
---
4. Prompt 2 cannot be matched to a persona
In `05_label_and_score_metrics.ipynb`, you may encounter cases where:
`P1 / P2 / P3` is not identified correctly
The detected name does not match the actual persona
Suggestions:
Check `unmatched_groups`
Check `unmatched_detail`
Manually review abnormal outputs
---
5. Duplicate results after reruns
The notebooks already try to:
Skip completed results
Remove duplicates with `drop_duplicates`
However, if you changed:
The model list
The prompt content
The dataset schema
Then it is recommended to:
Create a new `BASE_DIR`, or
Delete old output files before rerunning
---
Minimal Quick-Run Version
If you only want the shortest reproducible workflow:
Open all notebooks in Colab
Make sure `BASE_DIR` is identical in every notebook
Run `01`
Enter API keys and run `02`
Run `03`
Enter API keys and run `04`
Run `05`
Run `06`
---
Notes
The default target sample size is approximately 900
This project is better suited to Colab + Google Drive
Running locally is possible, but you must manually change every `BASE_DIR` path and handle file paths yourself
---
One-Sentence Summary
Run 01 to initialize, 02 to generate personas, 03 to structure the dataset, 04 to run phishing-vulnerability selection, 05 to label and score, and 06 to analyze the results.
