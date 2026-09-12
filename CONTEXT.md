# Eyelash Sofle 韌體設定

這個 repo 是 Eyelash Sofle 分體鍵盤的個人 ZMK 設定。它同時牽涉三方各自維護的專案，
以及四個外觀相近、職責卻完全不同的改鍵工具，術語極易互相污染。這份詞彙表用來固定說法。

## 上游關係

> **「官方」單獨使用是禁用詞。** 它可以同時指鍵盤賣家與 ZMK 專案，本專案歷來的溝通
> 混淆幾乎都源自這個詞。一律改說「上游」或「ZMK 專案」。

**上游**:
`a741725193/zmk-sofle`，這把鍵盤的設計者兼賣家所維護的設定 repo，也是本 repo 的 fork 來源。
_Avoid_: 官方、原廠、母版

**ZMK 專案**:
`zmkfirmware/zmk`，韌體框架本體的開源專案。
_Avoid_: 官方、上游

**cormoran fork**:
第三方個人維護的 ZMK 分支。上游自 2026-06 起改為指向它，本 repo 未跟進。
_Avoid_: DYA fork、官方 fork

## 硬體形態

**board**:
含有 MCU 的 PCB。
_Avoid_: 主板、控制器

**shield**:
不含 MCU、必須搭配控制器板才能構成一把鍵盤的 PCB。
_Avoid_: 擴充板、子板

**self-contained 鍵盤**:
MCU 與按鍵位於同一塊 PCB 的鍵盤。依 ZMK 的分類它只有 board、沒有 shield。本鍵盤屬於此類。
_Avoid_: 一體式、整合式

**central**:
分體鍵盤中負責與主機通訊的那一半。本鍵盤是左手。
_Avoid_: 主機端、master、主控

**peripheral**:
分體鍵盤中透過 BLE 把按鍵事件回報給 central 的那一半。本鍵盤是右手。
_Avoid_: 從機、slave、副手

## 連線

**profile**:
鍵盤記住的一組主機配對，本鍵盤有五組。指稱第幾組時務必說清楚基準：keymap 參數從 0
起算，螢幕上的圈圈從 1 起算。
_Avoid_: 通道、裝置槽、配對槽

**endpoint**:
鍵盤當下把按鍵事件送往何處，只有 USB 與 BLE 兩種。與 profile 正交 —— 走 USB 輸出的
同時，多個 BLE profile 仍可維持連線。
_Avoid_: 連線、輸出模式、通道

## 改鍵與繪圖工具

> **「Studio」單獨使用是禁用詞**，它可以指 ZMK Studio 或 DYA Studio，兩者需要的韌體不同。

**keymap-drawer**:
把 keymap 產生成 SVG 圖的工具。只產圖，不能改鍵。
_Avoid_: editor、drawer

**keymap-editor**:
網頁工具，讀取本 repo 的實體佈局檔，讓使用者以圖形方式編輯 keymap。
_Avoid_: drawer、繪圖工具

**ZMK Studio**:
ZMK 專案提供的即時改鍵工具。實體佈局由韌體回報，不讀取本 repo 的任何檔案。
_Avoid_: Studio

**DYA Studio**:
cormoran 維護的 Studio 分支，額外提供設定面板，必須搭配 cormoran fork 的韌體。本 repo 未採用。
_Avoid_: Studio

## keymap

**binding**:
keymap 中一顆鍵所對應的行為呼叫，例如 `&kp A`。
_Avoid_: 按鍵設定

**behavior**:
可被 binding 呼叫的動作定義，例如 `&kp`、`&mo`、巨集。
_Avoid_: 功能、動作

**layer**:
一整組覆蓋在基礎配置之上的 binding，索引從 0 起算。同一層有三種叫法 —— 索引
（`&mo 1`）、節點名（`sign`）、顯示名稱（`Sign`，螢幕上顯示的那行字）。顯示名稱自
2026-09-12 起改為與節點名一致，兩者可互相對照；指稱某一層時仍以**節點名**為準。
_Avoid_: 層級、模式、以索引指稱
