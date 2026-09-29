# 鬼谷倖存者 Demo-010 完整實作開發計畫書 (DOC-DEV-010 Implementation Plan)

本計劃依據《鬼谷倖存者 Demo-010 完整開發計劃書（DOC-DEV-010）》與用戶確認之決策（Q-17 至 Q-28）制定，針對 Demo-009 進行 76 項缺陷全面修復與架構重構，產出完整、無縮減、無依賴外部 CDN 的單一網頁檔案 `Demo-010.html`，並同步部署為入口 `index.html`。

---

## 一、 使用者意圖與專案核心目標

1. **零縮減完整交付**：打造無任何程式碼省略標記、具備完整仙俠題材與吸血鬼倖存者玩法的高品質單一 HTML 檔案（`Demo-010.html`），並提供精確程式碼行數。
2. **解決 76 項核心缺陷（ISSUE-101 ~ 176）**：
   - 解決受擊報錯、永不死亡、計時到不結算問題。
   - 解決幀率漂移導致的生怪、寶箱與環繞武器傷害不均問題（改採 1/60s 固定步長與時間積分）。
   - 解決存檔損毀與舊檔不相容問題（落實 A/B 雙槽交替寫入與 v9 遷移轉換補償機制）。
   - 移除未授權外部 CDN，改以原生 Canvas 預渲染水墨雜訊與 WebGL2 靈氣著色器雙背景切換。
3. **十階境界與專案原創材料完全對應**：落實「煉氣、築基、結晶、金丹、具靈、元嬰、化神、悟道、羽化、登仙」完整進階路線與對應突破素材。
4. **雙道友出戰與保底機制**：凡 70%、靈 25%、仙 5% 抽卡，40 抽保底出仙，出戰 2 位且可在局內受傷倒地。
5. **Boss 戰與天劫終局保證**：時間結束前 90 秒 Boss「大妖獸·當康」降臨；時間結束若未擊殺 Boss 則天劫降臨 30 秒，存活即天劫飛升，保證最遲 5 分 30 秒必定結算。

---

## 二、 系統架構與模組劃分（24 個 @MODULE 順序）

全檔將以嚴格的 `@MODULE:[NAME] v1 BEGIN / END` 標籤分段，所有內部邏輯封裝於 `window.GG`，拒絕任何全域變數污染：

```
Demo-010.html
├── 1.  @MODULE:MOD-HEAD v1          : CSP, UTF-8, Viewport (無縮放鎖定), Manifest
├── 2.  @MODULE:MOD-GUARD v2         : 外部請求防護與觀測
├── 3.  @MODULE:MOD-STYLE v1         : 基礎排版、佈局、HUD 與面板樣式
├── 4.  @MODULE:MOD-STYLE_ADD v1     : 仙俠字型、稀有度卡片、Boss 血條、Touch-action 防護
├── 5.  @MODULE:MOD-HUD v1           : DOM 結構（純原生，以 data-action 事件委派替代 inline onclick）
├── 6.  @MODULE:MOD-FEAT v1          : 瀏覽器環境能力偵測（WebGL2, Web Audio）
├── 7.  @MODULE:MOD-DOM v1           : 安全 DOM 工具（h, textContent, 防禦式選取）
├── 8.  @MODULE:MOD-PERF v2          : 效能量測（FPS, FrameTime, Stalls, p95 延遲）
├── 9.  @MODULE:MOD-QUALITY v2       : LOW / MID / HIGH 品質自動調適預算
├── 10. @MODULE:MOD-STABILITY v1     : safeLoop 主循環保護、itemTxn 事務交易、斷路器
├── 11. @MODULE:MOD-DATA v1          : 凍結資料庫（10 境界、11 材料、7 武器、4 宗門、丹藥、敵怪）
├── 12. @MODULE:MOD-FORMULA v1       : 公式統一庫（F-003 至 F-020，減傷、冷卻、Boss TTK、經驗）
├── 13. @MODULE:MOD-RNG v1           : Mulberry32 種子隨機產生器（保證可重現）
├── 14. @MODULE:MOD-CLOCK v2         : 三時鐘系統（Real/Game/Sim）、打擊停頓、五階段局勢狀態機
├── 15. @MODULE:MOD-SAVE v2          : 雙槽交替、FNV-1a 校驗、寫後讀回、v9 存檔遷移轉換
├── 16. @MODULE:MOD-RUN v1           : 本局臨時屬性計算（擲骰 + 永久 + 氣運，不跨局累加）
├── 17. @MODULE:MOD-GROWTH v1        : 局內經驗池、升級卡池（4 張）、詞綴疊加、重抽機制
├── 18. @MODULE:MOD-ECON v1          : 本局帳本（Ledger）、結算一次性入庫、靈果突破商店
├── 19. @MODULE:MOD-COMPANION v1     : 尋仙堂抽卡、40 抽保底、等級加成、2 位出戰陣容
├── 20. @MODULE:MOD-WAVE v2          : 時間積分式刷怪器、同屏上限、精英怪生成
├── 21. @MODULE:MOD-COMBAT v1        : 環繞武器定時判定、鋸齒閃電彈射、毒沼持續、點燃 Dot
├── 22. @MODULE:MOD-ART_* v1         : 背景雙引擎（Aura Shader / Ink Tile）、光暈快取、Emoji 怪物
├── 23. @MODULE:MOD-ENTITY v1        : 實體生命週期（玩家、道友、妖獸、子彈、AoE、掉落物）
├── 24. @MODULE:MOD-ULT v1           : 宗門大招（萬劍歸宗、焚天火雨、不壞金身）
├── 25. @MODULE:MOD-UI v1            : 頁內 Toast（無 alert）、Boss 警示橫幅、傷害飄字合併
├── 26. @MODULE:MOD-FLOW v1          : 彈窗隊列排程、主迴圈單一啟動保障、暫停協調
├── 27. @MODULE:MOD-HUB v1           : 洞府主介面（修為、靈果、突破、尋仙、背包、出戰）
├── 28. @MODULE:MOD-INPUT v1         : 虛擬搖桿、鍵盤 WASD/QE/Space、自動託管 AI 導航
├── 29. @MODULE:MOD-REPORT v3        : 效能與測試診斷面板（?report 啟動）
├── 30. @MODULE:MOD-TEST v2          : 黃金測試套件（自動驗證 F-003~F-018）
└── 31. @MODULE:MOD-CORE v1          : 遊戲入口 bootstrap、fitCanvas 自適應、1/60s 步進迴圈
```

---

## 三、 關鍵機制與技術實施細節

### 1. 戰鬥與時鐘公平性（F-003 ~ F-020）
- **固定步長**：`MOD-CLOCK` 採用 `1/60s` 固定步長模擬，單幀最高執行 5 步防禦螺旋延遲。
- **Boss 動態血量（F-008）**：`bossHp = 60 * clamp(playerDps, 0.6 * refDps, 1.4 * refDps)`，確保 Boss 擊殺時間穩定在 45~75 秒範圍內。
- **天劫終局（F-020）**：若在時間限制前未擊殺 Boss，天劫強制落雷 30 秒（預警 1.0s，每擊扣除 15% 最大 HP 真實傷害），存活即以 1.2 倍係數天劫飛升。

### 2. 存檔與無損相容遷移（MOD-SAVE v2）
- **雙槽輪轉**：使用 `guigu.save.v10.a` 與 `guigu.save.v10.b`，每次寫入以序號 `seq + 1` 遞增，並透過 `fnv1a` 校驗碼驗收。
- **Demo-009 舊檔平滑遷移**：
  - 自動偵測 `guigu.save.v9.a/b`。
  - 將中文境界名稱轉為標準 Key（如「結丹期」轉 `realm.jie_jing`）。
  - 將熟練度打擊次數 `/ 10` 轉換為擊殺數。
  - 將舊版跨局無上限膨脹之攻擊防禦屬性以靈果價格退還為靈石（上限 5,000 靈石）。
  - 保留 v9 原始檔案不刪除作為使用者本機備份。

### 3. 雙渲染引擎與視覺升級
- **WebGL2 靈氣著色器**：支援 WebGL2 且品質為 MID/HIGH 時，啟用原生 GLSL 片段著色器生成流動靈氣雲霧。
- **無縫平鋪水墨雜訊**：在不支援 WebGL2 或 LOW 品質時，自動降級為預渲染 256x256 雙層視差水墨雲霧圖塊，零卡頓。
- **光暈快取（MOD-ART_FX）**：建立 `Map<string, HTMLCanvasElement>` 快取最多 32 種光暈，完全杜絕 Canvas `shadowBlur` 造成的每幀效能雪崩。

---

## 四、 施工與驗收步驟

1. **施工準備**：
   - 讀取當前專案依賴與配置，確認靜態託管服務 Express 配置。
2. **完整組裝 `Demo-010.html`**：
   - 一氣呵成編寫完整無省略的單一 HTML 檔案，包含全部 CSS、DOM、JS 邏輯模組。
   - 確保程式碼行數完整豐富（預計約 2,200 ~ 2,600 行完整無壓縮代碼）。
3. **入口與版本同步**：
   - 將 `Demo-010.html` 同步複製為根目錄 `index.html`。
   - 在版本切換下拉選單（Version Switcher）中新增 `Demo-010 (天劫飛升版)` 並設為首選。
   - 更新 `metadata.json` 與 HTML head meta 標籤。
4. **編譯、測試與品質審查**：
   - 執行 `compile_applet` 驗證 Node.js / Express 構建與啟動狀態。
   - 驗證單元測試（F-003 冷卻、F-006 難度、F-008 Boss 血量、F-018 結算倍率、v9 遷移邏輯）。
   - 檢查控制台無語法錯誤、無未封裝全域變數、無外部 CDN 請求。
5. **交付報告**：
   - 統計總行數與模組清單。
   - 提供離線複製字串與部署說明。
