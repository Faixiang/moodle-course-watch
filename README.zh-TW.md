---
AIGC:
    Label: "1"
    ContentProducer: 001191440300708461136T1XGW3
    ProduceID: c6eb62ba6ec5a28db2139adf23a2fe4b_ae58f52ad16911f196c5525400248c00
    ReservedCode1: Tb7hXjGr6zwL9hqdGoQ1TfidGmYZQSJnvqVF0W/MEWjQWgnnrDGWmDy+DcMi97hSPc4z4x7Zmr2Zz7W4XBEuDNiLDPv2AijuDDaXJpA16q4GHUtVpvK1LtGwQdL/xOOlnT58Ox8TvLvkg19rz0ij23lPiFMNa3x0KG9UNR+EF19VLpcaqBnPM4mC4o0=
    ContentPropagator: 001191440300708461136T1XGW3
    PropagateID: c6eb62ba6ec5a28db2139adf23a2fe4b_ae58f52ad16911f196c5525400248c00
    ReservedCode2: Tb7hXjGr6zwL9hqdGoQ1TfidGmYZQSJnvqVF0W/MEWjQWgnnrDGWmDy+DcMi97hSPc4z4x7Zmr2Zz7W4XBEuDNiLDPv2AijuDDaXJpA16q4GHUtVpvK1LtGwQdL/xOOlnT58Ox8TvLvkg19rz0ij23lPiFMNa3x0KG9UNR+EF19VLpcaqBnPM4mC4o0=
---

🏫 Moodle Course Watch
---

[English](README.md) · [简体中文](README.zh-CN.md) · **繁體中文**

> **配套於 [moodle-mcp](https://github.com/Faixiang/moodle-mcp)** —— 該專案提供本 skill 所呼叫的 Moodle Web
> Service 工具，本倉庫提供其上的工作流程。主倉同時以 git submodule 形式掛載本倉庫（`skill/moodle-course-watch/`，複製主倉需加 `--recurse-submodules`），兩條路徑擇一即可。

![Moodle](https://img.shields.io/badge/Moodle-4.x-e6602c)
![Python](https://img.shields.io/badge/Python-3.9%2B-3776ab)
![License](https://img.shields.io/badge/License-MIT-blue)
![Mode](https://img.shields.io/badge/Read--only-yes-brightgreen)

一個可攜、唯讀的 Moodle 課程巡檢工具，適用於**任意** Moodle 站點：偵測新發布的課業、將缺漏的週次資料夾鏡像到本機、回報尚未提交的活動，並對已下載的材料進行專業講述、自動劃分重點等級——另附可選的複習筆記交付物。

永遠不會向 Moodle 提交、編輯或上傳任何內容。每次呼叫都是對標準 Web Service 的 `GET` 請求。

---

## 它能做什麼

| 事項 | 輸出 |
|---|---|
| **1. 新活動偵測** | 將 `cmid` 與本機基準比對，回報任何新活動的名稱與所屬章節 |
| **2. 缺漏週次下載** | 建立週次資料夾，以帶 token 的直連下載全部檔案，並校驗位元組數 |
| **3. 提交狀態** | 每個活動的 `submitted` / `draft` / `new`，附評分狀態與最後修改時間 |
| **4. 檔案內容講述** | 逐份專業講述：檔案定位、內容結構、**重點分級（必記 / 理解 / 了解）**、與前後週的銜接，以及無法讀取之處——於對話中給出，不產出報告檔案 |
| **5. 重點筆記** | 可選的 Markdown 交付物：定義表、含成立條件的公式表、例題步驟、常見錯誤、五分鐘回憶清單 |

## 觸發詞

用哪種語言思考就用哪種語言說。同一列指向同一件事。

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

## 為什麼它不是開箱即用

Moodle 部署在幾乎所有要緊的地方都不一樣：主機名、token 政策、模組命名、章節命名與限流策略。一個*靠猜*站點形狀的 skill 會安靜地回報「本週沒有新東西」——整整一個學期，而且看起來像成功，這是最糟的失敗方式。

所以在新站點上第一次執行時，它會停下來先問。答案寫進 `site-profile.md`，**沒有 profile 就不執行**。

| 問題 | 為什麼重要 |
|---|---|
| 站點根 URL | 一切都掛在 `https://<host>` 上，不能從課程名去猜 |
| token 如何發放 | `/login/token.php` + `moodle_mobile_app`、手動建立 key，或純 OIDC 根本無法簽發一般 token |
| Web Service 是否開啟 | 純 SSO 帳號經常連 token 都申請不到 |
| 課程 course id | 只能從 `core_enrol_get_users_courses` 取得，不要從 URL 猜 |
| 本機目錄與週次命名 | `Week 6`？`W6`？`Topic 6`？鏡像使用者既有的命名 |
| 哪些活動算課業 | 全是 `assign`，還是另有 `turnitintooltwo` / `quiz` / `workshop` / 自訂外掛 |
| 是否限流 | 激進的 WAF 在突發 POST 下回傳 502——呼叫要序列化，間隔約 0.15 秒 |

## 安裝

```bash
git clone https://github.com/Faixiang/moodle-course-watch.git
```

把整個資料夾放進你 agent 的 skills 目錄，例如：

```
<agent>/skills/custom/moodle-course-watch/
  SKILL.md
  site-profile.template.md   →  複製為 site-profile.md 並逐項填寫
  scripts/check_site.py
  scripts/submission_status.py
```

`site-profile.md` 保存你的主機、course id 與 token——它已被 **git 忽略**，絕不能提交。

## 依賴

- Python 3.9+，僅用標準函式庫（無需 `pip install`）。
- 一個暴露 Moodle Web Service 工具的本機 MCP server。參考實作：
  [moodle-mcp](https://github.com/Faixiang/moodle-mcp)——7 個工具，含 `moodle_course_files` 與
  `moodle_download_week`。以 `MOODLE_MCP_DIR` 指向它。

## 用法

```bash
# 1. 先探明站點，再談信任
python scripts/check_site.py --mcp-dir <dir>
#    → 站點名、Moodle 版本、函式數量、模組類型普查、章節名、duedate 覆蓋率

# 2. 跨課程的提交狀態
python scripts/submission_status.py --courses 28048,28196 --mcp-dir <dir>
python scripts/submission_status.py --courses 28048 --pattern "assignment|tutorial" --json
```

之後直接用英文、簡體或繁體對 agent 說話即可——每件事的說法見上文「觸發詞」。

## 下載直連——唯一值得單獨點名的坑

```
✅ https://<host>/webservice/pluginfile.php/<ctx>/<component>/<file>?forcedownload=1&token=<token>
❌ https://<host>/webservice/pluginfile.php/...            → 回傳 JSON，不是檔案
❌ https://<host>/mod/resource/view.php?id=<id>            → 需要瀏覽器工作階段
```

不帶 token 時，該端點會回傳一份錯誤文件，而其 HTTP 狀態碼仍可能是 `200`。務必校驗 `%PDF` 檔頭與位元組數，絕不能只看狀態碼。

## 盲區——說清楚，絕不掩蓋

- **Turnitin 與 quiz 的提交狀態**不在標準 Web Service 的暴露範圍內。活動可見，狀態不可見。本 skill 會標為*無法核驗*，而不是擅自寫成已交／未交。
- **`duedate` 為空**在從不設截止日的站點上很常見。會回報為「未設定」，絕不編造。
- **自訂外掛**未必出現在 `mod_assign_get_assignments` 下——承諾覆蓋範圍前先跑一次模組普查。
- **數學 PDF 的公式文字層亂碼**時逐頁視覺讀取，無法解析的公式列入「待確認」，不憑記憶重建。

## 範圍與邊界

- 設計上唯讀。不提交、不評分、不向 Moodle 回寫任何內容。
- 個人學術工具，與任何院校無關。站點的可接受使用政策由你自行判斷。

## 授權

MIT——見 [LICENSE](LICENSE)。
*（内容由AI生成，仅供参考）*
