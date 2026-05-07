# CarCounter - 車輛計數系統

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
git clone <repository-url>
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

### 3. 安裝依賴
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

## 設定 .env 檔案

在專案根目錄建立 `.env` 檔案，填入以下內容：

```env
# Roboflow API 金鑰（必填）
# 從 https://app.roboflow.com/settings/api 取得
ROBOFLOW_API_KEY=your_api_key_here

# Roboflow Workspace 名稱（必填）
# 通常是你在 Roboflow 的使用者名稱或組織名稱
ROBOFLOW_WORKSPACE=your_workspace_name

# CCTV 串流 URL（必填）
# 支援 RTSP、HTTP 串流等
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
├── .env                    # 環境變數配置（勿提交至版本控制）
├── .gitignore             # 忽略規則
├── origin.mp4             # 錄製的原始影片（自動生成）
├── labeled.mp4            # 標註後的計數結果影片（自動生成）
└── README.md              # 本檔案
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
   - 按 `Shift + Enter` 或點擊 ▶ 執行

   **Cell 2 - 錄製影片素材**
   ```python
   # 錄製 20 秒的 CCTV 串流
   # 輸出：origin.mp4
   ```
   - 執行此儲存格以從 CCTV 錄製 20 秒影片
   - 進度條會顯示錄製百分比
   - 完成後儲存為 `origin.mp4`

   **Cell 3 - 上傳、偵測、計數**
   ```python
   # 主要邏輯：
   # 1. 上傳 origin.mp4 到 Roboflow
   # 2. 逐幀推論並計數
   # 3. 即時列印偵測結果
   # 4. 輸出：labeled.mp4（含計數視覺化）
   ```
   - 執行此儲存格進行完整的偵測和計數流程
   - 控制台會列印每個新偵測的車輛
   - 最終輸出 `labeled.mp4`

### 方式二：命令列執行（進階）

如果要以指令列執行（不開 Jupyter GUI），使用：
```bash
jupyter nbconvert --to notebook --execute --inplace main.ipynb
```

## 輸出說明

### 控制台輸出
```
🚀 開始錄製純影片...
📂 儲存路徑: origin.mp4
⏱️  預計時長: 20 秒
...
【偵測成功】ID: 123 | 類型: car | 總量: 5
【偵測成功】ID: 124 | 類型: motorcycle | 總量: 6
...
=== 最終計數 === Total=42, car=35, motorcycle=7
Done! 400 frames at 20.0 FPS -> labeled.mp4
```

### 輸出檔案

| 檔案 | 說明 |
|------|------|
| `origin.mp4` | 從 CCTV 錄製的原始 20 秒影片 |
| `labeled.mp4` | 標註版本（含所有車輛類別計數、計數線） |

### 影片視覺化

`labeled.mp4` 中的每一幀都會顯示：
- **計數線**：y = 60 像素的青色水平線（車輛需要越過此線才會被計數）
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
