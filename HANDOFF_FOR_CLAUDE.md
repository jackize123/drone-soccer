# Drone Soccer / 星星大冒險 交接筆記

日期：2026-07-12（photoreal 大改版後更新）

## 目前狀態

- 專案位置：`C:\Users\user\drone-soccer`
- GitHub Pages 從 `main` 發佈：`https://jackize123.github.io/drone-soccer/stars3d.html`
- **photoreal 大改版進行中**，詳細路線圖與進度見 **`PHOTOREAL_ROADMAP.md`**（這份是主要交接文件）。
- 開發分支：`agent/improve-japan-scenery`（每個增量都是獨立可驗證的 commit）。

## 本次已完成（three.js r128 + examples/js addon，零升級破壞）

- INC-1 後製管線：EffectComposer（RenderPass→SSAO→Bloom→FXAA→Gamma）
- INC-2 PBR 材質 + PMREM 天空 IBL 環境反射；金鯱/金頂金屬材質
- INC-2b 程序式石垣 masonry 貼圖 + 地面法線
- INC-3 真 HDRI 環境光（CC0 Poly Haven，日間關卡）
- INC-6 GLTFLoader 管線 + CC0 寫實 boulder（熊本城關卡）
- 效能自動分級（`lowPerf`：手機/低記憶體退回 Lambert、關 SSAO、降 bloom）
- 全部 headless Chrome 截圖驗證、無 JS 錯誤、已 push

## 尚未完成

- INC-5 每關實景天空/地面照 → 需外部影像素材（本機無 imagegen 工具）
- 寫實日本城堡 glb → CC0 來源沒有，需委製或購買授權（現用 PBR 程序式城堡）

## 開發/驗證工具（scratchpad）

- `serve.mjs`：本地靜態伺服器（port 8765）
- `shot.mjs`：headless Chrome CDP 截圖器，可切關卡截圖比對
  用法：先起 serve.mjs，再 `node shot.mjs <前綴> <關卡,關卡>`（如 `node shot.mjs test 2,4,8`）

## 給下一棒的重點

- 目標：真實旅遊照片感。程式接口（RGBELoader/GLTFLoader/後製）都已就緒。
- 加寫實 decor 要注意：Poly Haven photoscan 模型 ~5MB/件，手機載入須節制（Draco 壓縮或改用 Kenney/Quaternius 低面數 CC0）。
- 每次改動請沿用 `shot.mjs` 截圖驗證後再提交；穩定才合併 `main` 上線。
