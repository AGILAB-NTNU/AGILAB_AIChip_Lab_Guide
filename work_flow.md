# 🚀 AGILAB AI 晶片軟硬整合開發流程與檢核規範

本文件定義了 AI 加速晶片（AI Accelerator）從演算法定調、硬體架構設計到 SOC 系統級整合的完整開發生命週期。每個階段皆包含明確的**實作內容**與**檢核評估方式（Checklists）**，以確保軟硬體銜接順暢並達成 PPA 目標。

## 📌 階段一：演算法規格與軟體定義 (Software Specification)

### 1. 實作內容

- **核心任務**：明確定義欲加速的軟體運算功能（例如：全連接層 Fully-Connected Layer、卷積層或注意力機制）。
- **參數釐清**：釐清算子所需的輸入/輸出引數、矩陣維度以及資料流型態。
- **資料格式與量化策略**：資料精度（例如：`INT8`、`FP16` 或自訂定點數 Fixed-point）。

### 2. 檢核評估方式 (Checklist)

- [ ] **浮點數 Golden Model 建立**：是否已使用 Python/PyTorch 建立未量化前的浮點數黃金模型，並確認演算法本身的預期準確度？
- [ ] **誤差容忍度定義**：是否已明確定義與浮點數對比後的硬體輸出「最大允許誤差許可範圍」？
- [ ] **邊界條件定義**：是否已窮舉所有極端輸入尺寸（如矩陣維度為 0、1 或極大值）的規格？

## 📌 階段二：Golden算法模擬與架構探索 (Golden Model Simulation)

### 1. 實作內容

- **核心任務**：使用 Python（如 NumPy）模擬硬體剪裁位元數後的溢位（Overflow）與精確度損失。
- **架構探索**：依據資料格式，初步評估硬體 Datapath 的基礎框架與資料流向（如 Weight-Stationary 或是 Output-Stationary）。

### 2. 檢核評估方式 (Checklist)

- [ ] **定點數模擬達標**：Python 定點數模擬的精確度與誤差，是否百分之百符合「階段一」設定的誤差許可範圍？
- [ ] **測試測資生成（Testbench Vector）**：是否已將 Python 模擬的 **輸入資料** 與 **預期正確答案（Golden Output）** 導出為文字檔（如 `.txt` 或 `.vec`），用作後續 RTL 驗證的模擬激勵源？

## 📌 階段三：硬體資料路徑開發 (Datapath RTL Design)

### 1. 實作內容

- **核心任務**：根據板子（如 PYNQ-ZU）的硬體資源限制，開發運算核心的 Datapath（利用 Verilog / SystemVerilog 或 Vivado HLS）。
- **時序優化**：實現流水線（Pipeline）架構，以優化硬體運算時脈。

### 2. 檢核評估方式 (Checklist)

- [ ] **RTL 語法與編譯**：RTL 程式碼是否通過語法檢查（Linting），且沒有任何潛在的問號/未定義狀態？
- [ ] **流水線與吞吐量驗證**：硬體 Pipeline 是否有非預期的 Bubble（空轉週期）？乘加器（MAC/DSP）的利用率是否達到架構預期？
- [ ] **資源預估安全線**：初步的邏輯合成資源預估，是否未超過開發板總資源的 **75%** （預留彈性給後續的系統整合）？

## 📌 階段四：控制器設計與局部驗證 (Controller & Standalone Simulation)

### 1. 實作內容

- **核心任務**：設計有限狀態機（FSM）控制器與暫存器映射表（Register Map），例如以 `CTRL`、`STATUS`、`BRAM_BASE_ADDR`、`VECTOR_LEN` 等控制暫存器定義啟動、狀態回報與資料位址/長度設定。
- **範例 Register Map**：

| Offset | Register Name | Access | Width | Description |
| --- | --- | --- | --- | --- |
| `0x00` | `CTRL` | RW | 32-bit | `[0]: Start Pulse (Self-clearing)` |
| `0x04` | `STATUS` | RO | 32-bit | `[0]: Done Flag (High when computation finished)` |
| `0x08` | `BRAM_BASE_ADDR` | RW | 32-bit | Starting address of vector data in BRAM |
| `0x0C` | `VECTOR_LEN` | RW | 32-bit | Length of vector elements (Default: 10) |

- **介面接腳對齊**：確認控制器與本地儲存區（BRAM）之間的讀寫控制訊號（`clk`、`rst`、`en`、`we`、`addr`、`din`、`dout`）對接無誤。

### 2. 檢核評估方式 (Checklist)

- [ ] **暫存器映射表定調**：Register Map 是否已撰寫成標準文件？是否已確實排除 **Multi-driven（多重驅動）** 的程式碼地雷（即同一個暫存器沒有在多個 always 塊中被賦值）？
- [ ] **RTL Simulation 通過**：在 Vivado Simulator  中載入「階段二」生成的測試測資，硬體運算結果與 Python Golden Output 是否完全二進位吻合（Bit-accurate）？
- [ ] **週期精確度評估**：透過波形圖，是否已精確記錄硬體核心運算單次任務所需的 **精確 Clock Cycles 數量**？

## 📌 階段五：系統級資料搬移架構 (System-level Data Movement)

### 1. 實作內容

- **核心任務**：設計記憶體佈局（Memory Layout）。CPU 將大量參數放置於主記憶體（DRAM），並規劃透過高效能通道搬移至 FPGA 本地儲存區（BRAM）。
- **通訊協定分流**：大數據量搬移採用 **AXI-Stream** 或 **AXI-Full (DMA)**；控制控制訊號採用 **AXI-Lite**。

### 2. 檢核評估方式 (Checklist)

- [ ] **記憶體邊界與對齊**：Memory Layout 的定址是否滿足 CPU 與 DMA 的對齊限制（如 32-bit 或 64-bit 對齊），且無緩衝區溢位（Buffer Overflow）風險？
- [ ] **BRAM 通道衝突排除**：若採用雙通道 BRAM（True Dual Port），是否已隔離 CPU 寫入端（Port B）與硬體加速器讀取端（Port A），確保兩端在同一時脈下**絕不會讀寫衝突**？
- [ ] **Address Editor 映射成功**：在 Vivado Block Design 中，所有自訂 IP 與 BRAM Controller 是否皆已成功分派到合法的基底底址（Base Address）？

## 📌 階段六：軟硬整合控制與閉環驗證 (SOC Integration & PYNQ Verification)

### 1. 實作內容

- **核心任務**：將生成完硬體位元流（`.bit`）與手過檔（`.hwh`）上傳至 PYNQ-ZU 板載 Linux 系統。
- **軟體驅動編寫**：在 Jupyter Notebook 中使用 Python `pynq` 庫編寫控制程式，實現「Python 倒資料 $\rightarrow$ AXI-Lite 啟動硬體 $\rightarrow$ 輪詢監聽 Done $\rightarrow$ 讀回答案」的閉環測試。

### 2. 檢核評估方式 (Checklist)

- [ ] **硬體載入成功**：Python 執行 `Overlay("./xxx.bit")` 時是否能順利加載，且 `ol.ip_dict.keys()` 能正確識別出加速器 IP 名字？
- [ ] **狀態機握手成功（Handshake Polling）**：Python 寫入 Start 後，硬體是否能正確運作並拉高 Done 訊號？Python 的 Polling 機制是否能順利跳出迴圈（無死鎖/超時現象）？
- [ ] **端到端全面驗證（End-to-End Pass）**：Python 從 BRAM 讀回的最終計算結果，與「階段一/二」的 Python 原始演算法結果比對，是否完全符合誤差許可範圍？

## 📌 階段七：晶片效能全面分析 (Final PPA Analysis)

### 1. 實作內容

- **核心任務**：當軟硬體驗證 100% 正確後，全面提取晶片在實體佈線後的各項物理指標，進行量化評估。

### 2. 檢核評估方式 (Checklist)

- [ ] **Performance（效能評估）**：結合實體運算時間與 Cycles 數，計算出晶片的實際吞吐量（Throughput，如 `GOPs` 或 `GFLOPs`）與硬體效率。
- [ ] **Power（功耗評估）**：利用 Vivado `Report Power` 工具，分析硬體在運作時的動態功耗（Dynamic Power）與靜態功耗（Static Power）比例是否符合功耗預算？
- [ ] **Area（面積與資源佔用）**：檢查最終的 `Utilization Report`，確保 `LUT`、`FF`、`DSP` 和 `BRAM` 的消耗百分比分佈合理，且無任何時序違例（Timing Violation，如 Slack 必須為正值）。
