# Graduation Thesis

## Overview

This repository contains the experimental code and artifacts for an undergraduate thesis on image classification and explainable AI (XAI). The workflow is organized into three stages:

1. Data preparation and debiasing
2. Baseline classifier training and comparison
3. XAI evaluation on top of the selected baseline model

The current project is built around a 6-class scene image dataset with the following labels:

- `buildings`
- `forest`
- `glacier`
- `mountain`
- `sea`
- `street`

## Repository Layout

| Path | Description |
| --- | --- |
| `notebook/` | Main experiment notebooks for data preparation, baselines, and XAI |
| `dataset/` | Raw dataset and local data copies |
| `outputs/` | Snapshot of generated experiment outputs |

## Notebook Organization

### 1. Data Preparation

| Notebook | Purpose |
| --- | --- |
| `notebook/data_prep.ipynb` | Builds the data manifest, detects duplicates, creates unified train/val/test splits, saves preprocessing config, and validates dataset integrity |

### 2. Baseline Classifiers

| Notebook | Model |
| --- | --- |
| `notebook/baseline/baseline_resnet.ipynb` | ResNet18 |
| `notebook/baseline/baseline_vgg.ipynb` | VGG16-BN |
| `notebook/baseline/baseline_vit.ipynb` | Vanilla ViT (`vit_b_16`) |
| `notebook/baseline/baseline_swin.ipynb` | Swin Transformer (`swin_t`) |
| `notebook/baseline/compare_baselines.ipynb` | Aggregates the four baseline results and selects the best model |

### 3. XAI Methods

| Notebook | Category | Method |
| --- | --- | --- |
| `notebook/xai/post-hoc/xai_lime.ipynb` | Post-hoc | LIME |
| `notebook/xai/post-hoc/xai_shap.ipynb` | Post-hoc | SHAP |
| `notebook/xai/intrinsic/xai_protopnet.ipynb` | Intrinsic | ProtoPNet |
| `notebook/xai/learned/xai_vibi_teacher_student.ipynb` | Learned explainer | VIBI Teacher-Student |
| `notebook/xai/end-to-end/xai_vibi_e2e.ipynb` | End-to-end | TB-VIBI / End-to-End VIBI |
| `notebook/xai/compare_xai.ipynb` | Summary | Aggregate and compare all saved XAI metrics |

## Dataset

The current dataset follows a standard scene-classification layout:

| Path | Description |
| --- | --- |
| `dataset/archive/seg_train/seg_train/<class>/` | Training images |
| `dataset/archive/seg_test/seg_test/<class>/` | Test images |
| `dataset/archive/seg_pred/seg_pred/` | Extra prediction images, not used in the main pipeline |

Dataset source:

- Kaggle: `https://www.kaggle.com/datasets/puneet6060/intel-image-classification`
- Dataset name: `Intel Image Classification`
- The repository assumes the extracted files are arranged under `dataset/archive/`.

Based on the current `split_all.csv`, the unified split already generated in this repository is:

| Split | Samples |
| --- | ---: |
| `train` | 12,629 |
| `val` | 1,402 |
| `test` | 3,000 |
| Total | 17,031 |

Training-set class distribution:

| Class | Training Samples |
| --- | ---: |
| `buildings` | 1,971 |
| `forest` | 2,044 |
| `glacier` | 2,164 |
| `mountain` | 2,260 |
| `sea` | 2,047 |
| `street` | 2,143 |

## Output Layout

The generated output snapshot currently stored in this repository lives under `outputs/` and is split into three parts:

| Path | Content |
| --- | --- |
| `outputs/data_prep/` | Manifests, duplicate reports, split files, preprocessing config, and validation reports |
| `outputs/baseline/` | Model weights, configs, training histories, and evaluation results for the four baseline models |
| `outputs/xai/` | Quantitative XAI metrics, per-sample results, explanation visualizations, and selected checkpoints |

### `data_prep` Artifacts

`outputs/data_prep/` currently contains:

- `data_manifest.csv`
- `data_manifest_clean.csv`
- `duplicates_report.csv`
- `split_all.csv`
- `split_train.txt`
- `split_val.txt`
- `split_test.txt`
- `preprocess_config.yaml`
- `class_distribution.json`
- `cleaning_log.json`
- `experiment_seeds.json`
- `verify_report.json`

### Current Baseline Results

The current best baseline is `swin_t`, selected by `test_macro_f1`.

| Model | Test Acc | Test Macro-F1 | Best Val Macro-F1 |
| --- | ---: | ---: | ---: |
| `swin_t` | 0.9420 | 0.9427 | 0.9496 |
| `vit_b_16` | 0.9367 | 0.9378 | 0.9390 |
| `vgg16_bn` | 0.9310 | 0.9320 | 0.9375 |
| `resnet18` | 0.9297 | 0.9309 | 0.9417 |

Each baseline directory follows the same structure:

- `outputs/baseline/<model_key>/best.pt`
- `outputs/baseline/<model_key>/config.json`
- `outputs/baseline/<model_key>/train_history.json`
- `outputs/baseline/<model_key>/metrics.json`
- `outputs/baseline/<model_key>/test_report.json`
- `outputs/baseline/<model_key>/confusion_matrix.csv`
- `outputs/baseline/summary/`

### Current XAI Result Snapshot

The saved XAI results currently use `swin_t` as the baseline model.

| Method | Variant | Samples | Mean Explain Time (s) | Deletion AOPC | Insertion AUC | Comprehensiveness | Sufficiency | Sparsity |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `lime` | `post-hoc` | 100 | 3.7599 | 0.4970 | 0.7251 | 0.1132 | 0.0720 | 0.9131 |
| `shap` | `post-hoc` | 100 | 20.2129 | 0.6057 | 0.6741 | 0.2167 | 0.6449 | 0.8642 |
| `protopnet` | `intrinsic` | 3000 | 0.0072 | 0.1640 | 0.5485 | 0.0308 | 0.0799 | 0.8282 |
| `vibi_teacher_student` | `learned` | 3000 | 0.0127 | 0.2472 | 0.6604 | 0.0833 | 0.0974 | 0.8367 |
| `vibi_e2e` | `end-to-end` | 3000 | 0.0067 | 0.2897 | 0.7463 | 0.0513 | 0.3203 | 0.8367 |

Notes:

- `Deletion AOPC` and `Comprehensiveness` are usually better when larger.
- `Sufficiency` is usually better when smaller.
- Higher `Sparsity` means a sparser explanation.
- `LIME` and `SHAP` were evaluated on 100 samples in the current snapshot.
- `ProtoPNet`, `VIBI Teacher-Student`, and `End-to-End VIBI` were evaluated on 3000 test samples.

Typical XAI outputs include:

- `per_sample_metrics.csv`
- `quant_metrics.json`
- `visuals/`

The consolidated XAI metrics table generated by `notebook/xai/compare_xai.ipynb` is written to:

- `outputs/xai/summary/xai_metrics_all.csv`

Some methods also save checkpoints:

- `outputs/xai/intrinsic/protopnet/swin_t/protopnet_best.pt`
- `outputs/xai/end-to-end/vibi_e2e/swin_t/tb_vibi_best.pt`

## Recommended Reproduction Order

Use Jupyter Notebook or VS Code Notebook and run the notebooks from top to bottom in the following order:

1. `notebook/data_prep.ipynb`
2. `notebook/baseline/baseline_resnet.ipynb`
3. `notebook/baseline/baseline_vgg.ipynb`
4. `notebook/baseline/baseline_vit.ipynb`
5. `notebook/baseline/baseline_swin.ipynb`
6. `notebook/baseline/compare_baselines.ipynb`
7. Run one or more XAI notebooks
8. `notebook/xai/compare_xai.ipynb`

If you want to stay aligned with the current output snapshot, use `swin_t` as the baseline key.

## Environment

The repository dependencies are:

- Python 3.10+
- `numpy`
- `pandas`
- `Pillow`
- `scikit-learn`
- `torch`
- `torchvision`
- `PyYAML`
- `lime`
- `shap`
- `scikit-image`

A GPU-enabled environment is recommended for baseline training and XAI experiments, especially for `SHAP` and the `VIBI` variants.
