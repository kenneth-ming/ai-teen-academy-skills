# AI 少年學院 · Skill 庫

這裡放 AI 少年學院課程中用到的所有 Hermes skill，**隨課程持續更新**。
學員不需要手動下載——在 LINE 跟自己的 Hermes 說「**幫我裝 <skill名>**」就會自動安裝。

## 🚀 怎麼安裝

1. 先確認你的 Hermes 已裝好「**Skill 安裝器**」meta-skill（`skills/skill_installer.md`，第一次要手動裝一次）。
2. 在 LINE 對 Hermes 說：
   ```
   幫我裝 ad_pipeline
   ```
3. Hermes 會自動從這個庫抓取、安裝到 `/opt/data/skills/`，裝完提醒你要不要重啟。

## 📦 Skill 清單

| Skill | 對應課程 | 功能 | 連結 |
|---|---|---|---|
| `skill_installer` | meta | 一句話裝 skill 的安裝器（先裝這個） | [檔案](ai-teen-academy-skills/skills/skill_installer.md) |
| `visual_gen` | 課程 3 | 萬能視覺生成（文字／圖 → 圖／影片） | [檔案](ai-teen-academy-skills/skills/visual_gen.md) |

> 之後每多一支 skill，就在這張表加一列、檔案丟進 `/skills/`。

## 📁 資料夾結構
- `/skills/` — 所有 skill 的**機器讀取版**（檔名 = skill 名，Hermes 從這裡抓）
- 課程影片、文字教材另放在 Skool / YouTube

## 🔄 版本
- raw 連結**永遠指向最新版**；改一次，全班下次安裝就拿到新的。
- 改動紀錄看 commit history。
- 小提醒：GitHub raw 有約 5 分鐘 CDN 快取，剛更新完可能要等幾分鐘才抓得到最新版。

## ⚠️ 使用規範
- 這些 skill 僅供學員在自己的 Hermes 環境學習、實作使用。
- Soul ID／真人素材只用「本人或已授權對象」，請勿用於未授權的第三人。
