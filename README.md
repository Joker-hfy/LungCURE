# LungCURE: Benchmarking Multimodal Real-World Clinical Reasoning for Precision Lung Cancer Diagnosis and Treatment

<p align="center">
  <a href="http://arxiv.org/abs/2604.06925"> <img src="https://img.shields.io/badge/Paper-Preprint-blue" alt="paper"/> </a>
  <a href="https://joker-hfy.github.io/LungCURE/"> <img src="https://img.shields.io/badge/Homepage-Website-green" alt="homepage"/> </a>
  <a href="https://github.com/Joker-hfy/LungCURE"> <img src="https://img.shields.io/badge/GitHub-Repository-purple" alt="github"/> </a>
  <a href="https://huggingface.co/datasets/Fine2378/LungCURE"> <img src="https://img.shields.io/badge/HuggingFace-Dataset-yellow" alt="huggingface"/> </a>
</p>




This repository supports:

- three core clinical tasks:

| Task | What it does | Supported inputs | Languages |
|---|---|---|---|
| Standalone TNM staging | Runs TNM staging only. | Text cases, image/PDF cases | Chinese, English |
| Standalone treatment | Runs treatment decision support only. | Text cases, image/PDF cases | Chinese, English |
| End-to-end TNM + treatment | Runs full TNM-to-treatment pipeline in one flow. | Text cases, image/PDF cases | Chinese, English |

- automatic evaluation against ground-truth treatment outputs.

---


---

## Repository Layout

```text
.
├── data_image/                       # Task input data directory for multimodal runs
├── data_text/                        # Task input data directory for text-case runs
├── gt/                               # Ground-truth files
├── judge/                            # Judge-model scripts for evaluation
├── LCAgent/                          # Agent code for TNM, treatment, and end-to-end pipelines
│   ├── end2end/
│   ├── TNM_staging/
│   └── treatment/
├── LC_patient_image/                 # Case data directory (LC_patient*), image/PDF cases
├── LC_patient_text/                  # Case data directory (LC_patient*), text cases
├── llm/                              # Pure-LLM code for TNM, treatment, and end-to-end baselines
│   ├── end2end/
│   ├── TNM_staging/
│   └── treatment/
├── results/                          # Output predictions and evaluation results
├── .gitignore
├── api_config.yaml                   # Provider/model configuration (local only)
└── README.md
```

---

## Environment Setup

### 0) Clone repository and enter project root

```bash
git clone https://github.com/Joker-hfy/LungCURE.git
cd LungCURE
```



### 1) Create conda environment `LungCURE`

```bash
conda create -n LungCURE python=3.12 -y
conda activate LungCURE
```

### 2) Install dependencies from requirements

Install all required packages using the repository dependency file:

```bash
pip install -r requirements.txt
```


---

## Configuration

Build `api_config.yaml` in the repository root by following the format of `api_config_example.yaml`.

```bash
cp api_config_example.yaml api_config.yaml
```

Then edit `api_config.yaml` and replace model names, base URLs and API keys, with your actual provider settings.

Typical fields used by scripts include:

- `active_provider`
- `evaluation.judge_provider`
- `providers.<name>.model`
- `providers.<name>.base_url`
- `providers.<name>.api_key`
- `providers.<name>.timeout`
- `providers.<name>.max_retries`
- `providers.<name>.retry_backoff_seconds`
- `providers.<name>.temperature`
- `providers.<name>.max_tokens`

---

## Data Format

### 1) Download case files from Hugging Face

Please download the case files from the Hugging Face dataset repo and place them in the project root while keeping the folder structure unchanged.


After download, ensure these folders are available in the repository root:

- `data_image/`
- `data_text/`
- `gt/`
- `LC_patient_image/`
- `LC_patient_text/`

### 2) What is stored in each folder

- `data_image/`: benchmark index JSON files for multimodal runs. These files point to case metadata and PDF paths.
- `data_text/`: benchmark index JSON files for text runs. These files point to case metadata and text paths (and may also include PDF paths).
- `gt/`: ground-truth JSON files containing clinician-annotated real-world case labels for evaluation. It contains two folders: `gt/tnm_gt/` and `gt/treatment_gt/`.
- `LC_patient_image/`: raw multimodal case files in PDF format (`.pdf`), typically split by language (`Chinese/`, `English/`).
- `LC_patient_text/`: raw text case files in Markdown format (`.md`), typically split by language (`Chinese/`, `English/`).

### 3) Expected JSON schema for `data_image` and `data_text`

Both folders store benchmark JSON files keyed by case ID. Language/seed are inferred from filenames such as `*_Chinese_seed42.json` and `*_English_seed3407.json`.

`data_image/*.json` example:

```json
{
  "LC_patient_0007": {
    "case_id": "LC_patient_0007",
    "Chinese_file_name": "LC_patient_image/Chinese/LC_patient_0007.pdf",
    "English_file_name": "LC_patient_image/English/LC_patient_0007.pdf"
  }
}
```

`data_text/*.json` example:

```json
{
  "LC_patient_0007": {
    "case_id": "LC_patient_0007",
    "Chinese_file_name": "LC_patient_image/Chinese/LC_patient_0007.pdf", // optional field
    "English_file_name": "LC_patient_image/English/LC_patient_0007.pdf", // optional field
    "chinese_md_name": "LC_patient_text/Chinese/LC_patient_0007.md",
    "english_md_name": "LC_patient_text/English/LC_patient_0007.md"
  }
}
```

---

## Run Inference

### A) TNM

#### 1. Agent Pipelines

Text-input agent pipeline:

```bash
OPENAI_API_KEY="your-api-key-here" \
OPENAI_BASE_URL="your-url-here" \
MODEL_NAME="gpt-5.2" \
python -u LCAgent/TNM_staging/run_benchmark_agent_simplified_Chinese.py \
  > logs/tnm_agent_text.log 2>&1
```

Multimodal-input agent pipeline (PDF):

```bash
OPENAI_API_KEY="your-api-key-here" \
OPENAI_BASE_URL="your-url-here" \
MODEL_NAME="gpt-5.2" \
python -u LCAgent/TNM_staging/run_ocr_benchmark_agent_simplified_Chinese.py \
  > logs/tnm_agent_ocr.log 2>&1
```

#### 2. LLM Baselines (Comparison Only)

Text baseline:

```bash
OPENAI_API_KEY="your-api-key-here" \
OPENAI_BASE_URL="your-url-here" \
MODEL_NAME="gpt-5.2" \
python -u llm/TNM_staging/run_benchmark_simplified_Chinese.py \
  > logs/tnm_llm_text.log 2>&1
```

PDF/image baseline:

```bash
OPENAI_API_KEY="your-api-key-here" \
OPENAI_BASE_URL="your-url-here" \
MODEL_NAME="gpt-5.2" \
python -u llm/TNM_staging/run_ocr_simplified_Chinese.py \
  > logs/tnm_llm_ocr.log 2>&1
```

### B) treatment

#### 1. Agent Pipelines

Text-input agent pipeline:

```bash
python LCAgent/treatment/run_agent.py \
    --workflow-yml LCAgent/treatment/非小细胞癌治疗-免提示-en.yml \
    --case-file LC_patient_text/LC_patient_0001.txt \
    --model gpt-5.2
```

Multimodal-input agent pipeline (PDF):

```bash
python LCAgent/treatment/run_agent.py \
    --workflow-yml LCAgent/treatment/非小细胞癌治疗-免提示-en.yml \
    --case-file LC_patient_image/LC_patient_0002.pdf \
    --read-pdf \
    --model gpt-5.2
```

#### 2. LLM Baselines (Comparison Only)

Text baseline:

```bash
python llm/treatment/run_llm.py \
    --case-file LC_patient_text/LC_patient_0001.txt \
    --model gpt-5.2
```

PDF/image baseline:

```bash
python llm/treatment/run_llm.py \
    --case-file LC_patient_image/LC_patient_0002.pdf \
    --model gpt-5.2
```

Output filename rules:

1) Workflow (`LCAgent`)
- If `--case-file` is provided and `--output-json` is not provided:
  `LCAgent/outputs/<case_filename>_agent_results.json`
- If `--case-file` is not provided (directory batch mode):
  `LCAgent/outputs/agent_results.json`

2) Direct LLM (`llm`)
- If `--case-file` is provided and `--output-json` is not provided:
  `llm/outputs/<case_filename>_llm_results.json`
- If `--case-file` is not provided (directory batch mode):
  `llm/outputs/llm_results.json`

### C) end2end

#### 1. Agent Pipelines

Text-input agent pipeline:

```bash
python -u LCAgent/end2end/workflow_text.py \
  --data-dir data_text \
  --provider gpt-5.2 \
  --output-dir results/results_end2end/outputs/agent_outputs/agent_text_outputs \
  > logs/end2end_agent_text.log 2>&1
```

Multimodal-input agent pipeline (PDF):

```bash
python -u LCAgent/end2end/workflow_image.py \
    --data-dir data_image  \
    --provider gpt-5.2 \
    --output-dir results/results_end2end/outputs/agent_outputs/agent_image_outputs \
    > logs/end2end_agent_image.log 2>&1
```

If you want to run partial-file tests for the two agent commands above, use either:

- `--input` to run one specific benchmark file.
- `--max-files` with `--max-cases` to limit test scale.

Examples:

```bash
# Text agent: test one input file
python -u LCAgent/end2end/workflow_text.py \
  --input benchmark_chinese_seed42.json \
  --provider gpt-5.2 \
  --output-dir results/results_end2end/outputs/agent_outputs/agent_text_outputs

# Multimodal agent: limit to first 1 file and first 5 cases
python -u LCAgent/end2end/workflow_image.py \
    --data-dir data_image  \
    --provider gpt-5.2 \
    --output-dir results/results_end2end/outputs/agent_outputs/agent_image_outputs \
    --max-files 1 \
    --max-cases 5 
```

Default agent output directories:

- Text: `results/results_end2end/outputs/agent_outputs/agent_text_outputs`
- Multimodal: `results/results_end2end/outputs/agent_outputs/agent_image_outputs`


#### 2. LLM Baselines (Comparison Only)

Text baseline:

```bash
python -u llm/end2end/llm_text.py \
    --data-dir data_text \
    --model gpt-5.2 \
    --output-dir results/results_end2end/outputs/llm_outputs/llm_text_outputs
    > logs/end2end_llm_text.log 2>&1
```

PDF/image baseline:

```bash
python -u llm/end2end/llm_image.py \
    --data-dir data_image \
    --model gpt-5.2 \
    --output-dir results/results_end2end/outputs/llm_outputs/llm_image_outputs \
    > logs/end2end_llm_image.log 2>&1
```

---

## Run Evaluation

### A) TNM

```bash
python judge/judge_TNMstaging.py
# Ensure you point the script to the generated result JSONs and your Ground Truth data.
```

### B) treatment
```bash
python judge/judge_treatment.py \
    --mode single \
    --pred-dir <YOUR PRED DIR> \
    --gt-dir <YOUR GT DIR> \
    --include "*.json" \
    --workers 1 \
    --metrics-dir outputs/judge/single\
```

Note: the judge extracts the seed from prediction filenames/content to match the corresponding GT files.

Default judge output rules (when `--metrics-dir` / `--result-root` are not provided):

- If `--pred-dir` is under `LCAgent`, output defaults to `LCAgent/outputs/judge/single` (paired mode uses `LCAgent/outputs/judge/paired`).
- If `--pred-dir` is under `llm`, output defaults to `llm/outputs/judge/single` (paired mode uses `llm/outputs/judge/paired`).

```bash
# TODO: add standalone treatment/CDSS evaluation commands
```

### C) end2end

Text-input agent outputs:

```bash
python judge/judge_end2end.py \
  --config api_config.yaml \
  --gt_dir gt/cdss_gt \
  --input_dir results/results_end2end/outputs/agent_outputs/agent_text_outputs \
  --output_dir results/results_end2end/metrics/agent_metrics/agent_text_metrics
```

Multimodal-input agent outputs:

```bash
python judge/judge_end2end.py \
  --config api_config.yaml \
  --gt_dir gt/cdss_gt \
  --input_dir results/results_end2end/outputs/agent_outputs/agent_image_outputs \
  --output_dir results/results_end2end/metrics/agent_metrics/agent_image_metrics
```

LLM baseline outputs can be evaluated by pointing `--input_dir` to the corresponding baseline folder.

Metric file naming convention:

`[model_name]_metric_[chinese|english]_seedN.json`

Expected input naming convention:

`[model_name]_output_[chinese|english]_seedN.json`

---

*Output Schema (Inference)*

Each case output typically includes:

- `case_id`
- `tnm_result`
  - `tnm_for_cdss`
  - `tnm_text`
  - `stage_result`
- `cdss_result`
- optional error/debug fields (`error`, `raw_model_output`, etc.)

Output file structure:

```text
results_end2end/
├── outputs/
│   ├── agent_outputs/
│   │   ├── agent_image_outputs/
│   │   └── agent_text_outputs/
│   └── llm_outputs/
│       ├── llm_image_output/
│       └── llm_text_output/
└── metrics/
    ├── agent_metrics/
    │   ├── agent_image_metrics/
    │   └── agent_text_metrics/
    └── llm_metrics/
        ├── llm_image_metrics/
        └── llm_text_metrics/
```

File naming convention:

`"[model_name]_[output/metric]_[chinese/english]_[seed + number].json"`

---

## Reproducibility Tips

- Fix model version and provider endpoint.
- Keep benchmark and GT sets immutable per experiment.
- Log command lines and seeds for each run.
- Use separate output folders per model/provider.

---

## Citation

If you use this repository in your research, please cite:

```bibtex
@misc{hao2026lungcurebenchmarkingmultimodalrealworld,
      title={LungCURE: Benchmarking Multimodal Real-World Clinical Reasoning for Precision Lung Cancer Diagnosis and Treatment}, 
      author={Fangyu Hao and Jiayu Yang and Yifan Zhu and Zijun Yu and Qicen Wu and Wang Yunlong and Jiawei Li and Yulin Liu and Xu Zeng and Guanting Chen and Shihao Li and Zhonghong Ou and Meina Song and Mengyang Sun and Haoran Luo and Yu Shi and Yingyi Wang},
      year={2026},
      eprint={2604.06925},
      archivePrefix={arXiv},
      primaryClass={cs.MM},
      url={https://arxiv.org/abs/2604.06925}, 
}
```

---

## License
This project is licensed under the MIT License. See the LICENSE file for details.

---

## Medical Disclaimer

This codebase is for research and benchmarking only. It is **not** a medical device and must not be used as the sole basis for diagnosis or treatment decisions.
