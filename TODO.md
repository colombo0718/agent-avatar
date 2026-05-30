# TODO.md — AA

## ⬡ MM 同步

| title | status | importance | energy | effort | due | next_action | tags |
|-------|--------|------------|--------|--------|-----|-------------|------|
| AA 初始骨架 + REGISTER.md | active | 1 | l | 60 | 2026-05-20 | 寫 REGISTER + schema | aa,scaffold |
| 葉衍君 profile（v0、最小可用）| active | 1 | m | 60 | 2026-05-21 | YY 啟動需用、寫月衡君 / 公領域工作型化身 | aa,yeyan |
| 克勞德管家 profile + 形象 prompt | active | 1 | m | 90 | 2026-05-22 | 英倫 × Ghibli、32-35 歲、Gemini 生圖 prompt | aa,claude-butler |
| LLM binding matrix 從 II 移過來 | active | 2 | l | 30 | 2026-05-22 | `ii-constellation-llm-mapping.md` → `docs/llm-binding-matrix.md` | aa,docs |
| 依依 / 小葉 / 小月 profile | queued | 2 | m | 120 | 2026-05-30 | II Phase 2 才用、先 sketch | aa,ii-trinity |
| NN 六貓入庫（status=static_ip）| queued | 2 | m | 180 | 2026-05-30 | 從 neko-numen 三份文件抽出來、重整成 AA 格式 | aa,nn-cats,ip |
| Soul / Avatar / Channel 三層架構文件 | queued | 2 | m | 60 |  | docs/soul-avatar-channel-3-layer.md | aa,architecture |
| status lifecycle 文件 | queued | 3 | l | 30 |  | docs/status-lifecycle.md | aa,governance |

## 立即動工（這週）

- [ ] `REGISTER.md`——所有 avatar 一覽表（簡表 + status + 通路）
- [ ] `schemas/avatar-profile.schema.json`——profile.md frontmatter 規範
- [x] `yeyan/profile.md` + `prompt-fragment.md` + `status.json`
- [x] `claude-butler/profile.md` + `prompt-fragment.md` + `visual-spec.md` + `status.json`

## v0.1（YY 上線同步）

- [ ] LLM binding matrix（從 II 移過來、含 9 角色 × 9 LLM）
- [ ] yeyan profile 提供給 YY harness 載入測試
- [ ] claude-butler profile 提供給 YY harness（fork 葉衍君後用）

## v0.2（II Phase 2 同步）

- [ ] 依依 profile（公關長、Kimi-k2.6）
- [ ] 小葉 profile（學習推薦、GLM-4.7-flash）
- [ ] 小月 profile（玩樂推薦、GPT-OSS-20b）

## v0.3（IP 線進場）

- [ ] NN 六貓全部入庫（status=static_ip → 未來分別升 active_llm）
- [ ] 跨遊戲 playbook 開始累積（墨墨 vs MD、雪雪 vs SP 等）

## ll-agent-runtime framework（2026-05-25 AA 協理升任後新主軸）

> 詳見 [`docs/cross-runtime-decomposition.md`](docs/cross-runtime-decomposition.md) + [`matrix-manager/meetings/2026-05-25-aa-promotion-and-cross-runtime-inventory.md`](../matrix-manager/meetings/2026-05-25-aa-promotion-and-cross-runtime-inventory.md)

- [x] v0.1 跨域盤點文件（三軸：cwsoft 阿全 / YY / EE）
- [ ] 四方 review（colombo + 執行長 + 公關長 + 秘書長）拍板選項 A / B / C
- [ ] 公關長補 EE 端 framework 需求清單（依依 hook）
- [ ] 秘書長補 YY 端 soul-shell 設計意圖 + 雙胞胎 soul 治理草案
- [ ] 若拍板選項 B：對接公關長、在 EE v0.3 上 PoC framework v0.1.0
- [ ] AvatarLoader 設計（AA 為唯一 source of truth、其他 repo pull、禁 inline）
- [ ] 補讀：`gtb_codex.py` / `expense-echo/src/yiyi.ts` / `src/vision.ts`

## 待釐清

- [ ] 阿全經理（cwsoft）的 avatar 要不要也進 AA？還是 LL / cwsoft 邊界區隔？
- [ ] avatar 跟 LINE OA channel 是 1:N 還是 1:1？（一個 avatar 上多個 channel 嗎）
- [ ] 雙胞胎 soul 分歧（laptop ↔ home jsonl 各自寫）的治理機制（YY 5/21 提、未解）
