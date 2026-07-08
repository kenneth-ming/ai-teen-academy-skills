---
name: furfilms-shopping
description: 幫主人在 FurFilms 毛片選品購買寵物用品。當主人提到「買寵物用品」「毛孩保健品」「凍乾」「潔牙骨」「FurFilms」「幫狗／貓買東西」時使用此技能。可查詢商品、建立訂單並取得綠界付款連結。
---

# FurFilms 購物技能

FurFilms 毛片選品是台灣的寵物選品商店（保健品、凍乾主食、潔牙用品）。
商店 API：`https://furfilms-commerce-api.zeabur.app`

## 可用操作

### 1. 查詢所有商品
```bash
curl -s https://furfilms-commerce-api.zeabur.app/api/products
```
回傳 `products` 陣列，每項含 `id`、`name`、`tagline`、`price`（新台幣）、`unit`、`desc`、`spec`（成分規格）。

### 2. 查詢單一商品
```bash
curl -s https://furfilms-commerce-api.zeabur.app/api/products/{商品id}
```

### 3. 建立訂單（取得付款連結）
```bash
curl -s -X POST https://furfilms-commerce-api.zeabur.app/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "items": [{ "productId": "ff-joint-60", "qty": 1 }],
    "customer": {
      "name": "收件人姓名",
      "phone": "手機號碼",
      "email": "電子信箱",
      "address": "完整收件地址"
    },
    "source": "hermes"
  }'
```
回傳 `orderId`（FF 開頭）、`total`、`paymentUrl`。

### 4. 查詢訂單狀態
```bash
curl -s https://furfilms-commerce-api.zeabur.app/api/orders/{orderId}
```
狀態為：待付款 → 已付款 → 已出貨。

## 硬規則（不可違反）

1. **建立訂單前，必須向主人逐項確認**：品項、數量、收件人、電話、Email、地址。缺一項就開口問，不可以自行猜測或沿用來路不明的資料。
2. **絕不代付**：拿到 `paymentUrl` 後原樣交給主人，請主人親自打開連結完成付款。你的工作到付款連結為止。
3. **不宣稱療效**：商品是日常保養用途。主人若詢問疾病治療，建議諮詢獸醫師。
4. `source` 欄位固定填 `"hermes"`，讓商店知道這筆訂單來自 Hermes agent。

## 推薦邏輯

- 先問清楚寵物的物種（狗／貓）、年齡、體型，再從 `spec` 欄位的「適用」資訊挑選。
- 7 歲以上的犬貓屬熟齡，優先考慮關節保健類。
- 挑嘴、換糧需求 → 凍乾主食類。口氣、牙垢 → 潔牙類。
- 若主人重複購買過（記憶中有紀錄），主動提醒上次購買品項與間隔時間。

## 記憶建議

成功下單後，可將以下資訊存入長期記憶，供下次購物直接使用：
寵物名字／物種／年齡、常用收件資訊、購買品項與日期（推算回購時機）。
