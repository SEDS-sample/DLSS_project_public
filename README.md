# DLSS_project
## Authors 
Leander Müller (01/1589827), Philipp Raumanns (01/1590830),
Lars Stützle (01/1163443), Leon Exeler (01/1588945)

# Task
This project compares the performance of two LLM-based approaches for classifying job postings into **ISCO-08** occupational categories:
 
1. **In-context learning** — prompting a pretrained LLM with task instructions and examples and letting the model choose from prechosen ISCO-codes, without updating model weights.
2. **Two-step fine-tuning** — a parameter-efficient fine-tuning (PEFT) pipeline that adapts the model to the classification task in two stages.
The resulting labels are used both to evaluate classification performance against a human-annotated gold standard and to support a downstream job market analysis.

## Overview 

DISCLAIMER: Job data files are not included in this repo, since we do not have permission to further distribute the data. The URL for the data can be found at the bottom section of this file.

 This project builds a pipeline that:
 
- Cleans and anonymizes raw job posting text (removing personal information such as emails, phone numbers, URLs, names, and organizations)
- Creates a human-labeled gold standard for evaluation
- Classifies job postings into ISCO-08 categories using two competing LLM strategies
- Compares the two approaches and applies the better-performing model to the full unlabeled dataset
- Job market analysis

## Files
- **`LLM_job_posts_ISCO.md`** - Task description as provided on the DLSS website
- **`Report_Final_Project.pdf`** - The report for the present final project. 
- **`requirements.txt`** 

### /data
 
| Path | Description |
|---|---|
| `raw/ISCO-08 EN Structure and definitions.xlsx` | ISCO-08 handbook as downloaded from https://ilostat.ilo.org/methods/concepts-and-dePinitions/classiPication-occupation/ |
| `raw/job_data.parquet` | NOT INCLUDED. Unlabeled LinkedIn job postings ([source](https://huggingface.co/datasets/xanderios/linkedin-job-postings)) |
| `gold_standard_done.xlsx` | NOT INCLUDED. The complete labeling of the gold standard set, i.e., including the cases resolved with an LLM|
| `gold_standard_final.xlsx` | NOT INCLUDED. Combines the ISCO-08 labels of the gold standard set from all group members to one (where possible). Indicates cases that need to be resolved with an LLM |
| `gold_standard_Name.xlsx` | NOT INCLUDED. Contains the labels of each group member for the job-ads in the gold standard set |
| `isco_lvl2.csv` | For ISCO-08 level 2 groups, this data contains titles, definitions, codes, major groups and formatted prompt texts. Created in ISCO_preprocessing |
| `isco_lvl3.csv` | For ISCO-08 level 3 groups, this data contains titles, definitions, codes, major groups and formatted prompt texts. Created in ISCO_preprocessing |
| `isco_soc_closswalk.xls` | Translation of ISCO codes to US Bureau of Labor Statistics framework |
| `isco_with_embeddings.parquet` | Embeddings for the ISCO class taxonomies |
| `job_ads_cleaned_state.parquet` | NOT INCLUDED. Cleaned job ad corpus inlcluding an additional column coding the state of the job location |
| `job_ads_cleaned.parquet` | NOT INCLUDED. Cleaned job-ad corpus |
| `job_ads_with_embeddings.parquet` | NOT INCLUDED. Embeddings for the job ads of the validation set |
| `labeled_corpus_final.parquet` | NOT INCLUDED. Job-ad corpus with ISCO-08 level 3 labels by our LLM |
| `labeled.csv` | NOT INCLUDED. Job postings with final ISCO labels |
| `merged_labels.csv` | NOT INCLUDED. Dataset of the 740 Job postings including all 3 rater labels |
| `national_M2023_dl.xlsx` | Official US labor market statistics by the US Bureau of Labor Statistics |

### /notebooks
 
| Notebook | Purpose |
|---|---|
| `data_cleaning_final.ipynb` | Preprocessing pipeline: normalizes, anonymizes, and cleans raw job posting text |
| `DeploymentAndTesting.ipynb` | Applies the final chosen model to the full unlabeled dataset and evaluates it |
| `in_context_labeling.ipynb` | Implements the In-Context LLM: Generating Embeddings + Decoder LLM  |
| `ISCO_preprocessing.ipynb` | Processes ISCO-08 EN Structure and definitions.xlsx into prompts for our models, as included in isco_level2 and isco_level3. |
| `labor_market_analysis.ipynb` | Data preparation, analysis and visualization for labor market analysis. |
| `merging_goldstandard.ipynb` | Merges labels from multiple raters into a single gold standard file |
| `preliminary_data_analysis.ipynb` | Data preparation, analysis and visualization for preliminary data analysis. Moreover, job locations in job_ads_cleaned.parquet are combined into states, which is saved in job_ads_cleaned_state.parquet. |
| `resolve_gold_standard_labels.ipynb` | Resolves disagreements between raters in the gold standard |
| `Two_Step.ipynb` | Implements the two-step PEFT fine-tuning approach |
 


## Usage
 
1. Run `ISCO_preprocessing.ipynb` to preprocess ISCO class texts
2. Run `data_cleaning_final.ipynb` to preprocess and anonymize the raw job postings.
3. Run `merging_goldstandard.ipynb` and `resolve_gold_standard_labels.ipynb` to produce the final gold standard.
4. Run `preliminary_data_analysis.ipynb` to obtain analyses and plots for the preliminary data analysis
5. Run `in_context_labeling.ipynb` to generate embeddings and run In-Context LLM 
6. Run `Two_Step.ipynb` to fine-tune the model using the two-step PEFT approach.
7. Run `DeploymentAndTesting.ipynb` to deploy the final model and generate predictions on the full dataset.
8. Run `labor_market_analysis.ipynb` to obtain analyses and plots for the labor market analyis.


## Methods
 
- **In-context learning:** zero-/few-shot prompting of a pretrained LLM for ISCO-08 classification. Letting, the Model choose from 5/10/15 through embeddings pre-selected ISCO-codes
- **Two-step fine-tuning:** parameter-efficient fine-tuning ([PEFT](https://github.com/huggingface/peft)) applied in two stages to adapt the base model to the classification task.

## Results
- Two-step fine-tuned model outperformed in-context learning (Macro F1 of 0.67 compared to 0.38 on the validation set), with continued pretraining on the ISCO-08 taxonomy providing substantial gains
- Performance varied by occupation. Rare major groups were classified less reliably due to limited training data
- Applied to the full corpus, the model produced plausible results, matching the gold-standard distribution (Professionals, Technicians, and Managers were the largest groups)
- Occupational composition remained stable over time, with slight geographic variation in the prevalence of professionals

## Data & Model Sources
 
- LinkedIn job postings dataset: [xanderios/linkedin-job-postings](https://huggingface.co/datasets/xanderios/linkedin-job-postings) (Hugging Face)
- ISCO-08 taxonomy: International Labour Organization (2012)
- ISCO-08 / 2010 SOC crosswalk: U.S. Bureau of Labor Statistics (2012)
- Base LLM: Meta Llama 3.1 (8B)
- Fine-tuning: Hugging Face [PEFT](https://github.com/huggingface/peft)

