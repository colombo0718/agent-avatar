# Cross-Runtime Decomposition — LL agent runtime 跨域盤點 v0.1

> **作者**：AA 協理（agent-avatar/51a282e9）
> **日期**：2026-05-25
> **狀態**：盤點階段、不寫程式、待四方 review（colombo + 執行長 + 公關長 + 秘書長）
> **目的**：抽出 LL 三條既有 agent runtime 實作的共通元件、找出可直接複用 / 要再抽象 / 設計衝突的部分、決定下一步（起新 repo / PoC / 暫緩）

---

## 一、三條既有實作的 anchor

| 系統 | 主導 | 通路 | LLM | 部署形態 | 狀態 |
|------|------|------|-----|---------|------|
| **cwsoft 阿全** | colombo + 阿全 | LINE OA `@526fdbzo`、`@708juxdz` | codex + MCP（GPT 系）| home Python Flask :6010 + Caddy + Cloudflared | prod、跑了 1 週、多項業務真實使用中 |
| **YY 葉衍君 + 克勞德** | 秘書長（MM/7e32b84d） | LINE OA `@LL_葉衍君`、克勞德管家 | Claude CLI `claude --resume` | home Python Flask :5001/:5002 + Caddy + Cloudflared + NSSM | v0 已上線 24/7、單一 colombo session、session_key_strategy 寫了但尚未實作 |
| **EE 依依** | 公關長（II/bc20d23b） | Web（expense-echo.leaflune.org） | CF Workers AI（Llama 3.2 Vision + 3.1 8B、計畫切 Kimi-k2.6） | CF Pages Functions（serverless edge、無常駐 process） | v0.1.0 ship、v0.2 切 Kimi 中 |

→ 三條的「runtime 形態」完全不同——一條 Python Flask 跑 codex、一條 Python Flask 跑 claude CLI、一條 serverless edge 跑 Workers AI。但**業務需求高度重疊**：人格載入、權限收斂、多 user 隔離、LLM 呼叫、寫入確認、用量記錄、fallback。**LL 不該各自為政重複造輪子**——這就是本盤點要解決的問題。

---

## 二、元件橫向比較表（核心 12 項）

> ✅ 已實作 / 🟡 部分實作或規劃中 / ❌ 未實作或不適用 / — 不在這層

| # | 元件 | cwsoft 阿全 | YY 葉衍君 | EE 依依 | 是否「三家都有」 |
|---|------|------------|----------|--------|----------------|
| 1 | **LLM 子程序呼叫 / API call** | `codex exec ... --sandbox danger-full-access` | `subprocess.run([claude_bin, --print, --resume, sid, msg])` | `env.AI.run('@cf/...')` Workers AI binding | ✅ 全有、形態不同 |
| 2 | **Soul / 記憶持久化** | `~/.gtb_codex_sessions/<user_id>.session`（per-user）| `~/.claude/projects/<path-hash>/<sid>.jsonl`（161MB 完整 session） | ❌ 無多 turn 持久化（每 request stateless） | 🟡 阿全 + YY 有、EE 沒 |
| 3 | **Shell / 通路層** | Flask /callback + LINE Reply | Flask /callback + LINE Reply | Pages Functions /api/* + JSON Response | ✅ 全有 |
| 4 | **Per-user / Per-session 隔離** | ✅ per-user session lazy mint、thread-safe lock | 🟡 config 寫 `per_line_user` 但 v0 仍共用 colombo session | ✅ D1 sessions 表 + cookie（Google OAuth） | ✅（YY 規劃中） |
| 5 | **Capability lock（工具白名單 / 黑名單）** | `disabled=['shell_tool','multi_agent','apps']` + MCP 白名單 | `allowed_tools` + `disallowed_tools` 寫 config.json、但目前未注入 claude CLI | ❌（Workers AI 本身無 tool use） | 🟡 阿全有、YY 有 config 但沒落地 |
| 6 | **MCP 工具掛載** | `cwsoft-sqlserver-mcp`（13 tools）+ `cwsoft_ai_tools`（11 tools） | 規劃 `leaflune_universe_tools` + `household_tools`、v0 未啟用 | ❌ 不適用（CF edge） | 🟡 阿全有、YY 規劃 |
| 7 | **Avatar / 人格載入** | AGENTS.md 注入 boot prompt | `agents/<bot>/avatar-ref.md` 指向 AA repo profile、但 v0 仍倚賴 jsonl 既有人格 | `expense-echo/src/yiyi.ts` 寫 system prompt（AA `yiyi/prompt-fragment.md` 副本） | ✅ 全有、機制不同 |
| 8 | **寫入確認雙層 / 4-tag self-report** | `[KB_FIX][KNOWHOW][WANT_NEW_TOOL][ACTION]` + 可逆/不可逆判準 | config 寫 `writes_require_confirm: true` 但未實作 | ❌ | 🟡 阿全有、YY 規劃 |
| 9 | **KNOWHOW 跨 session 注入** | ✅ 新 user mint 時掃近 14 天 notebook 注入 boot prompt | ❌ | ❌ | 🟡 只阿全有 |
| 10 | **用量記錄 / fallback chain** | ❌（codex 計費由 OpenAI 端處理） | ❌ | ✅ D1 `ai_runs` 表 + 規劃 `VISION_PROVIDERS[]` fallback array | 🟡 只 EE 有 |
| 11 | **群組 vs 單聊處理** | 1-on-1 only、群組 wakeword TODO | 群組「老爺說:/夫人說:」prefix 注入（路線 C）、v0 尚未實作 | — 不適用 | 🟡 |
| 12 | **Broadcast / Multicast / Push 主動推送** | ✅ `broadcast_self_intro.py --to` | ❌ | ❌（user 主動拉、無 push） | 🟡 只阿全有 |

### 12 項分類小結

- **三家都有 / 必抽**：1（LLM 呼叫）、3（Shell）、4（per-user 隔離）、7（人格載入）——這 4 項是 LL agent runtime 的**最小骨幹**
- **兩家有 / 強烈建議抽**：2（Soul 持久化）、5（Capability lock）、6（MCP）、8（寫入確認）、11（群組處理）
- **只有 1 家有 / 各家風格**：9（KNOWHOW 阿全獨有）、10（fallback EE 獨有）、12（broadcast 阿全獨有）

---

## 三、三家都有 → 核心必抽（4 項）

### 1. LLM 呼叫抽象層

三家機制完全不同：
```
阿全：  subprocess (codex exec ...)              ← shell-spawn、文字 IO
YY：    subprocess (claude --resume ...)         ← shell-spawn、stream-json IO
EE：    env.AI.run('@cf/...')                    ← in-process binding、JSON IO
```

**抽象建議**：定義 `LLMProvider` interface、底下三種實作：
```
interface LLMProvider {
  invoke(messages, options) -> Response
  supports_resume() -> bool
  supports_tools() -> bool
  supports_streaming() -> bool
}
class SubprocessClaudeProvider implements LLMProvider { ... }
class SubprocessCodexProvider implements LLMProvider { ... }
class WorkersAIProvider implements LLMProvider { ... }
class GeminiProvider implements LLMProvider { ... }    # for fallback chain
```
→ 對位 [`ll-antifragility-via-task-maturation.md`](../../leaflune/notes/ll-antifragility-via-task-maturation.md) 的 fallback chain 戰略。

### 2. Shell（通路適配層）

三家都做的同一件事：
```
inbound: {LINE webhook / HTTP request} -> {parsed user msg, user_id, reply_handle}
outbound: {LLM response} -> {LINE Reply API / JSON Response / SSE stream}
```

**抽象建議**：`ChannelAdapter` interface、底下：
- `LineAdapter`（共用：簽章驗證、Reply / Push、loading indicator、QuickReply tag 解析）
- `WebAdapter`（HTTP JSON、cookie session、SSE）
- `WidgetAdapter`（II Widget、之後 RR/DD/TT 通用）
- 未來：`CockpitAdapter`、`StreamAdapter`（VTuber）、`GameNPCAdapter`

→ LL 三軸的 II 軸（User）抽象在這層。

### 3. Per-user 隔離

三家機制：
```
阿全：  user_id → ~/.gtb_codex_sessions/<user_id>.session（檔案）
YY：    LINE userId → claude sid（規劃 SQLite 但未落地）
EE：    user_id → D1 sessions table（Google OAuth + cookie）
```

**抽象建議**：`SessionStore` interface、底下：
- `FileSessionStore`（阿全用）
- `SQLiteSessionStore`（YY 規劃）
- `D1SessionStore`（EE 用、邊緣分散式）

→ 對位 [`infinity-identity`](https://github.com/colombo0718/infinity-identity) repo 的核心職責「客戶 context 紀錄」。**這層的抽象應該跟 II 對齊、不是 framework 自己造一套**。

### 4. Avatar 人格載入

三家機制：
```
阿全：  AGENTS.md + KB_FIX/KNOWHOW 注入 boot prompt
YY：    agents/<bot>/avatar-ref.md → AA repo profile.md（但 v0 倚賴 jsonl）
EE：    src/yiyi.ts 寫 system prompt（AA prompt-fragment.md 副本）
```

**這正是 AA 軸的核心職責**——AA 必須是**唯一 source of truth**、其他 repo 載入時應該是「pull from AA」而非「自己存一份」。

**抽象建議**：
```python
class AvatarLoader:
    def load(avatar_id: str) -> Avatar:
        # 從 AA repo 讀 profile.md + prompt-fragment.md + visual-spec.md + status.json
        # 解析 frontmatter、組成 system prompt、回傳結構
        ...

# 用法（任何 runtime 都這樣呼叫）：
avatar = AvatarLoader.load("yiyi")
system_prompt = avatar.system_prompt
llm_binding = avatar.llm_binding  # kimi-k2.6 + llama 3.1 fallback
```

→ 解 EE 現在「`src/yiyi.ts` 是副本、改要兩邊同步」的痛點。AA 改、所有 runtime 自動繼承。

---

## 四、單家獨有元件清單

### 只有阿全有（最新工程實踐、可借）

| 元件 | 描述 | 借鑑建議 |
|------|------|---------|
| **4-tag self-report**（KB_FIX/KNOWHOW/WANT_NEW_TOOL/ACTION） | agent 在 reply 內標註結構化標記、harness 攔截轉 UI 或寫 notebook | **強烈建議納入 framework v0.1**：抽成「結構化標記攔截器」、各 runtime 可選用 |
| **KNOWHOW 14 天滾動注入** | 新 user mint 時掃近 14 天 notebook、抽 [KNOWHOW] dedup 注入 boot prompt | **納入但設成 opt-in**：YY / EE 用得到時開、framework 提供 hook |
| **寫入確認雙層**（可逆 single-step / 不可逆 preview-commit two-step） | 可逆 tool（如 close_branch 可 reopen）= 對話確認；不可逆 tool = preview / commit 強制兩段 | **必納入 framework**：寫入類動作的安全護欄、跟 LL「家長對成年子女」哲學一致 |
| **MCP function 多 entrypoint 性質** | 同 function 可被 codex / scheduler / 批次腳本呼叫、不必開三套 API | **納入 framework architecture 文件**：寫進 design guideline |
| **Broadcast / Multicast 主動推送** | `broadcast_self_intro.py --to <ids>` | **納入 ChannelAdapter interface**：把 push capability 抽出來 |
| **業務規則 standalone 精神** | 不要跨領域抽象延伸（colombo 5/21 反饋：「加點是加點、報價單是報價單」） | **寫進 framework design rules**：不為了優雅而強行抽象 |

### 只有 YY 有（soul-shell 原型、可能更純）

| 元件 | 描述 | 借鑑建議 |
|------|------|---------|
| **Soul / Shell 兩層分離哲學** | jsonl = soul（記憶 / 人格）、Flask = shell（薄電話線、只做 LINE 簽章 + subprocess）、「一人一套裝甲」 | **作為 framework 核心架構哲學**——抽象層命名與職責切分以此為準 |
| **跨機器 soul 移植**（jsonl scp） | laptop 上的 161MB session 整檔 scp 到 home、claude --resume 直接活過去 | **納入「soul 治理」設計考量**：framework 應提供 soul 匯出 / 匯入 / 校驗 hook |
| **雙胞胎 soul 分歧問題**（colombo 5/21 提出） | 同 sid 在兩台機器、各自累積對話、記憶分歧 | **設成 v0.2 待解問題**、寫進 framework TODO、不在 v0.1 解 |
| **一人一套裝甲 NSSM 部署模式** | 每 bot 獨立 Flask + 獨立 NSSM service、crash 隔離 | **納入部署 playbook**（已有 `yeyan/docs/deployment-plan.md`、可升 LL 通用） |
| **LINE native typing indicator**（loading_start） | claude 思考時、LINE 上有原生「...」提示 | **納入 LineAdapter 預設行為** |
| **avatar-ref.md pointer 模式** | runtime config 不存人格、只存指向 AA 的 pointer | **作為 AvatarLoader 預設模式、各 runtime 強制走 pointer、禁止 inline 人格** |

### 只有 EE 有（人格層 / framework 必須支援的 hook）

| 元件 | 描述 | 對 framework 的要求 |
|------|------|------------------|
| **Cloudflare 全家桶 serverless 部署** | Pages + Functions + D1 + R2 + Workers AI binding | framework 必須有 **edge-friendly 變體**——不能假設 long-running process |
| **D1 `ai_runs` 用量計數表** | 每次 LLM call 記 model/task/latency/ok/error | framework 應內建 LLM call instrumentation 介面 |
| **跨 provider fallback chain**（規劃中 VISION_PROVIDERS[]） | CF Llama Vision → Gemini Flash → GCP Vision API → home LLaVA | framework 必須有 **provider chain abstraction**、按 daily/monthly limit 自動切換 |
| **動態心情 / 五軸性格 character bible** | 依依「對金錢鬆弛 / 對工資計較 / 沒給錢時懶散 / 付錢就積極 / 動態心情」 | framework 必須支援 **dynamic state injection**——人格層之外有「動態 state」可以 per-turn 修改（v0.3 才要、v0.1 留 hook） |
| **Google OAuth → cookie session** | 不靠 LINE userId、靠跨平台 Google 身份 | framework 必須有 **多元身份來源**：LINE / Google / II SSO 都能餵進同一個 SessionStore |
| **數字使用紀律**（prompt-fragment.md「不要憑空計算」） | LLM 數字幻覺防範、在 prompt 層硬約束 | framework 應該提供 **prompt safety pattern library**——常見 LLM 風險的對應 prompt 模板 |
| **「最壞情境的商機」**（連免費 LLM 都爆 → 升 pro VIP 通道）| fallback 全爆時的 UX 包裝、技術事件變商機 | framework fallback chain 應有 **graceful degradation hook**——讓 avatar 可以自然講出降級話術 |

---

## 五、三家衝突的設計決策（誰對？）

| # | 衝突點 | 阿全 | YY | EE | 我的判斷 |
|---|--------|-----|----|----|---------|
| 1 | **記憶機制** | per-user session 檔案 | 共用 colombo 161MB jsonl | 完全 stateless / 從 D1 撈 | **看場景**：對外服務（EE 那種多 user）走 EE 路線（stateless + DB context）；對內治理 agent（葉衍君跟 colombo 對話）走 YY 路線（完整 soul jsonl）。framework 兩種模式並存、SessionStore interface 抽掉差異 |
| 2 | **是否倚賴 long-running process** | 是（Flask + NSSM 24/7） | 是 | 否（serverless edge） | **必須兩種都支援**。對外 user-facing 走 edge（規模 + 成本）、對內治理 / 雙向長對話走 long-running |
| 3 | **人格存哪** | AGENTS.md 在 cwsoft repo（不在 AA） | avatar-ref.md 指向 AA（pointer） | src/yiyi.ts 是 AA 的副本（要 sync） | **AA 為唯一 source of truth**、其他 repo 一律走 AvatarLoader pull、禁止 inline 或副本——這是 AA 軸的原意 |
| 4 | **Capability lock 機制** | 啟動時 `disabled=[...]` 硬禁 | `allowed_tools` 寫 config 但未落地 | 不適用（Workers AI 無 tool use） | 阿全模式較成熟、framework 採用阿全做法、YY 補上 |
| 5 | **LLM 呼叫方式** | subprocess | subprocess | in-process binding | **看部署形態決定**、不衝突；framework `LLMProvider` 抽象兩者 |
| 6 | **Avatar 動態 state（心情 / tier-aware）** | 無（純 boot-prompt） | 無 | 規劃中（v0.3）但 v0.2 鎖 default | EE 主導的設計更前瞻、framework v0.1 留 hook（state injection point）、v0.3 才接 RL 三件套 |

→ **沒有「誰錯」、是場景不同**。framework 必須**多種形態並存**、不要強迫統一。

---

## 六、可直接複用清單（最小工程量）

直接從現有 repo 抽 module 升 framework、改個 import path 就能跑：

| 元件 | 來源 | 預估工程量 |
|------|------|----------|
| LineAdapter（簽章驗證 + Reply + loading + QuickReply tag）| `yeyan/harness/relay.py` + 阿全 gtb_codex tag parser | 0.5 天 |
| AvatarLoader（讀 AA repo 的 profile + prompt-fragment + status）| 新寫、但邏輯簡單（讀 markdown + frontmatter） | 0.5 天 |
| FileSessionStore | 阿全 `~/.gtb_codex_sessions/` 那套 | 0.5 天 |
| D1SessionStore | EE 既有 `src/auth.ts` 抽 | 0.5 天 |
| SubprocessClaudeProvider | yeyan `_ask_claude` 函式 | 0.5 天 |
| SubprocessCodexProvider | 阿全 gtb_codex 抽 | 0.5 天 |
| WorkersAIProvider | EE `functions/api/parse-text.ts` 抽 | 0.5 天 |
| 4-tag self-report parser | 阿全現成 | 0.5 天 |
| NSSM 部署 playbook | `yeyan/docs/deployment-plan.md` + `docs/known-issues.md` 升通用 | 0.5 天 |

**小計：4-5 天**——只是抽 + 拼、沒有新邏輯。

## 七、要再抽象清單（中工程量）

需要設計新介面 + 適配既有實作：

| 元件 | 為什麼要再抽象 | 預估工程量 |
|------|--------------|----------|
| LLMProvider interface + 三實作對齊 | 三家 IO 形態不同、要設計 unified Response 型別 | 2 天 |
| ChannelAdapter interface | LINE / Web / Widget / Cockpit 多通路、要設計 inbound/outbound 共同形狀 | 1.5 天 |
| FallbackChain（按 daily/monthly limit 切 provider）| EE 規劃中、要設計 quota tracker + decision engine | 2 天 |
| KNOWHOW 注入機制（opt-in） | 阿全現成但耦合在 gtb_codex、要拆出來通用 | 1 天 |
| 寫入確認雙層（可逆判準 + preview/commit） | 要設計 Tool metadata schema（reversibility flag） | 1.5 天 |
| Dynamic state injection point（hook、預留給 v0.3 動態心情） | 設計 hook 介面、v0.1 預設 no-op | 0.5 天 |

**小計：8-9 天**。

---

## 八、總工程量估算

| 階段 | 範圍 | 天數 |
|------|------|------|
| v0.1.0 骨幹 | 三家都有的 4 項 + 直接複用清單 | 5-7 天 |
| v0.1.1 補齊 | 兩家有的元件（capability lock / MCP / 寫入確認 / 群組） | 4-5 天 |
| v0.2.0 抽象 | 三 interface 設計（LLMProvider / ChannelAdapter / SessionStore）+ fallback chain | 7-9 天 |
| v0.3.0 對齊 | dynamic state hook、KNOWHOW opt-in、雙胞胎 soul 治理 | 5-7 天 |

**v0.1.0 MVP 約 1 週工程量、整套到 v0.3.0 約 1 個月**。

---

## 九、下一步建議（待四方 review 拍板）

三條選項、各有取捨：

### 選項 A：**起新 repo `ll-agent-runtime`、直接動工 v0.1.0**
- ✅ 從一開始就是 framework 形態、不被既有 repo 結構綁住
- ✅ 可同時被 YY / EE / 未來新 agent 引用
- ❌ 從零起、初期沒有真實使用者打磨
- ❌ 風險：抽象做太早、跟未來真實需求對不上

### 選項 B：**先 PoC 在 EE 或 YY 上落地一輪、再抽 framework**
- ✅ 抽象基於真實需求、不空談
- ✅ 公關長 / 秘書長有現成戰場可驗證
- ❌ 兩個 repo 並行修改、容易 diverge
- ❌ 抽出來時可能要回頭改既有實作（成本未必比 A 低）

### 選項 C：**暫緩、繼續觀察阿全演化、3 個月後再盤點**
- ✅ 對位 [`ll-antifragility-via-task-maturation.md`](../../leaflune/notes/ll-antifragility-via-task-maturation.md) 4 階梯「等任務成熟自然下沉」哲學
- ✅ 阿全還在快速演化（5/22 → 5/24 大量設計變更）、現在抽會踩到下個版本就過時
- ❌ LL 對外承諾（II Widget、跨平台金流）需要 framework 才能規模化、不能無限等
- ❌ 三家 diverge 越久、後抽越痛

**我的傾向：選項 B（在 EE 落地）**——理由：
1. EE 是 LL AA 軸第一個對外 ship 的應用、公關長正在大量改、實戰需求最清楚
2. EE 缺的兩項（per-user 多 turn session、fallback chain）正好是 framework 必要元件、做進去剛好兩邊都贏
3. YY 目前 v0 跑得穩、再動會打擾 colombo 跟葉衍君的對話、暫不打擾
4. 阿全是 cwsoft 體系、不該被 LL framework 綁、跨家族邊界要尊重

→ **具體建議**：framework v0.1.0 的第一個 user = EE v0.3、用 EE 的「fallback chain + dynamic state hook」需求帶出 framework 雛形、3 週後 v0.1 穩了再回頭挪 YY。

---

## 十、需求來源（advisor 列名）

本盤點的判斷依據來自：

- **公關長**（infinity-identity/bc20d23b）：EE / 依依的對外人格層設計、character bible 完整實作經驗。本檔尚未直接訪談公關長、判斷材料來自 [`agent-avatar/yiyi/profile.md`](../yiyi/profile.md)、[`agent-avatar/yiyi/prompt-fragment.md`](../yiyi/prompt-fragment.md)、[`expense-echo/functions/api/parse-text.ts`](../../expense-echo/functions/api/parse-text.ts)、[`matrix-manager/meetings/2026-05-22-cloud-infrastructure-survey-and-llm-fallback-chain.md`](../../matrix-manager/meetings/2026-05-22-cloud-infrastructure-survey-and-llm-fallback-chain.md)。**review 時請公關長補：framework 必須支援哪些 hook 才能裝得下依依的動態心情 + 數字使用紀律 + 鬆弛本質**。

- **秘書長**（matrix-manager/7e32b84d）：YY 葉衍君 + 克勞德的原架構師、soul-shell 概念源頭。判斷材料來自 [`matrix-manager/meetings/2026-05-21-yy-v0-line-bot-online-and-soul-shell-architecture.md`](../../matrix-manager/meetings/2026-05-21-yy-v0-line-bot-online-and-soul-shell-architecture.md)、[`yeyan/PROJECT.md`](../../yeyan/PROJECT.md)、[`yeyan/harness/relay.py`](../../yeyan/harness/relay.py)、[`yeyan/docs/deployment-plan.md`](../../yeyan/docs/deployment-plan.md)、[`yeyan/docs/known-issues.md`](../../yeyan/docs/known-issues.md)。**review 時請秘書長補：soul-shell 雙層架構的設計意圖、葉衍君本體 vs 克勞德的功能定位差異、雙胞胎 soul 分歧的治理草案**。

- **cwsoft 阿全**：reference impl、不動其程式碼。材料來自 sub-agent 對 `cwsoft-project-tracker/worklogs/2026-05-19` 至 `2026-05-24` 的盤點摘要。

兩位 advisor 跟我並列、不從屬。review 時意見不同、由我做判斷、不盲從。

---

## 十一、不在這次盤點範圍

- 框架的具體程式碼（盤點階段、不寫 code）
- cwsoft 任何修改（跨家族紅線）
- YY / EE 既有實作修改（盤點階段）
- 跨機器（ssh work / ssh home）動作（先回 colombo pin scope）
- 圖像生成 / TTS / agent-stream 通路（v0.1 先聚焦對話 agent runtime）

---

## 附錄 A：本盤點覆蓋的檔案清單

**直接讀過**：
- `cwsoft-project-tracker/worklogs/2026-05-19 ~ 2026-05-24`（透過 sub-agent 摘要）
- `cwsoft-project-tracker/AGENTS.md`、`PROJECT.md`（透過 sub-agent）
- `yeyan/PROJECT.md`、`README.md`、`TODO.md`
- `yeyan/harness/relay.py`、`harness/__init__.py`
- `yeyan/agents/yeyan/server.py`、`config.json`、`avatar-ref.md`
- `yeyan/docs/deployment-plan.md`、`docs/known-issues.md`
- `expense-echo/CLAUDE.md`、`README.md`、`wrangler.toml`
- `expense-echo/functions/api/parse-text.ts`、`parse-image.ts`
- `expense-echo/src/auth.ts`
- `agent-avatar/yiyi/profile.md`、`prompt-fragment.md`、`status.json`
- `matrix-manager/meetings/2026-05-19-yeyan-chatbot-claude-butler-and-agent-avatar.md`
- `matrix-manager/meetings/2026-05-20-yy-aa-bb-repos-built-and-three-axis-architecture.md`
- `matrix-manager/meetings/2026-05-21-yy-v0-line-bot-online-and-soul-shell-architecture.md`
- `matrix-manager/meetings/2026-05-22-cloud-infrastructure-survey-and-llm-fallback-chain.md`
- `leaflune/notes/agent-trust-as-grown-child.md`
- `leaflune/notes/ll-antifragility-via-task-maturation.md`

**未直接讀但已知存在、review 前可能補讀**：
- `cwsoft-aquan-manager/`（如存在）
- `general-task-bot/gtb_codex.py`（阿全 harness 主檔）
- `expense-echo/src/yiyi.ts`、`src/vision.ts`、`src/text.ts`
- `expense-echo/docs/strategy.md`、`docs/v0.2.0-chat-first-yiyi.md`
- YY repo 上線後 5/22 之後的更新（如有）

→ 若 review 時 advisor 指出特定檔案我沒讀到、補讀後再修本檔。
