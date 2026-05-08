# CCTV_CarCounter - 車輛計數系統

使用 Roboflow 推論 API 和 CCTV 攝像機串流，自動偵測並計數經過指定線路的所有車輛類別。

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
# 從 https://app.roboflow.com/settings/api 取得
ROBOFLOW_API_KEY=your_api_key_here

# Roboflow Workspace 名稱（必填）
# 通常是你在 Roboflow 的使用者名稱或組織名稱
ROBOFLOW_WORKSPACE=your_workspace_name

# CCTV 串流 URL（必填）
# 例如：https://cctvtraffic.tycg.gov.tw/camera183/
CCTV_URL=https://your-cctv-stream-url/
```

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

   **Cell 1 - 安裝套件**（可選，僅首次運行）
   ```python
   %pip install --upgrade pip
   %pip install -r requirements.txt
   ```
   - 點擊 ▶ 執行

   **Cell 2 - 錄製影片素材**
   ```python
   # 錄製影片素材
   ```
   - 執行此儲存格以從 CCTV 錄製影片
   - 完成後儲存為 `origin.mp4`

   **Cell 3 - 上傳、偵測、計數**
   ```python
   # 上傳影片並取得標註、計數
   ```
   - 執行此儲存格以上傳影片到 Roboflow 推論
   - 完成後儲存為 `labeled.mp4`

### 輸出檔案

| 檔案 | 說明 |
|------|------|
| `origin.mp4` | 從 CCTV 錄製的原始影片 |
| `labeled.mp4` | 標註版本（含所有車輛類別計數、計數線） |

### 影片視覺化說明

`labeled.mp4` 中的每一幀都會顯示：
- **計數線**：y = 60 像素的青色水平線（以上的車輛太模糊，不計算）
- **計數文字**（左上角，青色）：
  ```
  Total: 42
  car: 35
  motorcycle: 7
  truck: 2
  ...
  ```
  （顯示所有被偵測到的類別）


## 參考資源

- **Roboflow 官網**：https://roboflow.com/
- **Roboflow Python SDK**：https://docs.roboflow.com/deploy/python
- **OpenCV 文件**：https://docs.opencv.org/
- **Jupyter Lab 使用教學**：https://jupyter.org/

## 授權

本專案使用 Roboflow 推論 API。詳細條款請參閱 Roboflow 服務協議。
