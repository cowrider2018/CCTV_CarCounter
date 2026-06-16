# CCTV_CarCounter - 車輛計數系統

使用 Roboflow 推論 API 和 CCTV 攝像機串流，自動偵測並計數跨越指定計數線段的所有車輛類別。

## 環境需求

### Python 版本
- **建議：Python 3.10**
- 支援：Python 3.8+
- 不支援：Python 3.12+（inference-sdk 相容性問題）

### 系統依賴
- ffmpeg（用於 CCTV 串流錄製和視頻編碼）
  - **Windows**: 可從 [ffmpeg.org](https://ffmpeg.org/download.html) 下載，或使用 `choco install ffmpeg`
  - **macOS**: `brew install ffmpeg`
  - **Linux**: `sudo apt-get install ffmpeg`

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
# Roboflow API 金鑰（必填）
# 從你的 https://app.roboflow.com/settings/api 取得
ROBOFLOW_API_KEY=your_api_key_here

# CCTV 串流 URL（必填）
# 例如：https://cctvtraffic.tycg.gov.tw/camera183/
CCTV_URL=https://your-cctv-stream-url/

# --- 以下為選填（進階自定義用）---
# 預設不需設定：程式碼已直接指向作者已發布的公開工作流 cowrider2018/carcounter。
# 只有當你想改用「自己的」工作流時才需取消註解並填入，詳見下方「使用作者的工作流 / 如何自定義」。
# ROBOFLOW_WORKSPACE=your_workspace_name
# ROBOFLOW_WORKFLOW=your_workflow_id
```

> 一般使用者只要填好「自己的」`ROBOFLOW_API_KEY` 與 `CCTV_URL` 即可，
> 程式會以你的金鑰呼叫作者已發布的 `cowrider2018/carcounter` 工作流（私有模型在 Roboflow 伺服器端執行）。

### 獲取 Roboflow API 金鑰
1. 前往 [Roboflow 官網](https://roboflow.com/) 並登入帳戶
2. 進入 [API 設定頁面](https://app.roboflow.com/settings/api)
3. 複製 **Private API Key**
4. 貼到 `.env` 的 `ROBOFLOW_API_KEY`

## 專案結構

```
carCounter/
├── main.ipynb              # 主程式（Jupyter Notebook）
├── requirements.txt        # Python 依賴清單
├── .env                    # 環境變數配置（勿上傳）
├── .gitignore             # Git 忽略規則文件
├── origin.mp4             # 錄製的原始影片（自動生成）
├── roboflow_data.db       # Roboflow 標註與時間戳記資料庫（SQLite，自動生成）
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

   **Cell 3 - 切分影片、上傳 Roboflow、存入 SQLite**
   ```python
   # 切分影片並上傳 Roboflow，將標註、時間存入 SQLite
   ```
   - 將 `origin.mp4` 以每 10 分鐘為一段切片，避免單次上傳過大導致 `RuntimeError`
   - 逐段上傳到 Roboflow 推論 API，取得每一影格的標註
   - 將標註與時間戳記原樣存入 `roboflow_data.db`（SQLite）

   **Cell 4 - 讀取 SQLite、繪製、輸出影片**
   ```python
   # 從 SQLite 解析標註、計數並重畫到原影片，輸出 labeled.mp4
   ```
   - 從 `roboflow_data.db` 讀取所有影格的標註
   - 解析標註，並對**跨越計數線段**的車輛進行去重計數（詳見下方「計數規則」）
   - 將標註與計數重畫回原影片，完成後儲存為 `labeled.mp4`

### 輸出檔案

| 檔案 | 說明 |
|------|------|
| `origin.mp4` | 從 CCTV 錄製的原始影片 |
| `roboflow_data.db` | Roboflow 回傳的標註與時間戳記（SQLite，由 Cell 3 產生、Cell 4 讀取） |
| `labeled.mp4` | 標註版本（含所有車輛類別計數、計數線） |

### 影片視覺化說明

`labeled.mp4` 中的每一幀都會顯示：
- **計數線段**：位於 `y = 100` 像素、`x = 0 ~ 160` 的青色水平線段（只畫出這一段，而非整條橫線）
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

- **計數線段**：`COUNT_LINE_Y = 100`、`COUNT_LINE_X_MIN = 0`、`COUNT_LINE_X_MAX = 160`，
  即 `y = 100`、`x ∈ [0, 160]` 的水平線段（可於 Cell 4 開頭調整這三個常數）。
- **跨線判定（雙向）**：依 `tracker_id` 追蹤每台車的中心點軌跡，當同一 id 前後兩次出現
  分屬計數線兩側（由上往下或由下往上皆計），即視為一次跨越。
- **x 範圍判定**：取該車前後兩點「連線」與 `y = 100` 的交點 x，交點落在 `[0, 160]` 才計數。
  使用連線交點（而非單一影格的中心 x）可在偵測框閃爍、接近線時掉幀的情況下仍正確計數。
- **去重**：每個 `tracker_id` 只計一次，計入其對應的車輛類別。
- **已知限制**：若車輛閃爍導致追蹤器重新指派新的 `tracker_id`（軌跡斷裂），
  以軌跡連線為基礎的方法無法將斷成兩段的軌跡接回，這類情況可能漏算。


## 使用作者的工作流 / 如何自定義

### 預設行為

程式碼已預設**直接指向作者已發布的公開工作流 `cowrider2018/carcounter`**。
你只需在 `.env` 填入「自己的」`ROBOFLOW_API_KEY`，程式就會以你的金鑰呼叫該工作流——
偵測、追蹤與分類所用的私有模型都在 Roboflow 伺服器端執行，你無須自行訓練或上傳模型。

工作流名稱與 workspace 定義於 `main.ipynb` 的 Cell 3：

```python
ROBOFLOW_WORKSPACE = os.environ.get("ROBOFLOW_WORKSPACE", "cowrider2018")
ROBOFLOW_WORKFLOW  = os.environ.get("ROBOFLOW_WORKFLOW", "carcounter")
```

### `workflow.JSON` 的用途

`workflow.JSON` 是上述 `carcounter` 工作流的**完整定義**，供想複製或自建工作流的人參考。

### 如何自定義（改用自己的工作流）

1. 在 Roboflow 進入自己的 workspace，建立或進入已存在的 workflow，點擊左上角 </> 符號開啟 JSON 編輯，複製 `workflow.JSON` 內容並填入或覆蓋
   （如果你想用自己的模型，也可以自行修改參數）。
2. 在 `.env` 取消註解並填入：
   ```env
   ROBOFLOW_WORKSPACE=你的_workspace_名稱
   ROBOFLOW_WORKFLOW=你的_workflow_id
   ```
3. 重新執行 `main.ipynb` 的 Cell 3，即會改用你自己的工作流（`.env` 的值會覆寫上述預設）。

## 參考資源

- **Roboflow 官網**：https://roboflow.com/
- **Roboflow Python SDK**：https://docs.roboflow.com/deploy/python
- **OpenCV 文件**：https://docs.opencv.org/
- **Jupyter Lab 使用教學**：https://jupyter.org/

## 授權

本專案使用 Roboflow 推論 API。詳細條款請參閱 Roboflow 服務協議。
