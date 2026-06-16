# trainYOLO

用 `dataset/`（Roboflow YOLO11 格式，4 類別：bus / car / motorcycle / truck）訓練一個 **YOLO11-nano** 偵測模型，輸出 `model.pt`。

## 檔案

| 檔案 | 用途 |
|------|------|
| `train_colab.ipynb` | **Colab 免費 GPU (T4) 訓練**（建議；本機 Intel UHD 630 無法加速） |
| `train.ipynb` | 本機訓練（Python 3.10 + `.venv`；CPU，較慢） |
| `dataset/` | 資料集（train 164 / valid 32 / test 13） |
| `model.pt` | 訓練輸出的最佳權重（執行後產生） |

## 快速開始 — Colab（建議）

1. 本機打包資料集：
   ```powershell
   Compress-Archive -Path dataset -DestinationPath dataset.zip -Force
   ```
2. 開 [Google Colab](https://colab.research.google.com/) 上傳 `train_colab.ipynb`，
   選 **Runtime → Change runtime type → T4 GPU**。
3. 由上而下執行所有 cell，在上傳 cell 選 `dataset.zip`。
4. 完成後瀏覽器自動下載 `model.pt`，放回專案根目錄。

## 快速開始 — 本機

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install ultralytics albumentations ipykernel
```
在 VSCode / Jupyter 選 `.venv` kernel，執行 `train.ipynb` 所有 cell。

## 重點設計

- **修正 data.yaml**：原檔相對路徑易解析錯誤，notebook 執行時動態產生絕對路徑的 `data_fixed.yaml`（不改原檔）。
- **資料增強**：注入 Albumentations 的模糊 (Blur/MedianBlur) 與雜訊 (GaussNoise/ISONoise)。
- **強化 motorcycle 召回**：含 motorcycle 影像過採樣 4x、`imgsz=1280`、`cls=1.0`、強化 scale/mosaic/mixup、`patience` early stopping；最後印出逐類 P/R。

> 部署時調低偵測 `conf`（如 `model.predict(conf=0.15)`）可進一步提升召回。
