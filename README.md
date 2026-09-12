# zmk-sofle

Eyelash Sofle 的 ZMK 韌體設定（個人分支）。

上游（產品資訊與購買）：<https://github.com/a741725193/zmk-sofle>

## 日常會動到的只有 `config/`

| 檔案 | 用途 |
| --- | --- |
| `config/eyelash_sofle.keymap` | **按鍵配置**，目前 4 層 |
| `config/eyelash_sofle.conf` | 韌體設定：RGB、背光、debounce、soft-off |
| `config/eyelash_sofle.json` | 實體佈局，給 [keymap-editor](https://nickcoutsos.github.io/keymap-editor/) 用 |
| `config/west.yml` | ZMK 來源與版本（釘在 `zmkfirmware/zmk` v0.3.0） |

改完 push，GitHub Actions 會自動編譯，到 Actions 頁面下載 artifact 裡的 `.uf2`。左右手都要刷。

改 keymap 有三種方式：直接編 `.keymap`、用 [keymap-editor](https://nickcoutsos.github.io/keymap-editor/)（讀 `config/eyelash_sofle.json` 取得實體佈局）、或用 [ZMK Studio](https://zmk.studio/)（佈局是從韌體內的 `zmk,physical-layout` 讀的，不經過這個 repo）。

## 目錄結構

```
.
├── build.yaml                  建置目標：左手(studio) / 右手 / settings_reset
├── config/                     ★ 日常設定，見上表
├── boards/arm/eyelash_sofle/   自建 board 定義（MCU 焊死在 PCB 上，非插拔式控制器）
├── keymap-drawer/              keymap 圖，CI 自動產生，請勿手動編輯
├── keymap_drawer.config.yaml   keymap 圖的樣式與圖示對應
├── zephyr/module.yml           讓本 repo 能被當成 ZMK module（board_root）
├── .github/workflows/
│   ├── build.yml               編譯韌體，釘在 ZMK v0.3.0
│   └── draw.yml                產生 keymap 圖並 commit 回本分支
├── AGENTS.md                   AI agent 指引
└── docs/agents/                agent skill 設定（issue tracker / triage / domain）
```

### `boards/arm/eyelash_sofle/`

這把鍵盤是 self-contained（nRF52840 直接焊在 PCB 上），所以在 ZMK 裡是 **board** 而不是 shield。

| 檔案 | 用途 |
| --- | --- |
| `eyelash_sofle.dtsi` | 共用硬體：矩陣掃描、編碼器、WS2812、PWM 背光、nice_view SPI |
| `eyelash_sofle-layouts.dtsi` | 實體佈局，64 顆鍵的座標（含拇指鍵旋轉），ZMK Studio 靠它繪圖 |
| `eyelash_sofle_left.dts` / `_right.dts` | 左右手各自的 board |
| `eyelash_sofle_left_defconfig` / `_right_defconfig` | 左右手的 Kconfig（左手為 split central） |
| `Kconfig.board` / `Kconfig.defconfig` | board 宣告與預設值 |
| `board.cmake` | 燒錄器設定 |
| `eyelash_sofle.keymap` | board 內建的預設 keymap，實際會被 `config/` 的蓋掉 |
| `eyelash_sofle.yaml` / `.zmk.yml` | Zephyr twister 與 ZMK 硬體 metadata |

## 與上游的關係

本分支停在**舊架構**（自建 board + 官方 ZMK v0.3.0）。上游已於 2026-06 改為
shield 架構並改用第三方的 ZMK fork（DYA Studio），本分支暫未跟進。

上游仍保留為 `upstream` remote 供對照：

```bash
git fetch upstream
git diff nimo upstream/main -- <path>          # 比對差異
git show upstream/main:<path>                  # 取回清理掉的檔案
```
