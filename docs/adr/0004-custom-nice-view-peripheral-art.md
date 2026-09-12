---
status: accepted
date: 2026-09-13
---

# 換掉右手螢幕的圖，但不複製也不 fork 上游的程式碼

右手螢幕上那張圖寫死在 ZMK 專案的 `app/boards/shields/nice_view/widgets/art.c`，
而本 repo 只透過 `config/west.yml` 釘 `zmkfirmware/zmk` v0.3.0，不持有那份原始碼。
做法是新增 shield `nice_view_photo`，**整個 shield 只有四個設定檔加一個 C 檔**：

```
boards/shields/nice_view_photo/
├── Kconfig.shield          Zephyr 靠它找到 shield；順便 select LVGL 需要的字型與元件
├── nice_view_photo.overlay 空的。Zephyr 靠 `<name>.overlay` 決定 shield 叫什麼名字
├── nice_view_photo.conf    只有一行 CONFIG_NICE_VIEW_WIDGET_STATUS=n
├── CMakeLists.txt          指到上游路徑編譯三個檔案，加一道 EXISTS 檢查
└── photo.c                 圖的點陣資料 + zmk_display_status_screen()
```

`util.c` / `bolt.c` / `peripheral_status.c` 本 repo 一行都不需要改，所以不複製，
由 `CMakeLists.txt` 直接指到 west workspace 裡的上游路徑編譯。

## 為什麼

圖是 `lv_img_dsc_t` 常數，編譯期決定，沒有任何 Kconfig 或 devicetree 開關能換掉。
而 `nice_view` 的 `CMakeLists.txt` 是整套一起編的 —— 只要它編譯，
`zmk_display_status_screen()` 與 `zmk_widget_status_init()` 就會被定義，再加上自己的
版本必然重複符號。所以**任何換圖方案都得先讓上游那套不編譯**，差別只在替代品從哪來。

選「指路徑而非複製」的理由：

- **沒有東西需要跟上游同步。** 那三個檔案本 repo 一行都不需要改，複製進來只會在
  ZMK 升級時多出三份要人工比對的檔案。
- **`peripheral_status.c` 的兩行差異用符號解決。** 它會在 `balloon` 與 `mountain`
  之間隨機挑一張；`photo.c` 讓這兩個名字都指向同一份點陣資料，隨機就沒有作用了。
  為了改兩行而複製 129 行不划算。
- **不動 `west.yml`。** [ADR 0001](0001-defer-shield-and-fork-migration.md) 的判準是
  穩定性優先、避免依賴浮動分支與個人 fork。為了一張圖去 fork ZMK 或改指第三方模組，
  代價與收益完全不成比例。
- **爆炸範圍侷限在右手的畫面。** 這個 shield 只在 peripheral 編譯，掛到 central 會在
  CMake 階段直接 `FATAL_ERROR`。就算整個壞掉，左手、按鍵、連線都不受影響。

## 考慮過但未採用

**把上游那幾個檔案複製進本 repo。** 最初就是這樣做的，共 6 個檔案約 380 行。好處是
自我完備、不依賴上游的目錄結構；壞處是其中 5 個檔案一行都沒改，卻從此要跟著上游維護。
判準是：沒有改動的東西不該複製。

**留一支 PNG → C 陣列的轉換腳本與來源 PNG。** 中間也做過。但那是兩個平常只會佔版面、
一年用不到一次的檔案，而這個 shield 的每個檔案都要有人讀。換圖時重做一次轉換即可。

**`CONFIG_ZMK_DISPLAY_STATUS_SCREEN_BUILT_IN=y`。** 改用 ZMK 內建的純文字狀態畫面，
一行設定就能達成。但它把圖整個拿掉了，而「換掉那張圖」正是需求本身。

**引第三方模組。** `build.yaml` 裡原本註解著上游舊架構留下的 `nice_view_custom`，
來自 `GPeye/urchin-peripheral-animation`。它追浮動分支、2024 年後上游自己也不再用，
與 ADR 0001 的判準牴觸。那行註解已一併移除。

**fork ZMK 直接改 `art.c`。** 改動最小，但代價是本 repo 從此得維護一份 ZMK 分支。
與 ADR 0001 明確拒絕的方向相同。

## 後果

- **依賴上游的目錄結構。** `CMakeLists.txt` 假設 `nice_view` 位在
  `${CMAKE_SOURCE_DIR}/boards/shields/nice_view`（`CMAKE_SOURCE_DIR` 在 ZMK 的建置裡
  就是 `zmk/app`，上游自己也是這樣取 include 目錄的）。若 ZMK 重組目錄，建置會失敗 ——
  這是刻意的：那裡有一道 `EXISTS` 檢查，會印出明確訊息而不是一串連結錯誤。
- **`balloon` / `mountain` 這兩個名字會出現在 `photo.c` 裡。** 它們與圖的內容無關，
  純粹是為了滿足上游的 `LV_IMG_DECLARE`。
- **`photo.c` 裡的 status screen 刻意與上游不同。** 上游的 `custom_status_screen.c`
  在 peripheral build 也 include central 版的 `widgets/status.h`，多配置兩個 68×68 的
  canvas buffer；這裡 include `peripheral_status.h`，省下約 9 KB RAM。
  （也不能直接編譯上游那個檔案 —— 它整段被 `CONFIG_NICE_VIEW_WIDGET_STATUS` 包住，
  拿來編會得到空白畫面。）
- 重新評估的觸發條件：ZMK 提供官方的自訂 art 機制、或上游把 `nice_view` 的目錄結構
  或 widget 介面大改。

## 換圖

直接改 `photo.c` 裡 `photo_map[]` 的位元組。格式是 **140×68 的 LVGL
`INDEXED_1BIT`**：前 8 個 byte 是調色盤（不要動），之後每列 18 bytes、高位在前、
**1 = 白**，共 68 列。顯示時整塊逆時針轉 90 度，所以眼睛看到的是 68 寬×140 高。

`.data_size` 用的是 `sizeof(photo_map)`，長度變了也不必手動改。

從圖片產生位元組的轉換工具刻意不留在 repo 裡，要用時重做一個即可 —— 上面那段格式
說明就是全部需要知道的東西。
