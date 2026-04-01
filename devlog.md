# AM32 Bootloader 開發日誌

---

## 2026-03-23 — ESC Tool 透過 PA2 通訊失敗問題

### 問題
燒錄 `AM32_L431_BOOTLOADER_PA2_V13.hex` 後，使用 ESC Tool 嘗試與 bootloader 通訊時，PA2 訊號線無法被拉到 GND，導致通訊失敗。

### 原因
PA2 同時被兩個外部裝置驅動，造成匯流排衝突：
1. **Nucleo 開發板**：ST-Link 透過焊接橋 SB13 將 PA2 連接為 USART2 TX（虛擬 COM Port），ST-Link 持續將 PA2 拉高。
2. **Pixhawk 6C**：Pixhawk 的 DShot 訊號輸出也接在 PA2，Pixhawk 主動驅動此線。

只要上述任一裝置連接，ESC Tool 就無法把 PA2 拉到 GND 進行通訊。

### 修改方式／解決方法
- 使用 ESC Tool 燒錄 APP 時，**先拔除 Pixhawk 的訊號線**，確保 PA2 只有 ESC Tool 介面卡連接。
- 接線方式（單線雙向）：
  ```
  ESC Tool 介面卡 TX/RX ──── PA2
  GND ──────────────────────── GND
  ```
- Nucleo 開發板若需要長期使用，可考慮切斷背面 **SB13** 焊接橋，斷開 ST-Link 與 PA2 的連線。

### 提醒
- PA2 是本專案 ESC APP 的 **DShot 訊號輸入腳**，Bootloader 與 APP 共用此腳，設計正確。
- 每次使用 ESC Tool 燒錄或調參前，**必須先拔除 Pixhawk 訊號線**。
- Bootloader HEX 使用 `AM32_L431_BOOTLOADER_PA2`，目標 MCU 為 STM32L432KC（以 L431 代號編譯）。

---

## 2026-04-01 — ESC Config Tool 連線設定補充

### 問題
ESC Config Tool 1.93 連線後顯示 `BAD OR NO ACK FROM ESC`，無法讀取設定值。

### 原因
ESC Config Tool 在未勾選 `USB / Arduino Connect` 時，使用全雙工模式通訊，無法正確處理單線半雙工（TX/RX 短接）的回音（echo），導致協定解析失敗。

### 解決方法
在 ESC Config Tool 中**必須勾選 `USB / Arduino Connect`**，工具才會以正確的半雙工模式與 AM32 Bootloader 通訊。

### ESC Tool 正確操作步驟（Nucleo 開發環境）
1. 拔除 Pixhawk 訊號線（避免 PA2 匯流排衝突）
2. USB-UART 介面卡 TX/RX 短接後接 PA2，GND 接 GND
3. 開啟 ESC Config Tool，**勾選 `USB / Arduino Connect`**
4. 選擇對應 COM Port，按 Connect
5. 給 ESC（Nucleo）上電
6. 按 M1 連線

### 提醒
- 未勾選 `USB / Arduino Connect` → `BAD OR NO ACK FROM ESC`
- 已勾選 `USB / Arduino Connect` → 正常連線並可燒錄 APP 韌體

---
