# REGISTER.md — Avatar 一覽表

> 本檔是 AA 的目錄、列出所有 avatar 的簡表。詳細設定見各 avatar 資料夾。
> 最後更新：2026-05-20

---

## active_llm（接 LLM、可對話）

| Avatar | 中文 | 場域 | 綁 LLM | 通路 | profile |
|--------|------|------|-------|------|---------|
| yeyan | 葉衍君 | 公領域、LL 推進 | TBD（待 YY v0 拍板）| LINE OA `@LL_葉衍君` | [`avatars/yeyan/`](avatars/yeyan/) |
| claude-butler | 克勞德管家 | 私領域、家庭 | TBD（同上） | LINE OA `@LL_克勞德` + 三人群組 | [`avatars/claude-butler/`](avatars/claude-butler/) |

---

## static_ip（未接 LLM、IP 形象資產）

| Avatar | 中文 | 江湖封號 | 倭鼠兇名 | 規劃綁 LLM | 來源 IP |
|--------|------|---------|--------|---------|--------|
| huahua | 花花 | 翩鴻驚羽 | 追魂帖 | Google Gemma-4 | NN 六貓 |
| xuexue | 雪雪 | 鐵面御史 | 勾命簿 | Meta Llama-4-scout | NN 六貓 |
| kongkong | 空空 | 禪武宗師 | 琉璃掌 | IBM Granite | NN 六貓 |
| momo | 墨墨 | 墨畫陰陽 | 山水囚 | DeepSeek-R1 | NN 六貓 |
| juju | 橘橘 | 豪俠鎮關 | 碾骨磨 | Alibaba Qwen3 | NN 六貓 |
| rourou | 柔柔 | 天音妙手 | 安魂曲 | Mistral | NN 六貓 |

---

## 規劃中 / 待建（II 群星會精靈）

| Avatar | 中文 | 角色 | 規劃綁 LLM | 對應 channel |
|--------|------|------|---------|------------|
| yiyi | 依依 | 公關長、跨平台身份 | Moonshot Kimi-k2.6 | II Widget on leaflune.org |
| xiaoye | 小葉 | 學習推薦員 | Zhipu GLM-4.7-flash | II Widget |
| xiaoyue | 小月 | 玩樂推薦員 | OpenAI GPT-OSS-20b | II Widget |

---

## 跨家族（cwsoft、未必納入 LL AA）

| Avatar | 中文 | 場域 | 綁 LLM | 通路 |
|--------|------|------|-------|------|
| aquan | 阿全經理 | cwsoft 客服 | Codex CLI | LINE OA `@526fdbzo` |

→ 是否納入待 colombo 拍板（LL / cwsoft 邊界）。

---

## Status 升降規則

```
static_ip ──補上 prompt-fragment.md + 綁 LLM──→ active_llm
active_llm ──暫停服務──→ dormant_llm
dormant_llm ──恢復──→ active_llm
任何 status ──保留檔案、移出編制──→ 退役（仍可重啟）
```

詳見 [`docs/status-lifecycle.md`](docs/status-lifecycle.md)。

---

## 總計

- active_llm：**2**（v0、實作中）
- static_ip：**6**（IP 資產備位）
- 規劃中：**3**（II 精靈）
- 跨家族：**1**（阿全、待定）

→ LL avatar 完整宇宙當前 **12 個**、未來會持續加入。
