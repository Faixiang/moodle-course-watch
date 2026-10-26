🏫 Moodle Course Watch
---

[English](README.md) · **简体中文** · [繁體中文](README.zh-TW.md)

![Moodle](https://img.shields.io/badge/Moodle-4.x-e6602c)
![Python](https://img.shields.io/badge/Python-3.9%2B-3776ab)
![License](https://img.shields.io/badge/License-MIT-blue)
![Mode](https://img.shields.io/badge/Read--only-yes-brightgreen)

一个可移植、只读的 Moodle 课程巡检工具，适用于**任意** Moodle 站点：检测新发布的课业、把缺失的周文件夹镜像到本地、报告尚未提交的活动，并对已下载的材料做专业讲述、自动划分重点等级——另附可选的复习笔记交付物。

永远不会向 Moodle 提交、编辑或上传任何内容。每次调用都是对标准 Web Service 的 `GET` 请求。

---

## 它能做什么

| 事项 | 输出 |
|---|---|
| **1. 新活动检测** | 将 `cmid` 与本地基线比对，报告任何新活动的名称与所属章节 |
| **2. 缺失周次下载** | 建立周文件夹，以带 token 的直链下载全部文件，并校验字节数 |
| **3. 提交状态** | 每个活动的 `submitted` / `draft` / `new`，附评分状态与最后修改时间 |
| **4. 文件内容讲述** | 逐份专业讲述：文件定位、内容结构、**重点分级（必记 / 理解 / 了解）**、与前后周的衔接、以及无法读取之处——在对话中给出，不产出报告文件 |
| **5. 重点笔记** | 可选的 Markdown 交付物：定义表、含成立条件的公式表、例题步骤、常见错误、五分钟回忆清单 |

## 触发词

用哪种语言想就用哪种语言说。同一行指向同一件事。

**English**

| Job | Say |
|---|---|
| New activities | *anything new in this course this week?* · *any new coursework posted?* |
| Missing-week download | *download Week 6* · *grab the files for Week 7* |
| Submission status | *what haven't I submitted yet?* · *which assignments are still open?* |
| File walkthrough | *what does this handout cover?* · *walk me through these slides* |
| Exam notes | *make me revision notes* · *build a key-points sheet* |

**简体中文**

| 事项 | 说法 |
|---|---|
| 新活动 | 看下这门课这周有没有新东西 · 有没有新发布的作业 |
| 按周下载 | 补下载 Week 6 · 把第 7 周的文件下下来 |
| 提交状态 | 我哪些作业没交 · 还有哪些作业没交 |
| 内容讲述 | 这份讲义讲了什么 · 这几份课件讲了什么 |
| 重点笔记 | 帮我做重点笔记 · 整理复习重点 |

**繁體中文**

| 事項 | 說法 |
|---|---|
| 新活動 | 看下這門課這週有沒有新東西 · 有沒有新發布的作業 |
| 按週下載 | 補下載 Week 6 · 把第 7 週的檔案下載下來 |
| 提交狀態 | 我哪些作業沒交 · 還有哪些作業沒交 |
| 內容講述 | 這份講義講了什麼 · 這幾份課件講了什麼 |
| 重點筆記 | 幫我做重點筆記 · 整理複習重點 |

## 为什么它不是开箱即用

Moodle 部署在几乎所有要紧的地方都不一样：主机名、token 政策、模块命名、章节命名与限流策略。一个*靠猜*站点形状的 skill 会安静地报「本周没有新东西」——整整一个学期，而且看起来像成功，这是最坏的失败方式。

所以在新站点上第一次运行时，它会停下来先问。答案写进 `site-profile.md`，**没有 profile 就不运行**。

| 问题 | 为什么重要 |
|---|---|
| 站点根 URL | 一切都挂在 `https://<host>` 上，不能从课程名去猜 |
| token 怎么发放 | `/login/token.php` + `moodle_mobile_app`、手工建 key，或纯 OIDC 根本无法签发普通 token |
| Web Service 是否开启 | 纯 SSO 账号经常连 token 都申请不到 |
| 课程 course id | 只能从 `core_enrol_get_users_courses` 取，不要从 URL 猜 |
| 本地目录与周命名 | `Week 6`？`W6`？`Topic 6`？镜像用户既有的命名 |
| 哪些活动算课业 | 全是 `assign`，还是另有 `turnitintooltwo` / `quiz` / `workshop` / 自定义插件 |
| 是否限流 | 激进的 WAF 在突发 POST 下返回 502——调用要串行，间隔约 0.15 秒 |

## 安装

```bash
git clone https://github.com/Faixiang/moodle-course-watch.git
```

把整个文件夹放进你 agent 的 skills 目录，例如：

```
<agent>/skills/custom/moodle-course-watch/
  SKILL.md
  site-profile.template.md   →  复制为 site-profile.md 并逐项填写
  scripts/check_site.py
  scripts/submission_status.py
```

`site-profile.md` 保存你的主机、course id 与 token——它已被 **git 忽略**，绝不能提交。

## 依赖

- Python 3.9+，仅用标准库（无需 `pip install`）。
- 一个暴露 Moodle Web Service 工具的本地 MCP server。参考实现：
  [moodle-mcp](https://github.com/Faixiang/moodle-mcp)——7 个工具，含 `moodle_course_files` 与
  `moodle_download_week`。用 `MOODLE_MCP_DIR` 指向它。

## 用法

```bash
# 1. 先探明站点，再谈信任
python scripts/check_site.py --mcp-dir <dir>
#    → 站点名、Moodle 版本、函数数量、模块类型普查、章节名、duedate 覆盖率

# 2. 跨课程的提交状态
python scripts/submission_status.py --courses 28048,28196 --mcp-dir <dir>
python scripts/submission_status.py --courses 28048 --pattern "assignment|tutorial" --json
```

之后直接用英文、简体或繁体对 agent 说话即可——每件事的说法见上文「触发词」。

## 下载直链——唯一值得单独点名的坑

```
✅ https://<host>/webservice/pluginfile.php/<ctx>/<component>/<file>?forcedownload=1&token=<token>
❌ https://<host>/webservice/pluginfile.php/...            → 返回 JSON，不是文件
❌ https://<host>/mod/resource/view.php?id=<id>            → 需要浏览器会话
```

不带 token 时，该端点会返回一份错误文档，而其 HTTP 状态码仍可能是 `200`。务必校验 `%PDF` 头与字节数，绝不能只看状态码。

## 盲区——说清楚，绝不掩盖

- **Turnitin 与 quiz 的提交状态**不在标准 Web Service 的暴露范围内。活动可见，状态不可见。本 skill 会标为*无法核验*，而不是擅自写成已交／未交。
- **`duedate` 为空**在从不设截止日的站点上很常见。会报为「未设置」，绝不编造。
- **自定义插件**未必出现在 `mod_assign_get_assignments` 下——承诺覆盖范围前先跑一次模块普查。
- **数学 PDF 的公式文本层乱码**时逐页视觉读取，无法解析的公式列入「待确认」，不凭记忆重建。

## 范围与边界

- 设计上只读。不提交、不评分、不向 Moodle 回写任何内容。
- 个人学术工具，与任何院校无关。站点的可接受使用政策由你自行判断。

## 许可

MIT——见 [LICENSE](LICENSE)。
