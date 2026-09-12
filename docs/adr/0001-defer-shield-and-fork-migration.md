---
status: accepted
date: 2026-09-12
---

# 暫緩遷移至 shield 架構與 cormoran fork

上游（`a741725193/zmk-sofle`）在 2026-06-05 的 commit `4848e21` 一次做了兩個決定：把
board 定義改寫成 shield 搭配 `nice_nano_v2`，以及把 `config/west.yml` 的 ZMK 來源從
`zmkfirmware/zmk` 改指第三方的 `cormoran/zmk`。**本 repo 兩者都不跟進**，維持自建
board + ZMK 專案 v0.3.0。

## 為什麼

原本最有力的三個遷移理由，查證後逐一失效：

- **卡鍵問題不會因遷移而修好。** `app/src/usb_hid.c` 的 `zmk_usb_hid_send_report()`
  沒有檢查 `k_sem_take` 的回傳值，寫入失敗時也不重送。zmkfirmware v0.3.0、zmkfirmware
  main、cormoran fork 三者的這段程式碼一字不差。
- **cormoran 的修正對本專案價值有限。** 該 fork 相對 v0.3.0 的 7 條 `fix` 中，6 條位在
  Studio RPC 路徑，只在實際使用 Studio 時才發作；其中一條還是 ZMK 官方 PR #3185 的
  backport。唯一的核心修正在 split GATT 訂閱，而本鍵盤未出現對應症狀。
- **時間壓力不存在。** HWMv2（Zephyr 4.1）確實會讓自建 board 需要整包重寫，但 ZMK 最新
  正式版仍是 2025-08-01 的 v0.3.0，`app/VERSION` 尚未 bump，v0.4.0 沒有時間表。

反面的代價則是具體的：新架構 `west.yml` 的六個相依中有四個追浮動分支；
`BOARD_ENABLE_DCDC_HV` 會從 `y` 退回 `nice_nano_v2` 的預設 `n`，續航變差；右手的
`ZMK_USB=n` 會失效；上游關閉了 Issues 功能，且有把打錯字的 `west.yml` 推上 main、
兩天後才修的前例。

## 考慮過但未採用

**只取 shield 重構，`west.yml` 留在 `zmkfirmware/zmk`。** 架構上最乾淨，也避開了對個人
fork 的依賴。但這個組合全世界只有本 repo 在跑 —— 當優先序是穩定性時，「沒有人踩過的
組合」比「架構正確」更危險。上游那套至少有一千多個 fork 在幫忙踩雷。

## 後果

- 遷移成本隨時間累積。拖得越久，與上游的落差越大，屆時一次補完的工作量也越大。
- 自建 board 的維護責任完全落在本 repo。上游已轉向 shield，不會再更新 board 定義。
- 重新評估的觸發條件：ZMK 發布 v0.4.0 且上游跟進、決定自行 fork ZMK 修 `usb_hid.c`、
  或需要 DYA Studio 的即時調參功能。

若日後決定遷移，`upstream` remote 保留著可供對照：

```bash
git diff nimo upstream/main -- <path>
```
