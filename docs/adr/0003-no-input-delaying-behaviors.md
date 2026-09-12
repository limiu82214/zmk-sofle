---
status: accepted
date: 2026-09-13
---

# 不採用會延後按鍵送出的 behavior

combo、hold-tap（`&mt` / `&lt`）、tap dance 一律**不採用**，keymap 維持 `&kp` / `&mo` /
`&to` 這類按下即送出的 behavior。判準是：**快速打字時的順暢度是本 keymap 的第一優先**，
而這類 behavior 的共通機制正是「先扣住事件，等判斷結果才送出」。

記下來是因為一把分體鍵盤的 keymap 完全沒有 combo 與 hold-tap 會顯得可疑，下一個人（或
下一個 agent）多半會當成疏漏而主動補上。

## 為什麼

combo 在 `app/src/combo.c` 的 `capture_pressed_key()` 回傳 `ZMK_EV_EVENT_CAPTURED`，把
press 事件整個扣下，直到 combo 成立或 `timeout-ms` 到期才由 `release_pressed_keys()`
放行。`zmk,combos.yaml` 的 `timeout-ms` 預設 50ms，且這個延遲加在**每一顆參與 combo 的
鍵**上，不是只有觸發的當下。

hold-tap 的代價則是把字元從「按下瞬間」延到「放開瞬間」，快速打字時單鍵按壓約 30–50ms，
那就是固定的 30–50ms 延遲。

**這個量級是感覺得到的**，這是否決的唯一理由。三種延遲的區分見 `CONTEXT.md` 的「延遲」。

## 考慮過但未採用

**`require-prior-idle-ms`。** v0.3.0 的 combo 與 hold-tap 都有這個屬性
（`zmk,combos.yaml`、`zmk,behavior-hold-tap.yaml`，預設皆為 `-1`）。`combo.c:159` 的
`is_quick_tap()` 判斷距上次敲鍵是否不足 N ms，`setup_candidates_for_first_keypress()`
只在它為 false 時才把 combo 列為候選 —— 也就是打字節奏中事件根本不會被扣住，零延遲。
**技術上這確實解掉了延遲問題**，不採用的原因在下一條。

**「省鍵位」作為理由本身不成立。** 這才是真正的否決點。ZMK 這整套 behavior 是為了在鍵位
不足的小鍵盤上擠出功能而設計的，本鍵盤有 60+ 顆鍵、`sign` 與 `number` 層還留著大量
`&none`。同理否決了 conditional layer（tri-layer）、`&caps_word`（大寫一律用 Shift）、
以及補綁 `&soft_off`（電力不是痛點）—— 這三者都不影響延遲，純粹是解決本 repo 沒有的問題。

**thumb 上的 `&lt 1 SPACE`。** 空白是高頻鍵，必然延後到放開才送出；而 thumb 的 layer 鍵
不能用 `require-prior-idle-ms` 補救，因為打字中途仍需要立刻切層。現行的 `&kp SPACE` 加
獨立 `&mo 1` 在這個優先序下就是正解。

## 後果

- ZMK 為小鍵盤設計的整個 behavior 家族（combo / hold-tap / sticky key / tap dance /
  caps word）已評估完畢，不必再逐一提案。
- 還能加的只剩不影響既有手感的功能：macro、分層 encoder（`sign` / `number` / `media`
  三層目前共用同一個 `&scroll_encoder`）、ZMK Studio。
- **重新評估的觸發條件**：鍵位真的不夠用，或改用鍵數明顯更少的鍵盤。屆時
  `require-prior-idle-ms` 是既有解法，不必重查。
