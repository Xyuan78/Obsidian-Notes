# 附录 B · 错误对照总表

> **这份附录解决什么问题**
> 源文档有 **8790 行代码 + 2926 行说明**，里面有 **71 处** 会让你的项目出问题的地方。
> 这些地方**不全是错别字**，更危险的是三类：
> - **照抄必报错**（你能发现，好办）
> - **不报错但结果错**（你发现不了，最坑）
> - **方法论错误**（指标虚高，你可能还挺高兴）
>
> 这份附录把 71 处归并成 **34 个编号**，按危险等级排，每条给：
> **❌ 原文怎么写 → 💥 会发生什么 → ✅ 正确写法 → 🔍 怎么自查**
>
> **怎么用**：写代码时遇到报错，B.6 有"按症状反查"；交付前用 [`附录E`](附录E_验收Checklist.md) 过一遍。

---

## B.0 编号规则与危险等级

### 四级分类

| 级别 | 名称 | 定义 | 数量 | 你的感受 |
|---|---|---|---|---|
| **S** | **Silent，方法论错误** | 代码能跑、指标好看，但**结论是假的** | **1** | 完全察觉不到 |
| **E** | **Error，必然报错** | 照抄就 `Traceback` | 23 | 一眼看到红字 |
| **W** | **Wrong，逻辑错误** | 能跑、不报错，但**结果不对** | 7 | 结果怪但不知哪错 |
| **N** | **Nit，环境陷阱** | 配置/文件名/编码层面的坑 | 3 | 排查半天 |

### 为什么 S 级单独列、而且只放 1 条

因为**它一条顶其他 33 条**。

E 级报错，你会去查、会去改，改完就没事。
W 级结果不对，你对比一下数字会发现。
**S 级什么都不发生。** 你跑出 AUC 0.81，很开心，写进报告，拿去答辩——然后被评委一句话问穿。

**如果这本手册你只记住一条，记 S-01。**

### 编号去重说明

源文档实测发现 71 处问题。归并原则：

- **同一个错误在不同章节重复出现** → 合并为一条（如 `y_proba` 参数名错了 4 次，记为 E-02）
- **同一类错误的不同实例** → 合并为一条（如 3 个地方都缺 `sys.path`，记为 PY-07）
- **纯排版/表述瑕疵** → 不收录
- **同一个 API 在两份文档里描述矛盾** → 取"能跑通的那个"，矛盾本身记入正文说明

所以：**71 处现象 → 34 个根因**。

---

## B.1 S 级 · 方法论错误（1 条，最重要）

### 🚨 S-01 · 流水线顺序导致数据泄漏

**这是全书唯一"改了原文档核心逻辑"的地方。**

#### ❌ 原文的流水线顺序

```python
# 源文档 pipeline_config.yaml 的 steps 顺序
steps:
  - load_data
  - preprocess              # ← 在这里 fit 了标准化器
  - feature_engineering
  - feature_selection       # ← 在这里 fit 了特征筛选器
  - split_data              # ← 切分在最后！
  - train
  - evaluate
```

对应到 Python：

```python
# 原文 orchestrator.py 的写法
data = load_data(path)
data = preprocess(data)              # 对全量数据 fit + transform
data = feature_engineering(data)
data = feature_selection(data)       # 对全量数据 fit + transform
train, val, test = split_data(data)  # 最后才切
model.fit(train)
```

#### 💥 会发生什么

**什么都不会发生。** 代码跑得干干净净，没有一行 Warning。

然后你的指标会是：

```
验证集 AUC = 0.813    ← 虚高
测试集 AUC = 0.807    ← 也虚高
KS = 0.462
```

看起来很漂亮。但是：

| 你以为 | 实际 |
|---|---|
| 模型有 0.81 的能力 | 模型只有 0.78 的能力 |
| 上线后效果 ≈ 0.81 | 上线后效果 ≈ 0.78 |
| 指标可信 | 指标不可信，且**你无法判断差多少** |

**为什么虚高？** 三个具体机制：

**机制 1 · 标准化器偷看了测试集**

`StandardScaler.fit(X_all)` 算出的 `mean_` 和 `std_`，包含了测试集的样本。
测试集的均值和方差，通过这两个参数，进入了训练过程。
模型在训练时"知道"了测试集的分布中心在哪。

**机制 2 · 特征筛选偷看了标签**

`FeatureSelector` 用**全量数据的标签**算了特征重要性，然后按重要性删列。
这意味着它筛选时已经看过了测试集的 `isDefault`（标签）。
被保留下来的特征，是"在测试集上也表现好"的特征——这就是**选择性偏差**。

**机制 3 · 缺失值填充偷看了分布**

`SimpleImputer(strategy="median")` 在 `fit` 时算中位数，同样用了全量数据。
虽然影响比前两个小，但仍然是泄漏。

#### ✅ 正确顺序

```yaml
# 本手册修正后的 pipeline_config.yaml
steps:
  - load_data
  - split_data              # ← 第 2 步就切分
  - preprocess              # ← 只对 train fit
  - feature_engineering
  - feature_selection       # ← 只对 train fit
  - train
  - evaluate
  - save_artifacts
```

```python
# 正确的 orchestrator 写法
data = load_data(path)
train, val, test = split_data(data)          # 先切

# fit 只碰 train，其他只 transform
preprocessor.fit(train[feature_cols])
X_train = preprocessor.transform(train[feature_cols])
X_val   = preprocessor.transform(val[feature_cols])
X_test  = preprocessor.transform(test[feature_cols])

# 特征筛选同理：只用训练集 + 训练集标签来 fit
selector.fit(X_train, y_train)
X_train = selector.transform(X_train)
X_val   = selector.transform(X_val)
X_test  = selector.transform(X_test)
```

**核心原则一句话**：

> **任何带 `fit` 的东西，`fit` 的时候只能看到训练集。**
> 这条规则适用于：Scaler、Imputer、Encoder、Selector、PCA、任何 sklearn 的 Transformer。

#### 🔍 怎么自查

```cmd
:: 方法 1：打印流水线顺序
python -c "from src.pipeline.orchestrator import TrainingPipeline; p=TrainingPipeline('configs'); print(p.steps)"
```

看到 `preprocess` 排在 `split_data` 前面 → **中了**。

```cmd
:: 方法 2：做对照实验，量化泄漏量
:: 把 configs/pipeline_config.yaml 里的顺序改成错的，复制一份到 configs_leak/
python scripts\train_pipeline.py --use-sample --skip-mlp --config-dir configs_leak
```

对比两次的 AUC：

| 场景 | AUC | 说明 |
|---|---|---|
| 正确顺序 | 0.78 | 真实水平 |
| 错误顺序 | 0.81 | 泄漏量 = 0.03 |

**本项目实测泄漏量约 2~3 个 AUC 点。** 这个数字会随数据规模变化：测试集越大，泄漏越严重。

```cmd
:: 方法 3：代码审查，检查 fit 的入参
```

打开 `src/pipeline/orchestrator.py`，找所有 `.fit(`：

```python
✅ preprocessor.fit(X_train)
✅ selector.fit(X_train, y_train)
❌ preprocessor.fit(X_all)        ← 泄漏
❌ selector.fit(X_all, y_all)     ← 泄漏
```

#### 🧠 为什么这个错误这么常见

因为它**看起来完全合理**。人的直觉是："先把数据洗干净，再切分，这样逻辑更清晰。"
而且绝大多数教程都这么写——它们只演示 `train_test_split`，不演示完整流水线，自然不会暴露这个问题。

Kaggle 上的高分 notebook 也经常这么写，因为竞赛只关心榜上分数，不关心上线。

**但真实工程里，"指标可信"比"指标好看"重要一百倍。**

---

## B.2 E 级 · 照抄必报错（23 条）

> 这 23 条按"你会在第几章遇到"排序。每条给：原文 / 报错 / 修正 / 自查。

---

### E-01 · `derived_features` 少一层缩进

**❌ 原文（`data_config.yaml`）**

```yaml
derived_features:
  - name: dti
    formula: "installment / monthly_income"
  - name: ltv
    formula: "loan_amount / (annual_inc * 3)"
```

**💥 报错**

```
KeyError: 'derived_features'
```
或者读到 `None`，然后遍历时报 `TypeError: 'NoneType' object is not iterable`。

**✅ 修正**

```yaml
derived_features:
  rules:                              # ← 少了这一层
    - name: dti
      formula: "installment / monthly_income"
    - name: ltv
      formula: "loan_amount / (annual_inc * 3)"
```

**原因**：代码里读的是 `config["derived_features"]["rules"]`，YAML 少了这层就取不到。

**🔍 自查**：`python -c "from src.utils.config_loader import ConfigLoader; print(ConfigLoader('configs').load_all().get('data').derived_features.keys())"`
应输出含 `rules` 的键列表。

---

### E-02 · 评估器参数名写错

**❌ 原文**

```python
result = evaluator.evaluate(y_true=y_val, y_proba=proba)
```

**💥 报错**

```
TypeError: Evaluator.evaluate() got an unexpected keyword argument 'y_proba'
```

**✅ 修正**

```python
result = evaluator.evaluate(y_true=y_val, y_pred_proba=proba)
```

**🔍 自查**：`grep -rn "y_proba=" src/ scripts/ tests/`（注意区分 `y_pred_proba=`）

> 源文档里这个错误出现 **4 次**，分属第 02、09 章的评估调用。

---

### E-03 · 配置用点号直接取

**❌ 原文**

```python
learning_rate = configs.model.xgboost.params.learning_rate
```

**💥 报错**

```
AttributeError: 'dict' object has no attribute 'model'
```

**✅ 修正**（二选一）

```python
# 方案 A：显式 get（推荐，最稳）
learning_rate = configs.get("model").xgboost.params.learning_rate

# 方案 B：如果 ConfigLoader 已经返回 AttrDict
learning_rate = configs.model.xgboost.params.learning_rate
```

**为什么**：`load_all()` 返回的是普通 `dict`，不是 `AttrDict`。要支持点号访问，得递归包一层 `AttrDict`（见第 02 章）。

**🔍 自查**：`python -c "from src.utils.config_loader import ConfigLoader; c=ConfigLoader('configs').load_all(); print(type(c))"`
输出 `<class 'dict'>` → 必须用 `.get()`；输出 `AttrDict` → 点号可用。

---

### E-04 · `EvalResult` 调了不存在的方法

**❌ 原文**

```python
print(result.summary())
```

**💥 报错**

```
AttributeError: 'EvalResult' object has no attribute 'summary'
```

**✅ 修正**

```python
print(result.to_dict())
# 或者直接打印字段
print(f"AUC={result.auc:.4f}, KS={result.ks:.4f}")
```

**🔍 自查**：`python -c "from src.utils.metrics import EvalResult; print([m for m in dir(EvalResult) if not m.startswith('_')])"`

---

### E-05 · `Preprocessor` 少传一份配置

**❌ 原文**

```python
preprocessor = Preprocessor(config)
```

**💥 报错**

```
TypeError: Preprocessor.__init__() missing 1 required positional argument: 'feature_config'
```

**✅ 修正**

```python
preprocessor = Preprocessor(
    data_config=configs.get("data"),
    feature_config=configs.get("features"),   # ← 别漏
)
```

**为什么**：预处理既要读 `data_config`（哪些是数值/类别/要丢弃的列），又要读 `features_config`（缩放方式、缺失值策略）。

**🔍 自查**：`python -c "import inspect; from src.data.preprocessor import Preprocessor; print(inspect.signature(Preprocessor.__init__))"`

---

### E-06 · `DataSplitter` 传了整块配置

**❌ 原文**

```python
splitter = DataSplitter(configs)
```

**💥 报错**

```
AttributeError: 'ConfigLoader' object has no attribute 'test_size'
```
或更隐蔽：用了默认值 0.2，你以为改的配置没生效。

**✅ 修正**

```python
splitter = DataSplitter(configs.get("data").split)   # 只传 split 这一块
```

**为什么**：`DataSplitter` 只关心 `test_size` / `val_size` / `stratify` / `random_state`，这些都挂在 `data.split` 下面。

**🔍 自查**：打印 `configs.get("data").split` 看结构对不对。

---

### E-07 · `_safe_eval` 没有注入列名

**❌ 原文**

```python
def _safe_eval(self, formula: str, row: pd.Series):
    return eval(formula, {"__builtins__": {}})
```

注释里写着"已注入列名"，实际没注入。

**💥 报错**

```
NameError: name 'installment' is not defined
```

**✅ 修正**

```python
def _safe_eval(self, formula: str, row: pd.Series):
    # 把这一行的所有列名 + 值注入命名空间
    namespace = {k: v for k, v in row.items()}
    namespace["__builtins__"] = {}          # 禁掉内置函数，防注入
    try:
        return eval(formula, namespace)
    except Exception:
        return np.nan                        # 单行算错不影响整批
```

**为什么**：`eval` 的第二个参数是 globals，公式里的 `installment` 要从这里找到。不注入就是 `NameError`。

**安全提醒**：`__builtins__: {}` 是必要的。不加的话，配置里写 `__import__('os').system('rm -rf /')` 会被执行。

**🔍 自查**：跑 `pytest tests\test_basic.py::test_feature_builder -v`，应 PASSED。

---

### E-08 · `use_label_encoder` 已废弃

**❌ 原文**

```python
model = XGBClassifier(
    use_label_encoder=False,
    eval_metric="auc",
)
```

**💥 报错**（XGBoost 2.0+）

```
TypeError: XGBClassifier.__init__() got an unexpected keyword argument 'use_label_encoder'
```

**✅ 修正**

```python
model = XGBClassifier(
    eval_metric="auc",     # 删掉 use_label_encoder 这一行
)
```

**为什么**：XGBoost 1.6 起这个参数被标记废弃，2.0 起**彻底删除**。源文档写的是 1.x 时代的写法。

**🔍 自查**：`python -c "import xgboost; print(xgboost.__version__)"`，≥ 2.0 就要注意。

---

### E-09 · `XGBModel.load` 被当类方法调用

**❌ 原文**

```python
model = XGBModel.load("models/xxx/xgb_model.json")
```

**💥 报错**

```
TypeError: load() missing 1 required positional argument: 'self'
```

**✅ 修正**

```python
model = XGBModel(**configs.get("model").xgboost.params)
model.load("models/xxx/xgb_model.json")     # 实例方法
```

**为什么**：`load` 是实例方法（要读 `self._feature_names`、要设置 `self.model`），不是 `@classmethod`。

**🔍 自查**：`python -c "import inspect; from src.models.xgb_model import XGBModel; print(inspect.signature(XGBModel.load))"`
参数里第一个是 `self` 就是实例方法。

---

### E-10 · `predict_proba` 返回了二维数组

**❌ 原文**

```python
def predict_proba(self, X):
    return self.model.predict_proba(X)      # 返回 (n, 2)
```

**💥 报错**

```
ValueError: y_pred should be 1d array, got (2000, 2)
```
或者 AUC 算出来 0.5（被当成多分类了）。

**✅ 修正**

```python
def predict_proba(self, X):
    proba = self.model.predict_proba(X)
    return proba[:, 1]                      # ← 只取正类那一列，(n,)
```

**为什么**：`XGBClassifier.predict_proba` 返回 `(n, 2)`，第 0 列是负类概率，第 1 列是正类。`roc_auc_score` 要一维。

**注意**：`Evaluator` 里也做了兼容：

```python
y_pred_proba = np.asarray(y_pred_proba).ravel()
```

但**模型层应该返回正确的形状**，不要依赖评估层兜底。

**🔍 自查**：`python -c "..."` 打印 `model.predict_proba(X_test).shape`，应为 `(n,)`。

---

### E-11 · 传给 Optuna 的是整块 model 配置

**❌ 原文**

```python
optimizer = OptunaOptimizer(config=configs.get("model"))
```

**💥 报错**

```
KeyError: 'search_space'
```
或者 Optuna 报参数名不存在（因为读到了错误的 `search_space`）。

**✅ 修正**

```python
optimizer = OptunaOptimizer(config=configs.get("model").xgboost.tuning)
#                                          ^^^^^^^^^^^^^^^^^^^^^^^^^^
```

**为什么**：`search_space` 挂在 `model.xgboost.tuning` 下面，不在 `model` 顶层。

**🔍 自查**：`python -c "...print(configs.get('model').xgboost.tuning.keys())"`，应含 `search_space`。

---

### E-12 · SHAP 三个方法签名全错

**❌ 原文**

```python
analyzer = SHAPAnalyzer(model, background)
exp = analyzer.explain(X_single)
vals = analyzer.get_shap_values(X)
imp = analyzer.importance()
```

**💥 报错**

```
TypeError: SHAPAnalyzer.__init__() takes 2 positional arguments but 3 were given
TypeError: explain() takes 1 positional argument but 2 were given
AttributeError: 'SHAPAnalyzer' object has no attribute 'get_shap_values'
```

**✅ 修正**

```python
analyzer = SHAPAnalyzer(max_background_size=1000)   # 构造只吃参数
analyzer.fit(model, X_background)                    # 模型和背景在这里给
exp  = analyzer.explain_single(X_single)             # 单笔
exps = analyzer.explain_batch(X_batch)               # 批量
imp  = analyzer.global_importance(X_sample)          # 全局
```

**为什么**：原文档把"构造"和"绑定模型"混在一起，且方法名不一致。

**🔍 自查**：`python -c "import inspect; from src.explain.shap_analyzer import SHAPAnalyzer; print(inspect.signature(SHAPAnalyzer.fit)); print([m for m in dir(SHAPAnalyzer) if not m.startswith('_')])"`

---

### E-13 · `DeepExplainer` 收到包装类

**❌ 原文**

```python
explainer = shap.DeepExplainer(model, background)    # model 是 MLPModel
```

**💥 报错**

```
AttributeError: 'MLPModel' object has no attribute '_modules'
```
或
```
AttributeError: 'MLPModel' object has no attribute 'children'
```

**✅ 修正**

```python
# 把内部的 nn.Module 拆出来
net = getattr(model, "network", model)
explainer = shap.DeepExplainer(net, background_tensor)
```

**为什么**：`DeepExplainer` 要求传入 `torch.nn.Module`，它会访问 `_modules` / `children()` 遍历网络层。`MLPModel` 是包装类，只有 `.network` 属性指向真正的 `nn.Module`。

`getattr(model, "network", model)` 的写法能同时兼容"包装类"和"直接传 nn.Module"两种情况。

**🔍 自查**：`python -c "print(type(model)); print(hasattr(model, 'network'))"`

---

### E-14 · `PromptManager` 传了路径字符串

**❌ 原文**

```python
pm = PromptManager("configs/prompts")
```

**💥 报错**

```
AttributeError: 'str' object has no attribute 'get'
```
或 `TypeError: PromptManager.__init__() got an unexpected keyword argument ...`

**✅ 修正**

```python
pm = PromptManager(configs.get("llm").prompts)
```

**为什么**：`PromptManager` 吃的是配置对象（里面有 `template_dir` / `default_system` 等字段），不是路径。

**🔍 自查**：`python -c "import inspect; from src.llm.prompts import PromptManager; print(inspect.signature(PromptManager.__init__))"`

---

### E-15 · 脱敏在拼消息之后（**安全级**）

**❌ 原文**

```python
messages = self._build_messages(prompt, system_prompt, history)
safe_messages = self._sanitize_input(messages)      # 太晚了
```

**💥 后果**

**不会报错，但脱敏等于白做。**

1. `_build_messages` 内部已经把原文组装进 message，日志里可能已经打印过原文
2. `_sanitize_input` 收到的是 `list[dict]`，正则替换要么失效、要么把 JSON 结构搞坏
3. 最终发给 API 的**可能是未脱敏的原文**

**✅ 修正**

```python
safe_prompt = self._sanitize_input(prompt)                     # 先脱敏
messages = self._build_messages(safe_prompt, system_prompt, history)  # 再拼
```

**为什么顺序重要**：
- 脱敏要作用于**原始文本**
- 一旦拼成 message 列表，结构变了，正则容易误伤
- 而且日志、异常堆栈都在 `_build_messages` 之后产生，晚脱敏等于没脱

**🔍 自查**：在 `_sanitize_input` 里加一行 `logger.debug(f"脱敏前: {text[:50]}")`，在 `_build_messages` 后加一行 `logger.debug(f"发送: {messages}")`。
跑一次，对比两处还有没有真实手机号。

---

### E-16 · LLM 调用参数名错

**❌ 原文**

```python
response = client.chat(prompt, system=system_prompt)
```

**💥 报错**

```
TypeError: DeepSeekClient.chat() got an unexpected keyword argument 'system'
```

**✅ 修正**

```python
response = client.chat(prompt, system_prompt=system_prompt)
```

**🔍 自查**：`python -c "import inspect; from src.llm.client import DeepSeekClient; print(inspect.signature(DeepSeekClient.chat))"`

---

### E-17 · Jinja2 的 `tojson` 没有 `ensure_ascii`

**❌ 原文**

```jinja
{{ data | tojson(ensure_ascii=False) }}
```

**💥 报错**

```
TypeError: tojson() got an unexpected keyword argument 'ensure_ascii'
```

**✅ 修正**

```jinja
{{ data | tojson }}
```

**为什么**：Jinja2 内置的 `tojson` 过滤器**不接受** `ensure_ascii` 参数（那是 Python `json.dumps` 的参数）。Jinja2 的 `tojson` 默认就是 UTF-8 不转义。

**如果确实需要控制编码**，在 Python 侧先 `json.dumps(..., ensure_ascii=False)` 再传进去，模板里直接 `{{ data_json }}`。

**🔍 自查**：模板里搜 `tojson(`，看括号里有没有参数。

---

### E-18 · `ArtifactManager` 少了 `base_dir`

**❌ 原文**

```python
am = ArtifactManager()
```

**💥 报错**

```
TypeError: ArtifactManager.__init__() missing 1 required positional argument: 'base_dir'
```

**✅ 修正**

```python
am = ArtifactManager(base_dir="models")
```

**为什么**：刻意设计成必传。如果给默认值 `"models"`，在不同工作目录下跑会存到不同地方，产物就散了。

**🔍 自查**：搜 `ArtifactManager(`，确认每个调用都带 `base_dir=`。

---

### E-19 · 以为 `load_latest` 会返回模型

**❌ 原文**

```python
artifacts = am.load_latest()
model = artifacts["model"]          # KeyError
proba = model.predict_proba(X)
```

**💥 报错**

```
KeyError: 'model'
```

**✅ 修正**

```python
artifacts = am.load_latest()
# 只返回 {'dir': ..., 'meta': ..., 'eval_results': ...}

# 模型要自己构造再 load
model = XGBModel(**configs.get("model").xgboost.params)
model.load(Path(artifacts["dir"]) / "xgb_model.json")

# 预处理器同理
with open(Path(artifacts["dir"]) / "preprocessor.pkl", "rb") as f:
    preprocessor = pickle.load(f)
```

**为什么这么设计**：`ArtifactManager` 不知道你要加载哪个模型（可能加载 XGB、可能是融合模型），不该替你做决定。

**🔍 自查**：`python -c "...print(am.load_latest().keys())"`

---

### E-20 · CLI 参数根本不存在

**❌ 原文**

```cmd
python scripts\train_pipeline.py --skip-tuning --run-id exp01 --skip-steps preprocess
```

**💥 报错**

```
error: unrecognized arguments: --skip-tuning --run-id exp01 --skip-steps preprocess
```

**✅ 修正**

`train_pipeline.py` **只有 4 个参数**：

```cmd
python scripts\train_pipeline.py --use-sample --skip-mlp --tune --config-dir configs
```

| 参数 | 作用 |
|---|---|
| `--config-dir` | 配置目录，默认 `configs` |
| `--use-sample` | 用合成数据而不是真实数据 |
| `--skip-mlp` | 跳过 MLP，只训 XGBoost（快得多） |
| `--tune` | 启用 Optuna 调参 |

**没有** `--skip-tuning`（是 `--tune` 的反向）、`--run-id`（自动生成）、`--skip-steps`（不支持）。

**🔍 自查**：`python scripts\train_pipeline.py --help`

> 这个错误特别坑，因为**文档里写了这些参数**，你照着敲就报错，然后你会怀疑自己。

---

### E-21 · `@st.cache_resource` 加在条件分支里

**❌ 原文**

```python
# app/pages/4_单笔预测.py
if "model" not in st.session_state:
    @st.cache_resource            # ← 在 if 里面
    def load_model():
        return XGBModel.load(...)
    st.session_state.model = load_model()
```

**💥 后果**

**不报错，但缓存完全不生效。**

每次交互都重新加载模型，页面卡 3~5 秒。你可能以为是"数据大加载慢"，其实是缓存没命中。

**✅ 修正**

```python
# 装饰器必须在模块顶层
@st.cache_resource
def load_model(model_dir: str):
    m = XGBModel(...)
    m.load(Path(model_dir) / "xgb_model.json")
    return m

# 页面逻辑里正常调用
model = load_model(st.session_state.get("model_dir", "models/latest"))
```

**为什么**：Streamlit 的缓存机制靠**函数对象**做 key。函数定义在 `if` 里，每次重跑脚本都会重新定义一个新函数对象，key 就变了，缓存永远不命中。

**同理**：`@st.cache_data` 也不能加在函数内部或循环里。

**🔍 自查**：在 `load_model` 里加 `print("loading...")`，连续点 3 次按钮。如果打印了 3 次 → 缓存失效。

---

### E-22 · `chart_utils` 调了不存在的函数

**❌ 原文**

```python
from app.components.chart_utils import plot_importance, plot_roc
fig = plot_importance(importance_df)
```

**💥 报错**

```
ImportError: cannot import name 'plot_importance' from 'app.components.chart_utils'
```

**✅ 修正**

`chart_utils.py` **只有 5 个函数**：

```python
from app.components.chart_utils import (
    target_distribution,   # 目标分布直方图
    missing_bar,           # 缺失率条形图
    importance_bar,        # 特征重要性条形图
    risk_gauge,            # 风险仪表盘
    shap_waterfall,        # SHAP 瀑布图
)
```

**🔍 自查**：`python -c "from app.components import chart_utils; print([f for f in dir(chart_utils) if not f.startswith('_')])"`

---

### E-23 · 单例导致测试互相污染

**❌ 原文**

```python
def test_a():
    cfg = ConfigLoader("configs").load_all()
    cfg["model"]["xgboost"]["params"]["learning_rate"] = 0.5   # 改了
    ...

def test_b():
    cfg = ConfigLoader("configs").load_all()
    assert cfg["model"]["xgboost"]["params"]["learning_rate"] == 0.05   # 挂了
```

**💥 报错**

```
AssertionError: assert 0.5 == 0.05
```

**症状特征**：单独跑 `test_b` 能过，一起跑就挂。顺序变了挂的测试也变。

**✅ 修正**

```python
import pytest
from src.utils.config_loader import ConfigLoader

@pytest.fixture(autouse=True)
def reset_config_singleton():
    """每个测试前后都重置单例"""
    ConfigLoader._instance = None
    yield
    ConfigLoader._instance = None
```

**为什么**：`ConfigLoader` 是单例，`_instance` 是**类属性**，跨测试存活。第一个测试改了它，第二个读到的是改后的。

**🔍 自查**：
```cmd
pytest tests\test_basic.py::test_config_loader -v      :: 单独跑
pytest tests\ -v                                       :: 一起跑
```
结果不一致 → 就是这个问题。

---

## B.3 W 级 · 不报错但结果错（7 条）

> 这 7 条比 E 级危险：**代码跑得通，你甚至看不出异常**。

---

### W-01 · `_apply_scaler` 用 `hasattr` 判断类型

**❌ 原文**

```python
def _apply_scaler(self, X):
    for col in self._numeric_cols:
        if hasattr(self.scaler, "mean_"):
            X[col] = self.scaler.transform(X[[col]])
        # 否则跳过 —— 打算"MinMax 不需要处理"
    return X
```

**💥 后果**

如果配置里选的是 `minmax` 或 `robust`，这些 scaler **没有 `mean_` 属性**（MinMax 是 `data_min_`/`data_max_`，Robust 是 `center_`/`scale_`）。
结果：**这些列根本没被缩放**，但你不知道。

对 XGBoost 影响不大（树模型对量纲不敏感），但 **MLP 会明显掉点**——神经网络对量纲极敏感。

**✅ 修正**

```python
def _apply_scaler(self, X):
    if not self._scaler_cols:                    # 没学过就跳过
        return X
    X = X.copy()
    X[self._scaler_cols] = self.scaler.transform(X[self._scaler_cols])
    return X
```

配合在 `fit` 里明确记录：

```python
def _fit_scaler(self, X):
    self._scaler_cols = self._numeric_cols.copy()   # 显式记住
    scaler_type = self.feature_config.scaling.method
    if scaler_type == "standard":
        self.scaler = StandardScaler()
    elif scaler_type == "minmax":
        self.scaler = MinMaxScaler()
    elif scaler_type == "robust":
        self.scaler = RobustScaler()
    else:
        raise ValueError(f"未知的缩放方式: {scaler_type}")
    self.scaler.fit(X[self._scaler_cols])
```

**🔍 自查**

```cmd
python -c "import pandas as pd; from src.data.preprocessor import Preprocessor; ...; print(X_train.describe().loc[['mean','std']].round(3))"
```

缩放后：`standard` 的 mean 应接近 0、std 接近 1；`minmax` 的 min 应 0、max 应 1。
**如果还是原始量级 → 中了。**

---

### W-02 · 相关性阈值设太低，特征被删过头

**❌ 原文**

```yaml
feature_selection:
  correlation:
    threshold: 0.80        # 相关性 > 0.8 就删一列
```

**💥 后果**

63 个特征被删到 31 个，AUC 从 0.78 掉到 0.765。
因为 0.8 会删掉"高度相关但各有独立信息"的配对（比如 `loan_amount` 和 `funded_amount` 相关 0.95 就算了，但 `annual_inc` 和 `monthly_income` 相关 0.85，删掉哪个都损失信息）。

**✅ 修正**

```yaml
feature_selection:
  correlation:
    threshold: 0.95        # 只删几乎完全重复的
```

**为什么**：相关性筛选的目的是"去冗余"，不是"去相关"。0.95 以上才基本可以认为两列在表达同一件事。

**🔍 自查**：打印筛选前后的特征数和验证集 AUC，对比几次不同阈值：

```cmd
python scripts\train_pipeline.py --use-sample --skip-mlp    :: threshold=0.95
:: 手动改成 0.8 再跑一次，对比 AUC 和特征数
```

---

### W-03 · CPU 上跑 MLP 50 轮，等到天荒地老

**❌ 原文**

```yaml
mlp:
  epochs: 50
  batch_size: 256
```

**💥 后果**

CPU 上 10 万条数据、50 轮，可能跑 20 分钟以上。你会以为"程序卡死了"。

**✅ 修正**（三选一）

```cmd
:: 方案 1：开发阶段先跳过 MLP
python scripts\train_pipeline.py --use-sample --skip-mlp

:: 方案 2：装 GPU 版 PyTorch（有 N 卡）
pip install torch --index-url https://download.pytorch.org/whl/cu124 -i https://pypi.tuna.tsinghua.edu.cn/simple

:: 方案 3：减小 epochs，加早停
```

```python
# 代码层面：加早停
best_loss = float("inf")
patience_counter = 0
for epoch in range(epochs):
    train_loss = self._train_one_epoch(...)
    val_loss = self._validate(...)
    if val_loss < best_loss:
        best_loss = val_loss
        patience_counter = 0
        torch.save(self.network.state_dict(), "best.pt")
    else:
        patience_counter += 1
        if patience_counter >= patience:
            logger.info(f"早停于第 {epoch} 轮")
            break
```

**另外**：Windows 上 `DataLoader(num_workers>0)` 容易卡死，统一设 `num_workers=0`。

**🔍 自查**：`python -c "import torch; print(torch.cuda.is_available())"`，`False` 就是 CPU。

---

### W-04 · 装了剪枝器但从来不触发

**❌ 原文**

```python
pruner = optuna.pruners.MedianPruner(n_startup_trials=5)
study = optuna.create_study(direction="maximize", pruner=pruner)

def objective(trial):
    params = self._suggest_param(trial)
    model = XGBModel(**params)
    model.fit(X_train, y_train, X_val, y_val)
    return model.best_score            # 只返回最终分数
```

**💥 后果**

**不会报错，也不会有任何提示。** 每个 trial 都跑到底，剪枝一次没生效，你以为省了时间，其实一点没省。

**✅ 修正**

```python
def objective(trial):
    params = self._suggest_param(trial)
    model = XGBModel(**params)

    # 关键：每轮上报中间分数 + 检查是否该剪
    for epoch in range(num_rounds):
        model.train_one_round()
        score = model.eval_on_val()

        trial.report(score, epoch)              # ← 上报
        if trial.should_prune():                # ← 检查
            raise optuna.TrialPruned()          # ← 主动终止

    return score
```

**为什么**：`MedianPruner` 需要"中间分数序列"才能判断某 trial 是否落后于中位数。只返回最终值，它拿不到任何中间信号，只能干看着。

**🔍 自查**

```python
print(study.trials_dataframe()[["number", "state"]])
```

如果 `state` 全是 `COMPLETE`，一个 `PRUNED` 都没有 → 剪枝没生效。

---

### W-05 · 报告写成了因果结论

**❌ 原文**（LLM 生成的报告里）

```
该客户违约风险较高，主要原因是负债收入比过高。
建议提高客户收入以降低违约风险。
```

**💥 后果**

不报错，但这是**专业硬伤**。SHAP 是相关性归因，不能说"提高收入就能降低风险"。
真实场景里，收入不是你能改变的，而且"收入低"和"违约"可能同时由第三个因素（比如失业）导致。

**✅ 修正**

改 prompt 模板，明确禁止因果表述：

```jinja
要求：
1. 只能基于下方提供的数据进行描述，不得推算未提供的数字。
2. 描述归因时使用"与……相关""该特征推高了风险评分"等表述。
3. 严禁使用"因为……所以会……""提高X就能降低Y"等因果句式。
4. 建议部分只能针对"审核流程"，不能针对"改变客户属性"。
```

**正确表述**：

```
该客户的负债收入比为 0.56，处于高风险区间，是本次评分的主要推高因素（+0.15）。
建议：转人工复核，重点核实收入证明的真实性与稳定性。
```

**🔍 自查**：搜报告输出里有没有"提高""降低""导致""因为…所以"这类因果词。

---

### W-06 · 指标虚高但不自知（S-01 的表现）

**💥 症状**

AUC 0.81，KS 0.46，看起来是个好模型。但：
- 和同类公开项目比，高得不正常
- 换个随机种子，波动很大
- 上线后效果差一截

**✅ 修正**

见 [`S-01`](#-s-01--流水线顺序导致数据泄漏)。这是它的"表现层"编号。

**🔍 自查**：按 `附录E` Step 4.1 的三条命令查。

---

### W-07 · 合成数据和真实数据 schema 不一致

**❌ 原文**

```python
# generate_sample_data.py
df["term"] = np.random.choice([36, 60], size=n)          # 整数
df["grade"] = np.random.choice(["A","B","C"], size=n)    # 只有 3 档
df["dti"] = np.random.uniform(0, 100, size=n)            # 百分比
```

**💥 后果**

换成真实数据后：

- `term` 是 `"36 months"` 字符串，你的代码按整数处理 → `TypeError`
- `grade` 真实有 A~G 七档，模型只见过 A/B/C → 遇到 D 报 `unseen category`
- `dti` 真实是 0~1 小数，你的模型学的 0~100 → 预测全错但**不报错**

**✅ 修正**

```python
# 严格对齐真实数据字典
df["term"] = np.random.choice(["36 months", "60 months"], size=n)   # 字符串
df["grade"] = np.random.choice(list("ABCDEFG"), size=n)             # 七档
df["dti"] = np.random.uniform(0, 0.6, size=n)                       # 小数
```

**根本方法**：先拿到真实数据（或它的数据字典），**照着字典造**，不要凭想象。

**🔍 自查**

```cmd
python -c "import pandas as pd; d=pd.read_csv('data/raw/sample_data.csv'); print(d.dtypes); print(d['term'].unique())"
```

和真实数据的 `dtypes` 逐列对比。

---

## B.4 N 级 · 环境陷阱（3 条）

### N-01 · `.env` 变成了 `.env.txt`

**❌ 你怎么建的**

在资源管理器里右键 → 新建 → 文本文档 → 改名 `.env`

**💥 后果**

文件实际叫 `.env.txt`，`load_dotenv()` 找不到，`api_key` 读到空，然后：
- 有 Key 但报"未配置"
- 你以为 Key 填错了，反复检查 Key 本身

**✅ 修正**

```cmd
:: 方法 1：命令行建（推荐）
type nul > .env

:: 方法 2：资源管理器里先开启"显示文件扩展名"
:: 查看 → 显示 → 文件扩展名（勾上）

:: 验证
dir /a | findstr env
```

**🔍 自查**：`dir /a` 看到的是 `.env` 还是 `.env.txt`。

---

### N-02 · YAML 里用了 Tab

**❌ 原文**

```yaml
model:
	xgboost:            # ← 开头是 Tab
		params:
```

**💥 报错**

```
yaml.scanner.ScannerError: while scanning for the next token
found character '\t' that cannot start any token
```

**✅ 修正**

全部换成 2 个空格：

```yaml
model:
  xgboost:
    params:
```

**为什么**：YAML 规范明确禁止用 Tab 做缩进（因为不同编辑器 Tab 宽度不同，会导致歧义）。

**🔍 自查**：编辑器开启"显示空白字符"（VS Code：`Ctrl+Shift+P` → `Toggle Render Whitespace`），或者：

```cmd
python -c "import pathlib; [print(f) for f in pathlib.Path('configs').rglob('*.yaml') if '\t' in f.read_text(encoding='utf-8')]"
```

有输出就是中了。

---

### N-03 · 模型名写成了不存在的东西

**❌ 原文**

```env
DEEPSEEK_MODEL=deepseek-flash
```

**💥 报错**

```
openai.NotFoundError: Error code: 404 - {'error': {'message': 'Model Not Exist'}}
```

**✅ 修正**

```env
DEEPSEEK_MODEL=deepseek-chat
```

可用的模型名（截至写作时）：

| 模型名 | 特点 |
|---|---|
| `deepseek-chat` | 通用对话，**本项目用这个** |
| `deepseek-reasoner` | 推理模型，慢但逻辑强，报告生成不必用 |

**🔍 自查**：`python -c "from openai import OpenAI; ..."` 或直接看官方文档的模型列表。

---

## B.5 按章节反查

写某一章时，先看这章有哪些坑。

| 章 | 涉及的错误编号 | 数量 |
|---|---|---|
| 00 开工准备 | N-01 | 1 |
| 01 配置层 | E-01, N-02 | 2 |
| 02 工具层 | PY-03, E-02, E-04, E-23 | 4 |
| 03 数据层 | E-05, E-06, W-01 | 3 |
| 04 特征层 | E-07, W-02 | 2 |
| 05 模型层 | E-08, E-09, E-10, W-03 | 4 |
| 06 调参层 | E-11, W-04 | 2 |
| 07 解释层 | E-12, E-13 | 2 |
| 08 LLM 层 | E-14, E-15, E-16, E-17, N-03, W-05 | 6 |
| 09 编排层 | E-18, E-19, W-06, **S-01** | 4 |
| 10 脚本层 | E-20, W-07 | 2 |
| 11 Web 层 | E-21, E-22 | 2 |
| 12 测试 | E-23 | 1 |

> 注：`E-23` 同时属于第 02 章（单例实现）和第 12 章（测试），只在第 12 章计入。
> 总计：1+2+4+3+2+4+2+2+6+4+2+2 = **34**。

---

## B.6 按症状反查（报错信息 → 编号）

**看到报错，直接 Ctrl+F 搜这段文字。**

| 报错信息（片段） | 编号 | 一句话解法 |
|---|---|---|
| `'xxx' 不是内部或外部命令` | 附录 D | 你在用 Linux 命令，查 D.1 |
| `KeyError: 'derived_features'` | E-01 | YAML 少了 `rules:` 那层 |
| `KeyError: 'search_space'` | E-11 | 传 `model.xgboost.tuning` |
| `KeyError: 'model'`（load_latest 后） | E-19 | 返回的字典里没有 model，要自己 load |
| `AttributeError: 'dict' object has no attribute 'model'` | E-03 | 用 `configs.get("model")` |
| `AttributeError: 'EvalResult' object has no attribute 'summary'` | E-04 | 用 `to_dict()` |
| `AttributeError: 'str' object has no attribute 'get'` | E-14 | PromptManager 传 config 不是路径 |
| `AttributeError: 'MLPModel' object has no attribute '_modules'` | E-13 | `getattr(model, "network", model)` |
| `AttributeError: cannot import name 'plot_xxx'` | E-22 | chart_utils 只有 5 个函数 |
| `TypeError: evaluate() got an unexpected keyword argument 'y_proba'` | E-02 | 是 `y_pred_proba` |
| `TypeError: Preprocessor.__init__() missing 1 required positional argument` | E-05 | 要传 data_config + feature_config |
| `TypeError: XGBClassifier.__init__() got an unexpected keyword argument 'use_label_encoder'` | E-08 | 删掉这一行 |
| `TypeError: load() missing 1 required positional argument: 'self'` | E-09 | 先实例化再 load |
| `TypeError: SHAPAnalyzer.__init__() takes 2 positional arguments but 3 were given` | E-12 | 构造只吃参数，fit 才给模型 |
| `TypeError: chat() got an unexpected keyword argument 'system'` | E-16 | 是 `system_prompt=` |
| `TypeError: tojson() got an unexpected keyword argument 'ensure_ascii'` | E-17 | 去掉参数 |
| `TypeError: ArtifactManager() missing 1 required positional argument: 'base_dir'` | E-18 | 传 `base_dir="models"` |
| `NameError: name 'installment' is not defined` | E-07 | `_safe_eval` 注入列名 |
| `ValueError: y_pred should be 1d array` | E-10 | `return proba[:, 1]` |
| `error: unrecognized arguments: --skip-tuning` | E-20 | 只有 `--tune` |
| `ImportError: cannot import name 'plot_importance'` | E-22 | 用 `importance_bar` |
| `AssertionError: assert 0.5 == 0.05`（测试间互相影响） | E-23 | 重置 `ConfigLoader._instance = None` |
| `yaml.scanner.ScannerError: found character '\t'` | N-02 | YAML 换空格 |
| `openai.NotFoundError: Model Not Exist` | N-03 | 用 `deepseek-chat` |
| `FileNotFoundError: configs/xxx.yaml` | — | 必须在**项目根目录**跑 |
| `ModuleNotFoundError: No module named 'src'` | PY-07 | 脚本开头插 `sys.path` |
| 模型加载慢 / 每次点击都转圈 | E-21 | `@st.cache_resource` 提到模块顶层 |
| 指标特别好但心里没底 | **S-01** | 查流水线顺序 |
| 缩放后数据量纲没变 | W-01 | 别用 `hasattr(mean_)` 判断 |
| 特征被删太多、AUC 下降 | W-02 | 相关性阈值调到 0.95 |
| MLP 训练极慢 | W-03 | `--skip-mlp` 或用 GPU |
| 剪枝从不触发 | W-04 | `trial.report` + `TrialPruned` |
| 报告出现因果表述 | W-05 | 改 prompt 禁止因果句 |
| 换真实数据后各种错 | W-07 | 对齐数据字典 |

---

## B.7 最容易中的 5 条（按"新手踩坑率"排序）

| 排名 | 编号 | 踩坑率 | 为什么容易中 |
|---|---|---|---|
| 1 | **E-02** | 极高 | `y_proba` vs `y_pred_proba`，太像了 |
| 2 | **E-03** | 极高 | `config.model` 这种写法太自然了 |
| 3 | **E-20** | 高 | 文档里写了不存在的 CLI 参数 |
| 4 | **E-09** | 高 | `load()` 看起来就像类方法 |
| 5 | **W-01** | 中高 | `hasattr` 那个判断看起来很"聪明" |

**但真正该警惕的是第 6 名：`S-01`。**
它踩坑率可能不是最高（因为很多人根本没意识到要检查），但**后果最严重**。

---

## B.8 一句话总结

```
E 级 23 条：让你跑不起来，改完就好
W 级  7 条：让你跑错，改完就好，但你得先发现
N 级  3 条：让你怀疑自己，改完就好
S 级  1 条：让你以为自己成功了
```

**记住 S-01。其他的，遇到再查。**

---

## 附录 B 自测

1. `S-01` 和 `W-06` 是什么关系？
2. 为什么 `E-15`（脱敏顺序）被归到 E 级而不是 W 级？
3. 你看到 `AttributeError: 'dict' object has no attribute 'model'`，该查哪条？
4. `_apply_scaler` 里用 `hasattr(self.scaler, "mean_")` 判断有什么问题？
5. `trial.should_prune()` 装了但不生效，最可能漏了什么？
6. 34 条里，哪一条"改完之后指标会变差"，而且是应该的？

<details>
<summary>答案</summary>

1. **W-06 是 S-01 的表现**。`S-01` 是根因（流水线顺序错），`W-06` 是你在外面看到的现象（指标虚高但不自知）。修 `S-01` 就等于修 `W-06`。
2. 因为 `E-15` 的**结果是功能失效**（脱敏等于没做），而且是有明确"正确写法"的确定性错误。它不是"结果偏差"，是"这一步根本没起作用"。从安全角度，它比一般 W 级严重。
3. **`E-03`**：`configs.model` 应改成 `configs.get("model")`。
4. `mean_` 只有 `StandardScaler` 有。`MinMaxScaler` 是 `data_min_`/`data_max_`，`RobustScaler` 是 `center_`/`scale_`。用 `hasattr(mean_)` 判断，会导致后两种 scaler **完全不生效**，且不报错。
5. 漏了在 objective 里 **`trial.report(score, step)`** 上报中间分数，以及检查后 **`raise optuna.TrialPruned()`**。不 report，剪枝器收不到信号。
6. **`S-01`**。修正后 AUC 会从 0.81 降到 0.78。**但这个下降是正确的**——它挤掉了虚高的水分，留下的是真实水平。

</details>

---

← 上一篇 [`附录A_知识点速查册.md`](附录A_知识点速查册.md) ｜ 下一篇 → [`附录E_验收Checklist.md`](附录E_验收Checklist.md) ｜ 返回 [`00_总览与学习路线.md`](00_总览与学习路线.md)
