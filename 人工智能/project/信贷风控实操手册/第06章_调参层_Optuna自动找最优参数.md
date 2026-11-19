# 第 06 章　调参层：Optuna 自动找最优参数

> 本章建 1 个文件 + 回填 1 个 · 预计 1~1.5 小时 · 难度 ★★☆ · 前置：第 05 章

---

## 1. 这一章在流水线上的位置

```
[原始CSV] →①数据层→ ②特征层→ ③模型层→ ④调参→ ⑤解释→ ⑥AI报告→ ⑦编排→ ⑧脚本→ ⑨网页
                                    ▲
你在这里：第 06 章　调参层
已经有的：能训练的三个模型
还没有的：解释、报告、网页
```

---

## 2. 本章你要做什么

一句话：**让程序自己试 50 组超参，挑出验证集 AUC 最高的那组。**

你不再需要手动改 `max_depth` 试半天。

---

## 3. 开工前检查

- [ ] 上一章输出的是 `models OK`
- [ ] `model_config.yaml` 里 `xgboost.tuning.search_space` 有 7 个参数
- [ ] `src/tuning/__init__.py` 存在且是**空的**

---

## 4. 本章知识点

| 编号 | 概念 | 一句话解释 | 为什么这里需要它 | 本项目用在哪 | 要求 |
|---|---|---|---|---|---|
| KP-AL-10 | 超参数 | 训练前人为设定的参数（树深、学习率） | 它们决定模型上限，但没法从数据里学 | `search_space` | 必懂 |
| KP-AL-11 | 网格搜索 vs 贝叶斯优化 | 前者穷举，后者根据历史结果猜下一个 | 50 维空间穷举是天文数字 | `TPESampler` | 必懂 |
| KP-AL-12 | TPE | 用历史 trial 建概率模型，挑最可能好的点 | Optuna 默认采样器 | `TPESampler` | 知道即可 |
| KP-AL-13 | 剪枝（Pruning） | 中途发现没希望就提前停 | 省一半以上时间 | `MedianPruner` | 知道即可 |
| KP-AL-14 | study / trial | 一次调参 = study，一次尝试 = trial | Optuna 的核心概念 | `create_study` | 必懂 |
| KP-EN-13 | 目标函数 | 输入一组超参、输出一个分数的函数 | Optuna 靠它判断好坏 | `objective` | 必懂 |

### 展开讲：网格搜索为什么不可行

假设有 7 个超参，每个取 10 个候选值：

```
组合数 = 10^7 = 1000 万种
每次训练 30 秒 → 需要 95 年
```

贝叶斯优化的思路不同：先随机试 10 组，观察"哪类参数组合分数高"（比如学习率 0.03~0.08 区间普遍好），然后**把下一次尝试集中在这个区域**。50 次就能找到接近最优的组合。

```
随机/网格：●●●●●●●●●● 均匀铺开，浪费
TPE：      ●● → 发现左边好 → ●●●●●● 集中左边
```

---

## 5. 分步操作

### 步骤 6.1　写 `src/tuning/optimizer.py`

**【目标】** 通用优化器：传入模型类和搜索空间，返回最佳参数。

**【文件】** `src/tuning/optimizer.py`（新建）

**【完整代码】**

```python
"""
Optuna 超参优化
===============
在配置定义的搜索空间中，用 TPE 算法寻找验证集表现最好的超参组合。
"""
from __future__ import annotations

from typing import Any, Callable, Dict, Optional

import optuna
import pandas as pd
from optuna.pruners import MedianPruner
from optuna.samplers import TPESampler
from sklearn.metrics import roc_auc_score

from src.utils.logger import get_logger

logger = get_logger("tuning.optuna")
optuna.logging.set_verbosity(optuna.logging.WARNING)


class OptunaOptimizer:
    """超参优化器。构造只收一个 tuning 配置块。"""

    def __init__(self, config):
        # config 传的是 configs.get("model").xgboost.tuning
        self.config = config
        self.search_space = config.search_space
        self.n_trials = config.n_trials
        self.timeout = config.timeout
        self.direction = config.direction
        self.metric = config.metric

        self.study: Optional[optuna.Study] = None
        self.best_params: Optional[Dict] = None

    def optimize(
        self,
        model_class: Callable,
        X_train: pd.DataFrame,
        y_train: pd.Series,
        X_val: pd.DataFrame,
        y_val: pd.Series,
        base_params: Optional[Dict] = None,
        custom_objective: Optional[Callable] = None,
    ) -> Dict[str, Any]:
        """
        参数:
            model_class: 模型构造函数，如 xgb.XGBClassifier
            base_params: 不参与优化、但每次都要带上的参数
        返回:
            最佳超参字典
        """
        base_params = base_params or {}

        objective_func = custom_objective or self._build_default_objective(
            model_class, X_train, y_train, X_val, y_val, base_params
        )

        self.study = optuna.create_study(
            direction=self.direction,
            sampler=TPESampler(seed=42),
            pruner=MedianPruner(n_startup_trials=5, n_warmup_steps=10),
            study_name=f"opt_{getattr(model_class, '__name__', 'model')}",
        )

        logger.info(
            f"开始超参优化 - n_trials={self.n_trials}, timeout={self.timeout}s, "
            f"direction={self.direction}"
        )

        self.study.optimize(
            objective_func,
            n_trials=self.n_trials,
            timeout=self.timeout,
            show_progress_bar=False,
        )

        self.best_params = dict(self.study.best_params)
        logger.info(
            f"优化完成 - 最佳 {self.metric}: {self.study.best_value:.4f}, "
            f"参数: {self.best_params}"
        )
        return self.best_params

    def _build_default_objective(
        self,
        model_class: Callable,
        X_train, y_train, X_val, y_val,
        base_params: Dict,
    ) -> Callable:
        """默认目标函数：采样 -> 训练 -> 验证集 AUC。"""

        def objective(trial: optuna.Trial) -> float:
            params = {}
            for name, cfg in self.search_space.items():
                params[name] = self._suggest_param(trial, name, cfg)

            full_params = {**base_params, **params}

            try:
                model = model_class(**full_params)
                try:
                    model.fit(
                        X_train, y_train,
                        eval_set=[(X_val, y_val)],
                        verbose=False,
                    )
                except TypeError:
                    model.fit(X_train, y_train)

                val_pred = model.predict_proba(X_val)
                if val_pred.ndim == 2:
                    val_pred = val_pred[:, 1]
                return float(roc_auc_score(y_val, val_pred))
            except Exception as e:
                logger.warning(f"Trial 失败: {e}")
                return 0.0

        return objective

    def _suggest_param(self, trial: optuna.Trial, name: str, cfg: Dict):
        """把 YAML 里的类型翻译成 Optuna 的采样调用。"""
        ptype = cfg.get("type", "float")

        if ptype == "int":
            return trial.suggest_int(name, cfg["low"], cfg["high"])
        if ptype == "float":
            return trial.suggest_float(
                name, cfg["low"], cfg["high"], log=cfg.get("log", False)
            )
        if ptype == "categorical":
            return trial.suggest_categorical(name, cfg["choices"])
        raise ValueError(f"不支持的参数类型: {ptype}")

    def get_history_df(self) -> pd.DataFrame:
        """所有 trial 的记录，Web 调参页画收敛曲线用。"""
        if self.study is None:
            return pd.DataFrame()
        return self.study.trials_dataframe()

    def get_param_importance(self) -> Dict[str, float]:
        """哪些超参对结果影响最大。"""
        if self.study is None:
            return {}
        try:
            return optuna.importance.get_param_importances(self.study)
        except Exception as e:
            logger.warning(f"计算参数重要性失败: {e}")
            return {}
```

**【逐块讲解】**

| 代码段 | 要点 |
|---|---|
| `config.search_space` | 直接来自 YAML，加参数不用改代码 |
| `TPESampler(seed=42)` | 固定种子，同一份数据两次调参结果一致 |
| `MedianPruner` | 想提前砍掉差的 trial（见 W-04 的限制说明） |
| `objective` 里的 `try/except` | 某组参数不合法（比如 `max_depth` 太大爆内存）时返回 0，不中断整个 study |
| `log=cfg.get("log", False)` | `learning_rate` 用对数采样，因为 0.01 到 0.3 按线性采会集中在高值区 |
| `trials_dataframe()` | 直接给 DataFrame，方便在网页上画图 |

**【验证】** 用小规模跑一次（2 个 trial、小数据）：

```bat
python -c "import numpy as np,pandas as pd; from src.utils import ConfigLoader; from src.tuning.optimizer import OptunaOptimizer; from xgboost import XGBClassifier; c=ConfigLoader('./configs').get('model').xgboost.tuning; c.n_trials=2; c.timeout=60; X=pd.DataFrame(np.random.randn(600,6),columns=list('abcdef')); y=pd.Series(np.random.binomial(1,0.2,600)); o=OptunaOptimizer(c); p=o.optimize(XGBClassifier,X[:400],y[:400],X[400:],y[400:],base_params={'n_estimators':50,'objective':'binary:logistic','eval_metric':'auc','tree_method':'hist'}); print('best:',p)"
```

期望：打印 `best: {...}`，含 `max_depth`、`learning_rate` 等键。

**【常见报错】**

| 报错 | 原因 | 修复 |
|---|---|---|
| `TypeError: OptunaOptimizer.__init__() takes 2 positional arguments` | 传了多个配置 | 只传 `tuning` 这一个块 |
| `AttributeError: 'AttrDict' object has no attribute 'choices'` | 用了 `categorical` 类型但没写 `choices` | 补 `choices: [...]` |
| 所有 trial 分数都是 0.0 | `objective` 里异常被吞了 | 把 `logger.warning` 改成 `logger.exception` 看堆栈 |
| 调参跑了一小时还没完 | `n_trials` 太大 + 数据量大 | 先设 `n_trials: 10` |

---

### 步骤 6.2　回填仪式：`src/tuning/__init__.py`

```python
"""超参优化模块"""
from src.tuning.optimizer import OptunaOptimizer

__all__ = ["OptunaOptimizer"]
```

**【验证】**

```bat
python -c "from src.tuning import OptunaOptimizer; print('tuning OK')"
```

---

## 6. 本章错误对照

### 错误 E-11　把整个 `model` 配置传给 `OptunaOptimizer`

**出现在**：原文档的调用示例

**❌ 原文写法**

```python
optimizer = OptunaOptimizer(configs.get("model"))
```

**✅ 修正写法**

```python
optimizer = OptunaOptimizer(configs.get("model").xgboost.tuning)
```

**💥 不修的后果**

```
AttributeError: 'AttrDict' object has no attribute 'search_space'
```

**🧠 为什么**　`OptunaOptimizer.__init__` 要读 `search_space`、`n_trials`、`timeout`、`direction`、`metric`，这五个字段都在 `tuning` 子块下面。传 `model` 进去，它找不到这些键。

**✅ 自查**　构造前先 `print(configs.get("model").xgboost.tuning.keys())`，能看到这五个键就对了。

---

### 错误 W-04　剪枝器装了，但其实没生效

**现象**　配置里设了 `MedianPruner`，但每个 trial 都完整跑完，没见哪个被提前终止。

**🧠 为什么**　剪枝要求目标函数**主动汇报中间进度**：

```python
for step in range(n_epochs):
    ...
    trial.report(intermediate_score, step)
    if trial.should_prune():
        raise optuna.TrialPruned()
```

本项目的目标函数是"训完整个模型 → 返回一个分数"，中间没有汇报点，所以剪枝器无事可做。

**✅ 怎么办**　不影响正确性，只是省不了时间。想要真剪枝，得把训练循环拆开、按 epoch 汇报。教学项目不必做，知道这个机制存在即可。**答辩被问到"剪枝怎么生效"时，说清这一点是加分项。**

---

## 7. 常见坑

| 现象 | 原因 | 解决 |
|---|---|---|
| 调参出来的最优参数还不如默认值 | 验证集太小，噪声大 | 增大验证集比例 |
| 每次调参结果都不一样 | 没固定 sampler 种子 / 数据切分变了 | 已设 `seed=42`；确认切分 `random_state` 固定 |
| `n_trials=50` 跑了 3 小时 | 每 trial 都在全量数据上训练 | 调参时用子样本（10 万行足够） |
| 最优参数落在搜索边界 | 真实最优可能在范围外 | 扩展 `low/high` |

---

## 8. 本章自测

**1. 为什么 `learning_rate` 的搜索要开 `log: true`？**

<details><summary>点开看答案</summary>

学习率的有效区间是跨数量级的（0.001~0.3）。线性采样会集中在 0.1~0.3 这段，小值区几乎采不到。对数采样让 0.001~0.01 和 0.1~0.3 获得相同的采样密度。

</details>

**2. 调参时能不能用测试集当验证集？**

<details><summary>点开看答案</summary>

不能。用测试集挑参数，等于让模型间接见过测试集，最终报告的成绩就是虚高的。必须用验证集调参，测试集只在最后用一次。

</details>

**3. `trial` 和 `study` 的关系？**

<details><summary>点开看答案</summary>

一个 study 是一次完整的调参任务（比如"给 XGBoost 找最优参数"）；一个 trial 是其中一次尝试（一组具体参数 + 一次训练 + 一个分数）。

</details>

---

## 9. 本章产物清单 + 下一步

**产物**

| 路径 | 行数约 | 导出 |
|---|---|---|
| `src/tuning/optimizer.py` | 130 | `OptunaOptimizer` |
| `src/tuning/__init__.py` | 4 | 同上 |

**一条命令自检**

```bat
python -c "from src.tuning import OptunaOptimizer; print('tuning OK')"
```

**你现在已经能做到**：让程序自动搜索超参，不再手动试。

**下一章**：解释层——用 SHAP 算清"这笔为什么被判高风险"，这是整个项目从"能预测"走向"能解释"的关键一步。
