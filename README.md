# CCTV_CarCounter - 車輛計數系統

使用 YOLO 模型偵測、ByteTrack 追蹤與 CCTV 攝像機串流，自動偵測並計數跨越指定計數線段的所有車輛類別。所有推論與追蹤皆在本地完成。

## 環境需求

### Python 版本
- **建議：Python 3.10+**
- 支援：Python 3.8+

### 系統依賴
- ffmpeg（用於 CCTV 串流錄製和視頻編碼）
  - **Windows**: 可從 [ffmpeg.org](https://ffmpeg.org/download.html) 下載，或使用 `choco install ffmpeg`
  - **macOS**: `brew install ffmpeg`
  - **Linux**: `sudo apt-get install ffmpeg`
- （選用）NVIDIA GPU + CUDA：有 GPU 時 Ultralytics 會自動使用以加速推論；無 GPU 時自動退回 CPU（較慢但可正常運行）。

## 安裝步驟

### 1. 複製或下載專案
```bash
# 如果有 git
git clone https://github.com/cowrider2018/CCTV_CarCounter.git
cd carCounter

# 或直接下載解壓
cd carCounter
```

### 2. 建立虛擬環境（推薦）
```bash
# Windows
python -m venv venv
.\venv\Scripts\activate

# macOS / Linux
python3 -m venv venv
source venv/bin/activate
```

### 3. 設定 .env 檔案

在專案根目錄建立 `.env` 檔案，填入以下內容：

```env
# CCTV 串流 URL（必填）
# 例如：https://cctvtraffic.tycg.gov.tw/camera011/
CCTV_URL=https://your-cctv-stream-url/

# 本地模型路徑（選填，預設 model.pt）
MODEL_PATH=model.pt

# 偵測信心門檻（選填，預設 0.25 = Ultralytics 預設）
CONFIDENCE=0.25
```

### 4. 放置本地模型

將你的 YOLO 偵測模型（Ultralytics `.pt` 格式）放在專案根目錄並命名為 `model.pt`，
或以 `.env` 的 `MODEL_PATH` 指向其他位置。詳見下方「本地模型設定」。

## 專案結構

```
carCounter/
├── main.ipynb              # 主程式（Jupyter Notebook）
├── model.pt                # 本地 YOLO 偵測模型（自行放入）
├── requirements.txt        # Python 依賴清單
├── .env                    # 環境變數配置（勿上傳）
├── .gitignore             # Git 忽略規則文件
├── origin.mp4             # 錄製的原始影片（自動生成）
├── detections.db          # 本地推論的標註與時間戳記資料庫（SQLite，自動生成）
├── labeled.mp4            # 標註後的計數結果影片（自動生成）
└── README.md              # 說明書
```

## 使用流程

### 方式一：Jupyter Notebook（推薦）

1. **啟動 Jupyter Lab**
   ```bash
   jupyter lab
   ```
   瀏覽器會自動開啟，進入 Jupyter 介面。

2. **開啟 `main.ipynb`**
   - 在檔案瀏覽器中雙擊 `main.ipynb`，或
   - 在 Jupyter 中導航到該檔案並開啟

3. **逐個執行儲存格（Cell）**
   - 依序點擊 ▶ 執行

   **Cell 1 - 安裝套件**（可選，僅首次運行）
   ```python
   %pip install --upgrade pip
   %pip install -r requirements.txt
   ```

   **Cell 2 - 錄製影片素材**
   ```python
   # 錄製影片素材
   ```
   - 執行此儲存格以從 CCTV 錄製影片
   - 完成後儲存為 `origin.mp4`

   **Cell 3 - 本地推論、追蹤、存入 SQLite**
   ```python
   # 逐幀本地推論（YOLO 偵測 + ByteTrack 追蹤），將標註、時間存入 SQLite
   ```
   - 載入 `MODEL_PATH` 指定的本地 YOLO 模型
   - 對 `origin.mp4` 逐幀執行 `model.track()`（內建 ByteTrack 追蹤），取得每一影格含 `tracker_id` 的標註
   - 將標註與時間戳記存入 `detections.db`（SQLite）
   - 偵測與追蹤皆在本地完成

   **Cell 4 - 讀取 SQLite、繪製、輸出影片**
   ```python
   # 從 SQLite 解析標註、計數並重畫到原影片，輸出 labeled.mp4
   ```
   - 從 `detections.db` 讀取所有影格的標註
   - 解析標註，並對**跨越計數線段**的車輛進行去重計數（詳見下方「計數規則」）
   - 將標註與計數重畫回原影片，完成後儲存為 `labeled.mp4`

### 輸出檔案

| 檔案 | 說明 |
|------|------|
| `origin.mp4` | 從 CCTV 錄製的原始影片 |
| `detections.db` | 本地推論回傳的標註與時間戳記（SQLite，由 Cell 3 產生、Cell 4 讀取） |
| `labeled.mp4` | 標註版本（含所有車輛類別計數、計數線） |

### 影片視覺化說明

`labeled.mp4` 中的每一幀都會顯示：
- **計數線段**：位於 `y = 120` 像素、`x = 0 ~ 160` 的青色水平線段（只畫出這一段，而非整條橫線）
- **計數文字**（左上角，黃色）：
  ```
  Total: 42
  car: 35
  motorcycle: 7
  truck: 2
  ...
  ```
  （顯示所有被偵測到的類別）

### 計數規則

計數的依據是「車輛是否**跨越**計數線段」：

- **計數線段**：`COUNT_LINE_Y = 120`、`COUNT_LINE_X_MIN = 0`、`COUNT_LINE_X_MAX = 160`，
  即 `y = 120`、`x ∈ [0, 160]` 的水平線段（可於 Cell 4 開頭調整這三個常數）。
- **跨線判定（雙向）**：依 `tracker_id` 追蹤每台車的中心點軌跡，當同一 id 前後兩次出現
  分屬計數線兩側（由上往下或由下往上皆計），即視為一次跨越。
- **x 範圍判定**：取該車前後兩點「連線」與 `y = 120` 的交點 x，交點落在 `[0, 160]` 才計數。
  使用連線交點（而非單一影格的中心 x）可在偵測框閃爍、接近線時掉幀的情況下仍正確計數。
- **去重**：每個 `tracker_id` 只計一次，計入其對應的車輛類別。
- **已知限制**：若車輛閃爍導致追蹤器重新指派新的 `tracker_id`（軌跡斷裂），
  以軌跡連線為基礎的方法無法將斷成兩段的軌跡接回，這類情況可能漏算。


## 本地模型設定

### 模型需求

- 格式：**Ultralytics YOLO `.pt`**（程式以 `from ultralytics import YOLO` 載入）。
- Cell 3 以 `model.track(..., tracker="bytetrack.yaml")` 進行偵測＋追蹤，`bytetrack.yaml`
  為 Ultralytics 內建，無需額外檔案。
- 模型的類別名稱（`model.names`）即為計數時的分組依據；若要區分 `car` / `motorcycle` /
  `truck` 等，模型需以對應類別訓練。

### 指定模型與門檻

於 `.env` 設定（皆為選填，有預設值）：

```env
MODEL_PATH=model.pt   # 你的模型路徑
CONFIDENCE=0.25       # 偵測信心門檻（Ultralytics 預設；過高會濾掉大多數偵測，可依模型表現調整）
```

## 參考資源

- **Ultralytics YOLO 文件**：https://docs.ultralytics.com/
- **Ultralytics 追蹤（ByteTrack）**：https://docs.ultralytics.com/modes/track/
- **OpenCV 文件**：https://docs.opencv.org/
- **Jupyter Lab 使用教學**：https://jupyter.org/

## 授權

本專案使用 Ultralytics YOLO 進行本地推論。請留意 Ultralytics 採 AGPL-3.0 授權，
商用時請參閱其授權條款。
