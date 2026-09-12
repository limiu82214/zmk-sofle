---
status: accepted
date: 2026-09-12
---

# 不啟用 USB boot protocol，BIOS 支援延後至 v0.4.0

這把鍵盤在 BIOS 畫面中無法使用（該機為 Legacy/CSM 開機：`msinfo32` 的「BIOS 模式」顯示
「舊版」，`shutdown /r /fw` 回報韌體不支援開機到韌體 UI）。ZMK 的 `CONFIG_ZMK_USB_BOOT`
正是為此而設，**但本 repo 不啟用它**，BIOS 支援延後到 ZMK v0.4.0 發布時，連同
[ADR 0001](0001-defer-shield-and-fork-migration.md) 的遷移一併處理。

## 為什麼

v0.3.0 的 `app/Kconfig` 把兩個東西綁在一起：

```kconfig
config ZMK_USB_BOOT
    select USB_HID_BOOT_PROTOCOL
    select USB_DEVICE_SOF        # ← 問題在這行
```

`select` 是強制的，使用者的 `.conf` 寫 `CONFIG_USB_DEVICE_SOF=n` 蓋不掉。

而 SOF 是每 1 ms 一次的 USB start-of-frame 中斷，ZMK 完全沒有使用它 ——
`app/src/usb.c` 的 callback 第一行就 `return`，註解寫明「not used within ZMK」。代價是
每秒 1000 次中斷加 1000 次工作佇列喚醒，且上游 PR #3070（2025-12-18 合併）指出它會造成
非預期的 USB 斷線，列出的症狀之一正是「在 BIOS 能用，進 OS 反而不行」。

#3070 刪掉了那行 `select`，但**沒有任何 tag 包含這個修正**：`v0.3-branch` 自 v0.3.0
以來只有三個文件 commit，`app/Kconfig` 裡那行原封不動；唯一含有修正的是 `main`。而
`main` 在該修正之前 8 天已經合併 Zephyr 4.1（PR #3060），HWMv2 會讓本 repo 的自建 board
需要整包重寫 —— 正是 ADR 0001 決定暫緩的事。兩者無法分離。

## 考慮過但未採用

**啟用 `ZMK_USB_BOOT` 並接受 SOF。** 為了進 BIOS 而讓日常使用的 USB 連線變得不穩定，
取捨不成立 —— 何況 #3070 描述的症狀正好是「BIOS 好了、OS 壞了」。

**把 `west.yml` 升到 `main`。** 除 ADR 0001 已記的理由外，這次查出的具體代價：board
目錄要改成 HWMv2 佈局、`board.yml` 得手寫（ZMK 遷移文件明說自動轉換腳本對分體設計不
可靠）、DC/DC 設定要從 Kconfig 移到 devicetree、`CONFIG_WS2812_STRIP` 必須刪除、
`&bootloader` 需要整套 boot retention 設定否則失效、`build.yaml` 的 board ID 全部要改。

**移除 `studio-rpc-usb-uart` snippet。** 它帶進 `CONFIG_USB_CDC_ACM`，使左手成為 3 個
介面的複合裝置（裝置分類碼從 `00/00/00` 變成 `0xEF/0x02/0x01`，外加 IAD），而 Legacy
BIOS 對複合裝置的容忍度較低。這是零風險的測試，但查遍 ZMK 的原始碼、文件、issues 與 PR
都找不到複合裝置與 BIOS 相容性的關聯紀錄，且 ZMK Studio 決定保留。

## 後果

- 需要進 BIOS 時得另外接一把鍵盤。
- **已排除、不必重查的原因**：NKRO —— `config/eyelash_sofle.conf` 與左手 defconfig 皆為
  `n`，而 ZMK 在 `app/Kconfig` 與文件兩處都把 NKRO 列為已知的 BIOS 不相容原因。
- **尚未驗證的原因**：BIOS 內 `USB Legacy Support` 之類的設定是否被關閉。這是韌體改不了
  的，需要借一把鍵盤進 BIOS 才能確認 —— 在做任何韌體改動之前，應該先排除這一項。
- **重新評估的觸發條件**：ZMK 發布 v0.4.0。release PR #3024 已開啟並持續滾動更新，這比
  ADR 0001 撰寫時的「v0.4.0 沒有時間表」更近一步。屆時 `ZMK_USB_BOOT` 不再帶 SOF。

## 附帶

若日後啟用 boot protocol，記得 ZMK 在 boot protocol 生效期間會**停送滑鼠與 consumer
report**（`app/src/usb_hid.c`）。本 keymap 用到 `&mmv` / `&mkp` 與旋鈕的 `C_MUTE`，但這
只在主機主動送出 `SET_PROTOCOL(Boot)` 時發生，一般作業系統不會。
