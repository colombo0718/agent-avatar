# PROJECT.md — AA（agent-avatar）

## 這是什麼

LL 全部 agent 的**「基因序列」倉庫**。核心思想：

> **用文字（markdown）的方式、存下所有 agent 的所有「基因序列」、
> 這樣 avatar 在各處都能連貫。**

每個 avatar = 一份完整的「DNA」：性格 / 形象 / 能力 / 系統提示詞 / LLM 綁定 / 通路相容性——用 markdown 寫成可移植、可重現、可繼承的序列。

### 基因序列解的 6 個連貫性問題

| 問題 | 解法 |
|------|------|
| 跨通路連貫（LINE / 直播 / VTuber / 遊戲 NPC 是同一個） | 同一份基因序列載到不同 channel |
| 抗 LLM 更換（Anthropic / OpenAI / Workers AI 任一家） | Soul 換、Avatar 不變 |
| 抗繪圖工具更換（Gemini → Midjourney → 自家 LoRA） | 同份 prompt 重生、長相一致 |
| 可被繼承（新 agent 繼承既有 avatar 部分基因） | markdown 段落可組合 |
| 可長壽（跨技術世代、3D / AR / 影片版） | 文字基因永不過期 |
| 可離開 AI 場域（印書 / 周邊 / NFT / cosplay） | 設計師讀基因序列就能畫 |

→ **AA 是 LL「avatar 護城河」的物理載體**——別家做 LL 競品、要從零累積 N 個 avatar 的基因序列、需要時間、不可一夜完成。

---

## 三軸定位

對應 [`agent-memory-architecture`](https://github.com/colombo0718/matrix-manager/blob/master/agent-memory-architecture.md) 三軸：

- **AA 軸（Agent）= 本 repo 的職責**
- LL 軸：MM、UNIVERSE 全域狀態
- II 軸：infinity-identity 客戶 context

→ Per-Agent 軸的所有設定錨定在這、跨 session 永久。

---

## 與其他 repo 的職責邊界

| 我（AA）| 我的事 | 我不管的事 |
|---------|------|---------|
| ✅ avatar 人格 / 形象 / 系統提示詞 | | |
| ✅ avatar 綁哪個 LLM、什麼 status（active_llm / static_ip）| | |
| ✅ avatar 通路相容（LINE / Widget / VTuber / Game NPC）| | |
| ✅ avatar 跨遊戲 playbook 累積 | | |
| | | ❌ 員工編制名冊（去 [`matrix-manager/memory/agents-register.json`](https://github.com/colombo0718/matrix-manager)）|
| | | ❌ 客戶 context（去 [`infinity-identity`](https://github.com/colombo0718/infinity-identity)）|
| | | ❌ LINE / cockpit 通道 harness（去 [`yeyan`](https://github.com/colombo0718/yeyan) / [`captain-cockpit`](https://github.com/colombo0718/captain-cockpit)）|

→ **MM register = 員工清單；AA = 每位員工的詳細個人檔案**。互相引用、各管各的。

---

## Avatar Status：active_llm vs static_ip

每個 avatar 有 `status` 欄位、區分**運行狀態**：

| status | 意思 | 例 |
|--------|-----|----|
| `active_llm` | 已綁定 LLM、可即時對話互動 | 葉衍君、克勞德、阿全（cwsoft 那邊） |
| `static_ip` | 純角色形象設定、未接 LLM、做 IP 資產 | NN 六貓（目前） |
| `dormant_llm` | 曾經 active、暫停運作中 | （未來如 agent 退役但保留檔案）|

升級路徑：static_ip → 補上 prompt-fragment.md + LLM 綁定 → 變 active_llm。

**這個 framing 讓 LL 的 IP 線跟 AI 線天然合流**——不是兩條獨立發展、是同一個 avatar 倉庫的兩種啟用狀態。

---

## 目錄結構

```
agent-avatar/
├── PROJECT.md
├── CLAUDE.md (from MM)
├── README.md
├── TODO.md
├── REGISTER.md                       ← 所有 avatar 目錄（簡表 + status + 通路）
│
├── schemas/                          ← avatar profile 格式規範
│   └── avatar-profile.schema.json
│
├── yeyan/                            ← 葉衍君（active_llm）
│   ├── profile.md                    ← 角色身份 / 性格 / 能力 / 場域
│   ├── prompt-fragment.md            ← system prompt 給 LLM 用
│   ├── visual-spec.md                ← 形象描述 / 生圖 prompt
│   └── status.json                   ← active_llm、綁的 LLM、版本
│
├── claude-butler/                    ← 克勞德管家（active_llm）
│   ├── profile.md
│   ├── prompt-fragment.md
│   ├── visual-spec.md
│   └── status.json
│
├── huahua/                           ← 花花、刀馬旦（static_ip、NN 六貓）
├── niuniu/                           ← 牛牛、捕快（static_ip、NN 六貓、原雪雪）
├── kongkong/                         ← 空空、武僧（static_ip、NN 六貓）
├── momo/                             ← 墨墨、書生（static_ip、NN 六貓）
├── juju/                             ← 橘橘、屠戶（static_ip、NN 六貓）
├── rourou/                           ← 柔柔、樂伎（static_ip、NN 六貓）
│
├── (依依 yiyi)                        ← 待建、II 公關長
├── (小葉 xiaoye)                      ← 待建、學習推薦
├── (小月 xiaoyue)                     ← 待建、玩樂推薦
│
└── docs/
    ├── status-lifecycle.md           ← static_ip → active_llm 升級流程
    ├── llm-binding-matrix.md         ← 9 角色 × 9 LLM 對應（從 II repo 搬來）
    └── soul-avatar-channel-3-layer.md ← Soul / Avatar / Channel 三層架構
```

→ **扁平結構**：avatar 資料夾直接在 repo 根、不再包 `/avatars/` 一層（2026-05-20 colombo 拍板：repo 名已是 agent-avatar、再 nest `/avatars/` 是 tautology）。

---

## 命名

- repo：`agent-avatar`（**單數概念**、概念是「agent × avatar 配對」）
- LL register 內代號：**AA**

---

## 開發規範

- avatar profile 用 markdown + frontmatter（人機都好讀）
- LLM 模型 ID 寫進 `status.json`、跟 [`infinity-identity/docs/ii-constellation-llm-mapping.md`](https://github.com/colombo0718/infinity-identity/blob/master/docs/ii-constellation-llm-mapping.md) 一致（之後可能移過來這邊）
- 形象 prompt 變動要記錄 version、之前的進 `docs/version-history/`

---

## 起源

2026-05-19 從 II repo 拆出。II 原本被塞了「身份 + 客服 widget + 內容投放 + agent 設定」四件性質不同的事、agent 設定獨立成 AA、II 退回專注客戶 context。

對應討論：[2026-05-19-yeyan-chatbot-claude-butler-and-agent-avatar.md](https://github.com/colombo0718/matrix-manager/blob/master/meetings/2026-05-19-yeyan-chatbot-claude-butler-and-agent-avatar.md)
