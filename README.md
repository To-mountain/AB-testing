# A/B 测试分析：网页新版效果评估

一个端到端 A/B 测试分析项目：对 5000 名用户的分流实验数据，完成数据质量校验、实验有效性检验（SRM / AA）、指标对比、统计显著性检验与置信区间、功效分析、分层分析及上线决策建议。

## 关键结果

| 指标 | 对照组 A | 实验组 B | 增量（B − A） | 显著性 |
| --- | --- | --- | --- | --- |
| 转化率（主指标） | 5.40% | 14.07% | +8.67pp（相对 +160.6%） | p = 3.99e-25，显著 |
| 平均停留时长 | 241.73 秒 | 243.30 秒 | +1.57 秒（+0.65%） | p = 0.639，不显著 |
| 浏览深度（PV/UV） | 7.582 | 7.493 | -1.17% | p = 0.425，不显著 |

**结论**：实验组版本显著提升转化率（95% CI 为 [+7.04pp, +10.30pp]），且未损害体验类指标，各设备/地区子组方向一致，建议灰度放量上线。注意：效应量异常大，上线前建议对分流与埋点口径做一次复核。

## 数据说明

数据来源：Kaggle 公开数据集 [AB Testing Practice](https://www.kaggle.com/datasets/adarsh0806/ab-testing-practice)（本仓库仅将其用于学习与演示，版权归原作者所有）。

`ab_testing.csv`（5000 行 x 7 列，每行一个用户）：

| 字段 | 说明 |
| --- | --- |
| User ID | 用户唯一标识 |
| Group | 分组：A = 对照组（旧版页面），B = 实验组（新版页面） |
| Page Views | 页面浏览量 |
| Time Spent | 停留时长（秒） |
| Conversion | 是否转化（Yes / No） |
| Device | 设备类型（Desktop / Mobile） |
| Location | 地区（Scotland / England / Northern Ireland / Wales） |

## 分析流程

notebook 按标准 A/B 测试分析流程组织：

1. **数据质量检查** — 缺失值、重复行、用户唯一性
2. **实验有效性校验** — SRM 样本比例失配检验（卡方）+ AA 协变量平衡检验（Device / Location 卡方独立性检验）
3. **指标对比** — 转化率 / 平均停留时长 / 浏览深度，增量统一按 `实验组 B − 对照组 A` 计算
4. **统计显著性检验** — 转化率采用双比例 z 检验，连续指标采用 Welch t 检验与 Mann-Whitney U 检验，并给出差异的 95% 置信区间
5. **功效分析** — 当前样本量下的统计功效与最小可检出效应（MDE）
6. **分层分析** — Device / Location 子组转化率对比
7. **可视化** — 转化率对比（含 95% CI）、停留时长分布对比
8. **结论与行动建议** — 完整结论文档

## 主要发现

- **分流健康**：SRM 检验 p = 0.591；AA 平衡检验 Device p = 0.053、Location p = 0.250，随机化有效
- **主指标显著提升**：转化率 5.40% → 14.07%，绝对提升 +8.67pp，z = 10.35，p = 3.99e-25，95% CI 完全在 0 之上
- **体验指标无恶化**：停留时长、浏览深度差异均不显著，提升并非以用户体验为代价
- **分层结论一致**：Desktop（+8.0pp）/ Mobile（+9.3pp）及各地区（+6.4pp ~ +10.4pp）子组均一致提升，无 Simpson 反转
- **功效充足**：检测该效应的功效约为 1.0；80% 功效下 MDE 约为 1.78pp

## 项目结构

```
.
├── AB-Testing.ipynb        # 完整分析 notebook（含运行输出与结论文档）
├── ab_testing.csv          # 实验数据（5000 用户，来源 Kaggle）
├── requirements-lock.txt   # 依赖版本锁定（完整环境）
├── LICENSE                 # MIT License
└── README.md
```

## 环境与运行

Python 3.10+，核心依赖：

```bash
pip install pandas numpy scipy statsmodels matplotlib jupyter
jupyter notebook AB-Testing.ipynb
```

或使用完整锁定环境：

```bash
pip install -r requirements-lock.txt
```

## 方法论备注

- 主指标为转化率，护栏关注停留时长与浏览深度
- 所有"提升/增量"均以 `实验组 B − 对照组 A` 计算，避免方向歧义
- 连续型指标同时给出参数检验（Welch t）与非参数检验（Mann-Whitney U），以应对偏态分布
- 显著性水平 alpha = 0.05（双尾）

## License

本项目的代码采用 [MIT License](LICENSE) 开源。数据来源为 Kaggle 公开数据集（版权归原作者所有，仅用于学习与研究）。
