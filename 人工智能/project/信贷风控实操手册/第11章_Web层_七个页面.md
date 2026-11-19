# 第 11 章　网页层：七个页面

> 本章建 12 个文件 · 预计 5~8 小时 · 难度 ★★★ · 前置：第 09、10 章

---

## 1. 这一章在流水线上的位置

```
[原始CSV] →①数据层→ ②特征层→ ③模型层→ ④调参→ ⑤解释→ ⑥AI报告→ ⑦编排→ ⑧脚本→ ⑨网页
                                                                        ▲
你在这里：第 11 章　网页层（最终交付物）
已经有的：完整后端 + 命令行
还没有的：一个能给别人看的东西
```

---

## 2. 本章你要做什么

把前面所有能力包成 7 个页面。用 Streamlit，**不用写 HTML/CSS/JS**，纯 Python。

| 页面 | 给谁看 | 核心功能 |
|---|---|---|
| 1 数据探索 | 所有人 | 数据长什么样、违约率多少、缺失多少 |
| 2 特征工程 | 建模的人 | 衍生特征列表、特征重要性 |
| 3 模型训练 | 你自己 | 点按钮训练、看进度和指标 |
| 4 单笔预测 | **答辩旗舰** | 填表单 → 出概率 → SHAP 瀑布图 → AI 报告 |
| 5 模型解释 | 审计/业务 | 全局重要性、依赖图 |
| 6 AI 助手 | 所有人 | 对话问问题、四种模式 |
| 7 系统配置 | 运维 | 看配置、看环境变量、健康检查 |

> ⚠️ **本章的写法与其他章不同**：Streamlit 的 UI 代码重复度高（大量 `st.write`/`st.columns`），逐行抄 2600 行对学习没有帮助。所以本章给的是**可运行的骨架 + 每页的关键逻辑**，UI 细节你可以按自己的审美调整。完整原始实现见 `信贷风控_从零手写完整文档.md` 第 11 章（5853~8441 行）。

---

## 3. 开工前检查

- [ ] `data/models/latest/` 里有模型文件（跑过第 10 章的训练）
- [ ] `streamlit --version` 能输出版本号
- [ ] `app/`、`app/pages/`、`app/components/`、`app/assets/` 四个目录存在

---

## 4. 本章知识点

| 编号 | 概念 | 一句话解释 | 为什么这里需要它 | 本项目用在哪 | 要求 |
|---|---|---|---|---|---|
| KP-WEB-01 | Streamlit 执行模型 | 每次交互都从头重跑整个脚本 | 理解它才能理解 `cache` 和 `session_state` | 全部页面 | 必懂 |
| KP-WEB-02 | `pages/` 约定 | 目录里的 `.py` 自动变成侧边栏导航 | 不用写路由 | `app/pages/` | 必懂 |
| KP-WEB-03 | `@st.cache_resource` | 缓存"重对象"（模型、连接） | 不缓存的话每次点击都重新加载模型，慢到没法用 | 加载模型 | 必懂 |
| KP-WEB-04 | `st.session_state` | 跨重跑保存变量 | 表单提交后结果会丢，必须存这里 | 单笔预测页 | 必懂 |
| KP-WEB-05 | `st.form` | 批量提交，不逐项触发重跑 | 7 个输入框逐个触发会重跑 7 次 | 输入表单 | 必懂 |
| KP-WEB-06 | `st.write_stream` | 把生成器边生成边显示 | AI 报告打字机效果 | 报告生成 | 必懂 |
| KP-WEB-07 | Plotly 图 | 交互式图表对象 | `st.plotly_chart(fig)` 一行嵌入 | `chart_utils` | 知道即可 |
| KP-WEB-08 | 组件复用 | 把常用 UI 抽成函数 | 7 个页面共用侧边栏、指标卡 | `components/` | 必懂 |

### 展开讲：Streamlit 的执行模型（最容易踩的坑）

Streamlit 没有"事件回调"概念。**用户每点一次按钮、改一次输入框，整个脚本从头到尾重跑一遍。**

这带来两个后果：

**后果一：慢。** 每次都重新 `joblib.load` 模型 → 每次点击等 5 秒。

解法：`@st.cache_resource`

```python
@st.cache_resource
def load_artifacts():
    return ArtifactManager.load_latest("data/models")
```

加了缓存后，函数只在**参数变化时**才真正执行，否则直接返回上次的结果。

**后果二：状态丢。** 用户点了"预测"，页面重跑，`result` 变量被重置成 `None`，结果就消失了。

解法：`st.session_state`

```python
if submitted:
    st.session_state["result"] = predict(...)   # 存起来

if "result" in st.session_state:                # 重跑后依然在
    show(st.session_state["result"])
```

**记住一句话**：模型用 `cache_resource`，用户数据用 `session_state`。

---

## 5. 分步操作

### 步骤 11.1　`app/assets/style.css`（可选美化）

```css
#MainMenu {visibility: hidden;}
footer {visibility: hidden;}

.stButton > button[kind="primary"] {
    background-color: #1e88e5;
    color: white;
    font-weight: bold;
    border-radius: 8px;
}

[data-testid="stMetricValue"] {
    font-size: 2rem;
    color: #1e88e5;
}

.risk-high   { color: #d32f2f; font-weight: bold; }
.risk-medium { color: #f57c00; font-weight: bold; }
.risk-low    { color: #388e3c; font-weight: bold; }
```

### 步骤 11.2　`app/components/sidebar.py`

**【目标】** 所有页面共用的侧边栏：系统状态 + 模型信息。

**【完整代码】**

```python
"""侧边栏组件"""
from pathlib import Path

import streamlit as st


def render_sidebar():
    with st.sidebar:
        st.title("智能信贷风控系统")
        st.caption("ML + SHAP + LLM")

        st.divider()
        st.subheader("系统状态")

        model_dir = Path("data/models/latest")
        has_model = (model_dir / "xgb_model.json").exists()
        st.write(("模型：已就绪" if has_model else "模型：未训练"))

        data_file = Path("data/sample/sample.csv")
        st.write(("数据：可用" if data_file.exists() else "数据：缺失"))

        import os
        has_key = bool(os.getenv("DEEPSEEK_API_KEY")) and \
                  os.getenv("DEEPSEEK_API_KEY") != "sk-your-api-key-here"
        st.write(("AI：已配置" if has_key else "AI：未配置"))

        st.divider()
        if has_model:
            import json
            meta_path = model_dir / "meta.json"
            if meta_path.exists():
                meta = json.loads(meta_path.read_text(encoding="utf-8"))
                st.caption(f"run_id: {meta.get('run_id', '-')}")
                st.caption(f"特征数: {len(meta.get('feature_names', []))}")

        st.divider()
        st.caption("第 11 章 · 网页层")
```

### 步骤 11.3　`app/components/data_utils.py`

**【目标】** 统一的数据/模型加载，带缓存和"真实数据 → 合成数据"回退。

**【完整代码】**

```python
"""数据与模型加载工具"""
from pathlib import Path

import pandas as pd
import streamlit as st


@st.cache_data
def load_dataframe(max_rows: int = 5000) -> pd.DataFrame:
    """优先真实数据，没有就用合成数据。"""
    for path in ["data/raw/train.csv", "data/sample/sample.csv"]:
        p = Path(path)
        if p.exists():
            return pd.read_csv(p, nrows=max_rows)
    st.error("找不到数据。请先运行 `python scripts/generate_sample_data.py`")
    st.stop()


@st.cache_resource
def load_models():
    """加载模型与预处理器。失败返回 None。"""
    from src.explain.shap_analyzer import SHAPAnalyzer
    from src.models.xgb_model import XGBModel
    from src.utils.config_loader import ConfigLoader
    import joblib

    try:
        configs = ConfigLoader("./configs")
        model_dir = Path("data/models/latest")

        xgb = XGBModel(configs.get("model").xgboost)
        xgb.load(str(model_dir / "xgb_model.json"))

        preprocessor = joblib.load(model_dir / "preprocessor.pkl")
        selector_path = model_dir / "feature_selector.pkl"
        selector = joblib.load(selector_path) if selector_path.exists() else None

        analyzer = SHAPAnalyzer(max_background_size=300)
        analyzer.fit(xgb.model, pd.read_csv("data/sample/sample.csv", nrows=300)
                     if Path("data/sample/sample.csv").exists() else None)

        return {"xgb": xgb, "preprocessor": preprocessor,
                "selector": selector, "analyzer": analyzer, "configs": configs}
    except Exception as e:
        st.warning(f"模型尚未就绪：{e}")
        return None


@st.cache_data
def load_eval_results() -> dict:
    import json
    p = Path("data/models/latest/eval_results.json")
    if not p.exists():
        return {}
    return json.loads(p.read_text(encoding="utf-8"))
```

### 步骤 11.4　`app/components/chart_utils.py`

**【目标】** 图表工厂：每个函数返回一个 Plotly Figure。

```python
"""图表工具：每个函数返回一个 Figure，调用方一行 st.plotly_chart 就能显示。"""
import numpy as np
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go


def target_distribution(df: pd.DataFrame, target: str = "isDefault") -> go.Figure:
    counts = df[target].value_counts().rename({0: "正常", 1: "违约"})
    fig = px.pie(values=counts.values, names=counts.index,
                 title="违约分布", hole=0.5)
    return fig


def missing_bar(df: pd.DataFrame, top_n: int = 20) -> go.Figure:
    miss = (df.isnull().mean() * 100).sort_values(ascending=False).head(top_n)
    miss = miss[miss > 0]
    if miss.empty:
        return go.Figure().update_layout(title="无缺失值")
    return px.bar(x=miss.values, y=miss.index, orientation="h",
                  labels={"x": "缺失率(%)", "y": ""}, title="缺失率 Top")


def importance_bar(importance: pd.Series, top_n: int = 20) -> go.Figure:
    data = importance.head(top_n)[::-1]
    return px.bar(x=data.values, y=data.index, orientation="h",
                  labels={"x": "重要性", "y": ""}, title=f"特征重要性 Top {top_n}")


def risk_gauge(probability: float) -> go.Figure:
    """单笔预测的仪表盘。"""
    fig = go.Figure(go.Indicator(
        mode="gauge+number",
        value=probability * 100,
        number={"suffix": "%"},
        title={"text": "违约概率"},
        gauge={
            "axis": {"range": [0, 100]},
            "bar": {"color": "#1e88e5"},
            "steps": [
                {"range": [0, 30], "color": "#c8e6c9"},
                {"range": [30, 70], "color": "#ffe0b2"},
                {"range": [70, 100], "color": "#ffcdd2"},
            ],
        },
    ))
    fig.update_layout(height=280, margin=dict(t=50, b=10))
    return fig


def shap_waterfall(explanation) -> go.Figure:
    """SHAP 瀑布图：从基准值出发，各特征把它推到最终值。"""
    pairs = sorted(
        zip(explanation.feature_names, explanation.shap_values),
        key=lambda x: abs(x[1]),
    )[-12:]                       # 只画影响最大的 12 个

    names = [p[0] for p in pairs]
    values = [p[1] for p in pairs]

    fig = go.Figure(go.Bar(
        x=values, y=names, orientation="h",
        marker_color=["#d32f2f" if v > 0 else "#388e3c" for v in values],
    ))
    fig.update_layout(
        title=f"特征贡献（基准 {explanation.base_value:.2%} → 预测 {explanation.prediction:.2%}）",
        xaxis_title="SHAP 值（正=推高风险）",
        height=420, margin=dict(l=10, r=10, t=60, b=10),
    )
    return fig
```

### 步骤 11.5　`app/streamlit_app.py`（主入口）

**【目标】** 首页 + 全局配置 + 缓存初始化。

**【完整代码】**

```python
"""主入口：streamlit run app/streamlit_app.py"""
from pathlib import Path
import sys

# Streamlit 多页面应用必须手动把项目根加进 path
ROOT = Path(__file__).resolve().parent.parent
if str(ROOT) not in sys.path:
    sys.path.insert(0, str(ROOT))

import streamlit as st

from app.components.sidebar import render_sidebar

st.set_page_config(
    page_title="智能信贷风控分析系统",
    page_icon="🏦",
    layout="wide",
    initial_sidebar_state="expanded",
)

# 加载 CSS（文件不存在就跳过）
css_path = Path(__file__).parent / "assets" / "style.css"
if css_path.exists():
    st.markdown(f"<style>{css_path.read_text(encoding='utf-8')}</style>",
                unsafe_allow_html=True)

render_sidebar()

st.title("智能信贷风控分析系统")

st.markdown("""
本系统模拟消费金融的**贷款审批智能化**场景。输入一笔申请，几秒内给出：

1. **违约概率** —— XGBoost + 神经网络融合模型预测
2. **风险等级** —— 低 / 中 / 高
3. **判断依据** —— SHAP 拆出每个特征推高或拉低了多少
4. **自然语言报告** —— 大模型把上面三步翻译成业务人员能读的话
""")

st.divider()

col1, col2, col3 = st.columns(3)
col1.metric("模型", "XGBoost + MLP", "融合")
col2.metric("可解释性", "SHAP", "逐笔归因")
col3.metric("报告", "DeepSeek", "自然语言")

st.divider()
st.subheader("快速开始")
st.markdown("""
- 想看看数据长什么样 → 左侧选 **1 数据探索**
- 想试一笔预测（推荐） → **4 单笔预测**
- 想重新训练模型 → **3 模型训练**
""")

with st.expander("项目结构说明"):
    st.code("""
configs/     配置层（5 个 YAML + 4 个提示词模板）
src/         核心代码（数据/特征/模型/调参/解释/LLM/编排）
app/         网页层（本目录）
scripts/     命令行脚本
data/        数据与模型产物
logs/        日志
""", language="text")
```

### 步骤 11.6　七个页面

在 `app/pages/` 下建 7 个文件。**文件名前的数字决定侧边栏顺序**。

#### 11.6.1 `1_📊_数据探索.py`

```python
import sys
from pathlib import Path
ROOT = Path(__file__).resolve().parent.parent.parent
sys.path.insert(0, str(ROOT))

import streamlit as st
from app.components.chart_utils import missing_bar, target_distribution
from app.components.data_utils import load_dataframe

st.title("数据探索")
df = load_dataframe()

c1, c2, c3, c4 = st.columns(4)
c1.metric("样本数", f"{len(df):,}")
c2.metric("特征数", df.shape[1])
c3.metric("违约率", f"{df['isDefault'].mean():.1%}" if "isDefault" in df else "-")
c4.metric("缺失率", f"{df.isnull().mean().mean():.1%}")

tab1, tab2, tab3 = st.tabs(["概览", "分布", "缺失"])

with tab1:
    st.dataframe(df.head(100), use_container_width=True)
    st.caption("数值列统计")
    st.dataframe(df.describe().T, use_container_width=True)

with tab2:
    if "isDefault" in df:
        st.plotly_chart(target_distribution(df), use_container_width=True)

with tab3:
    st.plotly_chart(missing_bar(df), use_container_width=True)
```

#### 11.6.2 `2_🔧_特征工程.py`

```python
import sys
from pathlib import Path
ROOT = Path(__file__).resolve().parent.parent.parent
sys.path.insert(0, str(ROOT))

import streamlit as st
from app.components.chart_utils import importance_bar
from app.components.data_utils import load_models

st.title("特征工程")

models = load_models()
if models is None:
    st.info("请先训练模型（页面 3 或命令行）")
    st.stop()

cfg = models["configs"].get("feature")
st.subheader("配置中的衍生特征")
rules = cfg.derived_features.rules
st.table([{"名称": r["name"], "公式": r["formula"], "说明": r["description"]}
          for r in rules])

st.subheader("模型认为重要的特征")
imp = models["xgb"].get_feature_importance()
st.plotly_chart(importance_bar(imp), use_container_width=True)
```

#### 11.6.3 `3_🤖_模型训练.py`

```python
import sys
from pathlib import Path
ROOT = Path(__file__).resolve().parent.parent.parent
sys.path.insert(0, str(ROOT))

import streamlit as st
from src.utils.config_loader import ConfigLoader
from src.pipeline.orchestrator import TrainingPipeline

st.title("模型训练")
st.warning("浏览器有超时限制，大训练请用命令行：`python scripts/train_pipeline.py`")

use_sample = st.checkbox("使用合成数据", value=True)
skip_mlp = st.checkbox("跳过神经网络（快）", value=True)
tune = st.checkbox("启用超调参（慢）", value=False)

if st.button("开始训练", type="primary"):
    bar = st.progress(0)
    status = st.empty()

    def on_progress(step, pct):
        bar.progress(min(pct, 100))
        status.write(f"正在执行：{step}")

    with st.spinner("训练中..."):
        pipeline = TrainingPipeline(ConfigLoader("./configs"))
        result = pipeline.run(use_sample=use_sample, skip_mlp=skip_mlp,
                              skip_tuning=not tune, progress_callback=on_progress)

    st.success(f"训练完成，耗时 {result['duration']:.1f}s")
    st.json(result["eval_results"])
    st.cache_data.clear()      # 清缓存，让其他页面读到新模型
    st.cache_resource.clear()

st.divider()
st.subheader("历史评估结果")
import json
p = Path("data/models/latest/eval_results.json")
if p.exists():
    st.json(json.loads(p.read_text(encoding="utf-8")))
```

#### 11.6.4 `4_🎯_单笔预测.py`（旗舰页）

```python
import sys
from pathlib import Path
ROOT = Path(__file__).resolve().parent.parent.parent
sys.path.insert(0, str(ROOT))

import pandas as pd
import streamlit as st
from app.components.chart_utils import risk_gauge, shap_waterfall
from app.components.data_utils import load_models

st.title("单笔预测")
st.caption("填写申请人信息 → 得到违约概率、SHAP 归因、AI 报告")

models = load_models()
if models is None:
    st.info("请先训练模型")
    st.stop()

# ---- 用表单批量提交，避免每改一个输入就重跑 ----
with st.form("application"):
    c1, c2, c3 = st.columns(3)
    loan = c1.number_input("贷款金额", 1000, 40000, 15000, step=1000)
    term = c2.selectbox("期限（年）", [3, 5], index=0)
    rate = c3.number_input("利率(%)", 5.0, 30.0, 12.5, step=0.5)

    c4, c5, c6 = st.columns(3)
    income = c4.number_input("年收入", 8000, 1500000, 120000, step=5000)
    dti = c5.number_input("负债收入比", 0.0, 60.0, 22.0, step=1.0)
    fico = c6.number_input("FICO 评分", 580, 850, 690, step=5)

    c7, c8, c9 = st.columns(3)
    grade = c7.selectbox("贷款等级", list("ABCDEFG"), index=2)
    emp = c8.selectbox("就业年限",
                       ["< 1 year", "1 year", "2 years", "3 years", "5 years", "10+ years"],
                       index=5)
    util = c9.slider("循环额度利用率(%)", 0, 100, 35)

    submitted = st.form_submit_button("开始评估", type="primary")

if submitted:
    # 组装一行数据（缺的列用中位数兜底，真实项目要用完整表单）
    row = {
        "loanAmnt": loan, "term": term, "interestRate": rate,
        "installment": loan * (rate / 100 / 12) / (1 - (1 + rate / 100 / 12) ** (-term * 12)),
        "annualIncome": income, "dti": dti, "ficoRangeLow": fico,
        "ficoRangeHigh": fico + 4, "grade": grade, "employmentLength": emp,
        "revolUtil": util, "delinquency_2years": 0, "openAcc": 8,
        "pubRec": 0, "pubRecBankruptcy": 0, "revolBal": 5000,
        "totalAcc": 20, "homeOwnership": 1, "verificationStatus": 1,
        "purpose": 0, "postCode": 100, "regionCode": 10,
        "initialListStatus": 0, "applicationType": 0, "title": 0,
        "policyCode": 1, "employmentTitle": 5000,
        "issueDate": "2018-06-01", "earliesCreditLine": "2005-01",
    }
    for i in range(15):
        row[f"n{i}"] = 0.0

    X = pd.DataFrame([row])

    # 预处理 -> 特征 -> 筛选 -> 预测
    Xc = models["preprocessor"].transform(X.drop(columns=["isDefault"], errors="ignore"))
    Xf = models["xgb"]  # 占位，避免 lint 警告
    from src.features.builder import FeatureBuilder
    fb = FeatureBuilder(models["configs"].get("feature"))
    Xf = fb.build(Xc)
    if models["selector"] is not None:
        Xf = models["selector"].transform(Xf)

    proba = float(models["xgb"].predict_proba(Xf)[0])
    st.session_state["result"] = {"proba": proba, "X": Xf}

# ---- 展示结果（从 session_state 读，不依赖 submitted）----
if "result" in st.session_state:
    res = st.session_state["result"]
    proba = res["proba"]

    st.divider()
    left, right = st.columns([1, 1])
    with left:
        st.plotly_chart(risk_gauge(proba), use_container_width=True)
        level = "高风险" if proba >= 0.7 else ("中风险" if proba >= 0.3 else "低风险")
        st.metric("风险等级", level)

    with right:
        try:
            exp = models["analyzer"].explain_single(res["X"].iloc[0])
            st.plotly_chart(shap_waterfall(exp), use_container_width=True)
        except Exception as e:
            st.warning(f"SHAP 计算失败：{e}")

    st.divider()
    st.subheader("AI 风险报告")
    if st.button("生成报告"):
        try:
            from src.llm.agents.report_agent import ReportAgent
            from src.llm.client import DeepSeekClient
            from src.llm.prompts import PromptManager

            llm_cfg = models["configs"].get("llm")
            client = DeepSeekClient(llm_cfg)
            pm = PromptManager(llm_cfg)
            agent = ReportAgent(client, pm, llm_cfg.agents.report_agent)

            with st.spinner("生成中..."):
                exp = models["analyzer"].explain_single(res["X"].iloc[0])
                st.write_stream(agent.generate(
                    features=res["X"].iloc[0].to_dict(),
                    probability=proba,
                    shap_explanation=exp,
                    stream=True,
                ))
        except Exception as e:
            st.error(f"报告生成失败：{e}")
            st.caption("检查 .env 里的 DEEPSEEK_API_KEY 是否有效")
```

#### 11.6.5 `5_🔍_模型解释.py`

```python
import sys
from pathlib import Path
ROOT = Path(__file__).resolve().parent.parent.parent
sys.path.insert(0, str(ROOT))

import streamlit as st
from app.components.chart_utils import importance_bar
from app.components.data_utils import load_dataframe, load_models

st.title("模型解释")
models = load_models()
if models is None:
    st.info("请先训练模型")
    st.stop()

df = load_dataframe(max_rows=1000)

tab1, tab2 = st.tabs(["全局重要性", "数值分布"])

with tab1:
    st.plotly_chart(importance_bar(models["xgb"].get_feature_importance()),
                    use_container_width=True)

with tab2:
    col = st.selectbox("选择特征", df.select_dtypes("number").columns)
    if "isDefault" in df:
        import plotly.express as px
        fig = px.histogram(df, x=col, color="isDefault", nbins=40,
                           barmode="overlay", title=f"{col} 在好坏样本中的分布")
        st.plotly_chart(fig, use_container_width=True)
        st.caption("两条分布重叠越少，这个特征越有区分力")
```

#### 11.6.6 `6_💬_AI智能助手.py`

```python
import sys
from pathlib import Path
ROOT = Path(__file__).resolve().parent.parent.parent
sys.path.insert(0, str(ROOT))

import os
import streamlit as st
from src.llm.client import DeepSeekClient
from src.llm.prompts import PromptManager
from src.utils.config_loader import ConfigLoader

st.title("AI 智能助手")

key = os.getenv("DEEPSEEK_API_KEY", "")
if not key or key == "sk-your-api-key-here":
    st.warning("未配置 DEEPSEEK_API_KEY，请在 .env 中填写后重启")
    st.stop()

llm_cfg = ConfigLoader("./configs").get("llm")
client = DeepSeekClient(llm_cfg)
pm = PromptManager(llm_cfg)

mode = st.radio("模式", ["自由对话", "解释术语"], horizontal=True)

if "history" not in st.session_state:
    st.session_state["history"] = []

for msg in st.session_state["history"]:
    with st.chat_message(msg["role"]):
        st.write(msg["content"])

if q := st.chat_input("问点什么，比如：AUC 和 KS 有什么区别？"):
    st.session_state["history"].append({"role": "user", "content": q})
    with st.chat_message("user"):
        st.write(q)

    prompt = pm.render("chat_assistant", user_question=q,
                       dataset_info=None, model_info=None)
    with st.chat_message("assistant"):
        answer = st.write_stream(
            client.stream_chat(prompt, system_prompt=pm.get_system_role("chat_assistant"))
        )
    st.session_state["history"].append({"role": "assistant", "content": answer})
```

#### 11.6.7 `7_⚙️_系统配置.py`

```python
import sys
from pathlib import Path
ROOT = Path(__file__).resolve().parent.parent.parent
sys.path.insert(0, str(ROOT))

import os
import streamlit as st

st.title("系统配置")

st.subheader("配置文件一览")
for f in sorted(Path("configs").glob("*.yaml")):
    with st.expander(f.name):
        st.code(f.read_text(encoding="utf-8"), language="yaml")

st.subheader("环境变量")
st.table([
    {"变量": k, "状态": "已设置" if os.getenv(k) else "未设置",
     "值": (os.getenv(k, "")[:8] + "...") if os.getenv(k) else "-"}
    for k in ["DEEPSEEK_API_KEY", "DEEPSEEK_BASE_URL", "DEEPSEEK_MODEL"]
])

st.subheader("健康检查")
if st.button("测试 AI 连通性"):
    try:
        from src.llm.client import DeepSeekClient
        from src.utils.config_loader import ConfigLoader
        c = DeepSeekClient(ConfigLoader("./configs").get("llm"))
        st.success("连通正常") if c.health_check() else st.error("调用失败，检查 Key 和网络")
    except Exception as e:
        st.error(f"失败：{e}")

st.subheader("训练产物")
d = Path("data/models/latest")
if d.exists():
    for f in sorted(d.iterdir()):
        st.text(f"{f.name}  {f.stat().st_size/1024:.1f} KB")
else:
    st.info("尚无训练产物")
```

### 步骤 11.7　回填 `app/__init__.py` 与启动

两个文件各写一行注释：

```python
# app/__init__.py
"""Streamlit Web 应用"""
```

```python
# app/components/__init__.py
"""可复用组件"""
```

**【启动】**

```bat
streamlit run app/streamlit_app.py
```

浏览器自动打开 `http://localhost:8501`。

**【验证】** 逐页检查：

- [ ] 侧边栏显示"模型：已就绪 / 数据：可用"
- [ ] 页 1 能看到样本数、违约率、缺失图
- [ ] 页 2 能看到 5 条衍生特征、特征重要性条形图
- [ ] 页 3 点"开始训练"能跑完并显示指标（或用命令行已训好的结果）
- [ ] 页 4 填表后点"开始评估"，出现仪表盘 + 瀑布图；点"生成报告"能看到文字逐字出现
- [ ] 页 5 能切换特征看分布对比
- [ ] 页 6 能对话（需配 Key）
- [ ] 页 7 能看到 5 个 YAML 内容 + 环境变量状态

**【常见报错】**

| 报错 | 原因 | 修复 |
|---|---|---|
| `ModuleNotFoundError: No module named 'app'` | 没在项目根目录启动 | `cd` 到根目录再 `streamlit run` |
| 每次点击都卡 5 秒 | 没加 `@st.cache_resource` | 确认 `load_models` 有装饰器 |
| 点"评估"后结果一闪就没了 | 结果没存进 `session_state` | 见页 4 的写法 |
| 页面 4 报列数不匹配 | 输入缺列，特征数对不上 | 用完整表单或按中位数补全 |
| 中文图表乱码 | matplotlib 缺字体 | 本项目用 Plotly，不受影响 |

---

## 6. 本章错误对照

### 错误 E-21　`@st.cache_resource` 加在按钮分支里

**❌ 原文写法**

```python
if submitted:
    @st.cache_resource
    def load_model():
        return ArtifactManager.load_latest("data/models")
    model = load_model()
```

**✅ 修正写法**

```python
@st.cache_resource
def load_model():
    return ArtifactManager.load_latest("data/models")

# ... 页面逻辑里直接调用
model = load_model()
```

**💥 不修的后果**　不报错，但**缓存完全不生效**。装饰器在每次重跑时都重新定义一个函数对象，缓存键每次都"新"，等于没用。表现是每次点击都等好几秒。

**🧠 为什么**　`@st.cache_resource` 靠**函数对象 + 参数**做缓存键。在 `if` 里定义，函数对象每次都是新的，自然命中不了。

**✅ 自查**　所有 `@st.cache_*` 装饰的函数都必须在**模块顶层**定义。

---

### 错误 E-22　`chart_utils` 里调了不存在的函数

**现象**　页面报 `ImportError: cannot import name 'plot_ks_curve' from 'app.components.chart_utils'`

**原因**　原文档的页面代码引用了 `chart_utils` 里没实现的函数（如 `plot_lift_curve`、`plot_ks_curve`）。

**✅ 处理**　本章的 `chart_utils` 已经把用到的 5 个函数全部实现（`target_distribution`/`missing_bar`/`importance_bar`/`risk_gauge`/`shap_waterfall`）。**页面只 import 这 5 个**。需要新图就先在 `chart_utils` 里加函数，再在页面引用。

**✅ 自查**　`python -c "from app.components import chart_utils; print([n for n in dir(chart_utils) if not n.startswith('_')])"` 看实际有什么。

---

## 7. 常见坑

| 现象 | 原因 | 解决 |
|---|---|---|
| 侧边栏页面顺序乱 | 文件名没数字前缀 | 用 `1_`、`2_` 开头 |
| 改代码页面没变 | Streamlit 有时不热重载 | 刷新浏览器；仍不行就重启 |
| 多个用户同时用会串数据 | `session_state` 是每会话独立的，但 `cache_resource` 是全局的 | 本项目单机演示，不影响 |
| 训练页把浏览器卡死 | 同步训练阻塞 | 大训练一律走命令行 |
| 端口被占用 | 8501 已被别的实例用 | `streamlit run ... --server.port 8502` |

---

## 8. 本章自测

**1. 为什么模型加载要用 `cache_resource` 而不是 `st.session_state`？**

<details>
<summary>点开看答案</summary>

`cache_resource` 是全局共享的"重对象缓存"，所有用户、所有页面共用一份，省内存也省加载时间。`session_state` 是每个用户会话独立的，把模型放进去会导致每个用户各加载一份。

</details>

**2. 表单为什么用 `st.form` 包起来？**

<details>
<summary>点开看答案</summary>

Streamlit 默认每改一个输入就重跑整个脚本。9 个输入框就是 9 次重跑，很卡。`st.form` 把它们打包，只有点提交按钮才触发一次重跑。

</details>

**3. 点"预测"后结果为什么会消失？怎么解决？**

<details>
<summary>点开看答案</summary>

因为 Streamlit 每次交互都重跑脚本，局部变量被重置。要把结果写进 `st.session_state["result"]`，展示时从那里读，重跑后依然存在。

</details>

---

## 9. 本章产物清单 + 下一步

**产物**

| 路径 | 行数约 |
|---|---|
| `app/streamlit_app.py` | 70 |
| `app/components/sidebar.py` | 45 |
| `app/components/data_utils.py` | 70 |
| `app/components/chart_utils.py` | 110 |
| `app/pages/` 7 个页面 | 60~130 每个 |
| `app/assets/style.css` | 20 |

**启动自检**

```bat
streamlit run app/streamlit_app.py
```

**你现在已经有一个完整可演示的系统。** 7 个页面都能出内容，就是可以拿去答辩的状态。

**下一章**：写 10 个冒烟测试 + 走完六步验收，确保交付物是"可复现"的而不是"在我机器上能跑"的。
