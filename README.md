# 🚀 NTNU EE- AGILAB - 硬體組 Git & Vivado Tcl 協作規範

本文件為實驗室硬體開發的標準作業程序 (SOP)，規範**Vivado Tcl 腳本** 的使用，確保跨裝置開發時不會出現路徑報錯。

---

## 🛠️ 開發環境 (Environment)
* **EDA Tool:** Xilinx Vivado (建議統一使用 2024.x 版本)
* **Target Board:** PYNQ-Z2 / FPGA SoC
* **Version Control:** Git

---

### 第二部分：SOP 指令與 Checklist


---

## ✍️ 導出教學 (Exporter SOP)
在 Commit 提交程式前，請務必更新 Tcl 腳本，以反映最新的電路修改：

1. **移除二進制依賴**：
   在 Sources 視窗中，將 `Utility Sources` -> `utils_1` 內的所有 `.dcp` 檔案按右鍵選擇 **Remove from Project**。
2. **重設合成狀態**：
   在左側 Design Runs 視窗中，對著 synth_1 按右鍵，點選 Reset Runs。(這會清空舊的編譯紀錄，確保 Tcl 導出時是乾淨的)
3. **導出專案腳本 (Project Tcl)**：
   `File` -> `Project` -> `Write Tcl...`
   * **關鍵：** **取消勾選** `Copy sources to project`。
4. **導出 BD 腳本 (Block Design Tcl)**：
   若有使用 AXI DMA或是其他有block diagram的專案，點選 `File` -> `Export` -> `Export Block Design...`。

---

## 🔄 重建與更新教學 (Rebuilder SOP)
當你從 GitHub `git pull` 更新後，請遵循「砍掉重練」原則：

### 1. 刪除本地舊專案
手動刪除由 Tcl 自動產生的專案資料夾（例如 `ProjectName/`）。

### 2. 執行一鍵重建
開啟 Vivado，在下方的 Tcl Console 中輸入：
```tcl
cd [你的專案路徑]
source project_config.tcl
source design_1.tcl
```

📊 硬體 Fit 流程檢查清單 (Hardware Fit Checklist)

Timing (WNS) > 0 ns：時序必須通過，否則硬體無法運作。

Utilization：監控 LUT/DSP 使用率，確保符合晶片資源。

Power Analysis：導出功耗分析報告。

