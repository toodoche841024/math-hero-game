# 數學奇航(Math Hero)專案總指揮文件

> **此文件的用途**:這是本專案的最高指導文件。Claude Code 在執行任何任務前,必須先完整閱讀本文件,理解專案全貌、目前進度、工作守則,然後才開始動工。所有工作都必須遵守「工作守則」章節,完成後必須執行「驗證流程」並回報證據。

---

## 1. 專案簡介與商業目標

**產品名稱**:數學奇航(品牌主視覺:深色星空 + 橙色)
**遊戲名稱**:星界召喚師 / Math Hero
**目標用戶**:國小到國中、對數學有挫折感的孩子(玩家),以及願意為孩子教育付費的家長(付費者)

**核心商業邏輯**:
- 孩子端:用遊戲化(召喚師、星球、守護神、收集、進度)讓孩子「想玩」,在遊玩中大量練習數學
- 家長端:用學習報告、進度可視化、安全的帳號機制讓家長「放心、看得到效果」,進而願意付費
- 訂閱制:免費版(限 1 個孩子帳號)→ 付費版(多孩子、進階報告、更多內容)

**所有開發決策的判斷標準**:這個改動是否讓「孩子更想玩」或「家長更願意付費」?兩者都不是的話,優先級降低。

---

## 2. 系統架構總覽

### 2.1 兩個獨立的 Repo / 部署

| | math-hero-game(已部署) | math-hero-platform(部署中) |
|---|---|---|
| GitHub | toodoche841024/math-hero-game | math-hero-platform(確認是否已建立) |
| Vercel | https://math-hero-game.vercel.app | 待確認 |
| 技術 | 單一 HTML 檔(index.html) | Next.js 16 App Router |
| 資料庫 | Supabase Auth + DB(public schema) | Prisma v5 + Supabase Postgres(platform schema) |
| 角色 | 遊戲本體(孩子玩的) | 入口平台(家長註冊、儀表板、子帳號登入) |

### 2.2 資料庫(同一個 Supabase 專案、兩個 Schema)

```
Supabase Postgres
├── public / auth schema   ← 遊戲原有資料(players, battle_rooms...)
└── platform schema        ← 平台資料(Prisma 管理)
    ├── User       家長帳號(NextAuth 管理)
    ├── Account    OAuth 連結帳號
    ├── Session    NextAuth session
    ├── Family     家庭(含 familyCode)
    ├── Child      孩子(nickname, avatar, pin, grade)
    ├── Progress   學習進度(levelId, score...)
    └── Badge      成就徽章
```

**重要約束**:
- 遊戲端資料表(public schema)不可隨意改 schema,任何欄位變更必須先列出影響範圍
- platform schema 由 Prisma migration 管理,不可手動在 Supabase 後台改結構

### 2.3 登入系統(雙軌制)

```
家長端                          子帳號端
NextAuth.js v5                  自建 JWT (jose) child-token
Email/Password + Google         家庭代碼 → 選角色 → PIN
→ /parent/dashboard             → /play/game
                                ↓
                          Supabase Bridge(用 Service Role Key
                          在 Supabase 產生 session,拿 access_token)
                                ↓
                          iframe 載入遊戲:
                          math-hero-game.vercel.app?type=child&at=TOKEN&rt=TOKEN
                                ↓ 遊戲端 initAuth() 接收
                          進入遊戲主畫面
```

### 2.4 平台頁面路由

```
/                     首頁(數學奇航,雙入口:家長/孩子)
├── /parent/register  家長註冊
├── /parent/login     家長登入
├── /parent/dashboard 家長儀表板(NextAuth 保護)
├── /play             子帳號登入(已登入自動跳過 → 家庭代碼 → 選角色 → PIN)
└── /play/game        遊戲頁(proxy.ts 保護,過場動畫 + iframe)
```

---

## 3. 遊戲世界觀與內容設計(內容擴充的依據)

> Claude Code 在新增任何故事、關卡、文案時,必須符合以下世界觀設定,不可自創衝突設定。

### 3.1 核心主題:修復與守護(非戰鬥打殺)

孩子是「星界召喚師」,用數學之力淨化被「混沌迷霧」侵蝕的星球、喚醒沉睡的守護神。基調是**修復、成長、夥伴**,避免血腥暴力。

### 3.2 五大星球與守護神

| 星球 | 數學主題 | 守護神 |
|---|---|---|
| 火焰星球 | 加減法 | 鳳凰焰羽 |
| 海洋星球 | 分數/小數 | 海龜長老燦瀾 |
| 雷霆星球 | 除法 | 閃電鹿迅光 |
| 月之星球 | 混合運算 | 月狐銀霜 |
| 星之聖殿 | 全運算 | 星龍永恆 |

### 3.3 火焰星球四大區域(已實作)

| 區域 | 運算 | 關卡 |
|---|---|---|
| 森林入口 | 加法 | 1–4 |
| 燃燒溪谷 | 減法 | 5–8 |
| 火焰深林 | 乘法 | 9–13 |
| 鳳凰聖殿 | 除法 | 14–18 |

### 3.4 既有遊戲機制(不可破壞)

- 陪伴角色「小星」全程引導
- 18 片拼圖收集 → 集滿喚醒守護神
- 能量瓶 UI、世界修復進度條、連擊(combo)系統
- 區域專屬對話與「{暱稱}少年/{暱稱}少女」性別化稱呼
- 星星碎片貨幣 → 商店兌換實體獎勵(家長設定,有季度上限)

---

## 4. ⚠️ 工作守則(Claude Code 必須遵守)

### 守則一:先診斷,再動手

任何修 bug 或改功能的任務,**禁止直接改 code**。必須先:
1. 用 grep / 閱讀找出相關程式碼的實際位置
2. 說明「目前的程式碼為什麼會產生這個現象」(根因)
3. 列出修改計畫(改哪些檔案、哪些函式)
4. 才開始修改

如果無法確定根因,先加 `console.log` 診斷碼,請使用者在瀏覽器執行並回報 console 輸出,再判斷。

### 守則二:改完必須自我驗證,禁止「宣稱完成」

本專案歷史上多次發生「回報完成但部署後沒生效」。因此:
- 每個修改完成後,必須執行第 5 章的驗證流程,並**貼出驗證證據**(grep 結果、測試輸出、HTTP 回應)
- 禁止只說「已完成」「應該可以了」。沒有證據 = 沒有完成
- 如果修改的是 index.html(單一檔案遊戲),必須 grep 確認新程式碼確實存在於檔案中,且舊程式碼已被移除(沒有殘留重複定義的同名函式——同名函式重複定義時,後面的會覆蓋前面的,這是過去「改了沒效果」的常見原因)

### 守則三:小步提交

- 一次只做一個任務,做完 → 驗證 → git commit(訊息用繁體中文描述做了什麼)→ 才做下一個
- 禁止一次改動超過 3 個不相關的功能

### 守則四:不可破壞既有功能

- 改動前先確認該區塊被哪些其他函式呼叫(grep 函式名)
- Supabase 相關邏輯(登入、存檔、橋接 token)是命脈,改動時必須額外小心,改完必測登入與存檔
- 不可刪除或重新命名 `initAuth()`、`?type=child&at=&rt=` 的參數協定(平台 iframe 依賴它)

### 守則五:對非技術使用者說人話

使用者是非技術背景的創業者。回報時:
- 先講結論(做了什麼、能不能用了)
- 需要使用者操作的步驟,寫成可直接照做的清單(到哪個網站、點什麼按鈕、貼什麼值)
- 技術細節放最後

---

## 5. 標準驗證流程(每次修改後必跑)

### 5.1 程式碼層驗證(本機)

```bash
# 1. 確認修改確實寫入檔案
grep -n "新增的關鍵函式或字串" index.html

# 2. 確認沒有重複定義(輸出應該只有 1 筆)
grep -c "function 該函式名" index.html

# 3. JS 語法檢查(從 HTML 抽出 script 後用 node 驗證)
#    若是 Next.js 專案:
npm run build   # build 過 = 語法與型別 OK
```

### 5.2 部署層驗證(線上)

```bash
# 部署後,抓取線上版本確認新程式碼已上線
curl -s https://math-hero-game.vercel.app/ | grep -c "新增的關鍵字串"
# 輸出 >= 1 = 部署成功;輸出 0 = 部署沒更新(檢查 git push 與 Vercel build log)
```

### 5.3 功能層驗證(請使用者協助)

程式與部署都確認後,給使用者一份**驗收清單**,格式:
```
請打開 https://math-hero-game.vercel.app/(用無痕視窗避免快取)
□ 步驟 1:○○○ → 應該看到 ○○○
□ 步驟 2:○○○ → 應該看到 ○○○
如果有任何一步不符,請按 F12 打開 Console,截圖紅色錯誤訊息給我
```

### 5.4 快取注意事項

- 單一 HTML 檔在 Vercel 上可能被瀏覽器快取。驗收一律用「無痕視窗」或強制重新整理(Ctrl+Shift+R)
- 若懷疑快取,在 URL 後加 `?v=隨機數字` 測試

---

## 6. 目前進度

### ✅ 已完成

| 項目 | 狀態 |
|---|---|
| platform 資料庫 schema | ✅ |
| 家長註冊 / 登入 / 儀表板 | ✅ |
| 新增孩子帳號(含 PIN) | ✅ |
| 家庭代碼查詢 + 更換 | ✅ |
| 子帳號 PIN 登入 | ✅ |
| 路由保護(proxy.ts) | ✅ |
| Session 持久化 | ✅ |
| Supabase 橋接(子帳號自動登入遊戲) | ✅ |
| 遊戲 iframe 嵌入(過場動畫) | ✅ |
| 遊戲 auto-login(?type=child) | ✅ |
| 品牌統一(數學奇航) | ✅ |
| Platform 第一次 git commit | ✅ |
| 🌊 海洋星(分數四則運算,第 19–28 關,10 關)上線 | ✅ 圖檔待補 |

> **海洋星(2026-07 新增)**:第二顆星球,分數四則運算。分數加減(19–23)+分數乘除(24–28)。守護神海龜長老燦瀾(第 28 關甦醒)。破完燃燒星(第 18 關)後解鎖。
> - 分數專用資料/邏輯:`genFrac`、`buildFracOptions`(四選一誘答)、`fracEqual`、分數 helper(`gcd`/`reduce`/`fAdd..fDiv`/`fracHTML`/`exprHTML`);`LEVELS['19'..'28']`(帶 `kind:'frac'`);`PLANET_OPS`/`PLANETS`/`curPlanet`;分數作答 UI 為**四選一選擇題**(`#frac-choices`/`setInputMode`/`renderFracChoices`/`chooseFrac`;鍵盤 1~4)。
> - 判定規則:答案需「數值相等」且「已約到最簡」;作答改為**四選一**(不再手動輸入分子/分母),選項含 1 正解＋3 最簡誘答。出題產生器保證答案非整數、加減結果為正。
> - **待辦(需使用者提供美術)**:`images/` 內 9 張圖 — `ocean_shallow_dead/alive`、`ocean_coral_dead/alive`、`ocean_current_dead/alive`、`ocean_temple_dead/alive`、`turtle_guardian.jpg`。未放前場景顯示純色底(藍)。

### ❌ 已知未修復 Bug(遊戲端)

| # | Bug | 備註 |
|---|---|---|
| B1 | 主選單顯示舊的機器人角色,而非玩家選擇性別後的守護者形象 | 需先診斷主選單渲染來源 |
| B2 | 跨運算區域時,區域過場對話沒有出現 | 檢查觸發條件與狀態旗標 |
| B3 | 區域專屬鼓勵文字沒有作用 | 可能與 B2 同根因 |

---

## 7. 開發路線圖(依序執行)

> 每個階段內的任務按順序做。每完成一個任務 → 驗證 → commit → 回報 → 下一個。

### 🔴 階段 A:平台上線(最優先)

**A1. 修改 layout.tsx 標題**
- 把舊名「數學星球」改為「數學奇航」
- 驗收:瀏覽器分頁標題顯示「數學奇航」

**A2. GitHub 建 repo + 推送 + Vercel 部署**
- 指引使用者:在 GitHub 建立 `math-hero-platform` repo(逐步教學)
- `git remote add` → `git push`
- 指引使用者在 Vercel 連接 repo
- 驗收:Vercel build 成功,拿到正式網址

**A3. 設定環境變數(共 8 個)**
- 列出 8 個環境變數清單與每一個去哪裡取得(Supabase 後台哪一頁、複製哪個 key)
- `NEXTAUTH_URL` 更新為 Vercel 正式網址後重新部署
- ⚠️ 安全守則:`SUPABASE_SERVICE_ROLE_KEY` 等機密只能放 Vercel 環境變數,**絕對禁止寫進程式碼或 commit 進 git**。若發現已寫死在 code 裡,立即改為讀取環境變數並提醒使用者到 Supabase 重新產生(rotate)該 key
- 驗收:正式網址可完成「家長註冊 → 新增孩子 → 子帳號登入 → 進入遊戲」全流程

**A4. 修復遊戲端三個已知 Bug(B1–B3)**
- 嚴格遵守守則一(先診斷)。三個 bug 分開處理、分開 commit
- 驗收:依 5.3 出驗收清單給使用者

**A5. Google OAuth 申請與接入**
- 指引使用者到 Google Cloud Console 申請(逐步:建專案 → OAuth 同意畫面 → 憑證 → redirect URI 填什麼)
- 填入 `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`
- 驗收:可用 Google 帳號註冊/登入家長帳號

### 🟡 階段 B:家長價值(讓家長看到效果 → 付費意願的基礎)

**B1. 家長 session 自動跳轉**
- 已登入家長訪問 /parent/login 直接導向儀表板

**B2. Progress / Badge 同步機制**
- 設計:孩子在遊戲中過關/得星星時,資料同步寫入 platform.Progress 與 platform.Badge
- 建議實作:遊戲端在過關結算時呼叫平台的 API route(或直接寫 Supabase,需評估 RLS 權限),Claude Code 先提出方案比較表給使用者選
- 驗收:玩一關後,platform.Progress 出現對應紀錄

**B3. 家長學習報告(儀表板升級)**
- 從 public.players + platform.Progress 彙整:關卡完成數、正確率、每週學習時間、最近活躍日
- 視覺化:每週練習時間長條圖、各運算類型正確率
- 設計原則:讓家長 10 秒內看懂「孩子有沒有在進步」
- 驗收:儀表板顯示真實遊戲數據,而非只有星塵數量

**B4. 孩子忘記 PIN → 家長可重設**

### 🟢 階段 C:遊戲內容豐富化(讓孩子更想玩)

> 此階段每個任務開工前,先向使用者提交簡短設計提案(玩法、文案、獎勵數值)取得同意再實作。

**C1. 新手引導 + 安置測驗(Placement Test)**
- 新玩家首次進入:小星引導劇情 → 5–8 題快速測驗 → 依結果建議起始關卡
- 目的:程度好的孩子不無聊、程度弱的孩子不挫折

**C2. 錯題本(Wrong Answer Notebook)**
- 答錯的題目自動收錄,可在「訓練場」重新挑戰
- 重新答對 → 給予「淨化徽章」之類的正向回饋(把「錯誤」包裝成「待淨化的迷霧」,符合世界觀)

**C3. 訓練場(Training Ground)**
- 無時間壓力的自由練習區,可自選運算類型與難度
- 完成訓練給少量星星(數值需低於正式關卡,避免刷分)

**C4. 每日任務擴充**
- 例:「今天完成 10 題乘法」「連續答對 5 題」「淨化 3 題錯題」
- 連續登入獎勵階梯化(第 7 天給大獎)

**C5. 海洋星球(分數/小數)— 第二星球上線**
- 比照火焰星球架構:4 個區域 × 關卡、守護神燦瀾、區域對話、拼圖收集
- 分數題型需特別設計輸入 UI(分子/分母輸入)— 先出 UI 提案

**C6. 故事過場強化**
- 每個區域解鎖/守護神甦醒時的劇情演出(對話 + 畫面效果)
- 文案基調:溫暖、鼓勵、強調「是你的努力讓星球復甦」

### 🔵 階段 D:商業化

**D1. 訂閱方案落地**
- 方案:免費(1 個孩子)/ 基礎 NT$149/月 / 進階 NT$349/月
- 先實作「方案限制」邏輯(免費版第 2 個孩子帳號 → 顯示升級頁),再接金流

**D2. 金流串接(ECPay 優先,台灣市場)**
- Claude Code 先產出 ECPay vs Stripe 比較與申請流程指引

**D3. 實體獎勵兌換後台**
- 家長設定獎勵品項與所需星星、季度兌換上限(機制已有雛形,補完平台端管理介面)

---

## 8. 給 Claude Code 的標準開工流程

每次新 session 開始時,依序執行:

1. **讀本文件**(CLAUDE.md)
2. **確認現況**:`git log --oneline -10` + `git status`,比對第 7 章路線圖,判斷目前做到哪
3. **跟使用者確認本次目標**:「根據路線圖,下一個任務是 ○○,要做這個嗎?還是你有其他優先事項?」
4. **執行**:遵守守則一(先診斷)→ 實作 → 守則二(驗證 + 證據)
5. **收尾**:commit → 用人話回報 → 更新本文件第 6 章的進度表(把完成項目打勾、新發現的 bug 記入)

> **本文件是活的**:Claude Code 每完成一個任務,有責任更新「目前進度」章節,讓下一個 session 的 Claude Code 能無縫接手。

---

## 9. 常見問題排查手冊

| 症狀 | 先檢查 |
|---|---|
| 改了 code 但遊戲沒變化 | ① 是否 push + Vercel 重新部署了 ② 瀏覽器快取(無痕測試)③ 同名函式重複定義被覆蓋(grep -c 函式名) |
| 子帳號進不了遊戲 | ① iframe URL 的 at/rt token 是否存在 ② Supabase Bridge 的 Service Role Key 環境變數 ③ 遊戲端 initAuth() 是否報錯(Console) |
| 家長登入後被踢回登入頁 | ① NEXTAUTH_URL 是否等於目前網址 ② NEXTAUTH_SECRET 是否有設 ③ cookie 網域問題 |
| Prisma 報錯 | ① DATABASE_URL 指向的 schema 是否為 platform ② migration 是否已執行(npx prisma migrate deploy) |
| Vercel build 失敗 | 看 build log 的第一個 error;通常是缺環境變數或型別錯誤 |
