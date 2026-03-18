# Graduation Thesis

## 项目简介

本项目是一个面向毕业论文的图像分类与可解释人工智能（XAI）实验仓库。整体流程分为三部分：

1. 数据准备与去偏差处理
2. 基线分类模型训练与比较
3. 基于最佳基线模型的多种 XAI 方法评估

当前项目围绕一个 6 类场景图像数据集展开，类别为：

- `buildings`
- `forest`
- `glacier`
- `mountain`
- `sea`
- `street`

## 仓库结构

| 路径 | 说明 |
| --- | --- |
| `notebook/` | 主实验 notebook，包括数据准备、baseline 和 XAI |
| `dataset/` | 原始数据与数据副本 |
| `outputs/` | 已生成的实验输出快照 |

## Notebook 组织

### 1. 数据准备

| Notebook | 作用 |
| --- | --- |
| `notebook/data_prep.ipynb` | 生成 manifest、重复检测、统一 train/val/test split、预处理配置与完整性校验 |

### 2. Baseline 分类模型

| Notebook | 模型 |
| --- | --- |
| `notebook/baseline/baseline_resnet.ipynb` | ResNet18 |
| `notebook/baseline/baseline_vgg.ipynb` | VGG16-BN |
| `notebook/baseline/baseline_vit.ipynb` | Vanilla ViT (`vit_b_16`) |
| `notebook/baseline/baseline_swin.ipynb` | Swin Transformer (`swin_t`) |
| `notebook/baseline/compare_baselines.ipynb` | 汇总四个 baseline 的指标并选出最佳模型 |

### 3. XAI 方法

| Notebook | 方法类别 | 方法名 |
| --- | --- | --- |
| `notebook/xai/post-hoc/xai_lime.ipynb` | Post-hoc | LIME |
| `notebook/xai/post-hoc/xai_shap.ipynb` | Post-hoc | SHAP |
| `notebook/xai/intrinsic/xai_protopnet.ipynb` | Intrinsic | ProtoPNet |
| `notebook/xai/learned/xai_vibi_teacher_student.ipynb` | Learned explainer | VIBI Teacher-Student |
| `notebook/xai/end-to-end/xai_vibi_e2e.ipynb` | End-to-end | TB-VIBI / End-to-End VIBI |
| `notebook/xai/compare_xai.ipynb` | Summary | 汇总并比较所有已保存的 XAI 指标 |

## 数据集说明

当前数据目录采用典型的场景分类数据组织方式：

| 路径 | 说明 |
| --- | --- |
| `dataset/archive/seg_train/seg_train/<class>/` | 训练图像 |
| `dataset/archive/seg_test/seg_test/<class>/` | 测试图像 |
| `dataset/archive/seg_pred/seg_pred/` | 额外预测图像，当前主流程未使用 |

数据集来源：

- Kaggle：`https://www.kaggle.com/datasets/puneet6060/intel-image-classification`
- 数据集名称：`Intel Image Classification`
- 本仓库默认你已经将下载并解压后的数据放在 `dataset/archive/` 下。

根据当前 `split_all.csv`，已经生成的统一数据划分统计如下：

| Split | 样本数 |
| --- | ---: |
| `train` | 12,629 |
| `val` | 1,402 |
| `test` | 3,000 |
| 总计 | 17,031 |

训练集类别分布：

| 类别 | 训练样本数 |
| --- | ---: |
| `buildings` | 1,971 |
| `forest` | 2,044 |
| `glacier` | 2,164 |
| `mountain` | 2,260 |
| `sea` | 2,047 |
| `street` | 2,143 |

## Output 目录说明

当前仓库中的实验输出快照位于 `outputs/` 下，主要分为三部分：

| 路径 | 内容 |
| --- | --- |
| `outputs/data_prep/` | 数据清单、去重报告、split 文件、预处理配置、验证报告 |
| `outputs/baseline/` | 四个 baseline 的模型权重、配置、训练历史与评估结果 |
| `outputs/xai/` | 五种 XAI 方法的量化评估、逐样本结果、解释图与部分方法的 checkpoint |

### `data_prep` 产物

`outputs/data_prep/` 当前包含：

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

### Baseline 当前结果

当前最佳 baseline 为 `swin_t`，依据指标为 `test_macro_f1`。

| 模型 | Test Acc | Test Macro-F1 | Best Val Macro-F1 |
| --- | ---: | ---: | ---: |
| `swin_t` | 0.9420 | 0.9427 | 0.9496 |
| `vit_b_16` | 0.9367 | 0.9378 | 0.9390 |
| `vgg16_bn` | 0.9310 | 0.9320 | 0.9375 |
| `resnet18` | 0.9297 | 0.9309 | 0.9417 |

对应输出目录结构为：

- `outputs/baseline/<model_key>/best.pt`
- `outputs/baseline/<model_key>/config.json`
- `outputs/baseline/<model_key>/train_history.json`
- `outputs/baseline/<model_key>/metrics.json`
- `outputs/baseline/<model_key>/test_report.json`
- `outputs/baseline/<model_key>/confusion_matrix.csv`
- `outputs/baseline/summary/`

### XAI 当前结果快照

当前已保存的 XAI 结果均基于 `swin_t` baseline。

| 方法 | 变体 | 样本数 | 平均解释时间(s) | Deletion AOPC | Insertion AUC | Comprehensiveness | Sufficiency | Sparsity |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `lime` | `post-hoc` | 100 | 3.7599 | 0.4970 | 0.7251 | 0.1132 | 0.0720 | 0.9131 |
| `shap` | `post-hoc` | 100 | 20.2129 | 0.6057 | 0.6741 | 0.2167 | 0.6449 | 0.8642 |
| `protopnet` | `intrinsic` | 3000 | 0.0072 | 0.1640 | 0.5485 | 0.0308 | 0.0799 | 0.8282 |
| `vibi_teacher_student` | `learned` | 3000 | 0.0127 | 0.2472 | 0.6604 | 0.0833 | 0.0974 | 0.8367 |
| `vibi_e2e` | `end-to-end` | 3000 | 0.0067 | 0.2897 | 0.7463 | 0.0513 | 0.3203 | 0.8367 |

说明：

- `Deletion AOPC` 和 `Comprehensiveness` 越大通常越好。
- `Sufficiency` 越小通常越好。
- `Sparsity` 越大表示解释更稀疏。
- `LIME` 和 `SHAP` 当前结果基于 100 个样本。
- `ProtoPNet`、`VIBI Teacher-Student` 和 `End-to-End VIBI` 当前结果基于 3000 个测试样本。

典型 XAI 输出包括：

- `per_sample_metrics.csv`
- `quant_metrics.json`
- `visuals/`

由 `notebook/xai/compare_xai.ipynb` 生成的 XAI 总表位于：

- `outputs/xai/summary/xai_metrics_all.csv`

部分方法还包含模型或解释器 checkpoint：

- `outputs/xai/intrinsic/protopnet/swin_t/protopnet_best.pt`
- `outputs/xai/end-to-end/vibi_e2e/swin_t/tb_vibi_best.pt`

## 推荐复现顺序

建议使用 Jupyter Notebook 或 VS Code Notebook 环境，按以下顺序从上到下运行：

1. `notebook/data_prep.ipynb`
2. `notebook/baseline/baseline_resnet.ipynb`
3. `notebook/baseline/baseline_vgg.ipynb`
4. `notebook/baseline/baseline_vit.ipynb`
5. `notebook/baseline/baseline_swin.ipynb`
6. `notebook/baseline/compare_baselines.ipynb`
7. 选择 XAI notebook 进行解释方法实验
8. `notebook/xai/compare_xai.ipynb`

如果希望与当前输出快照保持一致，优先使用 `swin_t` 作为 baseline key。

## 环境依赖

项目中使用到的核心依赖包括：

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

建议在具备 GPU 的环境中运行 baseline 和 XAI notebook，尤其是 `SHAP` 与 `VIBI` 相关实验。
