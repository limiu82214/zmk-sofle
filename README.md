# zmk-sofle

Eyelash Sofle 的 ZMK 韌體設定（個人分支）。

上游（產品資訊與購買）：<https://github.com/a741725193/zmk-sofle>

## 鍵位圖

![Eyelash Sofle keymap](keymap-drawer/eyelash_sofle.svg)

由 `.github/workflows/draw.yml` 使用 [keymap-drawer](https://github.com/caksoylar/keymap-drawer)
自動產生。每次 `config/` 有變動就會重畫並 commit 回本分支，不需手動更新。

## 日常會動到的只有 `config/`

| 檔案 | 用途 |
| --- | --- |
| `config/eyelash_sofle.keymap` | **按鍵配置**，目前 4 層 |
| `config/eyelash_sofle.conf` | 韌體設定：underglow、螢幕背光、debounce、soft-off |
| `config/eyelash_sofle.json` | 實體佈局，給 [keymap-editor](https://nickcoutsos.github.io/keymap-editor/) 用 |
| `config/west.yml` | ZMK 來源與版本（釘在 `zmkfirmware/zmk` v0.3.0） |

改完 push，GitHub Actions 會自動編譯，到 Actions 頁面下載 artifact 裡的 `.uf2`。左右手都要刷。

改 keymap 有三種方式：直接編 `.keymap`、用 [keymap-editor](https://nickcoutsos.github.io/keymap-editor/)（讀 `config/eyelash_sofle.json` 取得實體佈局）、或用 [ZMK Studio](https://zmk.studio/)（佈局是從韌體內的 `zmk,physical-layout` 讀的，不經過這個 repo）。

## BLE profile 的切換與斷線

都綁在 media layer 的左手側，與螢幕上那五個圈圈對應：

| 位置 | binding | 作用 |
| --- | --- | --- |
| 第一排，左起第 2–6 格 | `&bt BT_SEL 0`–`4` | 切到第 n 個（0 起算）profile |
| 第二排，左起第 2–6 格 | `&bt BT_DISC 0`–`4` | 斷開第 n 個（0 起算）profile，**配對保留** |
| 第四排，左半最右格 | `&bt BT_CLR` | 解除**當前** profile 的配對（斷線 + 忘記這台主機） |

> **螢幕上的編號比 keymap 參數大 1。** 螢幕畫的是 `i + 1`，`&bt` 吃的是 0-based index。
> 螢幕顯示「2」的那個圈圈，要用 `&bt BT_DISC 1` 才斷得掉。鍵位圖上的數字已經換算成
> 螢幕的編號，照圖按即可，不用心算。

五個圈圈排成骰子五點（四角加正中），編號畫在圈圈裡。圈圈的長相就是該 profile 的狀態：

| 圈圈 | 狀態 |
| --- | --- |
| 實線整圈 | 連線中 |
| 8 段虛線圈 | 已配對，但當下沒連上 |
| 沒有圈，只有數字 | 沒配對過 |
| 中間多一個實心點 | 這是**當前** profile（與連線狀態無關，三種都可能疊加） |

三件按下去之前該知道的事：

- **`&bt BT_DISC n` 斷的是「第 n 個（0 起算）profile」，不是「目前這條連線」。** ZMK 沒有
  「斷開當前連線」這個指令；唯一作用在當前 profile 的是 `BT_CLR`，但它會把配對一起刪掉。
- **對未配對或當下未連線的 profile 按 `BT_DISC`，韌體回 `-ENODEV`，畫面不會有任何反應。**
  看起來像「斷不開」，其實是本來就沒連著 —— 圈圈本來就不是實線整圈。斷成功的話，該
  profile 的圈圈會從實線整圈變成 8 段虛線圈。
- **斷開後主機可能會馬上自己連回來**，macOS 尤其如此。這在 BLE 協定層面擋不住 —— 鍵盤
  只能斷開，不能阻止對方重新發起連線。要確實甩開得在主機上操作。

所以**「想讓鍵盤別再打字到某台主機」不該用 `BT_DISC`**：

| 想達到 | 該用 | 說明 |
| --- | --- | --- |
| 按鍵改送到別台 | `&bt BT_SEL n` | 舊連線還在，但按鍵不再送過去。多數情況要的是這個 |
| 按鍵改走 USB | `&out OUT_USB` | endpoint 與 profile 正交，BLE 連線會留著 |
| 對方再也連不回來 | `&bt BT_CLR` | 配對一併刪掉，下次要重新配對 |

切 profile 和切 endpoint 都**不會**斷開既有的 BLE 連線 —— `zmk_ble_prof_select()` 只改
當前 profile、存檔、更新廣播，沒有一行去動連線。所以切走之後舊 profile 的圈圈仍可能是
實線整圈，那是真的還連著，只是按鍵沒往那邊送。

> **圈圈不是即時的。** widget 只訂閱 `zmk_endpoint_changed`、`zmk_usb_conn_state_changed`、
> `zmk_ble_active_profile_changed` 三個事件，沒有訂閱 BLE 連線事件；而 `ble.c` 的
> connected / disconnected callback 只在 `is_conn_active_profile()` 成立時才發事件。
> 結論：**非當前 profile 的圈圈可能是舊畫面**，切回該 profile 會強制重讀。

`&bt BT_CLR` 原本放在第二排 `BT_SEL 4` 的正下方，位置太好按，而那一格正是 `BT_DISC 4`
該在的地方。現已移到第四排左半最右，與 SEL / DISC 兩排隔著 endpoint 那排 —— 想斷線時
按錯而把配對清掉的機會小得多。

## 目錄結構

```
.
├── build.yaml                  建置目標：左手(studio) / 右手 / settings_reset
├── config/                     ★ 日常設定，見上表
├── boards/arm/eyelash_sofle/   自建 board 定義（self-contained，MCU 焊死在 PCB 上）
├── boards/shields/nice_view_photo/  右手螢幕上那張圖，見下節
├── keymap-drawer/              keymap 圖，CI 自動產生，請勿手動編輯
├── keymap_drawer.config.yaml   keymap 圖的樣式與圖示對應
├── zephyr/module.yml           讓本 repo 能被當成 ZMK module（board_root）
├── .github/workflows/
│   ├── build.yml               編譯韌體，釘在 ZMK v0.3.0
│   └── draw.yml                產生 keymap 圖並 commit 回本分支
├── AGENTS.md                   AI agent 指引
├── docs/adr/                   架構決策紀錄
└── docs/agents/                agent skill 設定（issue tracker / triage / domain）
```

### `boards/arm/eyelash_sofle/`

這把鍵盤是 self-contained（nRF52840 直接焊在 PCB 上），所以在 ZMK 裡是 **board** 而不是 shield。

| 檔案 | 用途 |
| --- | --- |
| `eyelash_sofle.dtsi` | 共用硬體：矩陣掃描、編碼器、WS2812、螢幕背光 PWM、nice_view SPI |
| `eyelash_sofle-layouts.dtsi` | 實體佈局，64 顆鍵的座標（含拇指鍵旋轉），ZMK Studio 靠它繪圖 |
| `eyelash_sofle_left.dts` / `_right.dts` | 左右手各自的 board |
| `eyelash_sofle_left_defconfig` / `_right_defconfig` | 左右手的 Kconfig（左手為 split central） |
| `Kconfig.board` / `Kconfig.defconfig` | board 宣告與預設值 |
| `board.cmake` | 燒錄器設定 |
| `eyelash_sofle.keymap` | board 內建的預設 keymap，實際會被 `config/` 的蓋掉 |
| `eyelash_sofle.yaml` / `.zmk.yml` | Zephyr twister 與 ZMK 硬體 metadata |

## 右手螢幕上那張圖

兩半各有一塊 nice!view，但畫面內容完全不同：左手是上面講的那組狀態顯示，右手則是
一張圖加上角落的電量與連線符號。這張圖在 ZMK 專案裡是寫死的（開機在兩張內建圖之間
隨機挑一張），本 repo 用自己的 shield `nice_view_photo` 把它換成固定的自訂圖。

整個 shield 只有四個設定檔加一個 C 檔，且不複製任何上游程式碼 —— 需要的三個檔案由
CMake 直接指到 ZMK 原始碼的路徑編譯。換圖就是直接改 `photo.c` 裡 `photo_map[]` 的
位元組（140×68 的 LVGL `INDEXED_1BIT`）。

格式細節與這個設計的理由都在 [ADR 0004](docs/adr/0004-custom-nice-view-peripheral-art.md)。

## 與上游的關係

本分支停在**舊架構**（自建 board + ZMK 專案 v0.3.0）。上游已於 2026-06 改為
shield 架構並把 ZMK 來源改指 cormoran fork，本分支兩者都未跟進，決定與理由記在
[ADR 0001](docs/adr/0001-defer-shield-and-fork-migration.md)。

上游仍保留為 `upstream` remote 供對照：

```bash
git fetch upstream
git diff nimo upstream/main -- <path>          # 比對差異
git show upstream/main:<path>                  # 取回清理掉的檔案
```
