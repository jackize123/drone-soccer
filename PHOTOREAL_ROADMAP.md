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

- [x] **INC-1 後製管線**（commit `0fb80ed`）✅ EffectComposer：RenderPass→SSAO→Bloom→FXAA→Gamma。bloom threshold 0.78 只讓亮光暈溢光；GammaCorrectionShader 解決 r128 composer 偏灰陷阱。已 headless 驗證日/夜關卡。
- [x] **INC-2 PBR 材質 + IBL**（commit `bd00c40`）✅ `lamb()`→`MeshStandardMaterial`；`metalMat()` 金鯱/金頂金屬反射；`PMREMGenerator` 從每關天空生 envMap 設 `scene.environment`。
- [x] **INC-2b 程序式貼圖**（commit `0ffb470`）✅ canvas 生成石垣 masonry map + `heightToNormal()` 法線，套城堡石垣；地面 noise 法線。
- [x] **INC-4 SSAO + FXAA**（併入 INC-1）✅ 含 lowPerf 自動關閉。
- [x] **INC-7 效能分級**（併入 INC-1/2）✅ `lowPerf`（deviceMemory≤4 或行動裝置 UA）自動退回 Lambert/Basic、關 SSAO、降 bloom 與 pixelRatio。
- [x] **INC-3 HDRI 環境光**（commit `b23193f`）✅ 下載 CC0 `assets/hdri/clear_puresky_1k.hdr`(Poly Haven,1.1MB);RGBELoader→PMREM,日間關卡用真 HDRI 天光,夜間維持程序 env。
- [x] **INC-6 glb 模型管線**（commit `de3d483`）✅ `loadGLB()`(快取/clone/關卡守衛/降級) + `glbRock()`,熊本城關卡用 CC0 Poly Haven 寫實 boulder(5.5MB,lazy-load)。
  - ⚠️ **重要發現**:CC0 來源(Poly Haven 521 個模型)**無**日本天守閣/鳥居/神社模型 → 寫實城堡 glb 需委製或購買授權。
  - ⚠️ Poly Haven 寫實 decor 偏重(~5.5MB/件,photoscan 高面數),手機載入須節制;要多用需 Draco 壓縮或改用低面數 CC0(Kenney/Quaternius)。
- [ ] **INC-5 每關實景天空/地面**：目前 10 關共用一張 `japan-travel-panorama.png` + 程序漸層天空 + HDRI 反射。要每關實景照需**外部影像素材**（imagegen 產生或 CC0）。⚠️ **仍需外部素材**（本機無 imagegen 工具)。

### 本次(2026-07-12)已完成：INC-1、2、2b、3、4、6、7 —— 全部 headless 驗證、無 JS 錯誤、已提交。
### 剩餘：INC-5(每關實景照,需 imagegen/CC0 影像)、寫實城堡 glb(CC0 無,需委製)。GLTFLoader/RGBELoader 接口已就緒,拿到合適素材即可接。

## 素材授權
- HDRI、boulder 皆來自 Poly Haven,授權 **CC0**(公眾領域,無需標示;仍於此註明來源以示尊重)。

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
