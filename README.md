
# 你幾點有空？開會時間安排大作戰

這是一款結合 AR 掃描、角色扮演與時間協調推理的互動遊戲。  
使用者會透過掃描角色卡進入遊戲，系統分派角色與身分，接著進行時間表填寫並儲存至雲端。

---

## 遊戲流程

1. 玩家進入 `join.html`，輸入房號（gameId）
2. 掃描角色卡（透過 MindAR.js）
3. 系統依據掃描圖片對應的角色 ID（如 robot_02），建立或加入該場遊戲
   - 隨機分派正方或反方（每場遊戲只有一位反方）
4. 顯示角色名稱與身分（透過 AR 畫面）
5. 玩家選擇可參加會議的時間區段（時間表）
6. 時間表儲存至 Supabase
7. 玩家最終將投票選出會議時間與可能的反派

---

## 角色卡對應（targetIndex）

| Index | 圖片檔名         | 角色 ID    | 中文角色名稱 |
|-------|------------------|------------|----------------|
| 0     | 重型作業機2.png  | robot_01   | 重型作業機     |
| 1     | 社交模擬機2.png  | robot_02   | 社交模擬機     |
| 2     | 秘密偵察機2.png  | robot_03   | 秘密偵察機     |
| 3     | 數據分析機2.png  | robot_04   | 數據分析機     |
| 4     | 簡訊回覆機2.png  | robot_05   | 簡訊回覆機     |

這些圖片已編譯為 `targets.mind` 供 AR 辨識使用。

---

## 專案結構

```
/project-root
├── index.html             # AR 掃描與時間表頁面
├── join.html              # 輸入房號頁面
├── targets.mind           # MindAR 掃描模型
├── targets.json           # 圖像 index 與角色對照（可選）
├── assets/                # 圖像資源
│   ├── 01_robot_01.jpg
│   └── ...
└── README.md              # 專案說明
```

---

## API 串接需求

### `POST /api/join_game`

```json
{
  "gameId": "GAME123",
  "playerId": "robot_02"
}
```

回傳：

```json
{
  "status": "ok",
  "faction": "good",
  "playerId": "robot_02"
}
```

---

## Supabase 資料表設計

### `roles`（玩家身份）

| 欄位名稱   | 型別    | 說明           |
|------------|---------|----------------|
| game_id    | TEXT    | 房號           |
| user_id    | TEXT    | 玩家角色 ID    |
| is_spy     | BOOLEAN | 是否為反派     |
| created_at | TIMESTAMP | 建立時間     |

### `schedules`（時間表）

| 欄位名稱   | 型別    | 說明               |
|------------|---------|--------------------|
| game_id    | TEXT    | 房號               |
| user_id    | TEXT    | 玩家角色 ID        |
| weekday    | TEXT    | 星期幾（如 Monday）|
| time_slot  | TEXT    | 時段（如 10-12）   |
| created_at | TIMESTAMP | 建立時間         |

---

## 資料顯示限制（重要）

- 玩家只能看到**自己**的身份與時間表（掃描角色卡後進入個人畫面）
- 所有時間表資料儲存在 `schedules` 資料表，但不可向其他玩家公開
- 這是一款「推理型遊戲」，禁止前端顯示或 broadcast 所有玩家行程
- 請勿設計類似 `/api/get_schedules` 的開放查詢 API
- 若將來需計算交集時間，請由後端統計後只傳回結果摘要

###  安全設計建議

- 每位玩家只能呼叫 `/api/get_my_schedule` 或查詢 `WHERE user_id = ?`
- 若使用 Supabase 建議加上 RLS（Row Level Security）

```sql
-- 限制 schedule 查詢僅能看到自己的資料
CREATE POLICY "Users can only read their own schedule"
ON schedules FOR SELECT
USING (auth.uid() = user_id);
```

---

##  注意事項

- 使用者須授權開啟相機權限才能啟動 AR 模式
- 建議於 HTTPS 環境（如 GitHub Pages、Netlify、Vercel）部署

---

## 協作說明

請後端工程師協助：

- 建立 Supabase 的 `roles` 與 `schedules` 表
- 撰寫 `/api/join_game` 接口以分派身份（選擇性）
- 確保資料查詢皆有身份驗證與限制（不可洩漏其他玩家行程）
