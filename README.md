# [ICML 2026] MedREK: Retrieval-Based Editing for Medical LLMs with Key-Aware Prompts.

[![📑 Paper](https://img.shields.io/badge/Paper-ICML%202026-blue.svg)](https://openreview.net/pdf?id=XFHjS8kw6X)
[![🤗 Dataset](https://img.shields.io/badge/🤗%20Dataset-HuggingFace-blue.svg)](https://huggingface.co/datasets/erinxia/MedVersa)

Source code for ICML 2026 (main) paper "[**MedREK: Retrieval-Based Editing for Medical LLMs with Key-Aware Prompts**](https://openreview.net/pdf?id=XFHjS8kw6X)".


## 📰 News

* [2026/11/22] 🌍 MedREK also works for **general knowledge editing**! On CounterFact, it reaches **~99% locality** on GPT2-XL and Llama3-8B-Instruct, so it is well suited to applications that need precise knowledge preservation. 👉 See [General Knowledge Editing on CounterFact](#-general-knowledge-editing-on-counterfact) for the results and how to train/test.
* [2025/04/30] 🎉 MedREK has been accepted by **ICML 2026 Main Conference**! 
* [2025/11/07] 🎉 MedREK has been accepted by ResponsibleFM @ NeurIPS 2025! 
* [2025/10/16] 🚀 Our paper is available on arXiv! 
* [2025/10/15] 🔥 We release the MedVersa dataset and code for MedREK!


## 👀 Introduction


- We present the **first batch-editing benchmark** in medical scenarios with broader coverage of medical subjects.

- We propose **MedREK**, a retrieval-based editing framework that integrates a **shared query-key module** for precise matching with an **attention-based prompt encoder** for informative guidance.

- Experimental results on various medical benchmarks demonstrate that MedREK achieves superior performance across different core metrics and provides the **first validated solution for batch-editing in medical LLMs**.

![fig](fig/medversa.png)


## 🔧 Setup

```bash
conda create -n medrek python=3.10 -y
conda activate medrek
git clone https://github.com/mylittleriver/MedREK
pip install --upgrade medrek 
pip install -r requirements.txt
```

First enter the `medrek` directory: `cd medrek`. Run all commands below from this directory.

In `utils/global_attrs.py`, change  `ROOT_PATH` and `MODEL_PATH` to your paths. With the default layout, `ROOT_PATH = ""` and `MODEL_PATH = "models"` work as is.

### Prepare models

Before training, download the following models into `medrek/models/`:

1. **RoBERTa-base (required for all runs).** MedREK uses it as the encoder of the query-key module and the prompt encoder. Its path is fixed to `models/roberta-base`:
   ```bash
   huggingface-cli download FacebookAI/roberta-base --local-dir models/roberta-base
   ```
2. **The LLM to edit.** Its folder name must match the `-mn` argument, because `train_medrek.py` loads the model from `models/<model_name>`:
   ```bash
   # Medical editing (MedVersa / MedCF)
   huggingface-cli download epfl-llm/meditron-7b --local-dir models/meditron-7b
   # General knowledge editing (CounterFact)
   huggingface-cli download openai-community/gpt2-xl --local-dir models/gpt2-xl
   huggingface-cli download meta-llama/Meta-Llama-3-8B-Instruct --local-dir models/llama3-8b-instruct
   ```
   Meditron-7B and Llama-3-8B-Instruct are gated on Hugging Face. Accept their licenses on the model pages and run `huggingface-cli login` before downloading.

The resulting layout should look like:
```
medrek/models/
├── roberta-base/
├── meditron-7b/
├── gpt2-xl/
└── llama3-8b-instruct/
```

### Prepare data

The training and evaluation data are included in `medrek/data/`, so no download is needed:
- `data/meta-train/` holds the training sets of MedCF, CounterFact, etc.
- `data/evaluation/` holds the test sets of MedCF, CounterFact, etc.
- `data/medversa/` holds the training and test sets of MedVersa.

## 📊 MedVersa

Please check the `/medversa` floder to see the `train\valid\test` split of our enhanced **Medical LLM Model Editing** Benchmark.


## ⚙️ Train MedREK

Please run:
``` bash
python train_medrek.py -mn 'meditron-7b' -dn 'medcf'  
```
Checkpoints will be saved in `train_records/recipe/meditron-7b/train_name/checkpoints/`.

You can view training information in `train_records/recipe/meditron-7b/train_name/logs/` through Tensorboard.

We also provide the implementation of [RECIPE](https://arxiv.org/abs/2405.03279) for medical LLM model editing. Please check the `RECIPE` floder.

## 💫 Evaluate MedREK

Please run:
```bash
python test_medrek.py -en 'medrek' -mn 'meditron-7b' -et 'batch' -dvc 'cuda:0' \
    -ckpt 'train_records/recipe/meditron-7b/train_name/checkpoints/a_checkpoint' \
    -dn 'medcf' -edn 100 \
```
You can check results in `eval_results/medrek`.


## 🌍 General Knowledge Editing on CounterFact

MedREK is not limited to the medical domain. It applies directly to **general knowledge editing** in LLMs. We evaluate MedREK on [CounterFact](https://rome.baulab.info/) in the batch-editing setting (100 edits per batch) with GPT2-XL and Llama3-8B-Instruct.

MedREK achieves a **very high locality score (~99%)** on both models, much higher than leading knowledge editing methods like [AlphaEdit](https://github.com/jianghoucheng/AlphaEdit), which means unrelated knowledge stays almost entirely unchanged after editing. This shows MedREK's strength in applications that require **precise knowledge preservation**.

| Model | Efficacy | Generality | Locality |
|:--|:--:|:--:|:--:|
| GPT2-XL | 84.37 | 89.15 | **98.94** |
| Llama3-8B-Instruct | 81.52 | 95.03 | **99.28** |

To reproduce, first train MedREK on CounterFact:
```bash
# GPT2-XL
python train_medrek.py -mn 'gpt2-xl' -dn 'cf' -tn 'gpt2xl_cf' -bs 8 -epo 120 --device 'cuda:0'

# Llama3-8B-Instruct
python train_medrek.py -mn 'llama3-8b-instruct' -dn 'cf' -tn 'llama3_cf' -bs 8 -epo 120 --device 'cuda:0' --model_parallel
```
Checkpoints will be saved in `train_records/medrek/<model_name>/epoch120_<train_name>/checkpoints/`.

Then evaluate with batch editing (100 edits):
```bash
# GPT2-XL
python test_medrek.py -en 'medrek' -mn 'gpt2-xl' -et 'batch' -dvc 'cuda:0' \
    -ckpt 'train_records/medrek/gpt2-xl/epoch120_gpt2xl_cf/checkpoints/epoch-92-i-115000-ema_loss-1.0983' \
    -dn 'cf' -edn 100

# Llama3-8B-Instruct
python test_medrek.py -en 'medrek' -mn 'llama3-8b-instruct' -et 'batch' -dvc 'cuda:0' \
    -ckpt 'train_records/medrek/llama3-8b-instruct/epoch120_llama3_cf/checkpoints/epoch-36-i-45000-ema_loss-0.9923' \
    -dn 'cf' -edn 100
```
The results above use the checkpoints at iteration 115000 (GPT2-XL) and 45000 (Llama3-8B-Instruct).


## 🙏 Acknowledgement
This repo is built upon the following projects: 

* [RECIPE](https://github.com/qizhou000/RECIPE)
* [MedLaSA](https://github.com/quqxui/MedLaSA)

We thank the authors for their codes.



## 📝 Citation
Please cite our work if you use our code or discuss our findings in your own research:
```bibtex
@article{xia2025medrek,
  title={MedREK: Retrieval-Based Editing for Medical LLMs with Key-Aware Prompts},
  author={Xia, Shujun and Lin, Haokun and Wu, Yichen and Zhou, Yinan and Li, Zixuan and Wan, Zhongwei and Xing, Xingrun and Zheng, Yefeng and Li, Xiang and Shan, Caifeng and others},
  journal={arXiv preprint arXiv:2510.13500},
  year={2025}
}
```
