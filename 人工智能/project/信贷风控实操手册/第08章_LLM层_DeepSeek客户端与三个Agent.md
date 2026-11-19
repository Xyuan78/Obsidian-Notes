# 第 08 章　AI 层：DeepSeek 客户端与三个 Agent

> 本章建 5 个文件 + 回填 2 个 · 预计 3~4 小时 · 难度 ★★★★ · 前置：第 07 章

---

## 1. 这一章在流水线上的位置

```
[原始CSV] →①数据层→ ②特征层→ ③模型层→ ④调参→ ⑤解释→ ⑥AI报告→ ⑦编排→ ⑧脚本→ ⑨网页
                                                      ▲
你在这里：第 08 章　AI 层
已经有的：预测 + SHAP 归因
还没有的：编排、脚本、网页
```

**这一章不需要 API Key 也能做完。** 客户端代码在没有 Key 时会警告但不会崩，三个 Agent 也能正常 import。真正的调用留到第 11 章网页里试。

---

## 2. 本章你要做什么

| 文件 | 做什么 |
|---|---|
| `client.py` | 封装 DeepSeek 调用：重试、限流、脱敏、审计、流式 |
| `prompts.py` | 加载并渲染 4 个 `.j2` 模板 |
| `agents/report_agent.py` | 预测 + SHAP → 人话风险报告 |
| `agents/feature_agent.py` | 数据集信息 → 特征建议（带可执行代码） |
| `agents/case_agent.py` | 判错的样本 → 原因诊断 |

**三个 Agent 都是同一个三段式**：

```
build_context（整理数据） → render（套模板） → call_llm（调模型） → parse（解析结果）
```

学会一个，另外两个就都会了。

---

## 3. 开工前检查

- [ ] 上一章输出的是 `explain OK`
- [ ] `configs/prompts/` 下有 4 个 `.j2` 文件
- [ ] `.env` 里 `DEEPSEEK_API_KEY` 已填（没填也能继续，只是调用时会失败）
- [ ] `src/llm/__init__.py` 和 `src/llm/agents/__init__.py` 都是**空的**

---

## 4. 本章知识点

| 编号 | 概念 | 一句话解释 | 为什么这里需要它 | 本项目用在哪 | 要求 |
|---|---|---|---|---|---|
| KP-LLM-01 | OpenAI 兼容协议 | 各家大模型都仿 OpenAI 的接口格式 | 装一个 `openai` 库就能调 DeepSeek | `DeepSeekClient` | 必懂 |
| KP-LLM-02 | system / user 消息 | system 定角色，user 提要求 | 角色设定决定输出风格 | `_build_messages` | 必懂 |
| KP-LLM-03 | temperature | 输出的随机程度，0 最稳 | 风控报告要稳定，用 0.1~0.3 | `default_params` | 必懂 |
| KP-LLM-04 | 流式输出 | 一个字一个字返回 | 网页"打字机效果"，用户体验关键 | `stream_chat` | 必懂 |
| KP-LLM-05 | 重试与退避 | 失败后等一会再试，间隔递增 | 网络抖动是常态 | `@retry` | 必懂 |
| KP-LLM-06 | 提示词模板 | 把变量填进固定文本 | 改话术不动代码 | `PromptManager` | 必懂 |
| KP-LLM-07 | 上下文注入 | 把计算结果塞进 prompt | 让 LLM 基于事实说话，抑制幻觉 | `_build_context` | 必懂 |
| KP-LLM-08 | JSON 输出约束 | 要求模型只返回 JSON | 程序要解析结构化结果 | `feature_agent` | 必懂 |
| KP-BIZ-07 | PII 脱敏 | 敏感字段不外发 | 金融合规硬要求 | `_sanitize_input` | 必懂 |
| KP-BIZ-08 | 审计日志 | 每次调用留痕 | 监管要求可追溯 | `_log_audit` | 知道即可 |

### 展开讲 1：为什么不让 LLM 直接算概率

新手常想：把客户资料丢给 GPT，让它判断违约风险不就行了？

不行，三个原因：

1. **不可复现**：同样输入，不同时间可能给不同答案
2. **不可验证**：它给的 73% 没有任何数学依据
3. **不可审计**：监管问"这个 73% 怎么来的"，你答不上来

所以正确分工是：

```
ML 模型  → 算出 0.73（可复现、可验证）
SHAP     → 拆出"哪 5 个因素贡献了多少"（可解释）
LLM      → 把上面这些翻译成业务人员能读的话（可读）
```

**LLM 只负责"翻译"，不负责"判断"。** 这是这套架构最重要的设计原则，也是答辩时的核心论点。

### 展开讲 2：怎么防止 LLM 编造数字

LLM 有个坏习惯：你给它 5 个特征，它可能自己编出第 6 个。防法有三层：

| 层 | 做法 |
|---|---|
| Prompt 层 | 明确写"只能使用给定数据，不得编造数值" |
| 数据层 | 所有数字都从 SHAP 对象里取，不手写 |
| 校验层 | 输出后检查关键数字是否都能在输入里找到（可选） |

本项目做了前两层。第三层是加分项，你可以在第 11 章自己加。

---

## 5. 分步操作

### 步骤 8.1　写 `src/llm/client.py`

**【目标】** 一个健壮的 API 客户端。

**【文件】** `src/llm/client.py`（新建）

**【完整代码】**

```python
"""
DeepSeek API 客户端
===================
封装重试、限流、脱敏、审计、流式输出。
"""
from __future__ import annotations

import json
import os
import re
import time
from datetime import datetime
from pathlib import Path
from typing import Dict, Generator, List, Optional

from openai import OpenAI
from tenacity import retry, retry_if_exception_type, stop_after_attempt, wait_exponential

from src.utils.logger import get_logger

logger = get_logger("llm.client")


class DeepSeekClient:
    """DeepSeek 客户端（OpenAI 兼容接口）。"""

    def __init__(self, config):
        # config 传的是 configs.get("llm")
        self.config = config
        self.deepseek_cfg = config.deepseek

        api_key = os.environ.get(self.deepseek_cfg.api_key_env)
        if not api_key or api_key == "sk-your-api-key-here":
            logger.warning(
                f"未找到有效的 {self.deepseek_cfg.api_key_env}。\n"
                f"请在 .env 中配置真实 Key，申请地址 https://platform.deepseek.com\n"
                f"（前 7 章不需要 Key，本章之后的功能才需要）"
            )

        self.client = OpenAI(
            api_key=api_key or "dummy-key",
            base_url=self.deepseek_cfg.base_url,
            timeout=self.deepseek_cfg.default_params.timeout,
        )

        self.model = self.deepseek_cfg.model
        self.default_params = dict(self.deepseek_cfg.default_params)
        self.default_params.pop("timeout", None)   # timeout 已在 client 上设

        self._request_times: List[float] = []

        self.audit_log_path = None
        if hasattr(config, "safety") and config.safety.audit_log.enabled:
            self.audit_log_path = Path(config.safety.audit_log.log_file)
            self.audit_log_path.parent.mkdir(parents=True, exist_ok=True)

    @retry(
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=1, min=2, max=30),
        retry=retry_if_exception_type(Exception),
        reraise=True,
    )
    def chat(
        self,
        prompt: str,
        system_prompt: Optional[str] = None,
        history: Optional[List[Dict]] = None,
        **kwargs,
    ) -> str:
        """非流式对话，返回完整字符串。"""
        self._check_rate_limit()

        # 注意顺序：先脱敏，再拼消息（否则脱敏白做）
        safe_prompt = self._sanitize_input(prompt)
        messages = self._build_messages(safe_prompt, system_prompt, history)
        params = {**self.default_params, **kwargs}

        t0 = time.time()
        response = self.client.chat.completions.create(
            model=self.model,
            messages=messages,
            stream=False,
            **params,
        )
        duration = time.time() - t0

        content = response.choices[0].message.content
        usage = response.usage.model_dump() if response.usage else {}

        self._log_audit(safe_prompt, content, duration, usage)
        logger.info(f"LLM 调用完成 - 耗时 {duration:.2f}s, usage={usage}")
        return content

    def stream_chat(
        self,
        prompt: str,
        system_prompt: Optional[str] = None,
        history: Optional[List[Dict]] = None,
        **kwargs,
    ) -> Generator[str, None, None]:
        """流式对话，逐字 yield。Web 打字机效果靠它。"""
        self._check_rate_limit()

        safe_prompt = self._sanitize_input(prompt)
        messages = self._build_messages(safe_prompt, system_prompt, history)
        params = {**self.default_params, **kwargs}

        full_response = ""
        t0 = time.time()

        try:
            stream = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                stream=True,
                **params,
            )
            for chunk in stream:
                delta = chunk.choices[0].delta
                if delta and delta.content:
                    full_response += delta.content
                    yield delta.content

            duration = time.time() - t0
            self._log_audit(safe_prompt, full_response, duration, {"stream": True})
            logger.info(f"LLM 流式调用完成 - 耗时 {duration:.2f}s")

        except Exception as e:
            logger.error(f"LLM 流式调用失败: {e}")
            # 流式没法抛异常给调用方，用文字提示
            yield f"\n\n[调用出错] {e}"

    # ---------------- 内部方法 ----------------

    def _build_messages(self, prompt, system_prompt=None, history=None) -> List[Dict]:
        messages = []
        if system_prompt:
            messages.append({"role": "system", "content": system_prompt})
        if history:
            messages.extend(history)
        messages.append({"role": "user", "content": prompt})
        return messages

    def _check_rate_limit(self) -> None:
        if not self.deepseek_cfg.rate_limit.enabled:
            return

        now = time.time()
        self._request_times = [t for t in self._request_times if now - t < 60]

        max_rpm = self.deepseek_cfg.rate_limit.requests_per_minute
        if len(self._request_times) >= max_rpm:
            wait_time = 60 - (now - self._request_times[0])
            logger.warning(f"触达限流 ({max_rpm}/min)，等待 {wait_time:.1f}s")
            time.sleep(max(wait_time, 1))

        self._request_times.append(now)

    def _sanitize_input(self, text: str) -> str:
        if not hasattr(self.config, "safety") or not self.config.safety.content_filter.enabled:
            return text
        for pattern in self.config.safety.content_filter.block_patterns:
            text = re.sub(pattern, "[REDACTED]", text, flags=re.IGNORECASE)
        return text

    def _log_audit(self, prompt, response, duration, usage) -> None:
        if self.audit_log_path is None:
            return
        record = {
            "timestamp": datetime.now().isoformat(),
            "model": self.model,
            "duration_sec": round(duration, 2),
            "usage": usage,
            "prompt_preview": prompt[:200],
        }
        try:
            with open(self.audit_log_path, "a", encoding="utf-8") as f:
                f.write(json.dumps(record, ensure_ascii=False) + "\n")
        except Exception as e:
            logger.warning(f"审计日志写入失败: {e}")

    def health_check(self) -> bool:
        """发一条 hello 测通不通。"""
        try:
            self.chat("hello", max_tokens=10)
            return True
        except Exception as e:
            logger.error(f"健康检查失败: {e}")
            return False
```

**【逐块讲解】**

| 代码段 | 要点 |
|---|---|
| `api_key or "dummy-key"` | 没 Key 也不让构造失败，方便前 7 章开发和导入测试 |
| `@retry(...)` | 失败重试 3 次，等待 2s → 4s → 8s（指数退避） |
| `safe_prompt = self._sanitize_input(...)` 在 `_build_messages` **之前** | 顺序反了脱敏就白做了（见 E-15） |
| `stream=True` + `yield delta.content` | 逐字返回，生成器天然支持"边生成边显示" |
| `_check_rate_limit` | 滑动窗口：只看最近 60 秒的请求数 |
| `hasattr(config, "safety")` | 配置里可能没有这一块，防御性写法 |

**【验证】** 不需要 Key 也能测构造：

```bat
python -c "from src.utils import ConfigLoader; from src.llm.client import DeepSeekClient; c=ConfigLoader('./configs'); cl=DeepSeekClient(c.get('llm')); print('model:', cl.model); print('client OK')"
```

期望输出：`model: deepseek-chat` 和 `client OK`。若没配 Key，会先打印一条 WARNING，**这是正常的**。

---

### 步骤 8.2　写 `src/llm/prompts.py`

**【目标】** 把 `.j2` 模板加载进来并渲染。

**【文件】** `src/llm/prompts.py`（新建）

**【完整代码】**

```python
"""
Prompt 管理器
=============
Jinja2 模板的加载、渲染、缓存。
"""
from __future__ import annotations

from pathlib import Path
from typing import Dict

from jinja2 import Environment, FileSystemLoader, Template

from src.utils.logger import get_logger

logger = get_logger("llm.prompts")


class PromptManager:
    """模板管理器。构造收整个 llm 配置块。"""

    def __init__(self, config):
        self.config = config
        self.templates_dir = Path(config.prompts.templates_dir)
        self.prompts_cfg = config.prompts

        if not self.templates_dir.exists():
            raise FileNotFoundError(
                f"Prompt 模板目录不存在: {self.templates_dir}\n"
                f"提示: 请在项目根目录运行"
            )

        self.env = Environment(
            loader=FileSystemLoader(str(self.templates_dir)),
            trim_blocks=True,
            lstrip_blocks=True,
        )
        self._template_cache: Dict[str, Template] = {}

    def render(self, template_name: str, **kwargs) -> str:
        """template_name 是 llm_config.yaml 里的键，如 risk_report。"""
        cfg = getattr(self.prompts_cfg, template_name, None)
        if cfg is None:
            raise KeyError(f"未配置的 Prompt 模板: {template_name}")

        template = self._get_template(cfg.template_file)
        try:
            return template.render(**kwargs)
        except Exception as e:
            logger.error(f"渲染模板 {template_name} 失败: {e}")
            raise

    def get_system_role(self, template_name: str) -> str:
        cfg = getattr(self.prompts_cfg, template_name, None)
        if cfg is None:
            return ""
        return cfg.get("system_role", "")

    def _get_template(self, template_file: str) -> Template:
        if template_file not in self._template_cache:
            self._template_cache[template_file] = self.env.get_template(template_file)
        return self._template_cache[template_file]

    def reload(self) -> None:
        """改完 j2 文件后调用，清缓存。"""
        self._template_cache.clear()
```

**【验证】**

```bat
python -c "from src.utils import ConfigLoader; from src.llm.prompts import PromptManager; pm=PromptManager(ConfigLoader('./configs').get('llm')); print(pm.render('risk_report', features={'贷款金额':50000}, probability=0.73, risk_level='高风险', shap_top_features=[{'feature':'dti','value':45.3,'shap_value':0.14}], top_n=5)[:120])"
```

期望：打印出渲染后的前 120 个字符，里面能看到"73.00%"和"45.3"。

---

### 步骤 8.3　写 `src/llm/agents/report_agent.py`

**【目标】** 预测 + SHAP → 中文风险报告。

**【文件】** `src/llm/agents/report_agent.py`（新建）

**【完整代码】**

```python
"""
风险报告生成 Agent
==================
ML 算概率，SHAP 说归因，LLM 翻成人话。
"""
from __future__ import annotations

from typing import Any, Dict

from src.llm.client import DeepSeekClient
from src.llm.prompts import PromptManager
from src.utils.logger import get_logger

logger = get_logger("llm.report_agent")


class ReportAgent:
    """风险报告 Agent。"""

    def __init__(self, client: DeepSeekClient, prompt_manager: PromptManager, config):
        # config 传的是 configs.get("llm").agents.report_agent
        self.client = client
        self.prompts = prompt_manager
        self.config = config
        self.top_n = config.top_n_features

    def generate(
        self,
        features: Dict[str, Any],
        probability: float,
        shap_explanation,
        stream: bool = False,
    ):
        """stream=True 返回生成器（打字机），False 返回完整字符串。"""
        context = self._build_context(features, probability, shap_explanation)
        prompt = self.prompts.render("risk_report", **context)
        system_prompt = self.prompts.get_system_role("risk_report")

        logger.info(f"生成风险报告 - 违约概率={probability:.4f}")

        if stream:
            return self.client.stream_chat(prompt, system_prompt=system_prompt)
        return self.client.chat(prompt, system_prompt=system_prompt)

    def _build_context(self, features, probability, shap_explanation) -> Dict[str, Any]:
        if probability < 0.3:
            risk_level = "低风险"
        elif probability < 0.7:
            risk_level = "中风险"
        else:
            risk_level = "高风险"

        return {
            "features": self._select_display_features(features),
            "probability": probability,
            "risk_level": risk_level,
            "shap_top_features": shap_explanation.get_top_features(self.top_n),
            "top_n": self.top_n,
        }

    @staticmethod
    def _select_display_features(features: Dict) -> Dict:
        """挑业务字段 + 英文字段名翻成中文。"""
        priority = [
            "loanAmnt", "term", "interestRate", "installment",
            "annualIncome", "dti", "employmentLength", "homeOwnership",
            "purpose", "grade", "ficoRangeLow", "ficoRangeHigh",
            "delinquency_2years", "openAcc", "totalAcc", "revolUtil",
        ]
        labels = {
            "loanAmnt": "贷款金额", "term": "贷款期限(年)", "interestRate": "利率(%)",
            "installment": "月还款额", "annualIncome": "年收入", "dti": "负债收入比",
            "employmentLength": "就业年限", "homeOwnership": "房屋所有权",
            "purpose": "贷款用途", "grade": "贷款等级",
            "ficoRangeLow": "FICO 评分下限", "ficoRangeHigh": "FICO 评分上限",
            "delinquency_2years": "近2年逾期次数", "openAcc": "未结信用额度数",
            "totalAcc": "总信用额度数", "revolUtil": "循环额度利用率",
        }

        result = {}
        for field in priority:
            if field in features:
                value = features[field]
                if isinstance(value, float):
                    value = round(value, 2)
                result[labels.get(field, field)] = value
        return result
```

**【逐块讲解】**

| 代码段 | 要点 |
|---|---|
| `get_top_features(self.top_n)` | 直接从 SHAP 对象取，**不手写数字** |
| 0.3 / 0.7 分档 | 业务定的阈值，可改配置 |
| `labels` 字典 | 中文标签让 LLM 输出更自然，也更准确 |
| `stream` 分支 | 同一个方法两种返回类型，调用方要注意 |

---

### 步骤 8.4　写 `src/llm/agents/feature_agent.py`

**【目标】** 给它字段和数据统计，让它提特征建议并输出 JSON。

**【文件】** `src/llm/agents/feature_agent.py`（新建）

**【完整代码】**

```python
"""
特征建议 Agent
==============
LLM 当数据科学助手：看数据 -> 提特征建议 -> 返回可执行代码。
"""
from __future__ import annotations

import json
import re
from typing import Any, Dict, List

import pandas as pd

from src.llm.client import DeepSeekClient
from src.llm.prompts import PromptManager
from src.utils.logger import get_logger

logger = get_logger("llm.feature_agent")


class FeatureAgent:
    """特征建议 Agent。"""

    FIELD_DESC = {
        "loanAmnt": "贷款金额",
        "interestRate": "贷款利率",
        "annualIncome": "年收入",
        "dti": "债务收入比",
        "revolUtil": "循环额度使用率",
        "delinquency_2years": "近2年逾期次数",
        "grade": "贷款等级",
    }

    def __init__(self, client: DeepSeekClient, prompt_manager: PromptManager, config):
        self.client = client
        self.prompts = prompt_manager
        self.config = config

    def suggest(
        self,
        df: pd.DataFrame,
        current_model: str = "xgboost",
        current_auc: float = 0.0,
        bottleneck: str = "",
    ) -> Dict[str, Any]:
        """返回 {"suggestions": [...], "general_advice": "..."}"""
        fields = [
            {
                "name": col,
                "dtype": str(df[col].dtype),
                "description": self.FIELD_DESC.get(col, "业务含义未公开"),
                "stats": f"均值={df[col].mean():.2f}" if pd.api.types.is_numeric_dtype(df[col]) else "",
            }
            for col in df.columns
        ]

        prompt = self.prompts.render(
            "feature_suggestion",
            sample_size=len(df),
            feature_count=df.shape[1],
            fields=fields,
            data_sample=df.head(5).to_string(),
            current_model=current_model,
            current_auc=current_auc,
            bottleneck=bottleneck,
            max_suggestions=self.config.max_suggestions,
        )
        system_prompt = self.prompts.get_system_role("feature_suggestion")

        raw = self.client.chat(
            prompt,
            system_prompt=system_prompt,
            temperature=0.5,        # 想让它有创意一点
        )
        return self._parse_json(raw)

    @staticmethod
    def _parse_json(text: str) -> Dict[str, Any]:
        """多层容错：直接解析 -> 剥 ```json 包裹 -> 截取第一个 {...}"""
        try:
            return json.loads(text)
        except json.JSONDecodeError:
            pass

        match = re.search(r"```(?:json)?\s*(\{.*?\})\s*```", text, re.DOTALL)
        if match:
            try:
                return json.loads(match.group(1))
            except json.JSONDecodeError:
                pass

        start, end = text.find("{"), text.rfind("}")
        if start != -1 and end > start:
            try:
                return json.loads(text[start:end + 1])
            except json.JSONDecodeError:
                pass

        logger.warning("无法解析 JSON，返回原文")
        return {"raw": text, "suggestions": [], "general_advice": ""}
```

**【逐块讲解】**

| 代码段 | 要点 |
|---|---|
| `FIELD_DESC` | 给 LLM 业务含义，建议才靠谱 |
| `temperature=0.5` | 要创意，比报告用的 0.3 高 |
| `_parse_json` 三层容错 | LLM 经常给 ```json 包裹或前后带解释文字 |
| 兜底 `{"raw": text}` | 解析失败也要返回结构，不能让页面崩 |

---

### 步骤 8.5　写 `src/llm/agents/case_agent.py`

**【目标】** 分析判错的样本，给出改进方向。

**【文件】** `src/llm/agents/case_agent.py`（新建）

**【完整代码】**

```python
"""
异常案例诊断 Agent
==================
给 LLM 一个判错的样本，让它诊断原因。
"""
from __future__ import annotations

from typing import Any, Dict

from src.llm.client import DeepSeekClient
from src.llm.prompts import PromptManager
from src.utils.logger import get_logger

logger = get_logger("llm.case_agent")


class CaseAgent:
    """案例诊断 Agent。"""

    def __init__(self, client: DeepSeekClient, prompt_manager: PromptManager, config):
        self.client = client
        self.prompts = prompt_manager
        self.config = config

    def analyze(
        self,
        features: Dict[str, Any],
        true_label: int,
        probability: float,
        shap_explanation,
    ) -> str:
        """返回 markdown 格式的诊断报告。"""
        predicted_label = int(probability >= 0.5)

        # 置信度高的错判才值得看
        confidence = max(probability, 1 - probability)
        if confidence < self.config.confidence_threshold:
            logger.info(f"置信度 {confidence:.2f} 低于阈值，跳过分析")
            return ""

        prompt = self.prompts.render(
            "case_analysis",
            true_label=true_label,
            true_label_text="违约" if true_label == 1 else "正常",
            predicted_label=predicted_label,
            predicted_label_text="违约" if predicted_label == 1 else "正常",
            probability=probability,
            confidence_level=f"{confidence:.2%}",
            features=features,
            shap_features=shap_explanation.get_top_features(8),
        )
        system_prompt = self.prompts.get_system_role("case_analysis")

        return self.client.chat(prompt, system_prompt=system_prompt, temperature=0.2)
```

---

### 步骤 8.6　回填仪式：两个 `__init__.py`

`src/llm/__init__.py`：

```python
"""LLM 应用模块"""
from src.llm.client import DeepSeekClient
from src.llm.prompts import PromptManager

__all__ = ["DeepSeekClient", "PromptManager"]
```

`src/llm/agents/__init__.py`：

```python
"""LLM Agents"""
from src.llm.agents.report_agent import ReportAgent
from src.llm.agents.feature_agent import FeatureAgent
from src.llm.agents.case_agent import CaseAgent

__all__ = ["ReportAgent", "FeatureAgent", "CaseAgent"]
```

**【验证】**

```bat
python -c "from src.llm import DeepSeekClient, PromptManager; from src.llm.agents import ReportAgent, FeatureAgent, CaseAgent; print('llm OK')"
```

期望输出：`llm OK`（无 Key 也应通过，这是本章验收红线）。

---

## 6. 本章错误对照

### 错误 E-14　`PromptManager("路径")`

**出现在**：原文档指导书的调用示例

**❌ 原文写法**

```python
pm = PromptManager("./configs/prompts")
```

**✅ 修正写法**

```python
pm = PromptManager(configs.get("llm"))
```

**💥 不修的后果**

```
AttributeError: 'str' object has no attribute 'prompts'
```

**🧠 为什么**　构造函数要读 `config.prompts.templates_dir` 和各模板的 `template_file`/`system_role`，这些都在 llm 配置里。它需要的是**配置对象**，不是路径字符串。

---

### 错误 E-15　脱敏在拼消息之后做，等于白做

**出现在**：原文档 `chat()` 的实现顺序

**❌ 原文写法**

```python
messages = self._build_messages(prompt, system_prompt, history)
params = {**self.default_params, **kwargs}
prompt = self._sanitize_input(prompt)      # 太晚了
```

**✅ 修正写法**

```python
safe_prompt = self._sanitize_input(prompt)         # 先脱敏
messages = self._build_messages(safe_prompt, system_prompt, history)
```

**💥 不修的后果**　不报错，但**敏感信息照样发出去了**。`messages` 里存的是脱敏前的原始 `prompt`，后面改 `prompt` 变量不会影响已构建的列表。

这是最危险的一类 bug：**安全机制看起来在，实际没生效**。金融场景下属于合规事故。

**🧠 为什么**　Python 里字符串是不可变对象。`self._sanitize_input(prompt)` 返回一个新字符串，只是重新绑定了 `prompt` 这个名字，`messages` 里那份旧的字符串对象不受影响。

**✅ 自查**　凡是"先构造、后加工"的代码，都要检查加工结果有没有真正被用上。

---

### 错误 E-16　`chat(system=...)` 参数名错

**❌ 原文写法**

```python
client.chat(prompt, system="你是风控专家")
```

**✅ 修正写法**

```python
client.chat(prompt, system_prompt="你是风控专家")
```

**💥 不修的后果**　`system` 会被 `**kwargs` 接住，然后传给 OpenAI SDK，报：

```
TypeError: Completions.create() got an unexpected keyword argument 'system'
```

**🧠 为什么**　`chat` 的签名是 `(prompt, system_prompt, history, **kwargs)`。写 `system=` 不会匹配 `system_prompt`，而是落进 `**kwargs`，最后原样传给底层 SDK。

**✅ 自查**　全项目搜 `system=`，改成 `system_prompt=`。

---

### 错误 E-17　Jinja2 的 `tojson` 不支持 `ensure_ascii`

**出现在**：原文档指导书 7.2 节的自定义 Agent 示例

**❌ 原文写法**

```jinja2
{{ features | tojson(indent=2, ensure_ascii=False) }}
```

**✅ 修正写法**

```jinja2
{% for k, v in features.items() %}
- {{ k }}: {{ v }}
{% endfor %}
```

或者：

```jinja2
{{ features | tojson }}
```

**💥 不修的后果**

```
jinja2.exceptions.TemplateAssertionError: No filter named 'tojson'
```

或（在支持 tojson 的环境里）：

```
TypeError: do_tojson() got an unexpected keyword argument 'ensure_ascii'
```

**🧠 为什么**　`tojson` 是 Flask 提供的过滤器，不是 Jinja2 内置的。纯 Jinja2 环境里根本没有它；即使在 Flask 里，它的参数也不含 `ensure_ascii`（内部固定用 HTML 安全转义）。本项目是纯 Jinja2，所以要改用循环手写。

**✅ 自查**　模板里避免用 `tojson`、`url_for` 这类 Flask 专属过滤器。

---

### 错误 N-03　模型名写 `deepseek-flash`

**❌ 原文写法**　`.env` 里 `DEEPSEEK_MODEL=deepseek-flash`

**✅ 修正写法**

```bash
DEEPSEEK_MODEL=deepseek-chat
```

**💥 不修的后果**

```
openai.NotFoundError: Error code: 404 - model 'deepseek-flash' does not exist
```

**🧠 为什么**　DeepSeek 官方可用的模型名是 `deepseek-chat`（通用对话）和 `deepseek-reasoner`（推理）。`deepseek-flash` 不在其中。

---

## 7. 常见坑

| 现象 | 原因 | 解决 |
|---|---|---|
| 401 Unauthorized | Key 错或没加载 | `python -c "import os;from dotenv import load_dotenv;load_dotenv();print(os.getenv('DEEPSEEK_API_KEY')[:8])"` |
| 404 model not exist | 模型名错 | 改成 `deepseek-chat` |
| 流式输出卡住不动 | 网页端没用 `st.write_stream` | 见第 11 章 |
| 报告里的数字和 SHAP 对不上 | LLM 自己编了 | prompt 加"只能用给定数据"，并降低 temperature |
| 报告太长/太短 | `max_tokens` 不合适 | 调 `default_params.max_tokens` |
| 每次运行结果差异大 | temperature 太高 | 报告类任务设 0.1~0.3 |

---

## 8. 本章自测

**1. 为什么 `DeepSeekClient` 在没 Key 时只警告不抛异常？**

<details>
<summary>点开看答案</summary>

为了让前 7 章的开发、import 测试、CI 流程不被阻断。真正调用时才失败，而且失败信息明确指向"去配 Key"。这是"延迟失败"的设计。

</details>

**2. 三个 Agent 的共同结构是什么？**

<details>
<summary>点开看答案</summary>

`build_context`（把模型输出、SHAP、特征整理成上下文）→ `render`（套 Jinja2 模板）→ `chat`（调 LLM）→ 对需要结构化的再做 `parse`。学会一个就都会了。

</details>

**3. 为什么 LLM 不能直接算违约概率？**

<details>
<summary>点开看答案</summary>

不可复现、不可验证、不可审计。监管问"这个数怎么来的"答不上来。正确分工是 ML 负责算、SHAP 负责拆、LLM 只负责翻译。

</details>

**4. 流式输出的方法为什么用 `yield` 而不是 `return`？**

<details>
<summary>点开看答案</summary>

`yield` 让函数变成生成器，每收到一段就交出去一段，调用方可以边收边显示。`return` 必须等全部生成完才返回，就失去了"打字机"效果。

</details>

---

## 9. 本章产物清单 + 下一步

**产物**

| 路径 | 行数约 |
|---|---|
| `src/llm/client.py` | 185 |
| `src/llm/prompts.py` | 60 |
| `src/llm/agents/report_agent.py` | 90 |
| `src/llm/agents/feature_agent.py` | 95 |
| `src/llm/agents/case_agent.py` | 55 |
| `src/llm/__init__.py` / `agents/__init__.py` | 各 6 |

**一条命令自检**

```bat
python -c "from src.llm.agents import ReportAgent,FeatureAgent,CaseAgent; print('llm OK')"
```

**你现在已经能做到**：把模型结果翻译成自然语言报告，且具备重试、限流、脱敏、审计四道工程保障。

**下一章**：编排层——用一条命令把前面 9 章串起来，从读数据一路跑到产出模型文件。
