---
name: ad-pipeline
description: Use when the user wants to produce a full multi-shot advertisement or branded short video (e.g. "幫我做一支廣告", "做一支 OO 的 Reels"). Orchestrates storyboard, Soul ID character consistency, brand lock, per-shot generation and assembly via the Higgsfield CLI.
version: 1.0.0
author: AI 少年學院
license: MIT
metadata:
  hermes:
    tags: [creative, advertising, video, higgsfield, pipeline]
    related_skills: [visual-gen]
---
# Skill：廣告 Pipeline（多鏡頭廣告自動產線 · CLI 版）

## 功能
接收一個廣告 brief，自動完成整條產線：**拆分鏡 → 鎖角色(Soul ID) → 鎖品牌 → 逐鏡生成 → 鏡頭接續 →（選配）配音／打分 → 交付**。
全程透過 **Higgsfield CLI**（終端機指令）執行，不走 MCP。

## 觸發時機
使用者說「幫我做一支廣告」「做一支 OO 產品的 Reels」「拍一支 N 秒的品牌影片」
「做一條 OO 的 UGC 短影音」等，描述了產品/品牌與想要的成片時。

## 前置：先確認 brief 五要素
1. 產品／品牌是什麼
2. 核心賣點（一句話）
3. 目標受眾
4. 平台與比例（IG Reels / TikTok → 9:16；YouTube → 16:9）
5. 秒數與調性（溫暖／高級／搞笑／快節奏…）

缺關鍵的先問 1–2 題，不要硬生。**這是廣告，不是隨手生圖，方向錯了整條產線白燒額度。**

## 執行步驟

### 第一步：寫分鏡（把廣告拆成 N 個鏡頭）
- 依秒數拆鏡（一般每鏡 5 秒）。每一鏡用「**八層漢堡公式**」寫一段英文 prompt
  （Medium → Shot → Angle → Movement → Focus → Subject → Lighting → Color）。
- 鏡頭之間要有**敘事弧**：Hook（抓住）→ 痛點／情境 → 產品登場 → CTA。
- **先把分鏡表列給使用者確認再開生成**：

  | 鏡號 | 秒數 | 畫面 | 運鏡 | 模型 | prompt 摘要 |
  |---|---|---|---|---|---|

### 第二步：鎖角色（需要固定人物／寵物／代言貓時）
若廣告要同一個人物貫穿全片，先訓練 Soul ID（用 3–5 張參考圖）：
```bash
higgsfield soul-id create --name <角色名> --soul-2 \
  --image ./ref1.jpg --image ./ref2.jpg --image ./ref3.jpg
higgsfield soul-id wait <soul_id>
```
之後每個鏡頭生成都帶 `--soul-id <soul_id>`，整支廣告就是同一張臉。

> **授權規則**：Soul ID 只用「**本人或已明確授權的對象**」的照片
> （你自己、簽約代言人、客戶提供且授權的素材、品牌的吉祥物寵物）。
> **不得**用未授權的真實第三人照片做換臉、代言或動態。

### 第三步：鎖品牌（Marketing Studio 品牌套件）
載入品牌色、字體、語氣，讓每一鏡產出都符合品牌調性、不會跑色。
（用 `marketing-studio` 相關指令掛上 brand kit；詳細參數先 `higgsfield model get` 確認。）

### 第四步：估成本（批量生成前「必做」）
逐鏡批量生成很燒額度，開生成前先估、並跟使用者確認總額度：
```bash
higgsfield generate cost <model> \
  --prompt "<該鏡 prompt>" --aspect_ratio 9:16 --duration 5
```
把「鏡頭數 × 每鏡成本 = 總額度」算給使用者看，確認後再開生成。

### 第五步：逐鏡生成（CLI 主力指令）
- 不確定某模型支援哪些 flag／enum → **先** `higgsfield model get <model>` 看 schema。
- 生**圖**鏡頭（KV、產品圖）：
  ```bash
  higgsfield generate create nano_banana_2 \
    --prompt "<五層 prompt>" --aspect_ratio 9:16 --resolution 2k --wait
  ```
- 生**影片**鏡頭：
  ```bash
  higgsfield generate create kling3_0 \
    --prompt "<八層 prompt>" --aspect_ratio 9:16 --duration 5 \
    --soul-id <soul_id> --mode pro --sound off --wait
  ```
- 角色一致 → 帶 `--soul-id`；以某張圖為起點 → `--start-image ./shot.png`。
- `--wait` 會卡住終端機直到生成完成；長批量也可不加 `--wait`，改用
  `higgsfield generate get <job_id>` / `higgsfield generate wait <job_id>` 輪詢。

### 第六步：鏡頭接續／過場（首尾幀、影生影）
- **首尾幀補間**（讓 A 鏡尾接 B 鏡頭，過場順）：
  ```bash
  higgsfield generate create kling3_0 \
    --prompt "<過場動態>" --start-image ./A_last.png --end-image ./B_first.png \
    --aspect_ratio 9:16 --duration 5 --wait
  ```
- **影生影改風格**（draw_to_video / reframe 等 workflow）：
  ```bash
  higgsfield generate workflow reframe \
    --video ./source.mp4 --aspect-ratio 9:16 --resolution 720p --wait
  ```

### 第七步：配音／旁白（選配）
```bash
higgsfield generate create text2speech_v2 \
  --prompt "<旁白文字>" --model elevenlabs \
  --voice_type preset --voice_id <voice_id> --wait
```
（先 `higgsfield voices list` 找聲音。）

### 第八步：品質檢核（選配，交付前）
用 Virality Predictor 對成片打分，看 hook 強度／留存風險：
```bash
higgsfield generate create brain_activity --video ./ad.mp4 --wait
```

### 第九步：交付
輪詢完成 → 下載各鏡 → 回傳清單與連結，
並用一兩句中文說明這支廣告的敘事邏輯（Hook 在哪、CTA 是什麼）。

## 重要規則
- 跟使用者溝通用繁體中文，prompt 本身用英文
- **分鏡表先確認、成本先估算，再開生成**——這是廣告產線，別白燒額度
- 不確定模型參數一律先 `higgsfield model get <model>`，不硬猜 flag
- 角色一致用 `--soul-id`；鏡頭接續用 `--start-image` / `--end-image`
- 每個運鏡／光線都要有敘事理由，不堆砌關鍵字
- token 是短效的，CLI 報「session expired / not authenticated」→ 重跑 `higgsfield auth login`

## 禁止事項
- Soul ID 與生成只用「本人或已授權對象」；不得用未授權第三人做換臉／代言
- 不生成暴力、色情、仇恨內容
- 不產出虛假、誤導性的廣告宣稱（注意廣告／消保法規）
- 不堆砌無視覺功能的關鍵字
