# 任務進度追蹤

## 狀態：全部完成 ✅

## 完成清單

### 頂部狀態列
- [x] 所有畫面都有 top-bar-placeholder（15 個畫面）
- [x] renderTopBar() 函式，左上我的收藏，右上暱稱+登出
- [x] updateUserDisplay() 呼叫 renderTopBar() 更新所有 placeholder
- [x] top-bar CSS（sticky, 44px, blur 背景）

### 帳號設定（四個 Tab）
- [x] s-account 完整重寫
- [x] Tab 0：基本資料（玩家ID、信箱、暱稱、頭像8個）
- [x] Tab 1：安全性（密碼重設信、修改密碼、刪除帳號）
- [x] Tab 2：個人化（音效開關）
- [x] Tab 3：我的成就（星星碎片、各關最佳紀錄表格）
- [x] switchAccTab() 函式
- [x] openAccountScreen() 完整重寫
- [x] updateDisplayName() 用新 ID
- [x] updatePassword() 用新 ID
- [x] sendPasswordReset() 新函式
- [x] requestDeleteAccount() 新函式
- [x] selectAvatar() 新函式（8種頭像）
- [x] toggleSound() 新函式
- [x] copyPlayerId() 新函式
- [x] renderAchievements() 新函式

### 好友系統完整重寫
- [x] s-friends 有我的資訊卡（頭像、暱稱、ID、星星數）
- [x] 四個 Tab（好友、待確認、新增、對戰紀錄）
- [x] switchFrTab() 函式
- [x] loadFriendsScreen() 完整重寫（含 is_online 更新）
- [x] renderFriendsList() 含頭像、在線狀態、挑戰/鼓勵/刪除按鈕
- [x] renderPendingList() 顯示鼓勵訊息和好友邀請
- [x] loadEncouragements() 新函式
- [x] markEncouragementRead() 新函式
- [x] doSearchPlayer() 支援暱稱或8碼玩家ID搜尋
- [x] sendEncouragement() + confirmEncouragement() 新函式
- [x] deleteFriend() 新函式
- [x] loadBattleHistory() 新函式（對戰紀錄 tab）
- [x] copyFriendId() 新函式

### C2 即時對戰系統
- [x] generateBattleQuestions 函式
- [x] challengeFriend 函式
- [x] acceptBattle 函式
- [x] subscribeToBattleRoom 函式
- [x] handleBattleRoomUpdate 函式
- [x] startBattle 函式
- [x] showBattleQuestion 函式
- [x] submitBattleAnswer 函式（Enter 也能送出）
- [x] finishBattle 函式
- [x] showBattleResult 函式（含勝利+10⭐、平局+3⭐提示）
- [x] checkBattleInvites 函式（含 avatar 支援）
- [x] s-battle-waiting 畫面（含 top-bar-placeholder, calc(100vh-44px)）
- [x] s-battle 畫面（更新版 UI）
- [x] s-battle-result 畫面（含 top-bar-placeholder, 星星說明）

### 主選單按鈕
- [x] 帳號設定 → openAccountScreen()
- [x] 好友 → goTo('s-friends')
- [x] 星際排行榜 → goTo('s-leaderboard')
- [x] 每日任務 → goTo('s-tasks')

### 登入後初始化
- [x] checkLoginStreak()
- [x] initDailyTasks()
- [x] checkBattleInvites()
- [x] is_online:true 更新
- [x] authSignOut() 設定 is_online:false

### 音效
- [x] sfx() 加入 sound_on 判斷

### 部署
- [x] TASK.md 所有項目為 [x]
- [x] copy index-v7.html index.html
- [x] git push 成功
