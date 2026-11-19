# 第 05 章　模型层：XGBoost、神经网络与融合

> 本章建 4 个文件 + 回填 1 个 · 预计 4~6 小时 · 难度 ★★★★ · 前置：第 04 章

---

## 1. 这一章在流水线上的位置

```
[原始CSV] →①数据层→ ②特征层→ ③模型层→ ④调参→ ⑤解释→ ⑥AI报告→ ⑦编排→ ⑧脚本→ ⑨网页
                            ▲
你在这里：第 05 章　模型层（全项目核心）
已经有的：干净数据 + 精选特征
还没有的：调参、解释、报告、网页
```

**这是全项目最硬的一章。** 四个文件、约 800 行。做完你就能第一次训出模型、看到 AUC 数字。建议分两次做：先做 5.1~5.3（XGBoost + 融合），休息一下再做 5.4（神经网络）。

---

## 2. 本章你要做什么

| 文件 | 角色 | 一句话 |
|---|---|---|
| `base.py` | 契约 | 规定"一个模型"必须有哪 4 个方法 |
| `xgb_model.py` | 主力 | 梯度提升树，表格数据的王者 |
| `mlp_model.py` | 补充 | 神经网络，能学特征间的复杂交互 |
| `ensemble.py` | 融合 | 把两个模型的预测加权合成一个 |

**为什么要"统一接口"**：如果三个模型各写各的方法名（`train` / `fit` / `learn`），上层代码就得写三个 if。有了 `BaseModel` 之后，pipeline 里只写 `model.fit(...)`，换模型不用改调用方。

---

## 3. 开工前检查

- [ ] 上一章输出的是 `features OK`
- [ ] `src/models/__init__.py` 存在且是**空的**
- [ ] `import torch` 能成功（`python -c "import torch;print(torch.__version__)"`）

---

## 4. 本章知识点

| 编号 | 概念 | 一句话解释 | 为什么这里需要它 | 本项目用在哪 | 要求 |
|---|---|---|---|---|---|
| KP-AL-01 | 梯度提升树 | 一棵一棵树接力，每棵修正前一棵的错 | XGBoost 的本质，表格数据最强基线 | `XGBModel` | 必懂 |
| KP-AL-02 | 早停 | 验证集指标不再涨就停 | 防止白训和过拟合 | `early_stopping` | 必懂 |
| KP-AL-03 | 正则化 | 惩罚模型复杂度 | 抑制过拟合 | `reg_alpha/lambda` | 知道即可 |
| KP-AL-04 | 神经网络 / MLP | 多层线性 + 非线性激活堆叠 | 能拟合特征间复杂交互 | `MLPNetwork` | 知道即可 |
| KP-AL-05 | 反向传播 | 从损失往回算每层该调多少 | PyTorch 的 `loss.backward()` 就是它 | `_train_one_epoch` | 知道即可 |
| KP-AL-06 | 损失函数 | 衡量预测和真实差多少 | 训练就是最小化它 | `WeightedBCELoss` `FocalLoss` | 必懂 |
| KP-AL-07 | 优化器 | 按梯度更新参数的规则 | Adam 最常用 | `_build_optimizer` | 知道即可 |
| KP-AL-08 | Dropout | 训练时随机丢神经元 | 防过拟合的常用手段 | `MLPNetwork` | 知道即可 |
| KP-AL-09 | 模型融合 | 多个模型结果加权合并 | 降低方差，通常比单模型稳 | `EnsembleModel` | 必懂 |
| KP-EN-11 | 抽象基类 | 规定子类必须实现哪些方法 | 保证三个模型接口一致 | `BaseModel` | 必懂 |
| KP-EN-12 | 特征对齐 | 预测时按训练时的列顺序排 | 顺序错了结果全乱，且不报错 | `X._feature_names` | 必懂 |

### 展开讲 1：XGBoost 到底在做什么

一句话：**一群"弱"决策树排队接力，每棵专门修正前面所有树的错误。**

```
第 1 棵树：预测 0.30，真实 1.00，错 0.70
第 2 棵树：专门学"还差 0.70"，预测 0.45 → 累计 0.75
第 3 棵树：还差 0.25，预测 0.18 → 累计 0.93
...
最后把所有树的输出加起来 = 最终预测
```

为什么它在表格数据上强：

| 优势 | 说明 |
|---|---|
| 不需要标准化 | 树只看大小关系，不看绝对数值 |
| 能处理缺失 | 自动学"缺失时走哪边" |
| 能抓非线性 | 天然会切分，不用人工设计 |
| 给特征重要性 | 顺便能解释"哪个特征有用" |

代价是它不擅长处理**特征之间的高阶交互**（比如 A 高且 B 低时才危险）。这正是神经网络补位的地方，也是做融合的理由。

### 展开讲 2：为什么要两个模型融合

XGBoost 和 MLP 的**错误模式不一样**：

- XGBoost 擅长"分段规则"，在边界附近容易硬切
- MLP 擅长"平滑组合"，在极端值上容易飘

把两者加权平均，等于让两种错误互相抵消一部分。实测通常能涨 0.5~2 个点的 AUC。

**但融合不是一定有用。** 如果 MLP 训得很差（AUC 比 XGB 低 0.05 以上），加权只会拖后腿。所以第 09 章会教你：融合后指标没涨，就退回单模型，别硬融。

---

## 5. 分步操作

### 步骤 5.1　写 `src/models/base.py`

**【目标】** 定义三个模型必须长什么样。

**【文件】** `src/models/base.py`（新建）

**【完整代码】**

```python
"""
模型基类
========
用抽象基类规定统一接口：fit / predict_proba / save / load。
"""
from __future__ import annotations

from abc import ABC, abstractmethod
from typing import Any, Dict, Optional

import numpy as np
import pandas as pd


class BaseModel(ABC):
    """所有模型的父类。子类必须实现 4 个抽象方法。"""

    def __init__(self, config: Any, name: str = "base_model"):
        self.config = config
        self.name = name
        self._is_fitted = False
        self._feature_names: Optional[list] = None
        self._train_history: Dict = {}

    # ---------- 子类必须实现 ----------

    @abstractmethod
    def fit(self, X_train, y_train, X_val=None, y_val=None, **kwargs) -> "BaseModel":
        """训练模型，返回 self 以支持链式调用。"""
        ...

    @abstractmethod
    def predict_proba(self, X: pd.DataFrame) -> np.ndarray:
        """返回正类（违约）概率，shape=(n,)。"""
        ...

    @abstractmethod
    def save(self, file_path: str) -> None:
        ...

    @abstractmethod
    def load(self, file_path: str) -> "BaseModel":
        ...

    # ---------- 父类已经实现，子类直接继承 ----------

    def predict(self, X: pd.DataFrame, threshold: float = 0.5) -> np.ndarray:
        """概率 -> 0/1 标签。"""
        proba = self.predict_proba(X)
        if proba.ndim == 2:
            proba = proba[:, 1]
        return (proba >= threshold).astype(int)

    def get_feature_importance(self) -> Optional[pd.Series]:
        """默认不支持，支持的模型自己重写。"""
        return None

    @property
    def is_fitted(self) -> bool:
        return self._is_fitted

    @property
    def feature_names(self) -> Optional[list]:
        return self._feature_names

    @property
    def train_history(self) -> Dict:
        return self._train_history

    def _check_fitted(self) -> None:
        if not self._is_fitted:
            raise RuntimeError(f"{self.name} 模型未训练，请先调用 fit()")
```

**【逐块讲解】**

| 代码段 | 作用 | 要点 |
|---|---|---|
| `ABC` + `@abstractmethod` | 强制契约 | 子类少实现一个方法，`__init__` 时直接报 `TypeError` |
| `predict` | 概率转标签 | 这个不用每个子类重写，放父类 |
| `_feature_names` | 记住训练时的列 | 预测时按这个顺序对齐，避免列顺序错乱 |
| `_check_fitted` | 未训练就调用会报错 | 比返回垃圾结果好得多 |

---

### 步骤 5.2　写 `src/models/xgb_model.py`

**【目标】** 把 `XGBClassifier` 包一层，套上统一接口。

**【文件】** `src/models/xgb_model.py`（新建）

**【完整代码】**

```python
"""
XGBoost 模型封装
"""
from __future__ import annotations

from pathlib import Path
from typing import Optional

import numpy as np
import pandas as pd
import xgboost as xgb

from src.models.base import BaseModel
from src.utils.logger import get_logger

logger = get_logger("models.xgb")


class XGBModel(BaseModel):
    """XGBoost 二分类器封装。"""

    def __init__(self, config, name: str = "xgboost"):
        super().__init__(config, name)
        self.params = dict(config.params)      # AttrDict -> 普通 dict
        self.model: Optional[xgb.XGBClassifier] = None

    def fit(self, X_train, y_train, X_val=None, y_val=None, **kwargs) -> "XGBModel":
        logger.info(f"开始训练 XGBoost - 训练样本: {len(X_train)}, 特征: {X_train.shape[1]}")

        # 没配 scale_pos_weight 就按负正比自动算
        if not self.params.get("scale_pos_weight"):
            n_pos = int((y_train == 1).sum())
            n_neg = int((y_train == 0).sum())
            self.params["scale_pos_weight"] = n_neg / max(n_pos, 1)
            logger.info(f"自动设置 scale_pos_weight = {self.params['scale_pos_weight']:.2f}")

        self.model = xgb.XGBClassifier(**self.params)

        eval_set = []
        if X_val is not None and y_val is not None:
            eval_set = [(X_train, y_train), (X_val, y_val)]
            if self.config.early_stopping.enabled:
                self.model.set_params(early_stopping_rounds=self.config.early_stopping.rounds)

        fit_kwargs = {"verbose": False}
        if eval_set:
            fit_kwargs["eval_set"] = eval_set

        self.model.fit(X_train, y_train, **fit_kwargs)

        if hasattr(self.model, "evals_result_"):
            self._train_history = self.model.evals_result_

        self._is_fitted = True
        self._feature_names = X_train.columns.tolist()

        if X_val is not None:
            from sklearn.metrics import roc_auc_score
            val_auc = roc_auc_score(y_val, self.model.predict_proba(X_val)[:, 1])
            logger.info(f"XGBoost 训练完成 - 验证集 AUC: {val_auc:.4f}")

        return self

    def predict_proba(self, X: pd.DataFrame) -> np.ndarray:
        """返回正类概率，shape=(n,)。"""
        self._check_fitted()
        if self._feature_names:
            X = X[self._feature_names]       # 按训练时列顺序对齐
        return self.model.predict_proba(X)[:, 1]

    def get_feature_importance(self, importance_type: str = "gain") -> pd.Series:
        """importance_type: weight / gain / cover"""
        self._check_fitted()
        score = self.model.get_booster().get_score(importance_type=importance_type)

        importance = pd.Series(0.0, index=self._feature_names)
        for k, v in score.items():
            if k in importance.index:
                importance[k] = v
            elif k.startswith("f"):
                idx = int(k[1:])
                if idx < len(self._feature_names):
                    importance[self._feature_names[idx]] = v
        return importance.sort_values(ascending=False)

    def save(self, file_path: str) -> None:
        self._check_fitted()
        path = Path(file_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        self.model.save_model(str(path))
        # XGBoost 的 JSON 不含特征名，单独存一份
        path.with_suffix(".features.txt").write_text(
            "\n".join(self._feature_names), encoding="utf-8"
        )
        logger.info(f"XGBoost 模型已保存至 {path}")

    def load(self, file_path: str) -> "XGBModel":
        path = Path(file_path)
        if not path.exists():
            raise FileNotFoundError(f"模型文件不存在: {path}")

        self.model = xgb.XGBClassifier()
        self.model.load_model(str(path))

        feature_path = path.with_suffix(".features.txt")
        if feature_path.exists():
            self._feature_names = feature_path.read_text(encoding="utf-8").splitlines()

        self._is_fitted = True
        logger.info(f"XGBoost 模型已从 {path} 加载")
        return self
```

**【逐块讲解】**

| 代码段 | 要点 |
|---|---|
| `dict(config.params)` | 配置是 `AttrDict`，转普通 dict 才能 `**` 展开 |
| `scale_pos_weight` 自动算 | 负样本数/正样本数，约 4:1，让模型重视少数类 |
| `eval_set=[(train),(val)]` | 早停需要验证集；同时监控训练集能看出是否过拟合 |
| `X = X[self._feature_names]` | **关键**：预测时列顺序必须与训练一致，否则结果错乱且不报错 |
| `path.with_suffix(".features.txt")` | `xgb.json` → `xgb.features.txt` |
| `load` 返回 self | 所以是 `model = XGBModel(cfg); model.load(p)`，不是 `XGBModel.load(p)` |

**【验证】**

```bat
python -c "import numpy as np,pandas as pd; from src.utils import ConfigLoader; from src.models import XGBModel" 2>nul
```

如果上面报错说 `cannot import name 'XGBModel'`，那是因为 `__init__.py` 还没回填——正常。直接测：

```bat
python -c "import numpy as np,pandas as pd; from src.utils import ConfigLoader; from src.models.xgb_model import XGBModel; c=ConfigLoader('./configs'); X=pd.DataFrame(np.random.randn(500,6),columns=list('abcdef')); y=pd.Series(np.random.binomial(1,0.2,500)); m=XGBModel(c.get('model').xgboost).fit(X[:350],y[:350],X[350:],y[350:]); print('AUC on train:', __import__('sklearn.metrics',fromlist=['x']).roc_auc_score(y[350:], m.predict_proba(X[350:])))"
```

期望：打印一个 0.4~0.7 之间的 AUC（随机数据没有真实信号，这个范围正常）。

---

### 步骤 5.3　写 `src/models/ensemble.py`

**【目标】** 把多个模型的结果加权合并。

**【文件】** `src/models/ensemble.py`（新建）

**【完整代码】**

```python
"""
模型融合
========
支持三种：加权平均 / 投票 / Stacking。
"""
from __future__ import annotations

import json
from pathlib import Path
from typing import Dict, Optional

import numpy as np
import pandas as pd

from src.models.base import BaseModel
from src.utils.logger import get_logger

logger = get_logger("models.ensemble")


class EnsembleModel(BaseModel):
    """融合模型。weighted_avg 最常用。"""

    def __init__(self, config, name: str = "ensemble"):
        super().__init__(config, name)
        self.method = config.method
        self.weights_cfg = dict(config.weights) if config.method == "weighted_avg" else {}

        self.models: Dict[str, BaseModel] = {}
        self.model_weights: Dict[str, float] = {}
        self.meta_learner = None

    def add_model(self, name: str, model: BaseModel, weight: Optional[float] = None) -> None:
        self.models[name] = model
        if weight is None:
            weight = self.weights_cfg.get(name, 1.0)
        self.model_weights[name] = weight
        logger.info(f"已添加基模型: {name} (权重: {weight})")

    def fit(self, X_train, y_train, X_val=None, y_val=None, **kwargs) -> "EnsembleModel":
        if not self.models:
            raise RuntimeError("请先调用 add_model() 添加基模型")

        for name, model in self.models.items():
            if not model.is_fitted:
                raise RuntimeError(f"基模型 {name} 未训练")

        if self.method == "weighted_avg":
            self._normalize_weights()
        elif self.method == "stacking":
            self._train_meta_learner(X_train, y_train)
        elif self.method != "voting":
            raise ValueError(f"未知融合方法: {self.method}")

        self._is_fitted = True
        self._feature_names = list(self.models.values())[0].feature_names
        logger.info(f"融合模型 ({self.method}) 已就绪")
        return self

    def predict_proba(self, X: pd.DataFrame) -> np.ndarray:
        self._check_fitted()

        base_preds = {}
        for name, model in self.models.items():
            preds = model.predict_proba(X)
            if preds.ndim == 2:
                preds = preds[:, 1]
            base_preds[name] = preds

        if self.method == "weighted_avg":
            return self._weighted_average(base_preds)
        if self.method == "voting":
            return self._voting(base_preds)
        return self._stacking_predict(base_preds)

    # ---------- 三种融合 ----------

    def _normalize_weights(self) -> None:
        total = sum(self.model_weights.values())
        if total > 0:
            self.model_weights = {k: v / total for k, v in self.model_weights.items()}

    def _weighted_average(self, base_preds: Dict[str, np.ndarray]) -> np.ndarray:
        result = np.zeros(len(next(iter(base_preds.values()))))
        for name, preds in base_preds.items():
            w = self.model_weights.get(name, 1.0 / len(base_preds))
            result += w * preds
        return result

    def _voting(self, base_preds: Dict[str, np.ndarray], threshold: float = 0.5) -> np.ndarray:
        votes = np.array([p >= threshold for p in base_preds.values()])
        return votes.mean(axis=0)

    def _train_meta_learner(self, X_train: pd.DataFrame, y_train: pd.Series) -> None:
        """Stacking：用基模型输出当新特征，再训一个逻辑回归。"""
        from sklearn.linear_model import LogisticRegression

        meta_features = []
        for model in self.models.values():
            preds = model.predict_proba(X_train)
            meta_features.append(preds[:, 1] if preds.ndim == 2 else preds)

        meta_X = np.column_stack(meta_features)
        self.meta_learner = LogisticRegression(max_iter=1000)
        self.meta_learner.fit(meta_X, y_train)

    def _stacking_predict(self, base_preds: Dict[str, np.ndarray]) -> np.ndarray:
        meta_X = np.column_stack(list(base_preds.values()))
        return self.meta_learner.predict_proba(meta_X)[:, 1]

    # ---------- 持久化 ----------

    def save(self, file_path: str) -> None:
        """只存"配置"，基模型各自单独存。"""
        self._check_fitted()
        path = Path(file_path)
        path.parent.mkdir(parents=True, exist_ok=True)

        info = {
            "method": self.method,
            "model_names": list(self.models.keys()),
            "weights": self.model_weights,
            "feature_names": self._feature_names,
        }

        if self.method == "stacking" and self.meta_learner is not None:
            import joblib
            joblib.dump(self.meta_learner, path.with_suffix(".meta.pkl"))

        path.write_text(json.dumps(info, ensure_ascii=False, indent=2), encoding="utf-8")
        logger.info(f"融合配置已保存至 {path}")

    def load(self, file_path: str) -> "EnsembleModel":
        path = Path(file_path)
        info = json.loads(path.read_text(encoding="utf-8"))

        self.method = info["method"]
        self.model_weights = info["weights"]
        self._feature_names = info["feature_names"]

        if self.method == "stacking":
            import joblib
            meta_path = path.with_suffix(".meta.pkl")
            if meta_path.exists():
                self.meta_learner = joblib.load(meta_path)

        self._is_fitted = True
        logger.warning("注意：基模型需单独加载后再 add_model() 添加")
        return self
```

**【逐块讲解】**

| 关键点 | 说明 |
|---|---|
| `weights_cfg` 只在 `weighted_avg` 时读 | 其他模式不需要权重 |
| `_normalize_weights` | 0.6 + 0.4 = 1.0，归一化是为了防止手写成 6 和 4 |
| `fit()` 里没训练基模型 | 它假设基模型**已经训好**了，只做权重处理 |
| `_voting` | 每个模型先投 0/1 再取平均，得到"比例"当概率 |
| `save()` 只存配置 | 基模型要各自 `save()`，加载时各自 `load()` 再 `add_model()` |
| Stacking 的隐患 | 用训练集预测当元特征会偏乐观（应该用 OOF），教学项目简化处理 |

---

### 步骤 5.4　写 `src/models/mlp_model.py`（神经网络）

**【目标】** PyTorch 手写一个可配置的多层感知机。

**【文件】** `src/models/mlp_model.py`（新建）

**【完整代码】**

```python
"""
MLP 模型（PyTorch）
===================
网络结构、损失、优化器、调度器全部由配置驱动。
"""
from __future__ import annotations

from pathlib import Path
from typing import Dict, List, Optional

import numpy as np
import pandas as pd
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

from src.models.base import BaseModel
from src.utils.logger import get_logger

logger = get_logger("models.mlp")


class MLPNetwork(nn.Module):
    """Input -> [Linear -> BN -> Act -> Dropout] x N -> Output"""

    def __init__(
        self,
        input_dim: int,
        hidden_layers: List[int],
        activation: str = "relu",
        dropout: float = 0.3,
        batch_norm: bool = True,
        output_dim: int = 1,
        output_activation: str = "sigmoid",
    ):
        super().__init__()

        act_map = {
            "relu": nn.ReLU,
            "gelu": nn.GELU,
            "leaky_relu": nn.LeakyReLU,
            "tanh": nn.Tanh,
        }
        Act = act_map.get(activation.lower(), nn.ReLU)

        layers = []
        prev_dim = input_dim
        for hidden_dim in hidden_layers:
            layers.append(nn.Linear(prev_dim, hidden_dim))
            if batch_norm:
                layers.append(nn.BatchNorm1d(hidden_dim))
            layers.append(Act())
            if dropout > 0:
                layers.append(nn.Dropout(dropout))
            prev_dim = hidden_dim

        layers.append(nn.Linear(prev_dim, output_dim))
        if output_activation == "sigmoid":
            layers.append(nn.Sigmoid())

        self.network = nn.Sequential(*layers)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.network(x)


class MLPModel(BaseModel):
    """MLP 分类器封装。"""

    def __init__(self, config, name: str = "mlp"):
        super().__init__(config, name)
        self.arch_cfg = config.architecture
        self.train_cfg = config.training
        self.device = self._setup_device(config.device)

        self.network: Optional[MLPNetwork] = None
        self._train_history = {"train_loss": [], "val_loss": [], "val_auc": []}

    @staticmethod
    def _setup_device(device_str: str) -> torch.device:
        if device_str == "cuda" and torch.cuda.is_available():
            return torch.device("cuda")
        return torch.device("cpu")

    def fit(self, X_train, y_train, X_val=None, y_val=None, **kwargs) -> "MLPModel":
        logger.info(f"开始训练 MLP - 设备: {self.device}, 样本: {len(X_train)}")

        self._feature_names = X_train.columns.tolist()
        self.network = MLPNetwork(
            input_dim=X_train.shape[1],
            hidden_layers=list(self.arch_cfg.hidden_layers),
            activation=self.arch_cfg.activation,
            dropout=self.arch_cfg.dropout,
            batch_norm=self.arch_cfg.batch_norm,
            output_activation=self.arch_cfg.output_activation,
        ).to(self.device)

        train_loader = self._create_loader(X_train, y_train, shuffle=True)
        val_loader = None
        if X_val is not None and y_val is not None:
            val_loader = self._create_loader(X_val, y_val, shuffle=False)

        criterion = self._build_loss(y_train)
        optimizer = self._build_optimizer()
        scheduler = self._build_scheduler(optimizer)

        best_val_auc = 0.0
        patience_counter = 0
        best_state = None

        for epoch in range(self.train_cfg.epochs):
            train_loss = self._train_one_epoch(train_loader, optimizer, criterion)
            val_loss, val_auc = (None, None)
            if val_loader:
                val_loss, val_auc = self._validate(val_loader, criterion)

            self._train_history["train_loss"].append(train_loss)
            if val_loss is not None:
                self._train_history["val_loss"].append(val_loss)
                self._train_history["val_auc"].append(val_auc)

            if scheduler is not None:
                if isinstance(scheduler, optim.lr_scheduler.ReduceLROnPlateau):
                    scheduler.step(val_loss if val_loss is not None else train_loss)
                else:
                    scheduler.step()

            msg = f"Epoch {epoch+1}/{self.train_cfg.epochs} - train_loss: {train_loss:.4f}"
            if val_loss is not None:
                msg += f", val_loss: {val_loss:.4f}, val_auc: {val_auc:.4f}"
            logger.info(msg)

            # 早停
            if val_auc is not None and self.train_cfg.early_stopping.enabled:
                if val_auc > best_val_auc + self.train_cfg.early_stopping.min_delta:
                    best_val_auc = val_auc
                    patience_counter = 0
                    best_state = {k: v.clone() for k, v in self.network.state_dict().items()}
                else:
                    patience_counter += 1
                    if patience_counter >= self.train_cfg.early_stopping.patience:
                        logger.info(f"早停触发 @ epoch {epoch+1}, 最佳 val_auc={best_val_auc:.4f}")
                        break

        if best_state is not None:
            self.network.load_state_dict(best_state)

        self._is_fitted = True
        logger.info(f"MLP 训练完成 - 最佳 val_auc: {best_val_auc:.4f}")
        return self

    def predict_proba(self, X: pd.DataFrame) -> np.ndarray:
        self._check_fitted()
        if self._feature_names:
            X = X[self._feature_names]

        self.network.eval()
        with torch.no_grad():
            x_tensor = torch.FloatTensor(X.values).to(self.device)
            probs = self.network(x_tensor).cpu().numpy().flatten()
        return probs

    # ---------------- 内部方法 ----------------

    def _create_loader(self, X, y, shuffle: bool = True) -> DataLoader:
        x_tensor = torch.FloatTensor(X.values)
        y_tensor = torch.FloatTensor(y.values).unsqueeze(1)
        dataset = TensorDataset(x_tensor, y_tensor)
        return DataLoader(
            dataset,
            batch_size=self.train_cfg.batch_size,
            shuffle=shuffle,
            num_workers=0,     # Windows 上必须 0，否则多进程会出问题
        )

    def _build_loss(self, y_train: pd.Series) -> nn.Module:
        if self.train_cfg.loss == "bce":
            if self.train_cfg.class_weight == "balanced":
                pos_weight = (y_train == 0).sum() / max((y_train == 1).sum(), 1)
                return WeightedBCELoss(pos_weight=float(pos_weight))
            return nn.BCELoss()
        if self.train_cfg.loss == "focal":
            return FocalLoss()
        raise ValueError(f"不支持的损失函数: {self.train_cfg.loss}")

    def _build_optimizer(self) -> optim.Optimizer:
        name = self.train_cfg.optimizer.lower()
        params = self.network.parameters()
        lr, wd = self.train_cfg.learning_rate, self.train_cfg.weight_decay

        if name == "adam":
            return optim.Adam(params, lr=lr, weight_decay=wd)
        if name == "adamw":
            return optim.AdamW(params, lr=lr, weight_decay=wd)
        if name == "sgd":
            return optim.SGD(params, lr=lr, momentum=0.9, weight_decay=wd)
        raise ValueError(f"不支持的优化器: {name}")

    def _build_scheduler(self, optimizer):
        name = self.train_cfg.scheduler
        if name == "cosine":
            return optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=self.train_cfg.epochs)
        if name == "step":
            return optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.5)
        if name == "plateau":
            return optim.lr_scheduler.ReduceLROnPlateau(optimizer, mode="min", patience=3)
        return None

    def _train_one_epoch(self, loader, optimizer, criterion) -> float:
        self.network.train()
        total_loss, n_samples = 0.0, 0

        for x_batch, y_batch in loader:
            x_batch = x_batch.to(self.device)
            y_batch = y_batch.to(self.device)

            optimizer.zero_grad()
            output = self.network(x_batch)
            loss = criterion(output, y_batch)
            loss.backward()
            optimizer.step()

            total_loss += loss.item() * x_batch.size(0)
            n_samples += x_batch.size(0)

        return total_loss / max(n_samples, 1)

    def _validate(self, loader, criterion):
        from sklearn.metrics import roc_auc_score

        self.network.eval()
        total_loss, n_samples = 0.0, 0
        all_preds, all_targets = [], []

        with torch.no_grad():
            for x_batch, y_batch in loader:
                x_batch = x_batch.to(self.device)
                y_batch = y_batch.to(self.device)
                output = self.network(x_batch)
                loss = criterion(output, y_batch)

                total_loss += loss.item() * x_batch.size(0)
                n_samples += x_batch.size(0)
                all_preds.extend(output.cpu().numpy().flatten())
                all_targets.extend(y_batch.cpu().numpy().flatten())

        avg_loss = total_loss / max(n_samples, 1)
        auc = roc_auc_score(all_targets, all_preds) if len(set(all_targets)) > 1 else 0.0
        return avg_loss, auc

    def save(self, file_path: str) -> None:
        self._check_fitted()
        path = Path(file_path)
        path.parent.mkdir(parents=True, exist_ok=True)

        checkpoint = {
            "state_dict": self.network.state_dict(),
            "input_dim": self.network.network[0].in_features,
            "feature_names": self._feature_names,
            "train_history": self._train_history,
        }
        torch.save(checkpoint, str(path))
        logger.info(f"MLP 模型已保存至 {path}")

    def load(self, file_path: str) -> "MLPModel":
        path = Path(file_path)
        if not path.exists():
            raise FileNotFoundError(f"模型文件不存在: {path}")

        checkpoint = torch.load(str(path), map_location=self.device, weights_only=False)
        self._feature_names = checkpoint["feature_names"]
        self._train_history = checkpoint.get("train_history", {})

        # PyTorch 存的是权重不是结构，所以要按同样配置重建网络
        self.network = MLPNetwork(
            input_dim=checkpoint["input_dim"],
            hidden_layers=list(self.arch_cfg.hidden_layers),
            activation=self.arch_cfg.activation,
            dropout=self.arch_cfg.dropout,
            batch_norm=self.arch_cfg.batch_norm,
            output_activation=self.arch_cfg.output_activation,
        ).to(self.device)
        self.network.load_state_dict(checkpoint["state_dict"])

        self._is_fitted = True
        logger.info(f"MLP 模型已从 {path} 加载")
        return self


# ================= 自定义损失 =================

class WeightedBCELoss(nn.Module):
    """加权交叉熵：给正样本更大权重。"""

    def __init__(self, pos_weight: float = 1.0):
        super().__init__()
        self.pos_weight = pos_weight

    def forward(self, pred, target):
        eps = 1e-7
        pred = torch.clamp(pred, eps, 1 - eps)
        loss_pos = -self.pos_weight * target * torch.log(pred)
        loss_neg = -(1 - target) * torch.log(1 - pred)
        return (loss_pos + loss_neg).mean()


class FocalLoss(nn.Module):
    """Focal Loss：让模型更关注难分样本。"""

    def __init__(self, alpha: float = 0.25, gamma: float = 2.0):
        super().__init__()
        self.alpha = alpha
        self.gamma = gamma

    def forward(self, pred, target):
        eps = 1e-7
        pred = torch.clamp(pred, eps, 1 - eps)
        pt = pred * target + (1 - pred) * (1 - target)
        alpha_t = self.alpha * target + (1 - self.alpha) * (1 - target)
        return (-alpha_t * (1 - pt) ** self.gamma * torch.log(pt)).mean()
```

**【逐块讲解】**

| 代码段 | 要点 |
|---|---|
| `nn.Sequential(*layers)` | 把层列表拼成一个前向管道 |
| `BatchNorm1d(hidden_dim)` | 每层归一化，训练更稳更快 |
| `nn.Dropout(0.3)` | 训练时随机丢 30%，推理时自动关闭 |
| `optimizer.zero_grad()` | 清空上一步的梯度，不清会累加 |
| `loss.backward()` | 反向传播，算出每个参数的梯度 |
| `optimizer.step()` | 按梯度更新参数 |
| `self.network.eval()` | 切到推理模式（关掉 dropout、BN 用统计值） |
| `torch.no_grad()` | 不建计算图，省内存提速 |
| `num_workers=0` | **Windows 必须设 0**，否则多进程 spawn 会报错 |
| `weights_only=False` | PyTorch 2.6+ 默认 True，读含自定义对象的 checkpoint 要显式关掉 |

**【验证】**

```bat
python -c "import numpy as np,pandas as pd; from src.utils import ConfigLoader; from src.models.mlp_model import MLPModel; c=ConfigLoader('./configs'); cfg=c.get('model').mlp; cfg.training.epochs=3; X=pd.DataFrame(np.random.randn(400,6),columns=list('abcdef')); y=pd.Series(np.random.binomial(1,0.2,400)); m=MLPModel(cfg).fit(X[:300],y[:300],X[300:],y[300:]); print('shape:', m.predict_proba(X[300:]).shape)"
```

期望输出：`shape: (100,)`（一维，不是 `(100,2)`）。

**【常见报错】**

| 报错 | 原因 | 修复 |
|---|---|---|
| `RuntimeError: DataLoader worker ... exited unexpectedly` | `num_workers` 不为 0 | 改成 0 |
| `ValueError: Expected more than 1 value per channel when training` | 最后一个 batch 只有 1 条样本，BN 无法计算 | 把 `batch_size` 调小或丢弃最后不足 2 条的 batch |
| `RuntimeError: mat1 and mat2 shapes cannot be multiplied` | 输入维度与网络不匹配 | 检查 `input_dim` 是否等于特征数 |
| `torch.load` 报 `UnpicklingError` | PyTorch 2.6+ 的 `weights_only` 默认变了 | 加 `weights_only=False` |

---

### 步骤 5.5　回填仪式：`src/models/__init__.py`

```python
"""模型模块"""
from src.models.base import BaseModel
from src.models.xgb_model import XGBModel
from src.models.mlp_model import MLPModel
from src.models.ensemble import EnsembleModel

__all__ = ["BaseModel", "XGBModel", "MLPModel", "EnsembleModel"]
```

**【验证】**

```bat
python -c "from src.models import XGBModel, MLPModel, EnsembleModel; print('models OK')"
```

> ⚠️ 这条命令会强制 `import torch`。如果这里报 `ModuleNotFoundError: No module named 'torch'`，说明第 00 章的 torch 没装好，**不是模型代码的问题**。

---

## 6. 本章错误对照

### 错误 E-09　`XGBModel.load()` 是实例方法，不是类方法

**出现在**：原文档附录 B 的 API 速查

**❌ 原文写法**

```python
model = XGBModel.load("xgb.json", configs.get("model").xgboost)
```

**✅ 修正写法**

```python
model = XGBModel(configs.get("model").xgboost)
model.load("data/models/latest/xgb.json")
```

**💥 不修的后果**

```
TypeError: XGBModel.load() missing 1 required positional argument: 'file_path'
```

或者更迷惑的 `AttributeError: 'str' object has no attribute 'exists'`——因为第一个参数被当成了 `self`。

**🧠 为什么**　`load` 在 `BaseModel` 里是普通实例方法（`def load(self, file_path)`），不是 `@classmethod`。它要往 `self` 上挂模型对象，必须先有实例。

**✅ 自查**　凡是 `类名.方法(...)` 的写法，先看那个方法有没有 `@classmethod` 或 `@staticmethod`。

---

### 错误 E-10　把 `predict_proba` 当成 `(n, 2)` 用

**出现在**：原文档指导书的评估示例

**❌ 原文写法**

```python
proba = model.predict_proba(X_test)     # 以为是 (n, 2)
y_pred = proba[:, 1]
```

**✅ 修正写法**

```python
proba = model.predict_proba(X_test)     # 本项目返回 (n,)
y_pred = (proba >= 0.5).astype(int)
```

**💥 不修的后果**

```
IndexError: too many indices for array: array is 1-dimensional, but 2 were indexed
```

**🧠 为什么**　本项目的 `XGBModel.predict_proba` 内部已经 `[:, 1]` 取过正类概率，返回的是一维。sklearn 原生接口返回二维，两者不同——这正是"包装层"带来的差异，很容易混。

**✅ 自查**　`print(model.predict_proba(X[:5]).shape)`，看到 `(5,)` 就是一维。

---

### 错误 W-03　MLP 在 CPU 上训 50 轮太慢

**现象**　按默认配置（`epochs: 50`、`batch_size: 512`）在 CPU 上训 80 万行，一轮约 30~60 秒，50 轮要半小时以上。

**✅ 建议**　第一次跑通时改小：

```yaml
mlp:
  training:
    epochs: 8
    batch_size: 1024
```

跑通验证流程用 8 轮足够。等确认整条链路没问题，再改回来做正式训练。

---

## 7. 常见坑

| 现象 | 原因 | 解决 |
|---|---|---|
| MLP 的 loss 一直是 0.69（等于 `-log(0.5)`） | 模型没学到东西 | 检查是否标准化了；检查学习率 |
| 验证集 AUC 一直是 0.5 | 标签和特征错位 | 检查 `y` 是否被当成特征喂进去了 |
| 训练 loss 降但验证 loss 升 | 过拟合 | 加 dropout、减层数、早停 |
| `predict_proba` 结果每次不一样 | 忘了 `eval()` 模式 | 推理前必须 `self.network.eval()` |
| 保存的模型加载后预测全一样 | 网络结构没重建对 | 确认 `load` 时用的 `arch_cfg` 和训练时一致 |

---

## 8. 本章自测

**1. 为什么 `BaseModel` 要定义 `predict` 而不把它设成抽象方法？**

<details><summary>点开看答案</summary>

因为 `predict` 的逻辑（概率过阈值变标签）对所有模型都一样，写在父类里三个子类都能复用。抽象方法只留给"每个模型实现必然不同"的部分：训练、出概率、存、取。

</details>

**2. XGBoost 训练前不标准化也能工作，为什么 MLP 必须标准化？**

<details><summary>点开看答案</summary>

树模型只看特征的大小关系（切分点），不看绝对数值，所以量纲无所谓。神经网络做的是加权求和，一个量纲 10 万的特征会直接主导梯度，导致其他特征失效、训练不收敛。

</details>

**3. 融合模型的 `save()` 为什么只存配置，不存基模型？**

<details><summary>点开看答案</summary>

因为基模型有各自的格式（xgb 是 json、mlp 是 pth），没法塞进同一个文件。所以融合层只记录"用了哪些模型、权重多少"，基模型各自存，加载时分别载入再 `add_model()`。

</details>

**4. 如果 MLP 的 AUC 只有 0.60，XGBoost 是 0.74，还该融合吗？**

<details><summary>点开看答案</summary>

不该。差距 0.14 说明 MLP 基本没学到东西，加权只会拖低整体。应该先修 MLP（检查标准化、学习率、训练轮数），或者干脆只用 XGBoost。

</details>

---

## 9. 本章产物清单 + 下一步

**产物**

| 路径 | 行数约 | 导出 |
|---|---|---|
| `src/models/base.py` | 80 | `BaseModel` |
| `src/models/xgb_model.py` | 130 | `XGBModel` |
| `src/models/ensemble.py` | 175 | `EnsembleModel` |
| `src/models/mlp_model.py` | 300 | `MLPModel` `WeightedBCELoss` `FocalLoss` |
| `src/models/__init__.py` | 7 | 四个类 |

**一条命令自检**

```bat
python -c "from src.models import XGBModel,MLPModel,EnsembleModel; print('models OK')"
```

**你现在已经能做到**：训练三个模型并拿到预测概率。**这是整个项目第一次真正"有智能"的时刻。**

**下一章**：调参层——让 Optuna 自动帮你找比默认值更好的超参组合。
