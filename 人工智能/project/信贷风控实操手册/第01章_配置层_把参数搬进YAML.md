# 第 01 章　配置层：把参数搬进 YAML

> 本章建 9 个文件 · 预计 1~2 小时 · 难度 ★☆☆ · 前置：第 00 章

---

## 1. 这一章在流水线上的位置

```
[原始CSV] →①数据层→ ②特征层→ ③模型层→ ④调参→ ⑤解释→ ⑥AI报告→ ⑦编排→ ⑧脚本→ ⑨网页
     ▲
你在这里：第 01 章　配置层（全项目的控制面板）
已经有的：目录、依赖、空包
还没有的：所有代码
```

配置层不在流水线的某一格上，它是**悬在整条流水线上方的控制面板**——后面每一层的代码都会来这里读参数。

---

## 2. 本章你要做什么

一句话：**把"会变的数字"全部从代码里搬出来，集中写进 5 个 YAML 文件。**

本章结束时你会得到：

```
configs/
├── data_config.yaml        数据在哪、哪列是标签、怎么切分
├── feature_config.yaml     缺失值怎么填、类别怎么编码、造哪些新特征
├── model_config.yaml       三个模型的全部超参
├── llm_config.yaml         AI 接口、四个 Agent 的行为
├── pipeline_config.yaml    训练流程有哪几步
└── prompts/
    ├── risk_report.j2          风险报告模板
    ├── feature_suggestion.j2   特征建议模板
    ├── case_analysis.j2        错误案例诊断模板
    └── chat_assistant.j2       对话助手模板
```

**为什么值得单独一章**：如果不做这件事，你的代码里会到处是 `learning_rate=0.05`、`max_depth=6`、`"./data/raw/train.csv"`。想改一个参数就得翻遍 40 个文件，而且无法知道"上次跑出 0.74 用的是哪套参数"。

---

## 3. 开工前检查

- [ ] 上一章最后一条命令输出的是 `骨架 OK`
- [ ] 我当前目录能看到 `configs` 文件夹（`dir` 确认）
- [ ] 我的 conda 环境是激活状态（提示符前有 `(credit_risk)`）

---

## 4. 本章知识点

| 编号 | 概念 | 一句话解释 | 为什么这里需要它 | 本项目用在哪 | 要求 |
|---|---|---|---|---|---|
| KP-EN-06 | 配置驱动 | 参数放外部文件，代码只负责读 | 改参数不改代码，实验可复现 | 全部 5 个 YAML | 必懂 |
| KP-PY-02 | YAML 语法 | 用缩进表示层级的配置格式，比 JSON 好读 | 手写配置不容易错，还能加注释 | 5 个配置文件 | 必懂 |
| KP-EN-07 | 环境变量插值 | 配置里写 `${VAR:-默认值}`，运行时替换成真值 | Key 不进配置文件，安全 | `llm_config.yaml` | 必懂 |
| KP-EN-08 | 模板引擎 | 把变量填进带占位符的文本 | 提示词与代码分离，改话术不动 Python | 4 个 `.j2` 模板 | 知道即可 |
| KP-BIZ-01 | 违约概率与目标列 | 要预测的 `isDefault`：1=违约，0=正常 | 全项目的因变量就是它 | `data_config.yaml` | 必懂 |
| KP-BIZ-02 | 分层抽样 | 切分时保证每份的违约率都与原始一致 | 不分层可能导致测试集几乎没有违约样本 | `split.stratify` | 知道即可 |
| KP-BIZ-03 | 类别不平衡 | 违约样本远少于正常样本，模型会偏向多数类 | 需要 `scale_pos_weight` / `class_weight` 补偿 | 模型配置 | 知道即可 |

### 展开讲：YAML 的三条铁律

YAML 靠**缩进**表示层级，所以对格式极其敏感。记住三条，能避免 90% 的报错：

1. **只能用空格，不能用 Tab**。一个 Tab 就会让整个文件解析失败。
2. **同级缩进必须完全对齐**。差一个空格，某个键就变到别的层级下面去了。
3. **冒号后面必须有一个空格**。`key:value` 是错的，`key: value` 才对。

```yaml
# ✅ 正确
split:
  test_size: 0.2
  random_state: 42

# ❌ 错误（第二行缩进多了一个空格）
split:
  test_size: 0.2
   random_state: 42
```

---

## 5. 分步操作

### 步骤 1.1　`configs/data_config.yaml` —— 数据配置

**【目标】** 告诉程序：数据放哪、哪列是标签、47 个字段各是什么类型、怎么切分。

**【文件】** `configs/data_config.yaml`（新建）

**【完整内容】**

```yaml
# ============================================================
# 数据配置：数据源路径、字段类型、数据集划分
# ============================================================

dataset:
  name: "天池零基础入门金融风控-贷款违约预测"
  source_url: "https://tianchi.aliyun.com/competition/entrance/531830"
  total_samples: 800000

# 数据路径（全部用相对路径，项目挪位置不用改）
paths:
  raw_dir: "./data/raw"
  processed_dir: "./data/processed"
  sample_dir: "./data/sample"
  model_dir: "./data/models"
  train_file: "train.csv"
  test_file: "testA.csv"
  sample_file: "sample.csv"

# 标签列：要预测的目标（1=违约，0=正常）
target_column: "isDefault"

# ID 列：不参与建模
id_column: "id"

# 字段按类型分组（不同类型用不同预处理方法）
columns:
  numerical:
    - loanAmnt            # 贷款金额
    - term                # 贷款期限
    - interestRate        # 贷款利率
    - installment         # 分期付款金额
    - annualIncome        # 年收入
    - dti                 # 债务收入比
    - delinquency_2years  # 近2年逾期次数
    - ficoRangeLow        # FICO 评分下限
    - ficoRangeHigh       # FICO 评分上限
    - openAcc             # 未结信用账户数
    - pubRec              # 公开记录数
    - pubRecBankruptcies  # 破产记录数
    - revolBal            # 循环余额
    - revolUtil           # 循环额度使用率
    - totalAcc            # 总账户数

  categorical:
    - grade               # 贷款等级 A-G
    - subGrade            # 子等级 A1-G5
    - employmentTitle     # 就业职称（脱敏）
    - employmentLength    # 就业年限
    - homeOwnership       # 房屋所有权
    - verificationStatus  # 收入认证状态
    - purpose             # 贷款用途
    - postCode            # 邮编（脱敏）
    - regionCode          # 地区代码
    - initialListStatus   # 初始列示状态
    - applicationType     # 申请类型
    - title               # 借款描述（脱敏）
    - policyCode          # 政策代码

  datetime:
    - issueDate           # 贷款发放日期
    - earliesCreditLine   # 最早信用记录日期

  anonymous:
    - n0
    - n1
    - n2
    - n3
    - n4
    - n5
    - n6
    - n7
    - n8
    - n9
    - n10
    - n11
    - n12
    - n13
    - n14

# 数据集划分
split:
  test_size: 0.2        # 20% 做测试集
  validation_size: 0.1  # 10% 做验证集
  random_state: 42      # 固定随机种子，保证可复现
  stratify: true        # 分层抽样，保证每份违约率一致

# 数据加载选项
loading:
  chunk_size: 100000    # 大文件分块读取的行数
  encoding: "utf-8"
  low_memory: false
```

**【逐块讲解】**

| 配置块 | 作用 | 小白要点 |
|---|---|---|
| `dataset` | 元信息，只用来展示 | 不参与计算，可随意改 |
| `paths` | 所有路径的出处 | 代码里不许出现硬编码路径，都从这里读 |
| `target_column` | 目标列名 | 全项目就靠这个字符串定位标签 |
| `columns` | 字段分四组 | 数值组做标准化、类别组做编码、时间组拆年月、匿名组原样保留 |
| `split` | 切分比例 | `stratify: true` 必须保留，否则小样本时测试集可能没违约 |
| `loading` | 读取参数 | 80 万行时不加 `chunk_size` 可能爆内存 |

**【验证】** 本章全部写完统一验证（步骤 1.7）。

---

### 步骤 1.2　`configs/feature_config.yaml` —— 特征工程配置

**【目标】** 定义缺失值、编码、标准化、衍生特征、异常值、特征筛选的全部规则。

**【文件】** `configs/feature_config.yaml`（新建）

**【完整内容】**

```yaml
# ============================================================
# 特征工程配置：缺失值、编码、标准化、衍生特征、异常值、筛选
# ============================================================

# ===== 缺失值处理 =====
missing_value:
  numerical_strategy: "median"    # mean / median / zero / drop
  categorical_strategy: "unknown" # mode / unknown / drop
  drop_threshold: 0.95            # 缺失率超过 95% 的列直接丢弃

# ===== 类别编码 =====
encoding:
  method: "label"                 # label / onehot / target / woe
  custom:
    grade:
      method: "label"
      mapping: {A: 1, B: 2, C: 3, D: 4, E: 5, F: 6, G: 7}
    employmentLength:
      method: "label"
      mapping:
        "< 1 year": 0
        "1 year": 1
        "2 years": 2
        "3 years": 3
        "4 years": 4
        "5 years": 5
        "6 years": 6
        "7 years": 7
        "8 years": 8
        "9 years": 9
        "10+ years": 10

# ===== 数值标准化 =====
scaling:
  method: "standard"   # standard / minmax / robust / none
  per_feature: true

# ===== 衍生特征 =====
# ⚠️ 注意层级：规则列表在 derived_features.rules 下面，不是 derived_features 下面
derived_features:
  enabled: true
  rules:
    - name: "monthly_income"
      formula: "annualIncome / 12"
      description: "由年收入推算月收入"

    - name: "installment_to_income_ratio"
      formula: "installment / (annualIncome / 12 + 1)"
      description: "月还款额占月收入比例（+1 防除零）"

    - name: "fico_avg"
      formula: "(ficoRangeLow + ficoRangeHigh) / 2"
      description: "FICO 评分均值"

    - name: "loan_to_income_ratio"
      formula: "loanAmnt / (annualIncome + 1)"
      description: "贷款金额占年收入比例"

    - name: "revol_util_bucket"
      formula: "pd.cut(df['revolUtil'], bins=[-1, 30, 60, 90, 200], labels=[0, 1, 2, 3])"
      description: "信用利用率分桶"

# ===== 时间特征 =====
datetime_features:
  issueDate:
    extract: [year, month, quarter, is_year_end]
  earliesCreditLine:
    extract: [year, credit_history_length]

# ===== 异常值处理 =====
outlier:
  method: "clip"          # iqr / zscore / clip / none
  lower_quantile: 0.01
  upper_quantile: 0.99
  apply_to: [annualIncome, revolBal, dti, installment]

# ===== 特征筛选 =====
feature_selection:
  enabled: true
  methods:
    - name: "variance"
      threshold: 0.01
    - name: "correlation"
      threshold: 0.95
    - name: "model_based"
      model: "xgboost"
      top_k: 50
```

**【逐块讲解】**

| 配置块 | 关键点 |
|---|---|
| `missing_value` | 数值用**中位数**而不是均值——收入这类字段有极端值，均值会被拉偏 |
| `encoding.custom` | `grade`（A~G）和 `employmentLength` 本身有**顺序**，所以用有序映射而不是 one-hot |
| `scaling` | 标准化是"减均值除标准差"，让不同量纲的字段可比。神经网络特别依赖这一步 |
| `derived_features.rules` | **本章最容易出错的地方**。代码里必须写 `config.derived_features.rules`，写成 `config.derived_features[0]` 会报 `KeyError: 0` |
| `outlier` | 截断法：超过 99 分位的压回 99 分位，低于 1 分位的抬到 1 分位。比直接删行温和 |
| `feature_selection` | 三道筛：方差太小 → 互相太相关 → 用 XGBoost 重要性保留 Top 50 |

---

### 步骤 1.3　`configs/model_config.yaml` —— 模型配置

**【目标】** 三个模型（XGBoost、神经网络、融合）的全部超参。

**【文件】** `configs/model_config.yaml`（新建）

**【完整内容】**

```yaml
# ============================================================
# 模型配置：XGBoost / MLP / 融合 / 评估 / 保存
# ============================================================

xgboost:
  enabled: true
  params:
    objective: "binary:logistic"
    eval_metric: "auc"
    booster: "gbtree"
    tree_method: "hist"
    device: "cpu"              # 有 GPU 且装了 GPU 版 torch 时可改 "cuda"

    max_depth: 6
    learning_rate: 0.05
    n_estimators: 1000

    min_child_weight: 1
    gamma: 0
    subsample: 0.8
    colsample_bytree: 0.8

    reg_alpha: 0.1
    reg_lambda: 1.0

    scale_pos_weight: 4        # 负正样本比，补偿类别不平衡
    random_state: 42
    n_jobs: -1
    verbosity: 1

  early_stopping:
    enabled: true
    rounds: 50

  tuning:
    enabled: true
    method: "optuna"
    n_trials: 50
    timeout: 1800
    direction: "maximize"
    metric: "auc"
    search_space:
      max_depth:        {type: "int",   low: 3,      high: 10}
      learning_rate:    {type: "float", low: 0.01,   high: 0.3, log: true}
      min_child_weight: {type: "int",   low: 1,      high: 10}
      subsample:        {type: "float", low: 0.6,    high: 1.0}
      colsample_bytree: {type: "float", low: 0.6,    high: 1.0}
      reg_alpha:        {type: "float", low: 1.0e-8, high: 10.0, log: true}
      reg_lambda:       {type: "float", low: 1.0e-8, high: 10.0, log: true}

mlp:
  enabled: true
  architecture:
    hidden_layers: [256, 128, 64]
    activation: "relu"         # relu / gelu / leaky_relu
    dropout: 0.3
    batch_norm: true
    output_activation: "sigmoid"

  training:
    batch_size: 512
    epochs: 50
    learning_rate: 0.001
    weight_decay: 1.0e-5
    optimizer: "adam"          # adam / adamw / sgd
    scheduler: "cosine"        # cosine / step / plateau / none
    loss: "bce"                # bce / focal
    class_weight: "balanced"
    early_stopping:
      enabled: true
      patience: 5
      min_delta: 0.0001

  device: "cpu"                # 有 GPU 时改 "cuda"

ensemble:
  enabled: true
  method: "weighted_avg"       # weighted_avg / stacking / voting
  weights:
    xgboost: 0.6
    mlp: 0.4
  stacking:
    meta_learner: "logistic_regression"
    cv_folds: 5

evaluation:
  metrics: [auc, ks, accuracy, precision, recall, f1, log_loss]
  threshold_optimization:
    enabled: true
    method: "youden"           # youden / f1 / fixed
    fixed_value: 0.5

persistence:
  save_dir: "./data/models"
  format:
    xgboost: "json"
    mlp: "pth"
  versioning: true
```

**【逐块讲解】**

| 超参 | 含义 | 调大/调小会怎样 |
|---|---|---|
| `max_depth` | 每棵树最多几层 | 越大越容易拟合噪声（过拟合） |
| `learning_rate` | 每棵树贡献多少 | 越小越稳但需要更多树；常配 `n_estimators` 一起调 |
| `n_estimators` | 最多多少棵树 | 有早停保护，设大一点无妨 |
| `subsample` / `colsample_bytree` | 每棵树随机用多少行/多少列 | 小于 1 相当于加噪声，防过拟合 |
| `reg_alpha` / `reg_lambda` | L1 / L2 正则化强度 | 越大模型越保守 |
| `scale_pos_weight: 4` | 正样本权重放大 4 倍 | 违约率约 20%，负正比约 4:1，这样能让模型重视少数类 |
| `hidden_layers` | 神经网络每层神经元数 | `[256,128,64]` 是逐层收窄的"漏斗"结构 |

---

### 步骤 1.4　`configs/llm_config.yaml` —— AI 接口配置

**【目标】** DeepSeek 接口参数、重试限流、四个 Agent 的行为、合规设置。

**【文件】** `configs/llm_config.yaml`（新建）

**【完整内容】**

```yaml
# ============================================================
# LLM 配置：DeepSeek 接口、Prompt 模板、Agent 行为、安全合规
# ============================================================

deepseek:
  # ${VAR:-默认值} 表示：优先读环境变量，读不到用默认值
  base_url: "${DEEPSEEK_BASE_URL:-https://api.deepseek.com}"
  api_key_env: "DEEPSEEK_API_KEY"
  model: "${DEEPSEEK_MODEL:-deepseek-chat}"

  default_params:
    temperature: 0.3      # 风控要稳定输出，温度调低
    max_tokens: 2000
    top_p: 0.9
    frequency_penalty: 0.0
    presence_penalty: 0.0
    timeout: 30

  retry:
    max_attempts: 3
    backoff_factor: 2
    initial_wait: 1
    max_wait: 30

  rate_limit:
    enabled: true
    requests_per_minute: 60

prompts:
  templates_dir: "./configs/prompts"

  risk_report:
    template_file: "risk_report.j2"
    system_role: |
      你是一名资深的金融风控分析师，专长于个人信贷风险评估。
      你需要基于机器学习模型的预测结果和 SHAP 特征贡献度，
      为业务人员生成清晰、专业、可操作的风险评估报告。
      要求：语言通俗易懂；重点突出风险与保护因素；给出明确审批建议；包含免责声明。

  feature_suggestion:
    template_file: "feature_suggestion.j2"
    system_role: |
      你是一名经验丰富的数据科学家，专长于金融领域的特征工程。
      基于数据字段说明和样本统计信息，为风控建模提供专业的特征衍生建议。

  case_analysis:
    template_file: "case_analysis.j2"
    system_role: |
      你是一名机器学习模型诊断专家。
      分析模型预测错误的样本，找出潜在原因并提供改进建议。

  chat_assistant:
    template_file: "chat_assistant.j2"
    system_role: |
      你是"智能信贷风控分析系统"的 AI 助手。
      回答关于数据、模型、风控业务的各类问题。简洁、准确、有依据，不确定就明说。

agents:
  report_agent:
    enabled: true
    top_n_features: 5
    include_recommendations: true
    output_format: "markdown"
    streaming: false

  feature_agent:
    enabled: true
    sample_size: 1000
    max_suggestions: 10
    include_code: true

  case_agent:
    enabled: true
    max_cases: 20
    confidence_threshold: 0.85

  chat_agent:
    enabled: true
    max_history_turns: 10
    streaming: true

safety:
  pii_filter:
    enabled: true
    fields_to_mask: ["id", "postCode", "employmentTitle", "title"]

  content_filter:
    enabled: true
    block_patterns: ["API[_ ]?KEY", "password", "secret"]

  audit_log:
    enabled: true
    log_file: "./logs/llm_audit.log"
    log_request: true
    log_response: false
```

**【逐块讲解】**

| 配置 | 说明 |
|---|---|
| `${DEEPSEEK_BASE_URL:-https://api.deepseek.com}` | 环境变量插值。`.env` 里没配就用冒号后的默认值 |
| `api_key_env: "DEEPSEEK_API_KEY"` | 只存"变量名"，不存 Key 本身。Key 在 `.env` |
| `temperature: 0.3` | 0 最确定、1 最发散。风控报告要稳定，所以调低 |
| `retry` + `rate_limit` | 网络抖动自动重试（指数退避：等 1s、2s、4s）；每分钟最多 60 次 |
| `prompts.*.system_role` | 定义 AI"扮演谁"。这部分会作为 system 消息发给模型 |
| `safety` | 金融合规三件套：脱敏、内容过滤、审计日志 |

---

### 步骤 1.5　`configs/pipeline_config.yaml` —— 流程编排配置

**【目标】** 把训练拆成步骤清单，每步指定"用哪个模块的哪个类"。

**【文件】** `configs/pipeline_config.yaml`（新建）

**【完整内容】**

```yaml
# ============================================================
# 流程编排配置：训练步骤清单、预测流程、输出路径
# ============================================================

global:
  experiment_name: "credit_risk_v1"
  random_seed: 42
  num_workers: 4
  log_dir: "./logs"

training_pipeline:
  steps:
    - {name: "split_data",         module: "src.data.loader",         class: "DataSplitter",    enabled: true}
    - {name: "preprocess",         module: "src.data.preprocessor",   class: "Preprocessor",    enabled: true}
    - {name: "feature_engineering",module: "src.features.builder",    class: "FeatureBuilder",  enabled: true}
    - {name: "feature_selection",  module: "src.features.selector",   class: "FeatureSelector", enabled: true}
    - {name: "train_xgboost",      module: "src.models.xgb_model",    class: "XGBModel",        enabled: true}
    - {name: "train_mlp",          module: "src.models.mlp_model",    class: "MLPModel",        enabled: true}
    - {name: "ensemble",           module: "src.models.ensemble",     class: "EnsembleModel",   enabled: true}
    - {name: "evaluate",           module: "src.utils.metrics",       class: "Evaluator",       enabled: true}
    - {name: "explain",            module: "src.explain.shap_analyzer", class: "SHAPAnalyzer",  enabled: true}
    - {name: "save_artifacts",     module: "src.pipeline.orchestrator", class: "ArtifactManager", enabled: true}

inference_pipeline:
  steps:
    - {name: "load_model",       enabled: true}
    - {name: "preprocess_input", enabled: true}
    - {name: "predict",          enabled: true}
    - {name: "explain_local",    enabled: true}
    - {name: "generate_report",  enabled: true}

outputs:
  predictions_path: "./data/predictions"
  reports_path: "./data/reports"
  metrics_path: "./data/metrics"
  shap_path: "./data/shap"

tracking:
  enabled: false
  backend: "none"
```

**【逐块讲解】**

**这里有一处刻意修改，必须说明。** 原文档的步骤顺序是：

```
load_data → preprocess → feature_engineering → feature_selection → split_data → ...
```

本手册改成：

```
split_data → preprocess → feature_engineering → feature_selection → ...
```

**为什么**：原顺序里，标准化器（算均值和标准差）、特征筛选器（算特征重要性）是在**包含测试集的全量数据**上 `fit` 的。等于在考试前偷偷看了答案——测试集的分布信息泄漏进了训练过程，指标会虚高，上线后掉得很难看。这叫**数据泄漏**，是机器学习最常见的严重错误。

先切分，之后所有 `fit` 只发生在训练集上，测试集只 `transform`，这才是正确做法。

细节在第 03 章和第 09 章展开，现在只需知道：**这份清单顺序是修正过的，不要改回去。**

---

### 步骤 1.6　`configs/prompts/` —— 4 个提示词模板

**【目标】** 用模板把"提示词"和"代码"分开。改话术不用改 Python。

**【说明】** Jinja2 模板语法只有三种符号：

| 语法 | 作用 | 例子 |
|---|---|---|
| `{{ 变量 }}` | 把值填进去 | `{{ probability }}` |
| `{% for x in list %}` | 循环 | 遍历特征列表 |
| `{% if 条件 %}` | 条件 | 只有正数才显示箭头 |

#### 1.6.1 `configs/prompts/risk_report.j2`

```jinja2
你需要基于以下机器学习模型的预测结果，为一笔贷款申请生成专业的风险评估报告。

## 申请人信息
{% for key, value in features.items() %}
- **{{ key }}**: {{ value }}
{% endfor %}

## 模型预测结果
- **违约概率**: {{ "%.2f" | format(probability * 100) }}%
- **风险等级**: {{ risk_level }}
- **使用模型**: XGBoost + MLP 融合模型

## SHAP 特征贡献度（Top {{ top_n }}）
以下为对预测结果影响最大的特征：

{% for item in shap_top_features %}
{{ loop.index }}. **{{ item.feature }}**
   - 当前值: {{ item.value }}
   - SHAP 值: {{ "%.4f" | format(item.shap_value) }} ({% if item.shap_value > 0 %}↑ 增加风险{% else %}↓ 降低风险{% endif %})
{% endfor %}

## 输出要求

请生成一份结构化的中文风险评估报告，包含以下部分：

### 一、风险摘要（2-3 句话）
对该申请的整体风险情况做出判断。

### 二、关键风险因素
分析 SHAP 值为正的特征，说明它们如何增加违约风险。

### 三、保护因素
分析 SHAP 值为负的特征，说明它们如何降低违约风险。

### 四、审批建议
基于综合分析给出明确建议（通过 / 拒绝 / 人工复核），并说明理由。

### 五、补充建议
如果建议通过，提出可能的风险缓释措施（如调整额度、提高利率、增加担保等）。

### ⚠️ 免责声明
报告由 AI 模型生成，仅供参考，最终决策需人工审核。

---

**注意事项：**
1. 语言要专业但通俗，避免过多技术术语
2. 数字要具体，引用 SHAP 值时保留关键信息
3. 建议要有依据，不要凭空臆测
4. 整体长度控制在 500 字左右
```

#### 1.6.2 `configs/prompts/feature_suggestion.j2`

```jinja2
你是一位经验丰富的金融风控数据科学家。请基于以下数据集信息，为提升模型预测能力提供专业的特征工程建议。

## 数据集背景
- **场景**: 个人信贷违约预测
- **目标变量**: isDefault (1=违约, 0=正常)
- **当前样本数**: {{ sample_size }}
- **当前特征数**: {{ feature_count }}

## 现有字段说明
{% for field in fields %}
- **{{ field.name }}** ({{ field.dtype }}): {{ field.description }}
  {% if field.stats %}- 统计：{{ field.stats }}{% endif %}
{% endfor %}

## 数据样本（前 5 行）
```
{{ data_sample }}
```

## 当前模型表现
- 模型: {{ current_model }}
- AUC: {{ current_auc }}
- 主要瓶颈: {{ bottleneck if bottleneck else "未指定" }}

## 输出要求

请提供 **{{ max_suggestions }} 个**有助于提升模型表现的特征工程建议，每个建议包含：

1. **特征名称**：建议的新特征名
2. **业务含义**：从业务角度解释为什么这个特征有用
3. **构造逻辑**：如何从现有字段计算
4. **预期效果**：可能对哪类样本的预测有帮助
5. **示例代码**：可执行的 Pandas 代码片段

请按以下 JSON 格式输出，不要包含 markdown 代码块标记：

```json
{
  "suggestions": [
    {
      "name": "monthly_burden_ratio",
      "business_meaning": "月还款额占月收入的比例，反映还款压力",
      "construction": "installment / (annualIncome / 12)",
      "expected_impact": "对高负债压力客户的违约预测有帮助",
      "code": "df['monthly_burden_ratio'] = df['installment'] / (df['annualIncome'] / 12 + 1)"
    }
  ],
  "general_advice": "整体特征工程方向的建议..."
}
```

注意：
- 建议必须基于现有字段，不能凭空假设新数据来源
- 优先考虑业务可解释性强的特征
- 避免过度工程，保持简洁实用
```

#### 1.6.3 `configs/prompts/case_analysis.j2`

```jinja2
你是机器学习模型诊断专家。请分析以下模型预测错误的案例，找出潜在原因并提供改进建议。

## 案例信息

### 真实标签
{{ true_label_text }}（{{ true_label }}）

### 模型预测
- 预测概率: {{ "%.4f" | format(probability) }}
- 预测标签: {{ predicted_label_text }}（{{ predicted_label }}）
- 预测置信度: {{ confidence_level }}

### 样本特征
{% for key, value in features.items() %}
- {{ key }}: {{ value }}
{% endfor %}

### SHAP 特征贡献
{% for item in shap_features %}
- {{ item.feature }}: {{ "%.4f" | format(item.shap_value) }}（值={{ item.value }}）
{% endfor %}

## 输出要求

请按以下结构进行分析：

### 一、错误类型判断
- 是 False Positive（误判违约）还是 False Negative（漏判违约）？
- 错误的严重程度评估

### 二、可能原因分析
从以下几个角度分析：
1. **数据角度**：是否存在数据质量问题、特征缺失、异常值？
2. **特征角度**：现有特征是否能充分刻画该客户？
3. **模型角度**：模型是否存在偏见或盲区？
4. **业务角度**：是否存在模型未涵盖的特殊业务场景？

### 三、改进建议
1. **短期优化**：当前可立即调整的措施
2. **中期改进**：需要新增的特征或数据
3. **长期方向**：模型架构或策略层面的改进

### 四、相似案例排查建议
该错误是否可能在其他客户身上重复出现？应该如何排查？

---

请保持专业、客观的分析视角，不要做未经数据支持的猜测。
```

#### 1.6.4 `configs/prompts/chat_assistant.j2`

```jinja2
你是"智能信贷风控分析系统"的 AI 助手，用于帮助用户理解数据、模型和业务。

## 当前系统上下文
{% if dataset_info %}
**数据集状态**:
- 样本数: {{ dataset_info.n_samples }}
- 特征数: {{ dataset_info.n_features }}
- 违约率: {{ dataset_info.default_rate }}
{% endif %}

{% if model_info %}
**模型状态**:
- XGBoost AUC: {{ model_info.xgb_auc }}
- MLP AUC: {{ model_info.mlp_auc }}
- 融合模型 AUC: {{ model_info.ensemble_auc }}
{% endif %}

## 你的能力范围
你可以回答以下类型的问题：
1. 数据相关：字段含义、数据分布、缺失情况等
2. 模型相关：模型原理、性能指标解读、超参含义等
3. 业务相关：风控基础知识、特征业务含义、风险逻辑等
4. 系统使用：如何使用本系统的各个功能

## 行为规范
1. **基于事实回答**：依据系统当前数据回答，不知道就说不知道
2. **简洁清晰**：直接回答问题，不要冗长铺垫
3. **专业准确**：金融术语要正确，数据要精确
4. **教学友好**：可以适当解释专业概念，方便初学者理解
5. **诚实边界**：超出能力范围或敏感问题要明确告知

## 用户问题
{{ user_question }}

请给出你的回答：
```

---

### 步骤 1.7　验证全部 9 个文件

**【目标】** 确认 5 个 YAML 语法正确、4 个模板能被加载。

**【操作】** 在项目根目录执行：

```bat
python -c "import yaml,glob;[print(f,'OK') for f in sorted(glob.glob('configs/*.yaml')) if yaml.safe_load(open(f,encoding='utf-8'))]"
```

**【验证】** 期望输出 5 行，每行一个文件名 + OK：

```
configs\data_config.yaml OK
configs\feature_config.yaml OK
configs\llm_config.yaml OK
configs\model_config.yaml OK
configs\pipeline_config.yaml OK
```

再验证模板能被 Jinja2 加载：

```bat
python -c "from jinja2 import Environment,FileSystemLoader; e=Environment(loader=FileSystemLoader('configs/prompts')); [print(n,'OK') for n in e.list_templates()]"
```

期望输出 4 个 `.j2` 文件名。

**【常见报错】**

| 报错 | 原因 | 修复 |
|---|---|---|
| `yaml.scanner.ScannerError: mapping values are not allowed here` | 冒号后面少了空格，或用了 Tab | 检查报错指的**行号**，该行冒号后补空格 |
| `yaml.parser.ParserError: expected <block end>` | 缩进不对齐 | 同级键必须列对齐 |
| `FileNotFoundError` | 不在项目根目录 | `cd` 回去 |
| 只输出 4 行 | 某个 YAML 是空的或被存成 `.yaml.txt` | `dir configs` 看真实文件名 |

---

## 6. 本章错误对照

### 错误 E-01　`derived_features` 的访问层级

**出现在**：第 4 章 `FeatureBuilder` 会读这个配置

**❌ 原文写法**（把 `derived_features` 当成列表直接遍历）

```python
for rule in config.derived_features:
    name = rule["name"]
```

**✅ 修正写法**

```python
for rule in config.derived_features.rules:
    name = rule["name"]
```

**💥 不修的后果**

```
TypeError: 'AttrDict' object is not iterable
```

或者遍历出 `enabled` 这个键，然后在 `rule["name"]` 处报 `TypeError: string indices must be integers`。

**🧠 为什么**　YAML 里 `derived_features` 下面有两个东西：`enabled: true` 和 `rules: [...]`。规则列表在 `rules` 这个子键里，不是顶层。YAML 的缩进层级直接对应 Python 的点访问层级：

```yaml
derived_features:      # config.derived_features
  enabled: true        # config.derived_features.enabled
  rules:               # config.derived_features.rules
    - name: xxx        # config.derived_features.rules[0].name
```

**✅ 自查**　凡是遍历配置里的列表，先想清楚那个列表在 YAML 的第几层缩进。

---

### 错误 N-02　YAML 里用了 Tab

**出现在**：手写 YAML 时

**❌ 原文写法**　用 Tab 键缩进

**✅ 修正写法**　全部改空格（每级 2 个空格）

**💥 不修的后果**

```
yaml.scanner.ScannerError: found character '\t' that cannot start any token
```

**🧠 为什么**　YAML 规范明确禁止用 Tab 做缩进（因为不同编辑器 Tab 宽度不同，会导致层级歧义）。

**✅ 自查**　VS Code 打开文件，右下角显示 `Spaces: 2` 才对；如果是 `Tab Size`，点它改成 "Indent Using Spaces"。

---

## 7. 常见坑

| 现象 | 原因 | 解决 |
|---|---|---|
| 改了 YAML 但程序行为没变 | `ConfigLoader` 是单例，只读一次 | 重启 Python 进程；开发时调 `configs.reload()` |
| `${DEEPSEEK_API_KEY}` 没被替换 | `.env` 没加载或变量名拼错 | 确认 `.env` 在项目根目录，变量名完全一致 |
| `model: "${DEEPSEEK_MODEL:-deepseek-chat}"` 变成空字符串 | `.env` 里写了 `DEEPSEEK_MODEL=`（空值） | 空值会被当成"存在但为空"，删掉这行或填真值 |
| 模板渲染报 `UndefinedError` | 模板里用了没传进去的变量 | 检查 Agent 调用时 `render()` 的关键字参数是否齐全 |
| YAML 里中文乱码 | 文件不是 UTF-8 | 编辑器另存为 UTF-8 |

---

## 8. 本章自测

**1. 为什么 `learning_rate` 不应该写死在 `xgb_model.py` 里？**

<details><summary>点开看答案</summary>

因为它是"会变的实验参数"。写死后改一次就要改代码、可能引入 bug、也无法记录"这次 0.74 用的是哪套参数"。放 YAML 里，改配置即可，且配置文件能进版本库，实验可复现。

</details>

**2. `derived_features` 下面如果要拿规则列表，代码怎么写？**

<details><summary>点开看答案</summary>

`config.derived_features.rules`。因为 YAML 里 `rules:` 是 `derived_features:` 的子键，缩进多一层。

</details>

**3. 本章的 `pipeline_config.yaml` 里，`split_data` 为什么排在最前面？**

<details><summary>点开看答案</summary>

避免数据泄漏。先切出训练/验证/测试集，之后标准化、特征筛选都只在训练集上 `fit`，测试集只 `transform`。否则测试集信息会泄漏进训练过程，指标虚高。

</details>

**4. `${VAR:-default}` 和 `${VAR}` 有什么区别？**

<details><summary>点开看答案</summary>

前者环境变量不存在时用 `default`；后者不存在时保留原文 `${VAR}`（本项目 `ConfigLoader` 的实现如此）。所以配置默认值要用 `:-` 写法。

</details>

---

## 9. 本章产物清单 + 下一步

**产物**

| 路径 | 行数约 |
|---|---|
| `configs/data_config.yaml` | 100 |
| `configs/feature_config.yaml` | 75 |
| `configs/model_config.yaml` | 110 |
| `configs/llm_config.yaml` | 95 |
| `configs/pipeline_config.yaml` | 40 |
| `configs/prompts/risk_report.j2` | 50 |
| `configs/prompts/feature_suggestion.j2` | 55 |
| `configs/prompts/case_analysis.j2` | 45 |
| `configs/prompts/chat_assistant.j2` | 40 |

**一条命令自检**

```bat
python -c "import yaml,glob;print('YAML OK' if all(yaml.safe_load(open(f,encoding='utf-8')) for f in glob.glob('configs/*.yaml')) else 'FAIL')"
```

期望输出：`YAML OK`

**你现在已经能做到**：把项目里所有可调参数集中管理，改参数不动代码。

**下一章**：写三个工具——**读配置**（把 YAML 变成能用点访问的对象）、**打日志**（出问题能查）、**算指标**（AUC/KS 怎么算）。这三个是后面每一层的地基。
