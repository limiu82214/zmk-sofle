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

## 螢幕

> **「螢幕」單獨使用是禁用詞。** 兩半各有一塊，硬體相同但畫面內容完全不同，共有的
> 只有電量。既有文件講的「螢幕上的圈圈」只存在左手，「螢幕上的圖」只存在右手 ——
> 不指明哪一半，就會出現「我的螢幕沒有圈圈」這種對不上的對話。

**nice!view**:
兩半各一塊的 160×68 單色螢幕，只有黑與白、沒有灰階。指硬體本身。
_Avoid_: 螢幕、OLED、小螢幕

**nice_view**:
ZMK 專案內建的 shield 名稱，提供螢幕的 devicetree 節點與下面兩套畫面。左右手都掛它。
_Avoid_: nice!view（那是硬體）、螢幕驅動

**status widget**:
左手（central）螢幕上的那組狀態顯示：五個 profile 圈圈、endpoint、layer 顯示名稱、
WPM 曲線、電量。右手沒有這組。
_Avoid_: 狀態列、主畫面、螢幕

**art**:
右手（peripheral）螢幕上佔掉絕大部分面積的那張 140×68 圖。開機時在 `balloon` 與
`mountain` 兩張之間隨機挑一張，之後不再變動 —— 它不是動畫，也不反映任何狀態。
右手螢幕剩下的 68×68 方塊只顯示電量與一個連線符號。
_Avoid_: 桌布、開機畫面、動畫、螢幕

**nice_view_custom**:
第三方 shield（`GPeye/urchin-peripheral-animation`），用來換掉 art。`build.yaml` 裡
那行註解掉的引用是上游舊架構的殘留，該模組已不在 `config/west.yml`，取消註解會編譯
失敗。本 repo 未採用。
_Avoid_: 自訂 shield、換圖那個 shield

## 延遲

> **「延遲」單獨使用是禁用詞。** 這條路徑上有三種延遲，量級相差一個數量級以上，該修的
> 位置也完全不同。混用會把「感覺得到的 50ms」和「感覺不到的 4ms」放上同一個天平。

**scan 延遲**:
按鍵實體閉合，到 kscan 認定它閉合之間的時間，由掃描與 debounce 決定。
_Avoid_: 延遲、防彈跳延遲

**傳輸延遲**:
按鍵事件從鍵盤送達主機所花的時間，取決於當下的 endpoint 是 USB 還是 BLE。
_Avoid_: 延遲、藍牙延遲

**behavior 判斷延遲**:
combo 與 hold-tap 為了判斷使用者意圖，扣住按鍵事件暫不送出的那段時間。三者中唯一
感覺得到的一種。
_Avoid_: 延遲、combo 延遲

## 照明

> **「背光」單獨使用是禁用詞。** 中文的「背光」會同時指向 underglow 與 backlight，
> 而兩者是不同硬體、不同 behavior、開機預設值還相反（underglow 預設關、backlight
> 預設開）。用同一個詞指稱兩者必然推出矛盾的結論。
>
> 更糟的是本鍵盤的 backlight **根本不照明按鍵，它是螢幕的背光**。把它當成照明功能
> 推理，會得出「反正沒亮、關掉沒差」這種結論，實際後果是螢幕全黑。

**underglow**:
鍵盤上的 WS2812 燈條，由 media layer 的 `&rgb_ug` 開關，開關狀態存進 settings。
_Avoid_: 背光、RGB 燈、氛圍燈

**backlight**:
ZMK 的 `zmk,backlight` 功能。在本鍵盤上它驅動的**不是按鍵照明，而是 nice!view
螢幕的背光**（`eyelash_sofle.dtsi` 的 `pwm_led_0`，接在 P1.13）。關掉它螢幕就
看不見，所以 `CONFIG_ZMK_BACKLIGHT_ON_START` 必須為 `y`，且刻意**不提供任何
binding** —— 它應該一直開著，能關掉只會製造「螢幕壞了」的誤判。2026-09-13 以
暫時加上的 `&bl BL_TOG` 實測確認後移除。
_Avoid_: 背光、燈光、按鍵背光

## 狀態與清除

**settings**:
鍵盤存在自身 flash 中、與韌體分開的執行期狀態：BLE 配對、underglow 開關、ZMK Studio
存過的 keymap 改動。**刷新韌體不會清除它** —— 韌體與 settings 位在互不重疊的兩個
flash 分區。與 `config/eyelash_sofle.conf` 那種編譯期設定是兩回事。
_Avoid_: 設定、儲存、NVS

> **「reset」單獨使用是禁用詞。** 四個動作都叫 reset，後果從「什麼都沒清」到「配對與
> keymap 存檔全沒」。指稱時一律用下列名稱。

**sys reset**:
`&sys_reset`，單純重開機，不清除任何 settings。
_Avoid_: reset、重置

**清除配對**:
`&bt BT_CLR`，只解除**當前** profile 的配對，其餘 settings 不動。
_Avoid_: reset、清除連線

**Restore Stock Settings**:
ZMK Studio 內的動作，刪除 Studio 存過的 keymap 改動，讓 `.keymap` 重新生效。不動配對。
_Avoid_: reset、還原、恢復出廠

**settings reset**:
刷 `settings_reset` 韌體，抹除整個 settings 分區 —— 配對與 Studio 存檔一併消失。
_Avoid_: reset、清機、恢復出廠
