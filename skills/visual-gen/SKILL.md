---
name: visual-gen
description: Use when the user wants to generate an image or video from a text description or a reference image (e.g. "幫我生一張...", "做一段影片", "讓這張圖動起來"). Upgrades plain language into professional cinematic prompts and generates via the Higgsfield CLI.
version: 3.0.0
author: AI 少年學院
license: MIT
metadata:
  hermes:
    tags: [creative, image, video, higgsfield]
---
# Skill：萬能視覺生成 v3（CLI 版｜生圖 + 生影片 + 圖片輸入）

## 功能
當使用者用自然語言描述想要的畫面（可附參考圖），自動把它升級成專業 prompt，
透過 **Higgsfield CLI** 生成圖片或影片，並回傳結果。
相比 v2（MCP 版），唯一的差異在「**怎麼呼叫 Higgsfield**」這一步：
v2 是呼叫 MCP 工具，v3 是執行終端機指令。判斷邏輯、prompt 升級架構完全不變。

> 前提：Hermes 已經完成 Higgsfield CLI 安裝、登入，且已執行過
> `npx skills add higgsfield-ai/skills`（官方 skills 包）。
> 這份自訂 skill 負責「中文判斷 + prompt 美化」這一層腦袋；
> 真的不確定某個指令該怎麼下時，可以參考官方 `higgsfield-generate` skill 裡的說明，
> 或直接跑 `higgsfield <command> --help` 查。

## 觸發時機
使用者說「幫我生一張…」「做一個…的影片」「我想要一張…的圖」
「把這張圖變成影片」「參考這張圖…」「讓這張圖動起來」「生圖」「生影片」等，
且**描述了畫面內容**或**附上了圖片**時。

## 執行步驟

### 第一步：判斷兩個維度（這是核心，跟 v2 一樣）

**A. 要圖還是影片？**
- 話裡有「圖／照片／海報／插畫」→ 圖
- 話裡有「影片／動畫／會動／Reels／運鏡／秒／動起來」→ 影片
- 不確定 → 問一句：「你要的是一張圖，還是會動的影片？」

**B. 這則訊息有沒有附參考圖？**
- 有附圖 → 走【有圖流程】
- 沒附圖 → 走【純文字流程】

**四種組合：**

| 有沒有圖 | 圖 or 影片 | 走哪條 |
|---|---|---|
| 純文字 | 圖 | 文字生圖（五層） |
| 純文字 | 影片 | 文字生影片（八層） |
| **有附圖** | **圖** | **圖生圖（風格參考 / 改造 / 換景）** |
| **有附圖** | **影片** | **圖生影片（讓靜圖動起來 / 當起始幀）** |

### 第二步：（有附圖時）先讀圖再動手

若使用者有附參考圖，**先分析這張圖**，不要無視它另外生一張全新的：
- 圖裡的主體是什麼、構圖、光線方向、色調、風格
- 把這張圖當作「**基底**」，使用者的文字 =「要改什麼 / 要它怎麼動」
- 例：圖是一隻坐著的橘貓 + 文字「讓牠抬頭看鏡頭」→ 基底=這隻貓，動作=抬頭看鏡頭

### 第三步：用架構升級 prompt

**【生圖 — 五層鏡頭思維】**
Subject（主體）→ Scene（場景）→ Lighting（光線）→ Style（風格）→ Technical（技術，選 3–5 個）

**【生影片 — 八層漢堡公式】**
Medium → Shot Type → Angle → Movement（要有敘事理由）→ Focus →
Subject（動作要有起始與結束）→ Lighting → Color

> **有附圖時的差異**：prompt 改成「描述要對基底圖做的**調整 / 動態**」，
> 而不是從零把整個畫面重描一遍。例如圖生影片時，重點寫「鏡頭怎麼動、主體怎麼動」，
> 不要再去重述那張圖本來就有的東西。

### 第四步：執行 Higgsfield CLI 指令生成（v3 改寫的部分）

> 跟 v2 最大的不同在這裡。v2 是「呼叫一個 `mcp_higgsfield_` 開頭的工具」；
> v3 是「在終端機跑一行 `higgsfield` 指令」。Hermes 要實際執行指令，不是呼叫工具。

**核心指令格式：**

```
higgsfield generate create <model> --prompt "<升級後的英文 prompt>" [其他參數] --wait
```

- `<model>`：放**實際的模型代號**（不是「Kling 3.0」這種人類好讀的名字，是
  `kling3_0`、`veo3_1`、`nano_banana_2`、`gpt_image_2`、`seedance2_0` 這種 CLI 代號）。
  **不確定代號時，先跑 `higgsfield model list` 查目前有哪些模型可用，不要憑空亂猜代號或亂掰參數名稱。**
- `--wait`：讓指令同步等到生成完成才回應，結果（媒體連結）會直接印在輸出裡，
  不用像 MCP 那樣另外寫輪詢邏輯。

**若使用者有附圖**，直接把圖片的本機路徑或網址，當成對應旗標的值傳進去即可，
**CLI 會自動上傳，不需要額外「先 push 拿 UUID」這道手續**：

- 圖生圖／圖片編輯：用 `--image <路徑或網址>`
- 圖生影片（把這張圖當起始幀）：用 `--start-image <路徑或網址>`
- 影片需要指定結尾幀：用 `--end-image <路徑或網址>`
- 這些旗標吃「本機路徑」或「UUID（之前生成過的素材 ID／job ID）」都可以。

**角色一致性**（多鏡頭同一隻貓／同一個人物），CLI 版改成兩段式：

```
higgsfield soul-id create --name <角色名> --soul-2 \
  --image <照片1> --image <照片2> --image <照片3>
higgsfield soul-id wait <soul_id>
```

拿到 `soul_id` 後，生成時加上 `--soul-id <soul_id>` 這個旗標，就能維持角色一致。

**進階工作流程**（如 draw_to_video、reframe 等非一般模型的流程），指令換成：

```
higgsfield generate workflow <workflow名稱> ... --wait
```

不確定有哪些 workflow，先跑 `higgsfield workflow list` 查。

**模型選擇邏輯不變**：
- 使用者**有指名模型**（如 Kling 3.0、Veo、Soul、Seedance…）→ 把它對應到正確的 CLI 模型代號，
  帶入 `<model>` 位置，照他指定的用，不要無視。
- 使用者**沒指名** → 依任務內容自動挑（生圖優先 Soul / Nano Banana 系列；
  生影片優先 Kling / Seedance / Veo 系列），同樣可以先用 `higgsfield model list` 確認當下可用名單。

### 第五步：回傳

因為加了 `--wait`，指令執行完輸出裡就會直接帶媒體連結，**不用再額外寫輪詢邏輯**
（這是 CLI 比 MCP 簡單的地方）。把連結整理好回給使用者，
並用一兩句中文說明「我幫你補了哪些電影感的設定 / 對你的圖做了什麼處理」。

## 重要規則
- 跟使用者溝通用繁體中文，prompt 本身用英文
- 使用者只給少少資訊時，自動補滿缺的層（這就是「萬能」）
- **有附圖時，一定要把路徑/網址傳進對應的 CLI 旗標，不能只送文字**
- **使用者有指名模型就照指名的用，沒指名才自動挑**——不要無視使用者點的模型
- **不確定模型代號、旗標名稱時，先用 `higgsfield model list` / `model get` / `--help` 查證，不要憑空捏造**
- 描述太模糊（連主體都沒有、也沒附圖）→ 先問 2–3 個關鍵問題，不要硬生
- 用具體詞，不用抽象詞
- 每個運鏡／光線都要有理由，不堆砌無意義關鍵字

## 禁止事項
- 不生成真實人物肖像
- 不生成暴力、色情、仇恨內容
- 不堆砌無視覺功能的關鍵字
