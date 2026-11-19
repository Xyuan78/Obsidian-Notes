# 附录 D · Windows 命令速查

> **这份附录解决什么问题**
> 你在网上搜到的教程、GitHub 上的 README、大模型给你的回答，**90% 给的是 Linux/Mac 命令**。
> 你在 Windows 的 cmd 里一敲，报"不是内部或外部命令"。
> 这不是你笨，是命令本身不一样。
>
> 本附录给三栏对照：**Linux 写法 → cmd 写法 → 说明**。
> 手册正文里所有命令，都是 cmd 可用的版本；这份附录负责让你**看得懂别人的教程**。

---

## D.0 先搞清楚：你手上是哪个"命令行"

Windows 上有三种命令行，长得像但语法不同。**本手册统一用 cmd**，因为 conda 在 cmd 里最稳。

| 名称 | 怎么打开 | 提示符长这样 | 本手册用吗 |
|---|---|---|---|
| **cmd**（命令提示符） | `Win + R` → 输入 `cmd` → 回车 | `C:\Users\Lenovo>` | ✅ 全用手册 |
| PowerShell | `Win + X` → `终端` 或 `Windows PowerShell` | `PS C:\Users\Lenovo>` | ❌ 不用（语法不同，容易踩坑） |
| Git Bash | 装了 Git 后右键 → `Git Bash Here` | `Lenovo@PC MINGW64 ~ $` | ⚠️ 可用但不推荐 |
| Anaconda Prompt | 开始菜单搜 `Anaconda Prompt` | `(base) C:\Users\Lenovo>` | ✅ 装完 conda 后用这个 |

> **怎么一眼认出自己在哪个里？**
> 提示符前面有 `PS` → PowerShell。
> 提示符前面有 `(base)` 或 `(credit_risk)` → conda 环境已激活（这是好事）。
> 两者都没有、提示符是 `C:\...>` → 纯净 cmd。

**🧠 知识点 KP-EN-26 · 为什么 conda 要用专用 Prompt**
`conda activate` 本质是执行一个 `.bat` 脚本改环境变量。在纯净 cmd 里第一次可能要先 `conda init cmd.exe`。
用 **Anaconda Prompt** 打开就自带这个初始化，省一步。
如果你在纯净 cmd 里敲 `conda` 报"不是内部或外部命令"，直接改用 Anaconda Prompt 即可。

---

## D.1 目录操作（最高频，先背这 8 个）

| 要做的事 | ❌ Linux / Mac | ✅ Windows cmd | 备注 |
|---|---|---|---|
| 看当前在哪个目录 | `pwd` | `cd`（不带参数）或 `echo %cd%` | cmd 里 `cd` 单独敲就打印当前路径 |
| 列出目录内容 | `ls` / `ls -la` | `dir` | 想看隐藏文件：`dir /a` |
| 进入子目录 | `cd src` | `cd src` | ✅ 一样 |
| 返回上一级 | `cd ..` | `cd ..` | ✅ 一样 |
| 回到用户主目录 | `cd ~` | `cd %USERPROFILE%` | cmd 不认 `~` |
| 跨盘符切换 | `cd /d/work` | `d:` 然后 `cd \work` | ⚠️ 见下方 D.1.1 |
| 创建目录（单层） | `mkdir data` | `mkdir data` | ✅ 一样 |
| **创建多层目录** | `mkdir -p data/raw/csv` | `mkdir data\raw\csv` | ✅ **cmd 的 mkdir 天生就能建多层，不需要 `-p`** |
| 创建空文件 | `touch a.py` | `type nul > a.py` | ⚠️ cmd 没有 `touch` |
| 删除文件 | `rm a.py` | `del a.py` | ⚠️ |
| 删除目录（空） | `rmdir data` | `rmdir data` | ✅ |
| 删除目录（连同内容） | `rm -rf data` | `rmdir /s /q data` | ⚠️ **危险，见 D.7** |
| 复制文件 | `cp a.py b.py` | `copy a.py b.py` | ⚠️ |
| 复制整个目录 | `cp -r src backup` | `xcopy src backup /E /I` | ⚠️ 见下方 D.1.2 |
| 移动 / 重命名 | `mv a.py b.py` | `move a.py b.py` | ⚠️ |
| 查看文件内容 | `cat a.py` | `type a.py` | 长文件别用，用编辑器 |
| 清屏 | `clear` | `cls` | |

### D.1.1 跨盘符：Windows 最容易卡住的一步

Linux 只有一个根 `/`，所有东西都挂在下面。Windows 有 `C:` `D:` `E:` 多个盘，**每个盘是独立的树**。

```cmd
:: ❌ 错的（很多教程会这么教你）
cd /d/work

:: ✅ 对的，两步
d:
cd \work
```

为什么？因为 `cd` 只能在同一棵树里走。要换树，先"站到那棵树的根"上。

**实战：切到你的项目**

```cmd
e:
cd \PythonCode\11.4
dir
```

应该能看到 `开发指导书.md`、`信贷风控_从零手写完整文档.md`、`信贷风控实操手册\` 这几项。

**🧠 小技巧：在当前文件夹直接开 cmd**
在文件资源管理器地址栏里输入 `cmd` 回车，直接在当前目录打开 cmd。
或者按住 `Shift` 右键空白处 → "在此处打开命令窗口"。
**这个技巧能帮你省掉 80% 的 `cd`。**

### D.1.2 复制整个目录

```cmd
:: ❌ 错的
cp -r src backup

:: ✅ 对的
xcopy src backup /E /I
```

参数含义：
- `/E` = 连空子目录一起复制
- `/I` = 如果目标是新目录，就当成目录处理（不加这个会问你"backup 是文件还是目录"）

**更简单的做法**：直接用资源管理器 `Ctrl+C` / `Ctrl+V`。命令不是目的，把事做完才是。

---

## D.2 Python 与 conda

| 要做的事 | 命令 | 说明 |
|---|---|---|
| 看 Python 版本 | `python --version` | 应该显示 3.10.x |
| 看 conda 版本 | `conda --version` | |
| 看所有环境 | `conda env list` | 当前环境前面有 `*` |
| 创建环境 | `conda create -n credit_risk python=3.10 -y` | `-y` = 别问我，直接装 |
| 激活环境 | `conda activate credit_risk` | 成功后提示符前出现 `(credit_risk)` |
| 退出环境 | `conda deactivate` | |
| 删除环境（重建用） | `conda remove -n credit_risk --all -y` | |
| 看当前环境装了哪些包 | `pip list` | |
| 导出现有依赖 | `pip freeze > requirements_lock.txt` | 精确版本快照 |

**⚠️ 每次新开一个 cmd 窗口，都要重新 `conda activate credit_risk`。**
环境不会跨窗口保持。忘了激活 → 跑到 base 环境 → 报"没有 pandas"。

### D.2.1 判断"我到底有没有激活成功"

```cmd
where python
```

- ✅ 激活成功：输出里包含 `anaconda3\envs\credit_risk\python.exe` 或 `miniconda3\envs\...`
- ❌ 没激活：输出里是 `C:\Users\Lenovo\AppData\Local\Programs\Python\Python312\python.exe` 或 `anaconda3\python.exe`（这是 base）

**这一条命令比 `conda env list` 更靠谱**，因为它告诉你"真正被调用的那个 python 在哪"。

---

## D.3 pip 安装

```cmd
:: 基础安装
pip install -r requirements.txt

:: 国内加速（推荐，否则 torch 要下半天）
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

:: 只装一个包
pip install pandas

:: 指定版本
pip install "numpy>=1.26,<3"

:: 装本地 whl 文件
pip install torch-2.6.0+cu124-cp310-cp310-win_amd64.whl

:: 卸载
pip uninstall xgboost
```

**⚠️ 引号必须加**：`pip install "numpy>=1.26,<3"`
在 cmd 里 `>` 和 `<` 是**重定向符号**（把输出写到文件 / 从文件读输入）。
不加引号，`pip install numpy>=1.26` 会被 cmd 理解成"把 numpy 装好后，把输出写进 1.26 这个文件"，然后创建一个叫 `=1.26` 的怪文件。

> **真实症状**：命令没报错，但目录里多了个叫 `=1.26` 的文件，numpy 版本也没变。
> 遇到这个现象，先检查是不是引号忘了。

### D.3.1 装 GPU 版 PyTorch（单独一步，不要写进 requirements.txt）

```cmd
:: CUDA 12.4 版（对应大多数 30/40 系显卡）
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu124 -i https://pypi.tuna.tsinghua.edu.cn/simple

:: 验证
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"
```

期望输出类似 `2.6.0+cu124 True`。
如果输出 `2.6.0+cpu False` → 装成 CPU 版了，重新装。

**为什么不能写进 requirements.txt？**
`--index-url` 会**替换**默认源，导致 pandas / numpy 这些包在 PyTorch 源里找不到，整体装失败。
详见第 00 章 0.4 节的完整解释。

---

## D.4 运行与调试

| 要做的事 | 命令 | 说明 |
|---|---|---|
| 运行脚本 | `python scripts\train_pipeline.py --use-sample` | 路径用 `\`（cmd）或 `/`（都行） |
| 以模块方式运行 | `python -m src.data.loader` | 能正确解析相对导入 |
| 启动 Streamlit | `streamlit run app\streamlit_app.py` | 会自动开浏览器 |
| 指定端口 | `streamlit run app\streamlit_app.py --server.port 8502` | 8501 被占用时用 |
| 跑测试 | `pytest tests\ -v` | `-v` 显示每个用例名 |
| 只跑一个测试 | `pytest tests\test_basic.py::test_config_loader -v` | |
| 交互式 Python | `python` | 退出用 `exit()` 或 `Ctrl+Z` 回车 |
| 退出卡住的程序 | `Ctrl + C` | 一次不行按两次 |

### D.4.1 `python` 和 `python -m` 的区别（很重要）

```cmd
:: 直接跑
python scripts\train_pipeline.py
```

此时 Python 把 `scripts\` 当作"入口目录"，`sys.path[0]` 是 `scripts\`。
脚本里写 `from src.data import DataLoader` 会**找不到 src**——因为 `src` 在上一级。

这就是为什么手册里三个脚本开头都有这一段：

```python
import sys
from pathlib import Path
sys.path.insert(0, str(Path(__file__).resolve().parents[1]))
```

**🧠 知识点 KP-PY-07 · 为什么要手动插 `sys.path`**
Python 导入模块时，会去 `sys.path` 这个列表里的每个目录下找。
用 `python scripts\x.py` 启动时，列表第一项是 `scripts\`，没有项目根目录，所以 `src` 找不到。
手动 `insert(0, 项目根)` 就是补上这一项。
`Path(__file__).resolve().parents[1]` = 当前文件 → 上一级目录 = 项目根。

---

## D.5 进程与端口

| 要做的事 | 命令 |
|---|---|
| 看谁占了 8501 端口 | `netstat -ano \| findstr :8501` |
| 根据 PID 杀进程 | `taskkill /PID 12345 /F` |
| 看所有 Python 进程 | `tasklist \| findstr python` |
| 杀掉所有卡死的 Python | `taskkill /IM python.exe /F` |

**典型场景：Streamlit 关了浏览器但服务还在跑，重新启动报"端口已被占用"。**

```cmd
:: 第 1 步：找到 PID（最后一列的数字）
netstat -ano | findstr :8501

:: 输出示例：
::   TCP    0.0.0.0:8501    0.0.0.0:0    LISTENING    18432
::                                                       ^^^^^ 这个

:: 第 2 步：杀掉
taskkill /PID 18432 /F
```

**⚠️ `taskkill /IM python.exe /F` 会杀掉所有 Python 进程**，包括你另一个窗口正在跑的训练。用之前先确认。

---

## D.6 环境变量

```cmd
:: 查看
echo %DEEPSEEK_API_KEY%

:: 临时设置（只对当前窗口有效，关掉就没了）
set DEEPSEEK_API_KEY=sk-xxxxxxxx

:: 永久设置（写进用户环境变量，要重开窗口生效）
setx DEEPSEEK_API_KEY "sk-xxxxxxxx"

:: 删除
set DEEPSEEK_API_KEY=
```

**⚠️ `set` 的坑：等号两边不能有空格**

```cmd
:: ❌ 错：变量名变成 "DEEPSEEK_API_KEY "（末尾带空格）
set DEEPSEEK_API_KEY = sk-xxx

:: ✅ 对
set DEEPSEEK_API_KEY=sk-xxx
```

这个空格会导致 `os.getenv("DEEPSEEK_API_KEY")` 返回 `None`，然后你排查半小时。

> **本项目其实不需要 `set`。**
> 因为 `DeepSeekClient` 用的是 `python-dotenv` 读 `.env` 文件。
> 你只要在项目根目录建一个 `.env`，写 `DEEPSEEK_API_KEY=sk-xxx` 就行。
> 用 `.env` 比 `setx` 好，因为：换电脑好带、不会污染系统、能进 `.gitignore` 保护。

---

## D.7 ⚠️ 危险命令清单

**这些命令会永久删数据，没有回收站。**

| 命令 | 后果 | 我的建议 |
|---|---|---|
| `rmdir /s /q 目录` | 连同所有内容静默删除 | 删之前先 `dir 目录` 确认 |
| `del /f /s /q *.*` | 当前目录及子目录所有文件全删 | **永远不要敲** |
| `format` | 格式化整个盘 | **永远不要敲** |
| `diskpart` | 分区操作 | **永远不要敲** |
| `rd /s /q C:\` | 试图删掉 C 盘 | 系统会拒绝，但别试 |

**三条替代做法（按优先级）：**

1. **删之前先改名**：`move 数据目录 数据目录_待删`，跑一天没问题再真删。
2. **用回收站**：资源管理器里 `Delete` 键 = 进回收站，可以还原。`Shift+Delete` = 永久删，别按。
3. **重要数据先备份**：`xcopy data data_backup /E /I`

**本手册里你唯一会用到的删除操作**是删 `logs\`、`models\`、`outputs\` 里的实验产物（重跑用）。
这些都能重新生成，删了不影响。

---

## D.8 编码问题：中文乱码

**症状**：`print("中文")` 输出一堆 `锟斤拷` 或者 `������`。

**原因**：cmd 默认代码页是 936（GBK），Python 输出 UTF-8，对不上。

**解决办法（三选一）**：

```cmd
:: 方法 1：当前窗口临时改成 UTF-8（推荐）
chcp 65001

:: 方法 2：设置 Python 环境变量
set PYTHONIOENCODING=utf-8

:: 方法 3：在 Python 文件第一行加编码声明（治标）
# -*- coding: utf-8 -*-
```

**推荐方法 1 + 2 组合**：开新窗口时先敲 `chcp 65001`。

**更彻底的做法**：在项目根目录建 `.env`，加一行

```
PYTHONIOENCODING=utf-8
```

然后在代码里用 `load_dotenv()` 加载。不过这对 `print` 到 cmd 无效，`chcp` 才是根上的解法。

> **不影响功能的情况**：如果只是 loguru 日志里中文乱码，但程序跑得对，可以先不管。
> 写文件时明确指定 `encoding="utf-8"` 更关键：
> ```python
> df.to_csv(path, index=False, encoding="utf-8-sig")   # Excel 打开不乱码
> ```
> 注意是 `utf-8-sig`（带 BOM），不是 `utf-8`。`utf-8` 存的中文 CSV 用 Excel 直接打开会乱码。

---

## D.9 本项目完整命令序列（照抄即可）

假设项目根目录是 `E:\PythonCode\11.4\credit_risk_system`。

### 第 1 次：从零到能跑

```cmd
:: === 1. 建环境 ===
conda create -n credit_risk python=3.10 -y
conda activate credit_risk

:: === 2. 进项目 ===
e:
cd \PythonCode\11.4\credit_risk_system

:: === 3. 装依赖（先普通版）===
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

:: === 4. 单独装 GPU 版 torch（有 N 卡才做；没 N 卡跳过）===
pip install torch --index-url https://download.pytorch.org/whl/cu124 -i https://pypi.tuna.tsinghua.edu.cn/simple
python -c "import torch; print(torch.cuda.is_available())"

:: === 5. 验证环境 ===
python -c "import pandas, numpy, sklearn, xgboost, shap, optuna, streamlit; print('OK')"

:: === 6. 造数据 ===
python scripts\generate_sample_data.py

:: === 7. 训练 ===
python scripts\train_pipeline.py --use-sample --skip-mlp

:: === 8. 跑测试 ===
pytest tests\ -v

:: === 9. 开网页 ===
streamlit run app\streamlit_app.py
```

### 之后每次重开窗口

```cmd
conda activate credit_risk
e:
cd \PythonCode\11.4\credit_risk_system
streamlit run app\streamlit_app.py
```

**就这三行。** 前 7 步只在第一次做。

---

## D.10 常见报错 → 对应命令

| 报错原文 | 真实原因 | 该敲什么 |
|---|---|---|
| `'xxx' 不是内部或外部命令` | 命令名拼错 / Linux 命令 / 没装 | 查本附录 D.1 对照表 |
| `'python' 不是内部或外部命令` | 没激活环境 / 没装 Python | `conda activate credit_risk` |
| `ModuleNotFoundError: No module named 'src'` | 没插 `sys.path` | 见 D.4.1 |
| `ModuleNotFoundError: No module named 'pandas'` | 环境没激活 / 没装依赖 | `where python` 再 `pip install -r requirements.txt` |
| `pip 不是内部或外部命令` | 同上 | 同上 |
| `Access is denied` / `拒绝访问` | 文件被别的程序占用 | 关掉编辑器 / 关掉 Streamlit 再试 |
| `端口已被占用` / `Address already in use` | 上次没关干净 | 见 D.5 |
| 中文乱码 | 代码页不对 | `chcp 65001` |
| `SyntaxError: unexpected character after line continuation character` | 路径里的 `\` 被当成转义 | 用原始字符串 `r"E:\data\x.csv"` 或双反斜杠 |
| `UnicodeDecodeError: 'gbk' codec can't decode` | 读文件没指定编码 | `pd.read_csv(path, encoding="utf-8")` |

---

## D.11 一张纸备忘（建议打印/截图）

```
┌─────────────────────────────────────────────┐
│  开新窗口第一件事                             │
│    conda activate credit_risk               │
│    where python   ← 确认路径对               │
├─────────────────────────────────────────────┤
│  Linux  →  Windows                          │
│    ls     →  dir          cp -r  → xcopy /E /I │
│    mkdir -p → mkdir       rm -rf → rmdir /s /q │
│    touch  →  type nul >   cat    → type      │
│    pwd    →  cd           clear  → cls       │
├─────────────────────────────────────────────┤
│  跨盘符：d: 然后 cd \path   （不是 cd /d/path）│
│  装包带引号："numpy>=1.26,<3"                 │
│  端口占用：netstat -ano | findstr :8501      │
│            taskkill /PID xxx /F             │
│  中文乱码：chcp 65001                        │
└─────────────────────────────────────────────┘
```

---

## 附录 D 自测

不看上面，回答：

1. Linux 的 `mkdir -p data/raw/csv` 在 cmd 里怎么写？
2. 你在 `C:\` 盘，想切到 `E:\PythonCode`，要敲几条命令？
3. `pip install numpy>=1.26` 会发生什么？
4. `where python` 输出 `anaconda3\python.exe`，说明什么？
5. 中文乱码，敲什么？
6. `python scripts\x.py` 里 `from src.data import DataLoader` 报 `ModuleNotFoundError`，原因是什么？

<details>
<summary>答案</summary>

1. `mkdir data\raw\csv` —— cmd 的 mkdir 天生能建多层，不需要 `-p`。
2. 两条：`e:` 然后 `cd \PythonCode`。
3. cmd 会把 `>` 当成重定向，创建一个叫 `=1.26` 的文件，numpy 版本不变。要加引号：`pip install "numpy>=1.26"`。
4. 你在 **base 环境**，没激活 `credit_risk`。敲 `conda activate credit_risk`。
5. `chcp 65001`。
6. `sys.path` 里只有 `scripts\`，没有项目根目录，所以找不到 `src` 包。脚本开头要 `sys.path.insert(0, str(Path(__file__).resolve().parents[1]))`。

</details>

---

← 返回 [`00_总览与学习路线.md`](00_总览与学习路线.md) ｜ 下一篇 → [`附录C_术语表.md`](附录C_术语表.md)
