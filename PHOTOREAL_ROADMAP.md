# 星星大冒險 — Photoreal 大改版路線圖

日期：2026-07-12　分支：`agent/improve-japan-scenery`（**尚未合併 main / 未上線**）

目標（來自使用者 + codex 建議）：把 `stars3d.html` 從 low-poly 積木風，改成接近真實旅遊照片的寫實 3D 場景。**跨多次對話進行**，每個增量都要「可驗證 + 可獨立提交」，任何時候分支都不能是壞的。

## 技術地基（已完成 ✅）

- three.js 版本鎖定：**r128**（`three.min.js` 內 `REVISION=128`，2021 版）。
- 決策：**不升級** three.js（避免 `sRGBEncoding`→`colorSpace` 等 API 破壞），改用 r128 對應的 `examples/js` UMD addon（掛在全域 `THREE` 上，與現有非模組架構相容）。
- 已 vendored 到 `lib/three-addons/`（全部為 r128 版本）：
  - Bloom 鏈：`CopyShader.js`、`LuminosityHighPassShader.js`、`EffectComposer.js`、`RenderPass.js`、`ShaderPass.js`、`MaskPass.js`、`UnrealBloomPass.js`
  - SSAO：`SSAOShader.js`、`SimplexNoise.js`、`SSAOPass.js`
  - 抗鋸齒：`FXAAShader.js`
  - 資產載入：`GLTFLoader.js`（glb 模型）、`RGBELoader.js`（HDRI `.hdr`）
- 現況 renderer（第 146–152 行）：已有 `antialias`、`outputEncoding=sRGB`、`ACESFilmicToneMapping`、`PCFSoftShadowMap` —— 好的起點。
- Render loop：`renderer.setAnimationLoop`（第 831 行），單一 `renderer.render(scene,camera)`（第 855 行）。

## 接線點（stars3d.html 具體行號，供後續對話定位）

- 第 142 行 `<script src="three.min.js">` 之後 → 加 addon `<script>`（**順序依相依性**，見下）。
- 第 152 行後（renderer 設定完、scene/camera 於 153/154 建立）→ 建 `EffectComposer` 需在 camera 之後。
- 第 826 行 `resize()` → 加 `composer.setSize(...)` 與 bloom 解析度。
- 第 855 行 `renderer.render(scene,camera)` → 換成 `composer.render()`。

addon 載入順序（相依）：CopyShader → LuminosityHighPassShader → EffectComposer → RenderPass → ShaderPass → MaskPass → UnrealBloomPass →（SSAO 用）SSAOShader、SimplexNoise、SSAOPass →（AA 用）FXAAShader → GLTFLoader / RGBELoader。

## 增量計畫（每步做完 → headless 截圖驗證 → 提交）

- [ ] **INC-1 Bloom + tone mapping**（視覺 CP 值最高、風險低）
  接 EffectComposer + RenderPass + UnrealBloomPass。本場景大量 additive sprite（星星光暈、太陽、燈籠）會因 bloom 大幅提升質感。
  ⚠️ r128 陷阱：composer 渲染到 render target 時的 gamma/sRGB。若畫面偏灰或過亮，於最後加 `GammaCorrectionShader` pass 或設定最終 target 的 encoding。**務必截圖比對**，不可盲提交。
- [ ] **INC-2 PBR 材質**：把 `lamb()`（`MeshLambertMaterial`）關鍵物件（城堡石垣/瓦/木、地面）改 `MeshStandardMaterial` + 程序式 roughness/normal 或貼圖。用已內建 `PMREMGenerator` 從天空/HDRI 產生環境反射，讓金屬鯱、金頂有真實反射。
- [ ] **INC-3 HDRI 環境光**：`RGBELoader` 載入每關 `.hdr`（或用 PMREM 從全景圖生 envMap），取代目前純方向光；配真實陰影。素材放 `assets/hdri/`。
- [ ] **INC-4 SSAO + FXAA 後製**：`SSAOPass` 加接觸陰影、`FXAAShader` 補 AA。注意行動裝置效能，需加「低效能自動關閉」開關。
- [ ] **INC-5 每關實景天空/地面**：目前 10 關共用一張 `japan-travel-panorama.png`。改為每關各自的實景天空（沖繩海灘、廣島夕陽、京都、北海道雪…）與地面材質。素材放 `assets/scenery/<level>/`。可用 imagegen 產生或找 CC0 素材。
- [ ] **INC-6 glb 寫實模型**：`GLTFLoader` 載入寫實天守閣（熊本/大阪/弘前）、鳥居、樹木、遠景建築，取代 `castle()`/`torii()` 積木。素材放 `assets/models/`。⚠️ 注意檔案大小（GitHub Pages + 手機載入）→ 用 Draco 壓縮或低面數。
- [ ] **INC-7 效能與行動裝置**：加畫質分級（高/中/低），依 `devicePixelRatio` 與偵測到的幀率自動降級（關 SSAO、降 bloom、用積木 fallback），確保學校電腦/手機能跑。

## 發佈流程（每個穩定里程碑）

1. 只在 `agent/improve-japan-scenery` 上開發、提交。
2. headless 驗證：本地 `node`(見 scratchpad `serve.mjs`) 起 server → Chrome `--headless=new --screenshot` 截圖各關比對。
3. **確認畫面正確且效能可接受，才** 合併到 `main` 並 push → GitHub Pages 自動更新
   `https://jackize123.github.io/drone-soccer/stars3d.html`。
4. 未驗證的版本**絕不**進 main（線上是給小朋友用的正式版）。

## 資產與版控決策

- `assets/scenery/japan-travel-panorama.png` — 程式執行時載入，**必須入庫**。
- `assets/castles/*.jpg`（15 張城堡實拍）— 目前僅參考素材，**不入庫**（12MB 未被程式使用）；已列入 `.gitignore`。若 INC-6 要當貼圖再議。
- `lib/three-addons/*` — **入庫**（離線可用地基）。

## 可選工具（使用者提到，尚未安裝）

- `playwright`：自動開遊戲、切各關做視覺回歸測試（INC 迭代很有用）。
- `screenshot`：批次擷取各關畫面比對。
- `cloudflare-deploy` / `vercel-deploy`：發測試版而不動正式 Pages。
建議在 INC-1 之後、要密集視覺迭代時再裝 playwright + screenshot。
