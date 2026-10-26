---
AIGC:
    Label: "1"
    ContentProducer: 001191440300708461136T1XGW3
    ProduceID: c6eb62ba6ec5a28db2139adf23a2fe4b_a208be9cd16811f196c5525400248c00
    ReservedCode1: fPnik/37NDShDnoT06k7x0nH+xPr+nydZpoUhV7RgSeHcUoX3d1EovQjN+Egx+ZAiaB8AV2sYfTvI8QhaNaxVi4FBZo1IHMVCO0A/RzaBP/KLoqsPU6KlcIysoSD+32fyzEV725rxdEAjHtL6lj99iHEOwgedr5/UkeoiI17n4Kcdt6ueITZF4dJ/nQ=
    ContentPropagator: 001191440300708461136T1XGW3
    PropagateID: c6eb62ba6ec5a28db2139adf23a2fe4b_a208be9cd16811f196c5525400248c00
    ReservedCode2: fPnik/37NDShDnoT06k7x0nH+xPr+nydZpoUhV7RgSeHcUoX3d1EovQjN+Egx+ZAiaB8AV2sYfTvI8QhaNaxVi4FBZo1IHMVCO0A/RzaBP/KLoqsPU6KlcIysoSD+32fyzEV725rxdEAjHtL6lj99iHEOwgedr5/UkeoiI17n4Kcdt6ueITZF4dJ/nQ=
---

🏫 Moodle Course Watch
---

**English** · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md)

![Moodle](https://img.shields.io/badge/Moodle-4.x-e6602c)
![Python](https://img.shields.io/badge/Python-3.9%2B-3776ab)
![License](https://img.shields.io/badge/License-MIT-blue)
![Mode](https://img.shields.io/badge/Read--only-yes-brightgreen)

A portable, read-only course watch for **any** Moodle site: it detects newly published coursework, mirrors missing
week folders to disk, reports what is still unsubmitted, and walks you through the downloaded material with graded
key points — plus an optional revision-notes deliverable.

Nothing is ever submitted, edited or uploaded to Moodle. Every call is a `GET` against the standard Web Service.

---

## What it does

| Job | Output |
|---|---|
| **1. New activities** | diffs `cmid` against a local baseline; reports name + section of anything new |
| **2. Missing-week download** | creates the week folder, downloads every file with a tokenised link, verifies byte size |
| **3. Submission status** | `submitted` / `draft` / `new` per activity, with graded state and last-modified time |
| **4. File walkthrough** | professional read of each file: positioning, content, **graded key points (must-know / understand / awareness)**, links to neighbouring weeks, and anything unreadable — delivered in chat, no report file |
| **5. Exam notes** | optional Markdown deliverable: definition table, formula table with conditions of validity, worked-example steps, common mistakes, five-minute recall list |

## Triggers

Say it in whichever language you think in. Each line maps to the same job.

**English**

| Job | Say |
|---|---|
| New activities | *anything new in this course this week?* · *any new coursework posted?* |
| Missing-week download | *download Week 6* · *grab the files for Week 7* |
| Submission status | *what haven't I submitted yet?* · *which assignments are still open?* |
| File walkthrough | *what does this handout cover?* · *walk me through these slides* |
| Exam notes | *make me revision notes* · *build a key-points sheet* |

**简体中文**

| 事项 | 说 |
|---|---|
| 新作业 | 看下这门课这周有没有新东西 · 有没有新发布的作业 |
| 按周下载 | 补下载 Week 6 · 把第 7 周的文件下下来 |
| 提交状态 | 我哪些作业没交 · 还有哪些作业没交 |
| 内容讲述 | 这份讲义讲了什么 · 这几份课件讲了什么 |
| 重点笔记 | 帮我做重点笔记 · 整理复习重点 |

**繁體中文**

| 事項 | 說 |
|---|---|
| 新作業 | 看下這門課這週有沒有新東西 · 有沒有新發布的作業 |
| 按週下載 | 補下載 Week 6 · 把第 7 週的檔案下載下來 |
| 提交狀態 | 我哪些作業沒交 · 還有哪些作業沒交 |
| 內容講述 | 這份講義講了什麼 · 這幾份課件講了什麼 |
| 重點筆記 | 幫我做重點筆記 · 整理複習重點 |

## Why it is not turnkey

Moodle deployments disagree on almost everything that matters: hostname, token policy, module naming, section naming
and rate limiting. A skill that *guesses* the site's shape will quietly report "no new work" all semester — the worst
possible failure, because it looks like success.

So the first run on a new site stops and asks. Answers go into `site-profile.md`, and **no profile means no run**.

| Question | Why it matters |
|---|---|
| Site root URL | Everything hangs off `https://<host>` — never guessed from a course name |
| How is the token issued | `/login/token.php` + `moodle_mobile_app`, a manual key, or OIDC where a plain token is impossible |
| Is Web Service enabled | SSO-only accounts frequently cannot mint a token at all |
| Course id | From `core_enrol_get_users_courses`, never from a URL |
| Local directory + week naming | `Week 6`? `W6`? `Topic 6`? Mirror what the user already has |
| Which activities count | `assign` everywhere, or `turnitintooltwo` / `quiz` / `workshop` / custom plugins |
| Rate limiting | Aggressive WAFs answer 502 on burst POSTs — keep calls serial, ~0.15 s apart |

## Install

```bash
git clone https://github.com/Faixiang/moodle-course-watch.git
```

Drop the folder into your agent's skills directory, e.g.

```
<agent>/skills/custom/moodle-course-watch/
  SKILL.md
  site-profile.template.md   →  copy to site-profile.md and fill in
  scripts/check_site.py
  scripts/submission_status.py
```

`site-profile.md` holds your host, course ids and token — it is **git-ignored** and must never be committed.

## Requirements

- Python 3.9+, standard library only (no `pip install`).
- A local MCP server exposing Moodle Web Service tools. Reference implementation:
  [moodle-mcp](https://github.com/Faixiang/moodle-mcp) — 7 tools including `moodle_course_files` and
  `moodle_download_week`. Point at it with `MOODLE_MCP_DIR`.

## Usage

```bash
# 1. probe the site before trusting it
python scripts/check_site.py --mcp-dir <dir>
#    → site name, Moodle version, function count, module-type census, section names, duedate coverage

# 2. submission status across courses
python scripts/submission_status.py --courses 28048,28196 --mcp-dir <dir>
python scripts/submission_status.py --courses 28048 --pattern "assignment|tutorial" --json
```

Then just talk to the agent, in English, 简体 or 繁體 — see [Triggers](#triggers) for the phrasing of each job.

## Download links — the one trap worth naming

```
✅ https://<host>/webservice/pluginfile.php/<ctx>/<component>/<file>?forcedownload=1&token=<token>
❌ https://<host>/webservice/pluginfile.php/...            → returns JSON, not a file
❌ https://<host>/mod/resource/view.php?id=<id>            → needs a browser session
```

Without a token the endpoint answers with an error document whose HTTP status may still be `200`. Always verify the
`%PDF` header and the byte count, never the status code alone.

## Blind spots — stated, never hidden

- **Turnitin and quiz submission states** are not exposed by the standard Web Service. The activity is visible; the
  status is not. The skill says *unverifiable* rather than marking 已交/未交.
- **Blank `duedate`** is common on sites that never set deadlines. It is reported as 未设置, never invented.
- **Custom plugins** may or may not appear under `mod_assign_get_assignments` — run a module census before promising
  coverage.
- **Scrambled formula layers** in maths PDFs are read visually page by page, and unparsable formulas are listed as
  待确认 instead of being reconstructed from memory.

## Scope and limits

- Read-only by design. No submission, no grading, no writes back to Moodle.
- Personal academic tool, not affiliated with any institution. Your site's acceptable-use policy is your call.

## License

MIT — see [LICENSE](LICENSE).
*（内容由AI生成，仅供参考）*
