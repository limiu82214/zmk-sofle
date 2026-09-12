# SPDX-License-Identifier: MIT

if(CONFIG_ZMK_DISPLAY AND CONFIG_SHIELD_NICE_VIEW_PHOTO)
  if(CONFIG_ZMK_SPLIT AND CONFIG_ZMK_SPLIT_ROLE_CENTRAL)
    message(FATAL_ERROR
      "nice_view_photo 只提供 peripheral 的畫面，不能掛在 central（左手）上。"
      "左手請維持 `shield: nice_view`。")
  endif()

  # 上游 nice_view 在 west workspace 裡的位置。ZMK 的建置中 CMAKE_SOURCE_DIR
  # 就是 zmk/app —— nice_view 自己的 CMakeLists 也是這樣取 ZMK 的 include 目錄的。
  set(NICE_VIEW_DIR ${CMAKE_SOURCE_DIR}/boards/shields/nice_view)
  if(NOT EXISTS ${NICE_VIEW_DIR}/widgets/peripheral_status.c)
    message(FATAL_ERROR
      "找不到上游的 nice_view widget：${NICE_VIEW_DIR}。ZMK 的目錄結構可能變了，"
      "見 docs/adr/0004-custom-nice-view-peripheral-art.md。")
  endif()

  zephyr_library_include_directories(
    ${CMAKE_SOURCE_DIR}/include
    ${NICE_VIEW_DIR}/widgets
  )

  # 上游的檔案直接編譯，不複製。這三個本 repo 一行都不需要改。
  zephyr_library_sources(${NICE_VIEW_DIR}/widgets/util.c)
  zephyr_library_sources(${NICE_VIEW_DIR}/widgets/bolt.c)
  zephyr_library_sources(${NICE_VIEW_DIR}/widgets/peripheral_status.c)

  # 本 repo 自己的：圖 + status screen，全部在這一個檔案裡。
  zephyr_library_sources(photo.c)
endif()
