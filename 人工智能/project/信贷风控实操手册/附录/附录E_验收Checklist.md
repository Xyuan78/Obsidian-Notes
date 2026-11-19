# 附录 E · 验收 Checklist

> **这份附录解决什么问题**
> 你按手册一路做下来，**怎么知道自己真的做完了**？
> "能跑起来"不等于"做对了"。很多人项目跑通了，指标却 0.99，一查是数据泄漏。
>
> 这份附录给你一套**可勾选、可复现**的验收流程：
> - **E.1 六步验收**：从环境到网页，每步给命令、期望输出、失败对策
> - **E.2 交付物清单**：61 个文件逐个勾
> - **E.3 34 个已知坑自查**：症状 → 怎么查 → 中了怎么修
> - **E.4 指标合理性**：你的 AUC 到底是好还是假的好
> - **E.5 一页纸验收单**：打印出来贴墙上

---

## E.0 怎么用这份清单

**建议做法**：把这份文件复制一份，命名 `我的验收记录.md`，边做边勾。

```markdown
- [x] 环境里 pandas 能 import
- [ ] 训练跑通
```

`- [ ]` 改成 `- [x]` 就是打勾。

**验收的黄金原则**：

> **必须在"干净的、和你不熟的"环境里验一次。**

你现在的环境已经被你折腾过几十次了，很多问题被"碰巧装上的包"掩盖了。
真正交付前，用 E.1 Step 0 的方法**新建一个环境**重跑一遍。这一步会暴露 80% 的隐藏问题。

---

## E.1 六步验收

### Step 0 · 环境能立起来

**目的**：证明"别人拿到你的 `requirements.txt` 能装出一模一样的环境"。

```cmd
:: 1. 建一个全新的环境（不要复用旧的）
conda create -n cr_verify python=3.10 -y
conda activate cr_verify

:: 2. 确认 python 位置对（关键！）
where python
```

**✅ 期望输出**：路径里含 `envs\cr_verify`

```cmd
:: 3. 装依赖
cd /d E:\PythonCode\11.4\credit_risk_system
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

:: 4. 核心库能 import
python -c "import pandas, numpy, sklearn, xgboost, shap, optuna, streamlit, plotly, yaml, jinja2, loguru, pydantic, dotenv, openai; print('ALL OK')"
```

**✅ 期望输出**：`ALL OK`

```cmd
:: 5. 版本对齐
python -c "import pandas,numpy,sklearn,xgboost,shap,optuna,torch; print('pandas',pandas.__version__); print('numpy',numpy.__version__); print('sklearn',sklearn.__version__); print('xgboost',xgboost.__version__); print('shap',shap.__version__); print('optuna',optuna.__version__); print('torch',torch.__version__, torch.cuda.is_available())"
```

**✅ 参考范围**：

| 库 | 参考版本 | 版本不对会怎样 |
|---|---|---|
| pandas | 2.0 ~ 2.2 | 2.x 的 `inplace` / `applymap` 行为有变 |
| numpy | 1.26 ~ 2.x | numpy 2.x 与老版 shap 有 ABI 冲突 |
| scikit-learn | 1.3 ~ 1.5 | 1.2 以前的 `OneHotEncoder` 默认 `sparse=True` |
| xgboost | 2.0+ | 2.0 起 `use_label_encoder` 已删除 |
| shap | 0.44+ | 老版本对 xgboost 2.x 支持不好 |
| optuna | 3.x / 4.x | 3.x 起 `study.optimize` 参数名有变 |
| torch | 2.x | 2.0+ 支持 `weights_only` 参数 |

**❌ 失败对策**

| 症状 | 原因 | 修法 |
|---|---|---|
| `'conda' 不是内部或外部命令` | 没装 conda / 没加 PATH | 改用 Anaconda Prompt |
| `where python` 显示 `anaconda3\python.exe` | 没激活环境 | `conda activate cr_verify` |
| 装 torch 卡住不动 | 走的默认源，网速慢 | 加 `-i https://pypi.tuna.tsinghua.edu.cn/simple` |
| `ERROR: Could not find a version that satisfies...` | 版本约束太严 | 放宽 `requirements.txt` 里的 `==` 为 `>=` |
| numpy 装完 shap 报 `_ARRAY_API not found` | numpy 2.x 与旧 shap 不兼容 | 升 shap 到 0.44+，或降 numpy 到 1.26 |

- [ ] Step 0 通过

---

### Step 1 · 配置能读进来

**目的**：证明 5 个 YAML 语法对、路径对、环境变量插值生效。

```cmd
python -c "from src.utils.config_loader import ConfigLoader; c = ConfigLoader('configs').load_all(); print('data:', c.get('data').path); print('target:', c.get('data').target_column); print('lr:', c.get('model').xgboost.params.learning_rate); print('key前6位:', str(c.get('llm').api_key)[:6])"
```

**✅ 期望输出**（示意）：

```
data: data/raw/cs-training.csv
target: isDefault
lr: 0.05
key前6位: sk-xxx
```

**关键检查点：**

| 检查 | 怎么查 | 不正常说明 |
|---|---|---|
| 5 个 YAML 都读到了 | `c.keys()` | 少一个 → 文件名拼错或不在 `configs/` 下 |
| `${VAR:-默认}` 被替换 | `print(c.get('llm').api_key)` | 打印出 `${DEEPSEEK_API_KEY}` 原文 → 插值没生效 |
| 没有 Key 也不崩 | 删掉 `.env` 再跑一遍 | 崩了 → 应该"只警告不中断" |
| 点号访问能用 | `c.get('model').xgboost` | `AttributeError` → `AttrDict` 没生效 |

**❌ 失败对策**

| 报错 | 原因 | 修法 |
|---|---|---|
| `yaml.scanner.ScannerError: found character '\t'` | YAML 里用了 Tab | 全换成 2 空格（见 N-02） |
| `FileNotFoundError: configs/xxx.yaml` | 工作目录不对 | 必须在**项目根目录**跑，不是在 `src\` 里 |
| `AttributeError: 'dict' object has no attribute 'xgboost'` | 没用 `AttrDict` 包 | 见第 02 章 `AttrDict` 实现 |
| `KeyError: 'derived_features'` | YAML 层级写错 | `derived_features.rules` 是**两层**，见 E-01 |

- [ ] Step 1 通过

---

### Step 2 · 单元测试全绿

**目的**：证明各模块单独能工作，接口没写错。

```cmd
pytest tests\ -v
```

**✅ 期望输出**：

```
tests/test_basic.py::test_config_loader PASSED          [  9%]
tests/test_basic.py::test_evaluator PASSED              [ 18%]
tests/test_basic.py::test_find_best_threshold PASSED    [ 27%]
tests/test_basic.py::test_splitter PASSED               [ 36%]
tests/test_basic.py::test_preprocessor PASSED           [ 45%]
tests/test_basic.py::test_feature_builder PASSED        [ 54%]
tests/test_basic.py::test_xgb_train PASSED              [ 63%]
tests/test_basic.py::test_prompt_manager PASSED         [ 72%]
tests/test_basic.py::test_shap PASSED                   [ 81%]
tests/test_basic.py::test_scripts_importable PASSED     [ 90%]
tests/test_basic.py::test_pipeline_end_to_end PASSED    [100%]

======================= 11 passed in 8.42s =======================
```

**关键：11 个全过，一个不能少。**

如果只过了 8 个就 `passed`，说明你有 3 个用例被 `@pytest.mark.skip` 跳过了，那是自己骗自己。

```cmd
:: 确认没有被跳过的
pytest tests\ -v --no-header -rs
```

`-rs` 会列出所有 skip 原因。

**❌ 失败对策**

| 报错 | 原因 | 修法 |
|---|---|---|
| 第 1 个测试过、第 2 个就挂了 | 单例测试污染 | 测试开头 `ConfigLoader._instance = None`（见 E-23） |
| `TypeError: evaluate() got an unexpected keyword argument 'y_proba'` | 参数名写错 | 是 `y_pred_proba`（见 E-02） |
| `NameError: name 'installment' is not defined` | 公式没注入列名 | `_safe_eval` 要注入（见 E-07） |
| `AttributeError: 'EvalResult' object has no attribute 'summary'` | 调了不存在的方法 | 用 `to_dict()`（见 E-04） |
| `ModuleNotFoundError: No module named 'src'` | 缺 `sys.path` 注入 | 见 PY-07 |
| `torch.cuda.OutOfMemoryError` | 显存不够 | 测试里用 CPU 或减小 batch |

- [ ] Step 2 通过

---

### Step 3 · 合成数据造得对

**目的**：证明数据 schema 和真实数据一致，标签分布合理。

```cmd
python scripts\generate_sample_data.py
```

**✅ 期望输出**：

```
[INFO] 生成 10000 条样本
[INFO] 违约率: 0.1987
[INFO] 保存至: data/raw/sample_data.csv
[INFO] 字段数: 47
```

**必须检查的 4 件事：**

```cmd
:: 1. 违约率在 15%~25% 之间（不能是 0 或 1）
python -c "import pandas as pd; d=pd.read_csv('data/raw/sample_data.csv'); print('违约率', round(d.isDefault.mean(),4)); print('样本数', len(d))"
```

```cmd
:: 2. 没有全空的列，没有全是同一个值的列
python -c "import pandas as pd; d=pd.read_csv('data/raw/sample_data.csv'); n=d.isnull().sum(); print('全空列:', list(n[n==len(d)].index)); print('缺失率>50%:', list((n/len(d))[n/len(d)>0.5].index))"
```

**✅ 期望**：两个都是空列表

```cmd
:: 3. 关键字段类型对（term 是字符串不是数字，grade 是 A~G）
python -c "import pandas as pd; d=pd.read_csv('data/raw/sample_data.csv'); print(d[['term','grade','home_ownership']].dtypes); print(d['term'].unique()[:5])"
```

**✅ 期望**：`term` 是 `object`，值是 `['36 months' '60 months']` 之类，**不是** `36` / `60`

```cmd
:: 4. 数值字段的量纲和真实数据接近（月收入不可能是 5 位数以外的离谱值）
python -c "import pandas as pd; d=pd.read_csv('data/raw/sample_data.csv'); print(d[['monthly_income','loan_amount','dti']].describe().round(2))"
```

**✅ 期望**：月收入中位数在 3000~8000 之间，`dti` 在 0~1 之间（不是 0~100）

**❌ 失败对策**

| 症状 | 原因 | 修法 |
|---|---|---|
| 违约率 0.001 或 0.99 | 截距项太大/太小 | 调 `intercept=-4.2` 那个值（见第 10 章） |
| `term` 是数字 36/60 | 按自己的想法造了 | 改成 `"36 months"`，对齐真实 schema（W-07） |
| `dti` 在 0~100 | 单位换成百分比 | 真实数据是小数，除以 100 |
| 字段数和 47 不符 | 少造了字段 | 对照第 10 章的字段表 |

- [ ] Step 3 通过

---

### Step 4 · 训练跑得出合理指标

**目的**：这是最关键的一步——指标好不好，直接决定项目成败。

```cmd
python scripts\train_pipeline.py --use-sample --skip-mlp
```

**✅ 期望输出**（示意，数字会浮动）：

```
[INFO] Step 1/8 load_data ...
[INFO] Step 2/8 split_data ...  train=6400 val=1600 test=2000
[INFO] Step 3/8 preprocess ...
[INFO] Step 4/8 feature_engineering ...  47 -> 63 特征
[INFO] Step 5/8 feature_selection ...    63 -> 38 特征
[INFO] Step 6/8 train ...                (xgboost 训练中)
[INFO] Step 7/8 evaluate ...             AUC=0.7821  KS=0.4102
[INFO] Step 8/8 save_artifacts ...       models/20250101_143022/
[INFO] 完成，耗时 42.3s
```

**E.4 有详细的指标判断表，这里只列红线：**

| 指标 | 红线 | 超出红线的含义 |
|---|---|---|
| 验证 AUC | `< 0.65` | 特征没造好 / 训练没跑对 |
| 验证 AUC | `> 0.92` | **疑似数据泄漏**，去查 Step 4.1 |
| 训练 AUC - 验证 AUC | `> 0.15` | 过拟合，要加正则/减特征 |
| KS | `< 0.20` | 区分能力不足 |
| KS | `> 0.65` | 同样疑似泄漏 |

#### Step 4.1 · 数据泄漏专项检查（**必做**）

这是全项目**最容易出错、又最不报错**的地方。用三条命令查：

```cmd
:: 检查 1：步骤顺序里 split 是不是第 2 位
python -c "from src.pipeline.orchestrator import TrainingPipeline; p=TrainingPipeline('configs'); print([s for s in p.steps])"
```

**✅ 期望**：列表第 2 项是 `split_data` 或 `split`。
**❌ 如果 `preprocess` 在 `split` 前面** → 你的指标是假的，虚高约 2 个 AUC 点。

```cmd
:: 检查 2：跑一次"故意泄漏"的版本，对比差距
python scripts\train_pipeline.py --use-sample --skip-mlp --config-dir configs_leak_test
```

（把 `pipeline_config.yaml` 里的 steps 顺序换成错的，跑一次，记住 AUC）

**如果正确顺序 AUC 0.78、错误顺序 AUC 0.80+，说明你的泄漏量约 0.02。**
**如果两者一样**，说明你的 `Preprocessor` 恰好没在 fit 时受影响，也要去代码里确认 `fit` 只接收了 `X_train`。

```cmd
:: 检查 3：确认 preprocess 那一步喂进去的是训练集
```

打开 `src/pipeline/orchestrator.py`，找到 `_step_preprocess`，确认：

```python
# ✅ 正确
preprocessor.fit(X_train)
X_train = preprocessor.transform(X_train)
X_val   = preprocessor.transform(X_val)      # 只 transform
X_test  = preprocessor.transform(X_test)     # 只 transform

# ❌ 错误（泄漏）
preprocessor.fit(X_all)                       # 全量 fit
```

**❌ 训练失败对策**

| 报错 | 原因 | 修法 |
|---|---|---|
| `ArtifactManager() missing 1 required positional argument: 'base_dir'` | 少传参数 | `ArtifactManager(base_dir="models")`（E-18） |
| `TypeError: load() missing 1 required positional argument: 'self'` | `load` 是实例方法 | 先 `m = XGBModel(...)` 再 `m.load(path)`（E-09） |
| `ValueError: feature_names mismatch` | 列顺序不一致 | 保存并复用 `_feature_names`（EN-12） |
| `XGBoostError: use_label_encoder` | 老参数 | XGBoost 2.0 已删，去掉（E-08） |
| `num_workers` 相关卡死 | Windows 多进程 | 设 `num_workers=0`（W-03 相关） |
| 训练 50 轮要 20 分钟 | CPU 跑 MLP | `--skip-mlp` 或用 GPU（W-03） |

- [ ] Step 4 通过
- [ ] Step 4.1 数据泄漏检查通过

---

### Step 5 · 网页能开、能预测、能出报告

```cmd
streamlit run app\streamlit_app.py
```

**✅ 期望**：浏览器自动打开 `http://localhost:8501`，左侧看到 7 个页面。

**逐页检查（每页至少点一次）：**

| # | 页面 | 必须验证的事 | 失败说明 |
|---|---|---|---|
| 1 | 数据探索 | 能看到样本数、违约率、直方图 | 读不到数据 → 路径或 CSV 编码 |
| 2 | 特征工程 | 能看到筛选前后的特征数 | 特征列表没生成 → 检查产物目录 |
| 3 | 模型训练 | 能看到历史训练记录 | 没记录 → `models/` 下没有 `meta.json` |
| 4 | 单笔预测 | **填表 → 得到概率 + 风险等级 + SHAP 图** | 核心页面，必须通 |
| 5 | 模型解释 | 能看到全局重要性条形图 | 图不出来 → 检查 `_feature_names` |
| 6 | AI 助手 | 能生成报告文字（或明确提示"未配置 Key"） | 没 Key 应该优雅降级，不是崩溃 |
| 7 | 系统配置 | 能看到当前配置（脱敏后的） | 显示了完整 API Key → **严重安全问题** |

**特别检查：**

```cmd
:: 检查 1：模型是不是只加载一次（不是每次都重载）
```

在页面上连续点 5 次按钮。如果每次都转圈 3 秒 → `@st.cache_resource` 没加对（E-21）。

```cmd
:: 检查 2：单笔预测页刷新后，上次的输入还在吗？
```

刷新页面（F5），如果输入全清空 → 正常。如果换个 tab 回来就清空 → `session_state` 没用上。

```cmd
:: 检查 3：AI 报告是不是"打字机"效果？
```

如果是"转圈 10 秒 → 一次性弹出全部文字" → `stream_chat` 的 `yield` 没写对，或者是 `chat()` 而不是 `stream_chat()`。

**❌ 网页失败对策**

| 报错 | 原因 | 修法 |
|---|---|---|
| `端口 8501 已被占用` | 上次没关干净 | `netstat -ano \| findstr :8501` 然后 `taskkill`（附录 D.5） |
| `StreamlitAPIException: st.cache_resource` 位置错 | 装饰器在 if 里 | 提到模块顶层（E-21） |
| `AttributeError: module 'chart_utils' has no attribute 'xxx'` | 函数名对不上 | 检查 5 个图表函数名（E-22） |
| `FileNotFoundError: models/latest` | 产物目录不存在 | 先跑完 Step 4 |
| 页面白屏 | Python 异常被吞 | 命令行窗口里看 traceback |
| 中文乱码 | 编码 | `chcp 65001`（附录 D.8） |

- [ ] Step 5 通过

---

### Step 6 · 换台机器还能跑（可选，但强烈建议）

**目的**：证明没有"只有你电脑上才有的魔法"。

**最省事的做法**：把项目复制到 U 盘或另一个目录，重命名，然后在**新目录 + 新环境**里跑 Step 0~5。

```cmd
:: 换个路径
xcopy /E /I credit_risk_system D:\test_deliver\credit_risk_system

:: 换新环境
conda create -n cr_fresh python=3.10 -y
conda activate cr_fresh
cd /d D:\test_deliver\credit_risk_system
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
python scripts\generate_sample_data.py
python scripts\train_pipeline.py --use-sample --skip-mlp
pytest tests\ -v
```

**最常见的"换机器就挂"三个原因：**

1. **硬编码的绝对路径** —— 代码里写了 `E:\PythonCode\11.4\...`
   → 全项目搜 `E:\` 和 `C:\Users`，改成相对路径
2. **依赖没写进 requirements** —— 你 `pip install` 过但没记
   → 用 `pip freeze` 对比
3. **软链接失效** —— `models/latest` 是指向别处的快捷方式
   → 本项目已改成复制，如果还报错就去查 `ArtifactManager`（E-19）

- [ ] Step 6 通过

---

## E.2 交付物清单（61 个文件）

### 根目录（4）

- [ ] `requirements.txt`
- [ ] `.env.example`
- [ ] `.gitignore`
- [ ] `README.md`

### configs/（9）

- [ ] `data_config.yaml`
- [ ] `model_config.yaml`
- [ ] `pipeline_config.yaml`
- [ ] `llm_config.yaml`
- [ ] `web_config.yaml`
- [ ] `prompts/report.j2`
- [ ] `prompts/feature.j2`
- [ ] `prompts/case.j2`
- [ ] `prompts/system.j2`

### src/（22）

**utils（3 + init）**
- [ ] `src/utils/config_loader.py`
- [ ] `src/utils/logger.py`
- [ ] `src/utils/metrics.py`

**data（2 + init）**
- [ ] `src/data/loader.py`
- [ ] `src/data/preprocessor.py`

**features（2 + init）**
- [ ] `src/features/builder.py`
- [ ] `src/features/selector.py`

**models（4 + init）**
- [ ] `src/models/base.py`
- [ ] `src/models/xgb_model.py`
- [ ] `src/models/mlp_model.py`
- [ ] `src/models/ensemble.py`

**tuning（1 + init）**
- [ ] `src/tuning/optimizer.py`

**explain（1 + init）**
- [ ] `src/explain/shap_analyzer.py`

**llm（5 + init）**
- [ ] `src/llm/client.py`
- [ ] `src/llm/prompts.py`
- [ ] `src/llm/agents/report_agent.py`
- [ ] `src/llm/agents/feature_agent.py`
- [ ] `src/llm/agents/case_agent.py`

**pipeline（1 + init）**
- [ ] `src/pipeline/orchestrator.py`

**13 个 `__init__.py`**
- [ ] `src/__init__.py`
- [ ] `src/utils/__init__.py`
- [ ] `src/data/__init__.py`
- [ ] `src/features/__init__.py`
- [ ] `src/models/__init__.py`
- [ ] `src/tuning/__init__.py`
- [ ] `src/explain/__init__.py`
- [ ] `src/llm/__init__.py`
- [ ] `src/llm/agents/__init__.py`
- [ ] `src/pipeline/__init__.py`
- [ ] `src/app/__init__.py`
- [ ] `src/app/components/__init__.py`
- [ ] `tests/__init__.py`

### scripts/（3）

- [ ] `scripts/generate_sample_data.py`
- [ ] `scripts/train_pipeline.py`
- [ ] `scripts/download_data.py`

### app/（12）

- [ ] `app/streamlit_app.py`
- [ ] `app/components/sidebar.py`
- [ ] `app/components/data_utils.py`
- [ ] `app/components/chart_utils.py`
- [ ] `app/assets/style.css`
- [ ] `app/pages/1_数据探索.py`
- [ ] `app/pages/2_特征工程.py`
- [ ] `app/pages/3_模型训练.py`
- [ ] `app/pages/4_单笔预测.py`
- [ ] `app/pages/5_模型解释.py`
- [ ] `app/pages/6_AI助手.py`
- [ ] `app/pages/7_系统配置.py`

### tests/（1）

- [ ] `tests/test_basic.py`

### 运行后自动生成的目录（不进 git）

- [ ] `data/raw/`（放 `cs-training.csv` 或 `sample_data.csv`）
- [ ] `data/processed/`
- [ ] `models/<run_id>/`（含 `xgb_model.json`、`xgb_model.features.txt`、`preprocessor.pkl`、`eval_results.json`、`meta.json`）
- [ ] `logs/`
- [ ] `outputs/`

---

## E.3 34 个已知坑自查

**这 34 条是源文档里被证实有问题的地方。** 你如果是照着手册写的，理论上都该避开；但人总会手滑，用这张表核一遍。

### E 级 · 照抄必报错（23 条）

| 编号 | 症状（你会看到什么） | 自查方法 | 正确写法 |
|---|---|---|---|
| E-01 | `KeyError: 'derived_features'` 或规则不生效 | 打开 `data_config.yaml` 看 `derived_features` 下有没有 `rules:` 这一层 | 是 `derived_features.rules` 两层结构 |
| E-02 | `TypeError: evaluate() got an unexpected keyword argument 'y_proba'` | 搜 `y_proba=` | 参数名是 `y_pred_proba` |
| E-03 | `AttributeError: 'dict' object has no attribute 'model'` | 搜 `configs.model` | `configs.get("model")` |
| E-04 | `AttributeError: 'EvalResult' object has no attribute 'summary'` | 搜 `.summary()` | 用 `to_dict()` |
| E-05 | `TypeError: Preprocessor.__init__() missing 1 required positional argument` | 看构造调用 | 要传**两份**配置：`data_config` + `feature_config` |
| E-06 | `AttributeError: 'DataSplitter' object has no...` 或切分结果不对 | 看调用处 | `DataSplitter(split_config)`，只传 `split` 那一块 |
| E-07 | `NameError: name 'installment' is not defined` | 跑测试 `test_feature_builder` | `_safe_eval` 要把列名注入命名空间 |
| E-08 | `XGBoostError: use_label_encoder` | 搜 `use_label_encoder` | XGBoost 2.0 已删，删掉这一行 |
| E-09 | `TypeError: load() missing 1 required positional argument: 'self'` | 搜 `XGBModel.load(` | `m = XGBModel(...); m.load(path)` |
| E-10 | `ValueError: y_pred should be 1d` 或形状爆炸 | 打印 `proba.shape` | 返回一维 `(n,)`，不是 `(n,2)` |
| E-11 | `KeyError` 或 Optuna 报参数不存在的错 | 看 `OptunaOptimizer` 构造 | 传 `config.xgboost.tuning`，不是整个 `model` |
| E-12 | `TypeError: fit() takes 1 positional argument but 3 were given` | 对照第 07 章签名 | `fit(model, background)` / `explain_single(X)` / `explain_batch(X)` |
| E-13 | `AttributeError: 'MLPModel' object has no attribute '_modules'` | 看 DeepExplainer 的入参 | `net = getattr(model, "network", model)` 拆出 `nn.Module` |
| E-14 | `TypeError: PromptManager.__init__() takes ...` | 看构造 | 传 config 对象，不是路径字符串 |
| E-15 | 脱敏日志显示"已脱敏"，但外发内容还是原文 | **抓一次实际请求体看** | 先 `_sanitize_input(prompt)`，再 `_build_messages()` |
| E-16 | `TypeError: chat() got an unexpected keyword argument 'system'` | 搜 `system=` | 是 `system_prompt=` |
| E-17 | `TypeError: tojson() got an unexpected keyword argument` | 搜 `tojson` | Jinja2 的 `tojson` 没有 `ensure_ascii` 参数 |
| E-18 | `TypeError: ArtifactManager() missing 1 required positional argument: 'base_dir'` | 搜 `ArtifactManager(` | 必须传 `base_dir="models"` |
| E-19 | `load_latest` 返回 `None` 或 `KeyError: 'model'` | 打印返回值 | 只返回 `{dir, meta, eval_results}`，模型要自己 load |
| E-20 | `error: unrecognized arguments: --skip-tuning` | `python scripts\train_pipeline.py --help` | 只有 4 个参数：`--config-dir` `--use-sample` `--skip-mlp` `--tune` |
| E-21 | 连续点击每次都转圈（缓存没生效） | 看装饰器位置 | `@st.cache_resource` 必须在模块顶层 |
| E-22 | `AttributeError: module 'chart_utils' has no attribute 'plot_xxx'` | `dir(chart_utils)` | 只有 5 个：`target_distribution` `missing_bar` `importance_bar` `risk_gauge` `shap_waterfall` |
| E-23 | 第 2 个测试莫名失败，单独跑又过 | `pytest -p no:randomly tests\` | 测试开头重置 `ConfigLoader._instance = None` |

### W 级 · 不报错但结果错（7 条）

| 编号 | 症状 | 自查方法 | 正确做法 |
|---|---|---|---|
| W-01 | 用了 MinMax/Robust 但数据没被缩放 | 打印 `X_train.describe()` 看范围 | `_apply_scaler` 不能用 `hasattr(self, "mean_")` 判断类型，要显式判断 `scaler_type` |
| W-02 | 特征被删得只剩十几个，AUC 掉了 | 打印筛选前后特征数 | 相关性阈值别设太低（0.8 → 建议 0.95） |
| W-03 | MLP 训 50 轮要 20 分钟 | 计时 | CPU 上先 `--skip-mlp`；或用 GPU（附录 D.3.1） |
| W-04 | 装了 `MedianPruner` 但 trial 从不提前停 | 看 `trial.should_prune()` 有没有被调用 | objective 里要 `trial.report()` + 抛 `TrialPruned` |
| W-05 | 报告里写"提高收入就能降低风险" | 读报告文字 | SHAP 是相关不是因果，措辞要改成"与该风险相关" |
| W-06 | 指标特别好（AUC 0.85+）但上线就崩 | 按 Step 4.1 查泄漏 | split 必须在 preprocess 之前 |
| W-07 | 换成真实数据后一堆报错 | 对比 `term` 字段类型 | 合成数据要按真实 schema：`"36 months"` 不是 `36` |

### N 级 · 环境与小陷阱（3 条）

| 编号 | 症状 | 自查方法 | 正确做法 |
|---|---|---|---|
| N-01 | `.env` 建成了 `.env.txt`，Key 读不到 | `dir /a` 看文件名 | 资源管理器"显示扩展名"，或用 `type nul > .env` 建 |
| N-02 | `yaml.scanner.ScannerError: found character '\t'` | 编辑器开"显示空白字符" | YAML 只用空格，绝不用 Tab |
| N-03 | `openai.NotFoundError: model deepseek-flash not found` | 看 `.env` 里的模型名 | 用 `deepseek-chat` 或 `deepseek-reasoner` |

### S 级 · 方法论风险（1 条，最重要）

| 编号 | 问题 | 怎么自查 | 正确做法 |
|---|---|---|---|
| **S-01** | 流水线顺序 `preprocess → feature_selection → split` 造成数据泄漏 | 打印 `pipeline.steps` 看顺序 | `split` 必须排第 2 位：`load → split → preprocess → feature_eng → feature_sel → train` |

> **S-01 单独列出来，是因为它比其他 33 条加起来都重要。**
> 其他 33 条会报错或结果明显不对，你会去查。
> S-01 **不报错，指标还特别好看**——它骗的是你，也是看你项目的人。

**自查进度**：34 条里你中了几条？

- 0~2 条：正常，可以交付
- 3~5 条：回去改，改完重跑 Step 4
- 6 条以上：建议从头对着手册第 03、04 章重写关键模块
- **中了 S-01**：你的所有指标推倒重来

---

## E.4 指标合理性判断

**这一段帮你回答"我的 AUC 0.78 算好吗"。**

### E.4.1 参考区间（合成数据，10000 条，20% 违约率）

| 指标 | 偏低 | 正常 | 偏高 | 可疑 |
|---|---|---|---|---|
| 验证 AUC | < 0.65 | **0.72 ~ 0.85** | 0.85 ~ 0.90 | > 0.90 |
| 验证 KS | < 0.20 | **0.30 ~ 0.50** | 0.50 ~ 0.62 | > 0.62 |
| 训练 AUC | — | 0.80 ~ 0.88 | — | > 0.95 |
| 训练-验证 AUC 差 | — | **< 0.06** | 0.06 ~ 0.12 | > 0.15 |
| LogLoss | > 0.60 | **0.35 ~ 0.55** | — | < 0.20 |
| 最优阈值 | — | **0.20 ~ 0.40** | — | 正好 0.50 |

**为什么最优阈值通常不是 0.5？**
因为样本不平衡（20% 违约）。Youden 指数找的是 `TPR - FPR` 最大点，在正样本少的情况下，这个点会往左偏。阈值 0.5 意味着只有"非常确定"才判违约，召回率会很低。

### E.4.2 指标"太好"时怎么判断是不是泄漏

按这个顺序查，找到第一个"是"就停：

```
1. 验证 AUC > 0.92？                        → 是 → 高度可疑
2. 训练-验证 AUC 差 < 0.01？                 → 是 → 可疑（正常模型不会这么齐）
3. pipeline.steps 里 preprocess 在 split 前？ → 是 → 确认泄漏（S-01）
4. 特征里有"事后信息"吗？                     → 是 → 标签泄漏
   （如：催收结果、结清日期、逾期后状态）
5. 特征数远大于样本数 / 10？                  → 是 → 高危过拟合
```

**第 4 条怎么查**：逐个看特征名，问自己"这个值在**放款那一刻**能拿到吗？"
拿不到的就是事后信息，必须删。

典型的事后特征（本项目里不该出现）：

- `collection_recovery_fee`（催收回收费用）
- `last_payment_date`（最后还款日）
- `settlement_status`（结清状态）

### E.4.3 指标"太差"时怎么救

按性价比排序：

| 优先级 | 动作 | 预期提升 | 耗时 |
|---|---|---|---|
| 1 | 检查数据有没有读错（列错位、单位错） | 可能 +0.15 | 10 分钟 |
| 2 | 检查特征筛选是不是删过头（W-02） | +0.02~0.05 | 5 分钟 |
| 3 | 加业务特征（负债收入比、账龄、额度使用率） | +0.03~0.08 | 1~2 小时 |
| 4 | 调 `scale_pos_weight` 和 `max_depth` | +0.01~0.03 | 30 分钟 |
| 5 | 跑 Optuna 调参（`--tune`） | +0.01~0.02 | 1 小时 |
| 6 | 加 MLP 融合 | +0.005~0.02 | 30 分钟 |
| 7 | 换更复杂的模型（LightGBM / CatBoost） | +0.005~0.01 | 1 小时 |

**注意第 1、2 项的预期提升比第 5、6 项大得多。**
新手常犯的错：指标不好就去调参，其实问题在数据。

---

## E.5 一页纸验收单（打印用）

```
┌──────────────────────────────────────────────────────────┐
│  信贷风控系统 · 交付验收单                                 │
│  验收日期：____________   验收人：____________            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  【Step 0】环境                                           │
│   □ 新环境 conda create 成功                              │
│   □ where python 指向新环境                               │
│   □ pip install -r requirements.txt 无报错                │
│   □ 14 个核心库全部 import OK                             │
│   □ torch 版本与 CUDA 状态已确认                          │
│                                                          │
│  【Step 1】配置                                           │
│   □ 5 个 YAML 全部读到                                    │
│   □ ${VAR:-默认} 插值生效                                 │
│   □ 没有 .env 也不崩溃（只警告）                          │
│                                                          │
│  【Step 2】测试                                           │
│   □ pytest 11 passed，0 skipped                           │
│                                                          │
│  【Step 3】数据                                           │
│   □ 违约率在 15%~25%                                      │
│   □ 无全空列、无全常量列                                  │
│   □ term 是字符串 "36 months"                             │
│   □ dti 在 0~1 区间                                       │
│   □ 字段数 = 47                                           │
│                                                          │
│  【Step 4】训练                                           │
│   □ 8 个 step 全部执行                                    │
│   □ 验证 AUC 在 0.72~0.85                                 │
│   □ 验证 KS 在 0.30~0.50                                  │
│   □ 训练-验证 AUC 差 < 0.06                               │
│   □ 产物目录有 5 个文件                                   │
│                                                          │
│  【Step 4.1】★ 数据泄漏专项（最重要）                     │
│   □ pipeline.steps 里 split 在第 2 位                     │
│   □ preprocess 只对 X_train 调用 fit                      │
│   □ 无事后特征（催收/结清/最后还款）                      │
│   □ 对比"错顺序"版本，AUC 差距已记录：____                │
│                                                          │
│  【Step 5】网页                                           │
│   □ 7 个页面全部能打开                                    │
│   □ 单笔预测出概率 + 等级 + SHAP 图                       │
│   □ 连续点击不重复加载模型                                │
│   □ AI 报告是打字机效果（或优雅提示无 Key）               │
│   □ 系统配置页不显示完整 API Key                          │
│                                                          │
│  【Step 6】换机复现（可选）                               │
│   □ 换路径 + 换环境后仍能跑通                             │
│   □ 无硬编码绝对路径                                      │
│                                                          │
├──────────────────────────────────────────────────────────┤
│  34 个已知坑自查：中了 ____ 条                            │
│  其中 S-01（数据泄漏）：□ 已排除  □ 未排除                │
│                                                          │
│  验收结论：  □ 通过    □ 有条件通过    □ 不通过           │
│                                                          │
│  签字：________________                                  │
└──────────────────────────────────────────────────────────┘
```

---

## 附录 E 自测

1. 为什么 Step 0 要求"新建一个环境"，不能在现有环境验？
2. `pytest tests\ -v` 显示 `11 passed, 2 skipped`，算通过吗？
3. 验证 AUC 0.94，你觉得是好消息还是坏消息？下一步该做什么？
4. `where python` 显示 `anaconda3\python.exe`，说明什么？
5. 34 个坑里，哪一条"不报错但最致命"？
6. 指标只有 0.66，你该先去调参还是先查数据？为什么？

<details>
<summary>答案</summary>

1. 因为现有环境被你反复 `pip install` 过，很多缺失的依赖被"碰巧装上的包"补上了。新环境才能暴露 `requirements.txt` 缺项。这是"我这能跑，别人那不能跑"的根源。
2. **不算**。有 skip 说明有用例被跳过了，通常是条件不满足（比如没数据就跳过训练测试）。要 `-rs` 看跳过原因，把前置条件补齐。
3. **坏消息**。0.94 超出合成数据的合理上限（0.85），高度疑似数据泄漏。下一步立刻做 Step 4.1 的三项检查，尤其是 `pipeline.steps` 的顺序。
4. 你在 **base 环境**，没激活项目环境。`conda activate credit_risk`。
5. **S-01（数据泄漏）**。其他坑要么报错要么结果明显异常，你会去查；S-01 不报错，指标还好看，它骗的是你和评审。
6. **先查数据**。E.4.3 的表里，查数据的预期提升（+0.15）比调参（+0.02）大一个量级。新手最常见的错误就是指标不好先调参。

</details>

---

← 上一篇 [`附录B_错误对照总表.md`](附录B_错误对照总表.md) ｜ 下一篇 → [`附录F_答辩话术与演示脚本.md`](附录F_答辩话术与演示脚本.md)
