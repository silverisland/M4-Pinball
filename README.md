# M4-Pinball

M4 PatchTST 的独立分位预测模型：9 个分位输出、容量归一化 Pinball loss，以及推理时的南网指标选点。

默认前 16 步使用 `p_star_l1`，其余使用 `p_star`；非负与容量限制开启，晴空裁剪和夜间归零默认关闭。
训练使用 Pinball loss；选模和早停使用决策后的南网短期 D+1 评分（越高越好），与 M8 默认口径一致。

## 安装与验证

建议 Python 3.10–3.12；本地已在 Python 3.12 / Torch 2.5.1 / Transformers 4.40.1 下验证。

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
python -m pytest forecast_backbones/m4_pinball/tests -q
```

GPU 环境请先按对应 CUDA 版本安装 PyTorch。本仓库不附带训练数据、模型权重或凭据。

## 训练

```bash
python -m forecast_backbones.m4_pinball.train \
  --config /path/to/overrides.yaml \
  --train /path/to/train.parquet \
  --valid /path/to/valid.parquet \
  --out runs/m4_pinball/exp1
```

默认固定容量列 `cap_mw`，外部预测列 `chronos`，按真实数据在配置中调整。
也支持 `dataset.capacity_by_station` 固定站点容量表；不使用未来标签估算容量。

[配置、数据格式、推理和完整使用说明](forecast_backbones/m4_pinball/README.md)

## 代码来源与范围

本仓库从本地 NextGenPVForecast 的 `80af0be` 版本导出，仅包含 `m4_pinball` 和运行所需依赖，不携带原仓库 Git 历史。
`forecast_backbones/m4_pinball/layers` 自带 PatchTST、RevIN 和输入整理实现，不依赖 `m4` 模型包；
`interfaces` 和 `pipelines` 是共享框架接口。
`tests/m8_decision_reference.py` 仅用于验证决策数值与原 M8 实现一致，不参与训练或推理。
不包含其他实验、业务数据或原 M4 点预测模型的训练代码。
