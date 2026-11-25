# Scrub Stretch

桌面版的「拖移刷音」工具（仿手機 audio stretch 類 app 的核心操作，自己實作，非逆向）。

**用法**：直接用 Chrome / Vivaldi / Edge 開 `index.html`（雙擊即可），把音訊或影片拖進去。

- 拖動中間的波形 = 刷動（變速不變調）；按住不動 = 持續播放 playhead 那一瞬間（凍結）
- 「變調」開關：開啟後像轉黑膠，速度越快音越高、倒拖會倒放、停住無聲（此時凍結不適用）
- 拖波形時游標照常移動；拖出視窗左右邊緣時才鎖定滑鼠（Pointer Lock）、改顯示替身游標，從另一邊繞回來（像 Blender），可以無限拖。放開或按 Esc 解除；有繞回過的話，真的游標會出現在離開視窗的那個邊緣（網頁無法移動系統游標）
- 底部總覽條可直接點/拖跳到任何位置；滾輪縮放
- 空白鍵播放、速度滑桿 0.1–2×、`R` 錄製、`←/→` 跳 1 秒（Shift 5 秒）、`Esc` 停止重播
- 錄製：每段 take 會存下
  - 拖移軌跡（JSON，可再載入重播）
  - 即時錄下的影音（mp4，瀏覽器不支援時為 webm；音訊檔則錄下波形畫面）
  - 依軌跡離線重算的 WAV（與即時聽到的相同演算法、無壓縮）

## 原理

`index.html` 內嵌 AudioWorklet：WSOLA（波形相似重疊相加）。每 N/2 個樣本放一顆以 playhead 為中心的
Hann 粒子，並在 ±N/4 內找與上一顆粒子自然延續最相像的位置，所以速度改變時音高不變，停住時會對齊週期無縫凍結。
變調模式則改成直接以 playhead 的速度讀原音（三次 Hermite 內插、兩級平滑讓速度連續、12Hz DC 阻隔讓停住時安靜），切換時先淡出約 5ms 再換引擎。
「重播片段」調 N（1024 / 2048 / 4096 樣本，48kHz 下約 21 / 43 / 85ms）；變調模式不用片段，所以會停用。

錄製時 worklet 以樣本為單位記錄所有控制事件，重播與 WAV 匯出都跑同一份事件，所以結果可重現。

## 影片拖動不卡的做法

`<video>` 每次 seek 都要從前一個關鍵影格開始解碼，拖動時會卡。載入影片後，背景會用第二個隱藏的 video 以 2 倍速跑一遍，
用 `requestVideoFrameCallback` 抓下每一格，用 `OffscreenCanvas.convertToBlob` 存成 JPEG（長片會降解析度控制記憶體），
2 倍速漏掉的影格再逐格 seek 補齊。拖動時畫快取裡的影格（`createImageBitmap` 解碼、LRU 保留約 150 張），
手停下來後才讓真正的 video seek 到位，換回原解析度畫面。

## 介面

依 [tasteskill 2.5](https://github.com/leonxlnx/taste-skill) 的 system / motion 規則（它本身是做公司網站的，hero、版面分屏、照片與 gate 不適用於工具介面，只取按鈕、色彩、字級、動態、無障礙這些部分）。
Instrument Sans / Fragment Mono 字型與 Phosphor 圖示都內嵌在 HTML 裡（離線可用）；色彩用 CSS 變數，跟著系統深色 / 淺色模式切換，畫布也讀同一組變數。

## 第三方素材

以下素材以 base64 或 SVG 內嵌在 `index.html` 裡：

- [Instrument Sans](https://github.com/Instrument/instrument-sans)、[Fragment Mono](https://github.com/weiweihuanghuang/fragment-mono)：SIL Open Font License 1.1
- [Phosphor Icons](https://phosphoricons.com/)：MIT License
