# 你幾點有空？開會時間安排大作戰

這是一款結合 AR 掃描、行程推理與角色扮演的桌遊網頁前端。  
使用者會透過掃描角色卡，進行身分分派並進入互動排程流程。

---

## 專案簡介

- 使用 MindAR.js 掃描實體角色卡（5 張角色）
- 掃描後傳送 playerId + gameId 到後端
- 後端分派玩家身份（正派/反派）並回傳
- 每場遊戲以 gameId 隔離資料，支援多房同時進行

---

## 專案結構

```
robot-meeting-game/
├── index.html             # 掃描角色卡頁面
├── join.html              # 輸入房號頁面（可選）
├── targets.mind           # MindAR 掃描模型檔案
├── targets.json           # MindAR 模型 metadata
├── assets/                # 角色圖片資源
│   ├── robot_01.png
│   ├── robot_02.png
│   └── ...
└── README.md              # 工程師協作說明文件
```

## API 串接需求

### `POST /api/join_game`

前端傳送：

```json
{
  "gameId": "GAME123",
  "playerId": "robot_03"
}
```

後端回傳：

```json
{
  "status": "ok",
  "faction": "good",  // 或 "evil"
  "playerId": "robot_03"
}
```

規則：
- 若該玩家已存在於此場遊戲，回傳已分派身分
- 若首次加入，隨機分派（預設每場一位反派）

---

## Supabase 資料表設計建議

資料表名稱：`players`

| 欄位名稱     | 型別      | 說明                      |
|--------------|-----------|---------------------------|
| id           | UUID      | 主鍵，自動產生            |
| game_id      | TEXT      | 遊戲房號                  |
| player_id    | TEXT      | 機器人角色 ID             |
| faction      | TEXT      | "good" / "evil"           |
| has_scanned  | BOOLEAN   | 是否已掃描（預設 true）   |
| schedule     | JSONB     | 行程資料（可選填）        |
| created_at   | TIMESTAMP | 自動產生時間戳記          |

可加上以下索引：

```sql
CREATE INDEX idx_game_id ON public.players (game_id);
CREATE INDEX idx_player_id ON public.players (player_id);
```

---

## API 延伸建議

### `GET /api/get_players?gameId=GAME123`

用於查詢某場遊戲的玩家與身分（如 DEBUG 回合）。

回傳：

```json
[
  { "playerId": "robot_01", "faction": "good" },
  { "playerId": "robot_02", "faction": "evil" }
]
```

---

## 前端操作流程

1. 玩家輸入房號（gameId）
2. 掃描角色卡（取得 playerId）
3. 呼叫 `/api/join_game` 拿到身分
4. 顯示身分動畫／文字提示
5. 進入行程排程階段（未來擴充）

---

## 協作說明

請後端協助：

- 以建立上述 Supabase 資料表，要麻煩協助索引
- 撰寫 `/api/join_game` 和 `/api/get_players` 的處理邏輯
- 確保每場 gameId 的資料獨立、互不干擾

---

任何問題可由前端使用者提出具體參數、畫面流程圖協助協調。  
預期支援單場 4~5 人遊戲，每人使用各自裝置。
