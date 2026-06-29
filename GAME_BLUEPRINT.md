# 數學奇航 遊戲藍圖（GAME_BLUEPRINT.md）

> **這份文件的用途**：把「數學奇航」這款遊戲拆解成**可重用的骨架 + 可抽換的內容**。
> 之後要做「玩法類似、但主題/學科/呈現不同」的新遊戲時，Claude Code 與你都以本文件為設計藍本：
> **固定沿用的骨架**（帳號、資料庫、同步、橋接）直接複製；**三顆旋鈕**（主題美術、學科題目、玩法機制）按新需求抽換。
>
> 文件分兩段：
> - **PART 1｜白話總覽** — 給你（非技術）快速看懂全貌與「怎麼換皮」。
> - **PART 2｜技術規格** — 給未來的 Claude Code 照著重建用，含函式名、資料表、狀態變數、可抽換點。
>
> 對應的程式：`index.html`（單一檔，約 6,100 行）。本文件提到的行號是撰寫當下的概略位置，改動後以函式名為準。

---

# PART 1｜白話總覽

## 1.1 一句話核心循環

> **孩子是「星界召喚師」，在限時關卡中答數學題 → 答對淨化迷霧、修復星球、賺星星與拼圖 → 集滿拼圖喚醒守護神 → 解鎖下一區域，並用星星跟家長兌換實體獎勵。**

基調是**修復與守護**（不是打殺）：答對 = 世界變美、能量恢復；答錯 = 溫柔鼓勵再試，不懲罰。

## 1.2 玩家旅程地圖

```
開啟遊戲
  → 登入 / 註冊（或從家長平台帶 token 自動登入）
  → 開場動畫（小星引導世界觀）
  → 選性別（決定主角頭像與「少年/少女」稱呼）
  → 標題主畫面（顯示主角、星星數、各功能入口）
  → 世界地圖（選星球）
  → 關卡選擇（依運算類型分區，需逐關解鎖）
  → 進入關卡：倒數 3-2-1 → 區域過場對話（跨區時）→ 關卡劇情 → 答題
        每答對：粒子特效＋能量瓶上升＋世界修復＋連擊＋專屬鼓勵語
        每答錯：迷霧效果＋安慰語，扣血但可再試
  → 關卡結算：評等 S/A/B/C、得星星、掉拼圖碎片
  → （集滿該星球拼圖）守護神甦醒演出
  → 回主畫面 → 社交（好友、即時對戰、排行榜）/ 每日任務 / 商店兌換 / 帳號設定
```

## 1.3 功能模組總覽（四大類）

| 類別 | 模組 | 一句話 | 換新遊戲時 |
|---|---|---|---|
| **🦴 核心骨架**（固定沿用） | 帳號系統 | Supabase 註冊/登入/密碼重設 | 幾乎不動 |
| | 資料同步 | 進度雲端存檔 + 本地 localStorage 雙寫 | 幾乎不動 |
| | 家長平台橋接 | 從家長端帶 token 進來自動登入 | 幾乎不動 |
| | 導覽系統 | 25 個畫面用 `goTo()` 切換 | 沿用，換畫面內容 |
| **📚 學習內容**（旋鈕②） | 關卡引擎 | 關卡定義、解鎖鏈、出題、計分評等 | **抽換題庫與難度** |
| | 場景/區域 | 每關對應的世界/區域/美術 | **抽換主題地圖** |
| | 劇情對話 | 開場、區域過場、關卡劇情、結算 | **重寫文案** |
| **🎮 互動社交**（旋鈕③，可選） | 拼圖收集 | 過關掉碎片，集滿喚醒守護神 | 可保留或改成別的收集物 |
| | 每日任務 | 每天指定目標給獎勵 | 沿用，改任務內容 |
| | 連續登入 | 連續登入階梯獎勵 | 沿用 |
| | 排行榜 | 全服分數排名 | 沿用 |
| | 好友系統 | 加好友、鼓勵訊息 | 沿用 |
| | 即時對戰 | 雙人即時答題對戰 | 沿用或關閉 |
| **💰 商業化**（可選） | 星星貨幣 | 過關賺星星 | 沿用 |
| | 商店兌換 | 星星換實體獎勵兌換碼 | 沿用 |
| | 家長模式 | 內建家長儀表板（進度/能力/報告/留言） | 沿用 |
| | Admin 後台 | 管理獎勵品項與兌換碼 | 沿用 |

## 1.4 三顆可抽換旋鈕（這就是「換皮」的核心）

做新遊戲時，你主要只動這三件事，骨架不用重寫：

**🎨 旋鈕①：主題與美術**
- 世界觀（星界召喚 → 例如：海洋探險、魔法學院、太空任務）
- 角色與守護神名稱、星球/區域名稱
- 圖片資產（`images/` 內的 dead/alive 場景圖、角色頭像、精靈圖）
- 配色（深色星空 + 橙色 → 換成新主題色）

**📚 旋鈕②：學科與題目**
- 把「數學出題」換成新學科（英文單字、成語、自然科…）
- 改的是：題目產生邏輯（`genQ`）、關卡定義（`LEVELS`）、運算分組（`OPS`）、答題輸入 UI（數字鍵盤 → 可能要改成選擇題、拼字…）

**🕹️ 旋鈕③：玩法機制**
- 哪些社交/收集/對戰模組要保留、移除或改設計
- 關卡結構（18 關線性解鎖 → 可改成開放式、分支、無限刷）
- 獎勵數值與評等規則

## 1.5 換皮做新遊戲：最快路徑（概念版）

1. 複製整個 `index.html` → 新專案
2. **保留不動**：帳號（`initAuth`/`authSignIn`…）、同步（`syncFromCloud`…）、橋接協定、Supabase 資料表結構
3. **換旋鈕②**：改 `LEVELS`、`OPS`、`genQ`、答題輸入 UI
4. **換旋鈕①**：換 `images/`、`SCENE_MAP`、配色 CSS、所有對話文案常數
5. **調旋鈕③**：決定保留/移除哪些模組（拼圖、對戰、商店…）
6. 沿用同一個 Supabase 專案或新建一個（表結構照搬）
7. 照 PART 2 第 7 章的「抽象化檢查表」逐項確認

> 技術細節、每個模組「改哪個函式、碰哪張表」都在 PART 2。

---

# PART 2｜技術規格

## 2.0 技術棧與檔案結構

| 項目 | 內容 |
|---|---|
| 前端 | **單一 `index.html`**，原生 HTML/CSS/JS，無框架、無 build。所有畫面、樣式、邏輯都在同一檔 |
| 後端 | **Supabase**（Auth + Postgres + Realtime）。透過 `@supabase/supabase-js` client（變數 `db`） |
| 即時 | Supabase Realtime（對戰房、對戰邀請用 channel 訂閱） |
| 本地儲存 | `localStorage`（離線進度、設定、性別、星星），與雲端雙寫 |
| 部署 | GitHub → Vercel 自動部署（push 到 `main` 即上線）；`vercel.json` 設定 |
| 連線常數 | `SUPA_URL`、`SUPA_KEY`（publishable/anon key，可公開）、`ADMIN_ID`（管理員 user id） |

**單一檔的程式分區（由上而下）**：CSS → 各畫面 HTML（`<div id="s-...">`）→ JS：連線/Auth → 資料同步 → 關卡引擎常數 → SVG 角色 → 音效 → 出題/計分 → 視覺特效 → 劇情對話 → 關卡流程 → 結算 → 導覽 → 各功能模組（每日任務、排行榜、好友、對戰、商店、Admin、家長模式）→ 啟動。

> ⚠️ **單一檔最大地雷**：同名函式重複定義時，後者覆蓋前者，造成「改了沒效果」。改動時務必 `grep -c "function 名稱"` 確認唯一。

## 2.1 資料模型

### 2.1.1 Supabase 資料表

| 表 / View | 用途 | 關鍵欄位（推斷） |
|---|---|---|
| `players` | 玩家主檔（= auth user） | `id`(=auth uid), `email`, `display_name`, `avatar`, `star_fragments`, `is_online`, `last_seen` |
| `children` | 子帳號資料（家長平台建立的孩子） | `id`, `nickname`, `grade`, 對應 player |
| `level_progress` | 每關進度 | `player_id`, `level_id`, `score`, `grade`, `treasure` |
| `child_stats` | 孩子統計彙整 | `player_id`, 星星、累計等 |
| `answer_logs` | 每題作答紀錄（供能力分析） | `player_id`, `level_id`, `question`, `answer`, `is_correct`, `time_spent` |
| `friendships` | 好友關係 | `requester_id`, `receiver_id`, `status`(pending/accepted) |
| `encouragements` | 好友鼓勵訊息 | `sender_id`, `receiver_id`, `message`, `read` |
| `battle_rooms` | 即時對戰房 | `id`, 雙方 id、題目、雙方答題狀態、勝負 |
| `battle_answers` | 對戰每題作答 | `room_id`, `player_id`, 答案 |
| `shop_items` | 商店獎勵品項 | `id`, `name`, `desc`, `stars_required`, `badge_required`, `active` |
| `shop_codes` | 兌換碼庫存 | `item_id`, `code`, 是否已用 |
| `redemptions` | 兌換紀錄 | `player_id`, `item_id`, `code`, `season` |
| `parent_child_links` | 家長↔孩子綁定 | `parent_id`, `child_player_id` |
| `parent_messages` | 家長給孩子的留言 | `parent_id`, `child_id`, `text` |
| `public_leaderboard`（View） | 排行榜（聚合 players 分數） | 唯讀視圖 |

> 寫新遊戲時這 14 張表**結構可整套沿用**（旋鈕②/①不影響 schema）。只有當玩法機制大改（旋鈕③，例如移除對戰）才需刪表。

### 2.1.2 localStorage（離線層，物件 `SS`，行 2083）

| key | 用途 |
|---|---|
| `mc_stars` | 星星碎片（離線） |
| `mc_lv_<id>` | 某關最佳紀錄（存在 = 已通關 = 解鎖下一關依據） |
| `mc_tr` | 已獲得寶物 id 陣列 |
| `player_gender` | `boy` / `girl`，決定頭像與稱呼 |
| `sound_on` | 音效開關 |
| （每日任務、連續登入 key） | 見對應模組 |

`SS` 提供 `stars/addStars/lv/saveLv/unlocked/treasures/hasTr/addTr`。`unlocked(id)` 用 `UNLOCK_CHAIN` 判斷前一關是否通關。

## 2.2 畫面清單（25 個 `s-*`，用 `goTo(id)` 切換）

| id | 畫面 | 職責 |
|---|---|---|
| `s-auth` | 登入/註冊 | Email 密碼、切換 tab |
| `s-reset-password` | 重設密碼 | recovery token 進來後 |
| `s-opening` | 開場動畫 | 世界觀劇情（`OPENING_LINES`） |
| `s-choose-gender` | 選性別 | `chooseGender()` |
| `s-title` | 標題主畫面 | 主角頭像、星星、各功能入口 |
| `s-planet` | 世界地圖 | 選星球 |
| `s-world` | 世界/區域中介 | （導覽中繼） |
| `s-levels` | 關卡選擇 | 依 `OPS` 分區、解鎖狀態 |
| `s-intro` | 關卡前介紹 | 關卡名、魔物名 |
| `s-game` | **答題主畫面** | HUD、場景、數字鍵盤、能量瓶 |
| `s-celebrate` | 過關慶祝 | `CELEB_DATA` 里程碑 |
| `s-win` / `s-lose` | 勝/敗結算 | 評等、星星、再挑戰 |
| `s-collection` | 收藏 | 拼圖、守護神圖鑑 |
| `s-shop` | 商店 | 星星兌換實體獎勵 |
| `s-admin` | 管理後台 | 品項/兌換碼/兌換紀錄 |
| `s-parent` | 家長模式 | 儀表板（多 tab） |
| `s-mailbox` | 信箱 | 家長留言 |
| `s-leaderboard` | 排行榜 | 全服排名 |
| `s-friends` | 好友 | 好友/待確認/新增/對戰紀錄 |
| `s-tasks` | 每日任務 | 任務清單與獎勵 |
| `s-account` | 帳號設定 | 基本/安全/個人化/成就 4 tab |
| `s-battle-waiting` | 對戰等待 | 配對中 |
| `s-battle` | 對戰中 | 雙人即時答題 |
| `s-battle-result` | 對戰結算 | 勝負、星星 |

導覽核心：`goTo(id)`（行 3590）切換顯示，並在進入 `s-title` 時 `updateTitleHero()`、進入 `s-planet` 時重置 `lastZone`。`renderTopBar()` 渲染所有畫面頂部狀態列。

## 2.3 核心引擎（旋鈕②的主戰場）

### 2.3.1 關卡定義 `LEVELS`（行 2048）— **可抽換內容資料**
每關一個物件：
```js
'1': { op:'+', name:'一位數＋一位數', time:90,
       sc:[[3,100],[5,90],[7,80],[8,70],[9,60]],  // [秒數上限,得分] 越快越高分
       df:50,                                       // 超時的保底分
       S:900, A:800, B:700,                         // 累積分評等門檻（S/A/B，其餘 C）
       mon:'imp', mnm:'火焰惡魔',                    // 魔物（迷霧化身）
       tr:{i:'🏅', n:'火焰徽章'} },                  // 過關寶物
```
共 18 關（旋鈕②：換成你的學科時，改這張表的 `name`/`mon`/`tr` 與難度數值）。

### 2.3.2 運算分組 `OPS`（行 2068）— **可抽換**
```js
{ id:'add', name:'加法', icon:'＋', ids:['1','2','3','4','5'] }
```
4 組（加/減/乘/除），對應關卡選擇畫面的分區。新學科改成你的單元分組。

### 2.3.3 解鎖鏈 `UNLOCK_CHAIN`（行 2074）
線性 1→18。`SS.unlocked(id)` 判斷前一關通關才開放。要改成開放式/分支關卡時動這裡。

### 2.3.4 出題 `genQ(id)`（行 2438）— **旋鈕②核心**
`switch(id)` 為每關量身產生題目，回傳 `{text, ans}`。
- 例：`case'1': a=r(1,9);b=r(1,9);ans=a+b;text=\`${a} ＋ ${b} = ?\``
- 除法關卡（15-18）有「保證整除/限定範圍」的特殊枚舉邏輯。
> 換學科時整個 `genQ` 重寫：例如英文單字題回傳 `{text:'蘋果', ans:'apple'}`，並把答題 UI 從數字鍵盤改成拼字/選擇。

### 2.3.5 計分與評等（行 2469）
- `calcPts(id, secs, wrongs)`：依 `sc` 門檻取基礎分，每錯一次 `-50`，最低 0。
- `gradeV(g)`：S=4/A=3/B=2/C=1（排行榜/能力換算用）。
- 評等門檻在每關的 `S/A/B`。

### 2.3.6 場景/區域 `SCENE_MAP`（行 2845）— **旋鈕①核心**
每關 → `{ dead, alive, zone }`：迷霧（dead）圖、淨化後（alive）圖、區域名。
答對時 `scene-alive` 淡入覆蓋 `scene-dead`，視覺上「世界被修復」。
- `getPlanetType(id)`（行 2493）：1-8 burning / 9-16 ocean / 17-24 thunder / 25-32 moon / >32 hall——**已預留 5 星球擴充位**。
> 換主題時換 `images/` 與 `zone` 名稱即可，邏輯不動。

### 2.3.7 劇情對話常數 — **旋鈕①（文案）**
| 常數 | 用途 | 行 |
|---|---|---|
| `OPENING_LINES` | 開場世界觀 | 2740 |
| `ZONE_DIALOGS` | 跨區域過場（key=區域名） | 2866 |
| `LEVEL_DIALOGS` | 特定關卡劇情（key=關卡 id） | 3066 |
| `CELEB_DATA` | 里程碑慶祝（如第 14/18 關） | 3511 |
| `zoneEncourage` | 各區域答對專屬鼓勵語 | 2882 |
| `GAME_ENCOURAGE_MSGS` / `COMFORT_MSGS` / `FAIL_MSGS` | 通用鼓勵/安慰/失敗語 | 2664/2679/2075 |
通用對話引擎 `showDialog(lines)`（行 3092）逐句播放（頭像＋名字＋打字機）。換主題只改文案常數。

### 2.3.8 收集系統 `PLANET_PUZZLES`（行 2891）
每星球一組：`{ name, levels:[...], spiritImg, spiritName }`。過關掉 1 片拼圖，集滿該星球所有關卡 → 守護神甦醒（`showSpiritAwaken`）。`renderPuzzle()`/`renderSpiritGallery()` 渲染收藏畫面。

## 2.4 登入與資料同步（🦴 骨架，固定沿用）

### 2.4.1 `initAuth()`（行 1589）— 啟動入口，處理三種進場
1. **密碼重設**：URL hash 含 `type=recovery` + `access_token` → 設 session → 顯示重設畫面。
2. **家長平台橋接（重要協定，勿動）**：query 含 `?type=child&at=<token>&rt=<token>` → `db.auth.setSession({access_token:at, refresh_token:rt})` → 清掉 URL → 孩子自動登入。
3. **一般 session**：有 session → `loadPlayerData` → `loadOrCreateChild` → `syncFromCloud` → `checkLoginStreak` → `initDailyTasks` → `checkBattleInvites` → `subscribeToBattleInvites` →（管理員顯示後台鈕）→ `afterLogin`；無 session → `goTo('s-auth')`。

> ⚠️ 平台 iframe 依賴 `initAuth()` 與 `?type=child&at=&rt=` 參數協定，**新遊戲若也要接家長平台，必須保留這組協定原樣**。

### 2.4.2 帳號函式
`authSignUp`/`authSignIn`/`authSignOut`、`showForgotPassword`/`doResetPassword`、`translateError`（Supabase 錯誤訊息中文化）、`switchAuthTab`。`loadPlayerData`（行 1634）：查無 `players` 列就自動建一筆。

### 2.4.3 同步
- `syncFromLocal` / `syncFromCloud` / `forceFullSync`：localStorage ↔ Supabase 雙向。
- `logAnswer`（行 2007）：每題寫 `answer_logs`（供家長能力分析）。
- `saveProgressToCloud`（行 2023）：過關寫 `level_progress` + `child_stats`。

## 2.5 各功能模組規格

> 格式：**模組** — 關鍵函式 ｜ 資料表 ｜ 可抽換點 / 備註

- **關卡流程** — `startLevel`(3176) → `runCountdown` → `setupGame` →（區域過場）→ 關卡劇情 → `nextQ`/`submit`/`handleInput` → `endGame`(3397) → `showCelebration`/`s-win`/`s-lose`。狀態 `GS`（行 2481，每關重置：血量、題序、分數、連擊、答錯數）。
- **答題輸入** — `handleInput(v)`(3715) 數字鍵盤。｜旋鈕②：換學科要改這裡的輸入型態。
- **視覺特效** — `spawnParticles`/`animateParticles`（粒子）、`updateWorldRestore`（迷霧消退）、`updateEnergyBottle`/`resetEnergyBottle`（能量瓶）、`showSceneText`/`showHeroGlow`/`showWrongFogEffect`/`showComboPopup`/`showSpiritAwaken`。
- **音效** — `ac`/`unlockAudio`/`tone`/`sfx(n)`（WebAudio 合成，無音檔），受 `sound_on` 控制。
- **星星貨幣** — `addStarFragments`(3757)；顯示 `refreshStarsDisplay`。
- **每日任務** — `initDailyTasks`/`updateDailyTask(type,value)`/`checkDailyTaskRewards`/`renderTasksScreen`。｜localStorage 計數 + 獎勵星星。
- **連續登入** — `checkLoginStreak`(3850)：每日首登階梯獎勵。
- **排行榜** — `loadLeaderboard`/`renderLeaderboard`｜View `public_leaderboard`。
- **好友系統** — `loadFriendsScreen`、`doSearchPlayer`/`searchPlayer`（暱稱或 8 碼 ID）、`sendFriendRequest`/`acceptFriend`/`declineFriend`/`deleteFriend`、鼓勵 `sendEncouragement`/`loadEncouragements`/`markEncouragementRead`、`loadBattleHistory`｜`friendships`/`encouragements`/`players`｜`isPlayerOnline` 用 `last_seen` 判線上。
- **即時對戰（PvP）** — `challengeFriend`→`acceptBattle`→`subscribeToBattleRoom`(Realtime)→`startBattle`→`syncBattleQuestion`→`submitBattleAnswer`→`tryAdvanceQuestion`→`finishBattle`→`showBattleResult`；邀請 `checkBattleInvites`/`subscribeToBattleInvites`/`showBattleInvitePopup`/`acceptBattleInvite`。題目 `generateBattleQuestions(seed,...)`（同 seed 雙方同題）｜`battle_rooms`/`battle_answers`｜獎勵：勝 +10⭐、平 +3⭐。
- **帳號設定** — `openAccountScreen`/`switchAccTab`、`updateDisplayName`/`updatePassword`/`sendPasswordReset`/`requestDeleteAccount`、`selectAvatar`（8 頭像 `AVATAR_MAP`）、`toggleSound`、`renderAchievements`、`copyPlayerId`。
- **商店兌換** — `loadShop`/`openRedeemModal`/`confirmRedeem`/`showRedeemSuccess`/`copyRedeemCode`｜`shop_items`/`shop_codes`/`redemptions`｜需星星數＋有時需徽章門檻；季度 `getCurrentSeason`/`getPlayerBadgeCount`。
- **Admin 後台** — `openAdmin`/`switchAdminTab`、`adminLoadItems`/`adminAddItem`/`adminToggleItem`、`adminUploadCodes`/`adminLoadItemsForSelect`、`adminLoadRedemptions`｜限 `isAdmin()`（比對 `ADMIN_ID`）。
- **家長模式（內建）** — `enterParentMode`/`switchMode`/`switchViewingChild`、`loadParentDashboard`/`loadParentQuickSummary`/`loadParentOverview`/`loadParentAbility`/`loadParentReport`、留言 `loadParentMessages`/`sendParentMessage`/`openParentMail`/`checkParentMessages`、綁定 `linkChildAccount`/`linkChildFromAccount`/`unlinkChild`/`loadParentLinkedChildren`｜`parent_child_links`/`parent_messages`/`answer_logs`/`level_progress`/`children`。
- **模式切換/啟動** — `afterLogin`(5335)→`navigateAfterLogin`(5382)→`showModeSelect`/`enterGameMode`/`enterParentMode`。

## 2.6 視覺 / 音效 / 角色資產

- **背景**：`initStars`（星空）、`initFire`（火焰粒子）。
- **場景圖**：`images/<主題>_<區域>_dead.jpg` / `_alive.jpg`（旋鈕①整批換）。
- **角色頭像**：`images/boy.jpg` / `girl.jpg`（依 `player_gender`），用於標題(`updateTitleHero`)、遊戲內(`setupGame` 的 `scene-hero`)、勝利畫面(`showCelebration`)。
- **SVG 魔物（程式繪製，無圖檔）**：`heroSVG`/`impSVG`/`golemSVG`/`dragonSVG`/`demonSVG`，對應表 `MON_SVG_FN`、`MON_ICON`。｜旋鈕①可改成圖檔或新 SVG。
- **音效**：純 WebAudio 合成（`tone`/`sfx`），無需音檔，換主題可沿用。

## 2.7 抽象化指南：把這款複製成新遊戲（檢查表）

> 依序執行；每步驟標註屬於哪顆旋鈕。

**Step 0｜複製骨架（不動）**
- [ ] 複製 `index.html` 到新專案；沿用 Supabase 連線方式與 14 張表結構
- [ ] 保留：`initAuth`、`?type=child&at=&rt=` 橋接協定、`authSign*`、`syncFrom*`、`logAnswer`、`saveProgressToCloud`、`goTo`、`renderTopBar`
- [ ] 換 `SUPA_URL`/`SUPA_KEY`/`ADMIN_ID` 為新專案值

**Step 1｜旋鈕②：學科與題目**
- [ ] 重寫 `genQ(id)` 產生新學科題目（回傳 `{text, ans}`）
- [ ] 改 `LEVELS`（關名、難度 `sc`、評等門檻、`mon`/`tr`）
- [ ] 改 `OPS`（單元分組）與 `UNLOCK_CHAIN`（解鎖結構）
- [ ] 視題型改答題輸入 UI（`handleInput` / `s-game` 鍵盤區）→ 選擇題/拼字/連連看…
- [ ] 確認 `calcPts`/`gradeV` 規則是否沿用

**Step 2｜旋鈕①：主題與美術**
- [ ] 換 `images/`（場景 dead/alive、角色頭像、精靈圖）
- [ ] 改 `SCENE_MAP`（關卡→圖+區域名）、`getPlanetType`（星球分段）
- [ ] 改 `PLANET_PUZZLES`（守護神/收集物名稱與圖）
- [ ] 重寫文案常數：`OPENING_LINES`/`ZONE_DIALOGS`/`LEVEL_DIALOGS`/`CELEB_DATA`/`zoneEncourage`/`*_MSGS`
- [ ] 換配色 CSS 與標題、品牌字樣

**Step 3｜旋鈕③：玩法機制（決定保留/移除）**
- [ ] 列出要保留的模組（拼圖、每日任務、連登、排行榜、好友、對戰、商店、家長、Admin）
- [ ] 移除的模組：刪畫面 `s-*` + 對應函式 + 主畫面入口（注意別留孤兒呼叫）
- [ ] 調整獎勵數值（星星、對戰勝負分、每日任務獎勵）

**Step 4｜驗證（套用 CLAUDE.md 守則二）**
- [ ] `grep -c "function 名稱"` 確認無重複定義
- [ ] 本機開檔走一遍：登入→選關→答題→結算→各模組入口不報錯（F12 Console）
- [ ] push → Vercel → 線上 `curl | grep` 確認新碼上線
- [ ] 出實機驗收清單給使用者

---

## 附錄：關鍵常數/函式速查

| 想改什麼 | 去找 |
|---|---|
| 題目內容 | `genQ`(2438)、`LEVELS`(2048) |
| 關卡分組/解鎖 | `OPS`(2068)、`UNLOCK_CHAIN`(2074)、`SS.unlocked` |
| 計分評等 | `calcPts`(2469)、`gradeV`(2475)、各關 `sc`/`S`/`A`/`B` |
| 場景美術 | `SCENE_MAP`(2845)、`getPlanetType`(2493)、`images/` |
| 劇情文案 | `OPENING_LINES`/`ZONE_DIALOGS`/`LEVEL_DIALOGS`/`CELEB_DATA`/`zoneEncourage` |
| 登入/橋接 | `initAuth`(1589)、`?type=child&at=&rt=` |
| 存檔同步 | `syncFromCloud`/`saveProgressToCloud`/`logAnswer` |
| 收集/守護神 | `PLANET_PUZZLES`(2891)、`renderPuzzle`/`showSpiritAwaken` |
| 對戰 | `challengeFriend`→`finishBattle`、`battle_rooms` |
| 商店/兌換 | `loadShop`/`confirmRedeem`、`shop_items`/`shop_codes`/`redemptions` |
| 家長儀表板 | `loadParentDashboard` 系列、`parent_child_links`/`parent_messages` |
| 畫面切換 | `goTo(id)`(3590)、25 個 `s-*` |

> 本藍圖隨遊戲演進更新；新增模組時請補進 2.5 與速查表。
