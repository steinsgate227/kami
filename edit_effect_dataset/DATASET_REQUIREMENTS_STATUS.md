# Dataset Requirements Status

更新时间：2026-06-26

## 结论

- 主数据集已满足 no-leakage schema、config-isolated split、pattern-isolated split、任务/negative tier 分层统计。
- 旧主数据集的 QoR 子集仍偏容易：`edit_only`、`shuffled A_result` 与真实 `A_result` 差距很小，说明旧 QoR 里模型仍可能靠 edit/config pattern 取巧。
- 新增的 A_result-targeted causal 小数据集已经开始证明综合反馈有用：`edit_only` 明显下降，真实 `A_result + edit` 明显高于 `shuffled A_result + edit`。
- 当前最薄弱点：targeted causal 样本量还小，QoR optimization 需要继续补到 3000–5000 条，结构特征仍太粗，多步 trajectory 还需要同一状态分叉。

## 主数据集

| 项目 | 数量 |
|---|---:|
| all pairs | 14,747 |
| train pairs | 11,464 |
| test pairs | 3,283 |
| unique A | 399 |
| unique edit patterns | 2,290 |
| train/test A_config overlap | 0 |
| train/test edit_pattern overlap | 0 |

| 任务 | 数量 |
|---|---:|
| feasibility_repair | 11,043 |
| qor_optimization | 3,704 |

| Negative tier | 数量 |
|---|---:|
| easy_negative | 3,820 |
| hard_negative | 5,582 |
| medium_negative | 5,345 |

## QoR-only 评估

QoR-only 只看 `SUCCESS -> better SUCCESS`，用于检查模型是不是能做真实性能优化。

| 项目 | 数量 |
|---|---:|
| QoR pairs | 3,704 |
| unique A | 82 |
| unique edit patterns | 656 |
| train/test A_config overlap | 0 |

| Feature set | Accuracy | Precision | Recall | F1 | AUC | Confusion matrix |
|---|---:|---:|---:|---:|---:|---|
| edit_only | 0.9757 | 0.9881 | 0.9622 | 0.9750 | 0.9851 | tn=353 fp=4 fn=13 tp=331 |
| a_config_edit | 0.9743 | 0.9822 | 0.9651 | 0.9736 | 0.9862 | tn=351 fp=6 fn=12 tp=332 |
| a_result_edit | 0.9772 | 0.9881 | 0.9651 | 0.9765 | 0.9867 | tn=353 fp=4 fn=12 tp=332 |
| shuffled_a_result | 0.9772 | 0.9881 | 0.9651 | 0.9765 | 0.9868 | tn=353 fp=4 fn=12 tp=332 |
| edit_structure | 0.9772 | 0.9881 | 0.9651 | 0.9765 | 0.9855 | tn=353 fp=4 fn=12 tp=332 |
| full | 0.9729 | 0.9822 | 0.9622 | 0.9721 | 0.9855 | tn=351 fp=6 fn=13 tp=331 |
| full_no_structure | 0.9757 | 0.9852 | 0.9651 | 0.9750 | 0.9862 | tn=352 fp=5 fn=12 tp=332 |

解释：QoR 旧数据中 `edit_only=0.9757`、`a_result_edit=0.9772`、`shuffled_a_result=0.9772`，差距几乎没有；这说明旧 QoR 样本还不能证明模型真的依赖综合反馈。

## A_result-targeted causal 数据集

| 项目 | 数量 |
|---|---:|
| examples | 117 |
| train | 88 |
| test | 29 |
| positive | 59 |
| negative | 58 |
| valid contrast templates | 19 |
| no-leakage passed | True |
| no-leakage violations | 0 |

| Feature set | Accuracy | Precision | Recall | F1 | AUC | Confusion matrix |
|---|---:|---:|---:|---:|---:|---|
| edit_only | 0.5862 | 0.9167 | 0.5000 | 0.6471 | 0.6364 | tn=6 fp=1 fn=11 tp=11 |
| a_config_edit | 0.9310 | 1.0000 | 0.9091 | 0.9524 | 0.9610 | tn=7 fp=0 fn=2 tp=20 |
| a_result_edit | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 | tn=7 fp=0 fn=0 tp=22 |
| shuffled_a_result_edit | 0.4828 | 0.8182 | 0.4091 | 0.5455 | 0.6688 | tn=5 fp=2 fn=13 tp=9 |
| structure_edit | 0.5862 | 0.9167 | 0.5000 | 0.6471 | 0.6364 | tn=6 fp=1 fn=11 tp=11 |
| full | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 | tn=7 fp=0 fn=0 tp=22 |
| majority baseline | 0.2414 | 0.0000 | 0.0000 | 0.0000 | 0.5000 | tn=7 fp=0 fn=22 tp=0 |

解释：targeted causal 数据的方向是对的：同一类 edit 在不同 A_result 压力下标签会翻转。`edit_only` 明显低，`shuffled A_result` 明显低，真实 `A_result + edit` 最高。

## Targeted synthesis 进展

| 项目 | 数量 |
|---|---:|
| candidates | 340 |
| completed synthesis | 178 |
| remaining | 162 |

| 状态 | 数量 |
|---|---:|
| SUCCESS | 81 |
| EXCEEDED | 59 |
| TIMEOUT | 38 |

## A_result-dependent QoR 扩展

目标：补强 `SUCCESS -> better SUCCESS`，并让同一类 edit 在不同 `A_result` 压力下可能产生不同 label，避免模型只背 edit pattern。

| 项目 | 数量 |
|---|---:|
| 新 QoR candidates | 642 |
| checked variants | 3,487 |
| skipped existing signatures | 2,018 |
| skipped duplicate candidate signatures | 827 |
| existing unique signatures used for dedup | 1,097 |

候选分布：

| Kernel | Candidates |
|---|---:|
| atax | 120 |
| bicg | 120 |
| gemm | 182 |
| gemm_blocked | 220 |

已综合新 QoR candidates：

| 项目 | 数量 |
|---|---:|
| results | 11 |
| SUCCESS | 11 |
| unique config signatures | 11 |
| duplicate config signatures | 0 |

说明：目前新跑的 11 条都是 `atax/resource_room`，有些综合结果 latency/resource 相同，但配置 signature 不重复；结果相同不等于配置重复。

## 已满足 / 未满足

| 要求 | 状态 | 说明 |
|---|---|---|
| no-leakage schema | 已满足 | 公开样本只含 `A_config`、`A_result`、candidate config/edit、label；target/B result 只进 private audit。 |
| config-isolated split | 已满足 | train/test 不共享 `A_config`。 |
| pattern-isolated split | 已满足 | train/test 不共享 edit pattern。 |
| leave-one-kernel-out | 已有 | 已在 strict eval 中做过；GEMM 泛化仍是难点。 |
| feasibility vs QoR 分开评估 | 部分满足 | QoR-only 已单独评估；feasibility 仍需要在最新报告里补同格式表。 |
| hard negative 三档 | 已满足 | easy/medium/hard 已分层，QoR-only 也分 tier 评估。 |
| A_result 依赖证明 | 初步满足 | targeted causal 子集证明有效，但样本量还小。 |
| 结构特征证明 | 未满足 | 当前结构特征没有显著提升，需要解析 loop/access/path。 |
| multi-step 真实反馈 | 未满足 | trajectory 有了，但仍偏容易；需要同一状态分叉。 |

## 下一步

1. 继续生成非重复 targeted QoR candidates，优先 `gemm/gemm_blocked` 的 SUCCESS seed。
2. 构造同一 edit 在不同 `A_result` 下 label 相反的 QoR 样本，而不是只靠 feasibility/overflow。
3. 增强结构解析：loop nest、tripcount、target loop 层级、array access path、unroll/partition 是否同访存路径。
4. 扩展 multi-step trajectory 为同一 A_state 下 good/bad 分叉，并比较 real feedback vs shuffled feedback。
5. 对 GEMM/gemm_blocked 做 false positive、false negative、高置信错误样本分析。

## 核心文件

- `all_edit_preferences.jsonl`
- `train_edit_preferences.jsonl`
- `test_edit_preferences.jsonl`
- `a_result_targeted_causal_examples.jsonl`
- `a_result_targeted_causal_train.jsonl`
- `a_result_targeted_causal_test.jsonl`
- `a_result_targeted_causal_audit_private.jsonl`
- `a_result_targeted_causal_eval_report.json`
- `qor_only_ablation_report.json`
- `DATASET_REQUIREMENTS_STATUS.md`
