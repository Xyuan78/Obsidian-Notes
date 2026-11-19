# Obsidian Notes

个人笔记库，用 [Obsidian](https://obsidian.md) 编写和维护。内容以编程为主：语言基础、前端、Java 后端、人工智能课程笔记，另有科研与项目文档。共 200+ 篇 Markdown。

## 目录结构

```
.
├── 编程/                  # 语言与技术栈知识库
│   ├── 人工智能/          # 机器学习课程笔记（30 篇，含索引）
│   ├── 前端/              # HTML / CSS / JavaScript / Vue
│   ├── Java/              # Java 基础到并发
│   ├── Python/            # Python 语法、数据分析、Web
│   ├── SpringMVC/         # MVC、SSM 整合、SSM 实战
│   └── MyBatis/           # 映射、动态 SQL、关联查询
├── 实训笔记/              # 实训课程按天整理的笔记与项目记录
├── 人工智能/              # 信贷风控项目文档与实操手册
├── 科研/                  # 论文阅读记录、调优实验
├── 数据库/                # Obsidian Bases 视图
└── 图片/                  # 笔记引用的截图与配图
```

## 入口索引

| 分区 | 入口 | 内容 |
|---|---|---|
| 人工智能课程 | [00-人工智能学习索引](编程/人工智能/00-人工智能学习索引.md) | Python 工具 → 机器学习 → XGBoost → 深度学习 → SHAP → Optuna → 项目实战 |
| 前端 | [00-前端学习入口](编程/前端/00-前端学习入口.md) | HTML、CSS、JavaScript、Vue 四条线 |
| Python | [Python学习路线](编程/Python/Python学习路线.md) | 语法基础到数据分析与 Web 开发 |
| 实训 | [00-实训笔记索引](实训笔记/00-实训笔记索引.md) | Java / Spring 系列、Vue、项目复盘 |
| 信贷风控 | [实操手册总览](人工智能/project/信贷风控实操手册/00_总览与学习路线.md) | 从配置层到 Web 层的完整工程化流程 |

## 阅读方式

用 Obsidian 打开本文件夹作为 vault，即可获得 wikilink 跳转、双链面板与图谱。直接用 GitHub 浏览也能正常阅读，只是 `[[双链]]`、Mermaid 图和 LaTeX 公式不会渲染。

笔记里用到的 Obsidian 特性：

- 内部链接 `[[笔记名]]`，跨目录按文件名解析；
- callout（`> [!warning]`、`> [!tip]`）标注易错点与经验值；
- 公式用 LaTeX（`$...$` / `$$...$$`），流程图用 Mermaid；
- 表格用于多维度对比。

## 笔记约定

- frontmatter 只保留 `title` 与 `tags`；
- 每篇末尾有「相关笔记」和「来源」，来源标注课程日期与原始文件名，方便回溯；
- 课程笔记以成稿文档为准，语音转写只用于补充老师强调点和课堂代码；涉及的推导和课堂数字都复算过，与源材料不一致处会在正文注明。

## 更新

笔记库本身是一个 git 仓库，`main` 分支即当前内容：

```bash
git add -A
git commit -m "说明这次改了什么"
git push origin main
```
