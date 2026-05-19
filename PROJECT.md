# PROJECT.md — AA（agent-avatar）

## 這是什麼

LL 全部 agent 的**角色形象資料庫**。核心思想：

> **每個 agent 都有鮮明的角色形象（avatar）。**
> agent 跟 avatar 是 1:1 配對、不是兩個獨立的東西。

每個 avatar = 一份完整的角色檔案：性格 / 形象 / 能力 / 系統提示詞 / LLM 綁定 / 通路相容性。

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
├── avatars/                          ← 每個 avatar 一個資料夾
│   ├── yeyan/                        ← 葉衍君
│   │   ├── profile.md                ← 角色身份 / 性格 / 能力 / 場域
│   │   ├── prompt-fragment.md        ← system prompt 給 LLM 用
│   │   ├── visual-spec.md            ← 形象描述 / 生圖 prompt
│   │   └── status.json               ← active_llm、綁的 LLM、版本
│   │
│   ├── claude-butler/                ← 克勞德管家
│   │   ├── profile.md
│   │   ├── prompt-fragment.md
│   │   ├── visual-spec.md
│   │   └── status.json
│   │
│   ├── (依依 yiyi)                    ← 待建、II 公關長
│   ├── (小葉 xiaoye)                  ← 待建、學習推薦
│   ├── (小月 xiaoyue)                 ← 待建、玩樂推薦
│   ├── (墨墨 momo)                    ← 待建、書生（NN 六貓）
│   ├── (橘橘 juju)                    ← 待建、屠戶（NN 六貓）
│   ├── (花花 huahua)                  ← 待建、刀馬旦（NN 六貓）
│   ├── (雪雪 xuexue)                  ← 待建、捕快（NN 六貓）
│   ├── (空空 kongkong)                ← 待建、武僧（NN 六貓）
│   └── (柔柔 rourou)                  ← 待建、樂伎（NN 六貓）
│
└── docs/
    ├── status-lifecycle.md           ← static_ip → active_llm 升級流程
    ├── llm-binding-matrix.md         ← 9 角色 × 9 LLM 對應（從 II repo 搬來）
    └── soul-avatar-channel-3-layer.md ← Soul / Avatar / Channel 三層架構
```

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
