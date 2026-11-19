# 第 07 章　解释层：用 SHAP 说清"为什么"

> 本章建 1 个文件 + 回填 1 个 · 预计 1.5~2 小时 · 难度 ★★★ · 前置：第 05 章

---

## 1. 这一章在流水线上的位置

```
[原始CSV] →①数据层→ ②特征层→ ③模型层→ ④调参→ ⑤解释→ ⑥AI报告→ ⑦编排→ ⑧脚本→ ⑨网页
                                              ▲
你在这里：第 07 章　解释层
已经有的：能预测的模型
还没有的：AI 报告、网页
```

---

## 2. 本章你要做什么

模型说"这个人违约概率 73%"。业务人员下一句一定问：**为什么？**

这一章就是回答这个问题。你会得到：

- **单笔解释**：这个人为什么是高风险，哪 5 个因素起了作用，各推高/拉低多少
- **全局解释**：整体上哪些特征最重要（画条形图用）

**这是整个项目最有"产品感"的部分**，也是答辩时最能打动人的环节。

---

## 3. 开工前检查

- [ ] 上一章输出的是 `tuning OK`
- [ ] `import shap` 能成功
- [ ] `src/explain/__init__.py` 存在且是**空的**

---

## 4. 本章知识点

| 编号 | 概念 | 一句话解释 | 为什么这里需要它 | 本项目用在哪 | 要求 |
|---|---|---|---|---|---|
| KP-AL-15 | SHAP | 用博弈论的 Shapley 值公平分配"功劳" | 回答"为什么是这个预测" | `SHAPAnalyzer` | 必懂 |
| KP-AL-16 | 可加性 | 预测值 = 基准值 + 所有特征贡献之和 | 这是瀑布图能画出来的原理 | `explain_single` | 必懂 |
| KP-AL-17 | 全局 vs 局部解释 | 整体哪些特征重要 vs 这一笔为什么这样 | 两种用途，都要有 | `global_importance` / `explain_single` | 必懂 |
| KP-AL-18 | 三种 Explainer | Tree（快）/ Deep（神经网络）/ Kernel（通用但慢） | 不同模型要用不同解释器 | `_auto_select_explainer` | 知道即可 |
| KP-BIZ-06 | 归因方向 | SHAP 为正 = 推高违约风险 | 报告里要说清"这个因素是加分还是减分" | `direction` 字段 | 必懂 |
| KP-EN-14 | 背景数据集 | 计算基准期望值用的样本 | 不抽稀会很慢 | `max_background_size` | 知道即可 |

### 展开讲 1：SHAP 值到底怎么读

想象三个人合伙赚了 100 万，怎么分？按各自贡献分才公平。Shapley 值就是数学上唯一满足"公平性公理"的分法。

套到模型上：

```
基准值（所有人的平均违约概率）= 20%
这个人的预测值                = 73%
差 53 个百分点，是谁贡献的？
```

SHAP 把这 53 个点分摊给每个特征：

| 特征 | 取值 | SHAP 值 | 读法 |
|---|---|---|---|
| `installment_to_income_ratio` | 0.62 | +0.21 | 月还款占收入 62%，把风险推高 21 个点 |
| `dti` | 45.3 | +0.14 | 负债收入比高，推高 14 个点 |
| `annualIncome` | 48000 | +0.09 | 收入偏低，推高 9 个点 |
| `fico_avg` | 705 | −0.08 | 信用分还行，拉低 8 个点 |
| `grade` | C | +0.05 | 等级 C，推高 5 个点 |

```
基准 20% + 21 + 14 + 9 - 8 + 5 + ...（其余特征）= 73%
```

**这个加法必须成立**，这是 SHAP 的数学保证，也是瀑布图的原理。

### 展开讲 2：为什么解释器要分三种

| 解释器 | 适用 | 速度 | 原理 |
|---|---|---|---|
| `TreeExplainer` | 树模型（XGBoost/LightGBM） | 很快 | 利用树结构精确计算 |
| `DeepExplainer` | 神经网络 | 中等 | 用反向传播近似 |
| `KernelExplainer` | 任意模型（黑盒） | **很慢** | 反复扰动输入、观察输出变化 |

选错会很难受：给 XGBoost 用 `KernelExplainer`，1000 个样本可能要跑十几分钟。所以代码里做了**自动识别**。

---

## 5. 分步操作

### 步骤 7.1　写 `src/explain/shap_analyzer.py`

**【目标】** 三个能力：单笔解释、批量解释、全局重要性。

**【文件】** `src/explain/shap_analyzer.py`（新建）

**【完整代码】**

```python
"""
SHAP 模型解释
=============
同时支持全局解释（哪些特征重要）和局部解释（这一笔为什么这样）。
"""
from __future__ import annotations

from dataclasses import dataclass
from typing import Any, Dict, List, Optional

import numpy as np
import pandas as pd
import shap

from src.utils.logger import get_logger

logger = get_logger("explain.shap")


@dataclass
class SHAPExplanation:
    """单笔解释结果。"""
    base_value: float
    shap_values: np.ndarray
    feature_values: np.ndarray
    feature_names: List[str]
    prediction: float

    def get_top_features(self, top_n: int = 5) -> List[Dict]:
        """取贡献最大的 N 个特征，带方向说明。"""
        top_idx = np.argsort(-np.abs(self.shap_values))[:top_n]
        return [
            {
                "feature": self.feature_names[i],
                "value": self._format_value(self.feature_values[i]),
                "shap_value": float(self.shap_values[i]),
                "direction": "↑ 增加风险" if self.shap_values[i] > 0 else "↓ 降低风险",
            }
            for i in top_idx
        ]

    @staticmethod
    def _format_value(value):
        if isinstance(value, (int, np.integer)):
            return int(value)
        if isinstance(value, (float, np.floating)):
            return round(float(value), 4)
        return str(value)


class SHAPAnalyzer:
    """SHAP 分析器。构造只收一个"背景集大小"。"""

    def __init__(self, max_background_size: int = 1000):
        self.max_background_size = max_background_size
        self.explainer: Optional[Any] = None
        self.background_data: Optional[pd.DataFrame] = None
        self.feature_names: Optional[List[str]] = None
        self.expected_value: Optional[float] = None

    def fit(
        self,
        model: Any,
        background_data: pd.DataFrame,
        explainer_type: str = "auto",
    ) -> "SHAPAnalyzer":
        """
        参数:
            model: 训练好的模型。可以是原生模型，也可以是本项目的包装类
            background_data: 背景集，抽 1000 行足够
            explainer_type: auto / tree / kernel / deep
        """
        if len(background_data) > self.max_background_size:
            background_data = background_data.sample(
                n=self.max_background_size, random_state=42
            )

        self.background_data = background_data
        self.feature_names = background_data.columns.tolist()

        if explainer_type == "auto":
            explainer_type = self._auto_select_explainer(model)

        logger.info(f"使用 {explainer_type} explainer")

        if explainer_type == "tree":
            self.explainer = shap.TreeExplainer(model)

        elif explainer_type == "kernel":
            self.explainer = shap.KernelExplainer(
                model.predict_proba if hasattr(model, "predict_proba") else model,
                background_data,
            )

        elif explainer_type == "deep":
            import torch
            # 本项目的 MLPModel 是包装类，DeepExplainer 要的是里面的 nn.Module
            net = getattr(model, "network", model)
            try:
                device = next(net.parameters()).device
            except StopIteration:
                device = torch.device("cpu")
            bg_tensor = torch.FloatTensor(background_data.values).to(device)
            self.explainer = shap.DeepExplainer(net, bg_tensor)

        else:
            raise ValueError(f"不支持的 explainer 类型: {explainer_type}")

        self.expected_value = self._get_expected_value()
        logger.info(f"SHAP explainer 已就绪 - 基准值: {self.expected_value:.4f}")
        return self

    def explain_single(self, sample: pd.Series) -> SHAPExplanation:
        """解释单个样本。"""
        if self.explainer is None:
            raise RuntimeError("请先调用 fit()")

        sample_df = sample.to_frame().T if isinstance(sample, pd.Series) else sample
        if self.feature_names:
            sample_df = sample_df[self.feature_names]

        shap_values = self.explainer.shap_values(sample_df)

        # 多分类会返回 list，取正类
        if isinstance(shap_values, list):
            shap_values = shap_values[1] if len(shap_values) > 1 else shap_values[0]

        shap_array = shap_values[0] if np.asarray(shap_values).ndim == 2 else shap_values
        shap_array = np.asarray(shap_array).ravel()

        prediction = self.expected_value + float(shap_array.sum())

        return SHAPExplanation(
            base_value=float(self.expected_value),
            shap_values=shap_array,
            feature_values=sample_df.iloc[0].values,
            feature_names=self.feature_names,
            prediction=float(prediction),
        )

    def explain_batch(self, X: pd.DataFrame) -> np.ndarray:
        """批量 SHAP 值，shape=(n_samples, n_features)。"""
        if self.explainer is None:
            raise RuntimeError("请先调用 fit()")

        if self.feature_names:
            X = X[self.feature_names]

        shap_values = self.explainer.shap_values(X)
        if isinstance(shap_values, list):
            shap_values = shap_values[1] if len(shap_values) > 1 else shap_values[0]

        return np.asarray(shap_values)

    def global_importance(self, X: pd.DataFrame, top_n: int = 20) -> pd.Series:
        """全局重要性 = 所有样本 SHAP 绝对值的平均。"""
        shap_values = self.explain_batch(X)
        importance = np.abs(shap_values).mean(axis=0)
        result = pd.Series(importance, index=self.feature_names)
        return result.sort_values(ascending=False).head(top_n)

    def _auto_select_explainer(self, model: Any) -> str:
        """根据模型类型选解释器。"""
        name = model.__class__.__name__.lower()

        if any(k in name for k in ["xgb", "lgbm", "cat", "tree", "forest", "gbm"]):
            return "tree"
        if hasattr(model, "network") or hasattr(model, "forward"):
            return "deep"
        return "kernel"

    def _get_expected_value(self) -> float:
        ev = self.explainer.expected_value
        if isinstance(ev, (list, np.ndarray)):
            arr = np.asarray(ev).ravel()
            ev = arr[1] if len(arr) > 1 else arr[0]
        return float(ev)
```

**【逐块讲解】**

| 代码段 | 要点 |
|---|---|
| `max_background_size=1000` | 背景集越大越准但越慢，1000 是速度/精度的平衡点 |
| `_auto_select_explainer` | 靠类名猜：`XGBModel` 含 `xgb` → tree；含 `network` 属性 → deep |
| `getattr(model, "network", model)` | **关键修正**：包装类要拆出里面的 `nn.Module` |
| `sample_df[self.feature_names]` | 列顺序对齐，跟模型训练时一致 |
| `expected_value + shap_array.sum()` | 可加性，这是 SHAP 的核心性质 |
| `np.argsort(-np.abs(...))` | 按**绝对值**排序，因为要看"影响最大"，不管是正是负 |

**【验证】**

```bat
python -c "import numpy as np,pandas as pd; from src.utils import ConfigLoader; from src.models.xgb_model import XGBModel; from src.explain.shap_analyzer import SHAPAnalyzer; c=ConfigLoader('./configs'); X=pd.DataFrame(np.random.randn(400,6),columns=list('abcdef')); y=pd.Series(np.random.binomial(1,0.2,400)); m=XGBModel(c.get('model').xgboost).fit(X[:300],y[:300],X[300:],y[300:]); a=SHAPAnalyzer().fit(m.model, X[:100]); e=a.explain_single(X.iloc[0]); print('基准:',round(e.base_value,4),'预测:',round(e.prediction,4)); print(e.get_top_features(3))"
```

期望：打印基准值、预测值，以及 3 个特征的贡献列表（含 `feature`、`value`、`shap_value`、`direction`）。

**【常见报错】**

| 报错 | 原因 | 修复 |
|---|---|---|
| `TypeError: SHAPAnalyzer.__init__() takes ... argument` | 把 model 传给了构造函数 | 构造不传 model，`fit` 时才传 |
| `AttributeError: 'MLPModel' object has no attribute '_modules'` | DeepExplainer 收到了包装类 | 用 `getattr(model, "network", model)` 拆开 |
| `KeyError: ['xxx'] not in index` | 输入特征列与背景集不一致 | 检查列名有没有拼错 |
| SHAP 算得极慢 | 用了 KernelExplainer 解释树模型 | 显式传 `explainer_type="tree"` |

---

### 步骤 7.2　回填仪式：`src/explain/__init__.py`

```python
"""模型解释模块"""
from src.explain.shap_analyzer import SHAPAnalyzer, SHAPExplanation

__all__ = ["SHAPAnalyzer", "SHAPExplanation"]
```

**【验证】**

```bat
python -c "from src.explain import SHAPAnalyzer, SHAPExplanation; print('explain OK')"
```

---

## 6. 本章错误对照

### 错误 E-12　三个方法的签名全错了

**出现在**：原文档指导书的 API 速查（第 2840 行附近）

**❌ 原文写法**

```python
a = SHAPAnalyzer(model)          # ① 构造时传 model
a.fit(X_background)              # ② fit 只传背景集
exp = a.explain(X_single)        # ③ 方法叫 explain
exp.get_top_features(top_n=5)
```

**✅ 修正写法**

```python
a = SHAPAnalyzer(max_background_size=1000)   # ① 构造只收背景集大小
a.fit(model, X_background)                   # ② fit(model, background)
exp = a.explain_single(X_single)             # ③ 方法叫 explain_single
exp.get_top_features(top_n=5)
```

**💥 不修的后果**　三步分别报三个不同的错：

```
TypeError: SHAPAnalyzer.__init__() takes from 1 to 2 positional arguments but 3 were given
TypeError: fit() missing 1 required positional argument: 'background_data'
AttributeError: 'SHAPAnalyzer' object has no attribute 'explain'
```

**🧠 为什么**　设计上把"解释器"和"被解释的模型"分开了：分析器是**可复用**的（可以先后解释 XGB 和 MLP），所以模型在 `fit` 时才绑定，不在构造时绑定。方法名加 `_single` 是为了和 `explain_batch` 对称。

**✅ 自查**　记住三件事：构造不传模型、`fit(model, bg)`、单笔用 `explain_single`。

---

### 错误 E-13　`DeepExplainer` 收到的是包装类不是 `nn.Module`

**出现在**：原文档 `fit` 的 deep 分支

**❌ 原文写法**

```python
elif explainer_type == "deep":
    background_tensor = torch.FloatTensor(background_data.values)
    self.explainer = shap.DeepExplainer(model, background_tensor)
```

（这里的 `model` 是 `MLPModel` 实例）

**✅ 修正写法**

```python
elif explainer_type == "deep":
    net = getattr(model, "network", model)     # 拆出真正的 nn.Module
    bg_tensor = torch.FloatTensor(background_data.values)
    self.explainer = shap.DeepExplainer(net, bg_tensor)
```

**💥 不修的后果**

```
AttributeError: 'MLPModel' object has no attribute '_modules'
```

或更深的 `RuntimeError: Expected a 'cuda' device type for generator but found 'cpu'`。

**🧠 为什么**　`shap.DeepExplainer` 内部要遍历 `model._modules` 拿到网络层结构。`_modules` 是 `nn.Module` 的属性，而 `MLPModel` 只是个普通 Python 类（它**持有**一个 `network`，但自己不**是**网络）。必须把里面的 `self.network` 递进去。

**✅ 自查**　传给 `DeepExplainer` 的对象，`isinstance(obj, torch.nn.Module)` 必须是 `True`。

---

### 错误 W-05　把 SHAP 值当成"因果"

**现象**　报告里写"因为月还款占比高，所以这个人会违约"。

**问题**　SHAP 说明的是"模型为什么这么判"，不是"现实中为什么会违约"。这两件事不一样：

- 模型可能因为"邮编"给高分，但那只是相关性
- 换一个模型，SHAP 值就变了

**✅ 正确表述**　"模型认为……对该判断贡献最大"，而不是"……导致了他违约"。

**哪里要注意**　第 08 章喂给 LLM 的 prompt 里，措辞必须是"模型的风险判断依据"，不是"违约原因"。这是金融合规的硬要求。

---

## 7. 常见坑

| 现象 | 原因 | 解决 |
|---|---|---|
| SHAP 值加起来不等于预测值 | 用了 sigmoid 后的概率做近似 | 树模型的 SHAP 是 log-odds 空间，转概率后不严格可加，属正常 |
| `explain_single` 报维度错 | 传了 DataFrame 而不是 Series | 支持两种，但 Series 更稳 |
| 全局重要性每次不同 | 背景集是随机抽样的 | 固定 `random_state=42` 已处理 |
| 解释结果全是 0 | 模型其实没训练（`_is_fitted=False`） | 确认 fit 过 |
| 内存爆 | 一次解释几万条 | 用 `explain_batch` 分批，或抽样 |

---

## 8. 本章自测

**1. SHAP 值 +0.21 是什么意思？**

<details><summary>点开看答案</summary>

这个特征把该样本的预测结果朝"违约"方向推高了 0.21（在 log-odds 空间）。业务上的说法是"这个因素显著增加了风险"，但不能说成"导致违约"。

</details>

**2. 为什么全局重要性要用绝对值再平均？**

<details><summary>点开看答案</summary>

因为正负会相互抵消。一个特征在某些样本上强烈推高、在另一些上强烈拉低，直接平均接近 0，但它其实很重要。取绝对值才能衡量"影响强度"。

</details>

**3. 给 XGBoost 用 KernelExplainer 会怎样？**

<details><summary>点开看答案</summary>

能算，但极慢——它要反复扰动输入、每次调一次 `predict_proba`。1000 个样本可能要十几分钟甚至更久。树模型应该用 `TreeExplainer`。

</details>

---

## 9. 本章产物清单 + 下一步

**产物**

| 路径 | 行数约 | 导出 |
|---|---|---|
| `src/explain/shap_analyzer.py` | 175 | `SHAPAnalyzer` `SHAPExplanation` |
| `src/explain/__init__.py` | 4 | 同上 |

**一条命令自检**

```bat
python -c "from src.explain import SHAPAnalyzer,SHAPExplanation; print('explain OK')"
```

**你现在已经能做到**：说清"这一笔为什么被判高风险"，精确到每个特征贡献了多少。

**下一章**：AI 层——把 SHAP 的数字化结果，翻译成业务人员能读的自然语言报告。这是整个项目和传统机器学习项目最大的差异点。
