---
title: Raspberry Pi Pico 2 を Rust で動かす。
theme: default
background: /background.png
fonts:
  sans: Noto Sans JP
  mono: Fira Code
  weights: 600,900
class: text-center
mdc: true
favicon: /favicon.ico
# seoMeta:
#  ogImage: https://cover.sli.dev
---

# Raspberry Pi Pico 2 を Rust で動かす。

2025/07/23

@yutannihilation

---
layout: image-right
image: "/icon.jpg"
backgroundSize: 70%
---

# ドーモ！

## 名前:

湯谷啓明 (@yutannihilation)

## 自己紹介:

- ニンジャスレイヤーが好きです。

---

# Raspberry Pi Pico 2 とは

- Raspberry Pi Pico の後継機
- 2024年8月に発売
- 公式にサポートされているのは C/C++ での開発だが、そろそろ Rust まわりも整ってきた
  - 発売前から Rust 開発者にチップが提供されていた

---

# Pico → Pico 2 の違い

- RP2040 → RP2350A
- 全体的に性能強化
- セキュリティ機能の強化
- 価格も少し高い
- 注意点：[GPIO のプルダウン入力モードがうまく動かない](https://aloseed.com/it/pico-vs-pico2/)というけっこう深刻なバグがある

<v-clicks>

→ 正直、初心者が Pico 2 を選ぶ理由はほぼない

</v-clicks>


---

# 必要なもの

- Raspberry Pi Pico 2
- debug probe

---
layout: image-right
image: "/pico2.jpg"
---

# Raspberry Pi Pico 2

- 秋月とかで900円くらいで買える
- いろいろ種類がある
  - 無印
  - H
  - W
  - WH

---

# Raspberry Pi Pico 2

- 無印: ピンヘッダなし（自分ではんだ付けする）
- H: ピンヘッダあり
- W: 無線モジュール
- WH: 無線モジュール、ピンヘッダあり
- 公式以外にもいろいろ出てる

---
layout: image-right
image: "/probe.jpg"
---

# Debug Probe

- 秋月とかで2000円くらいで買える
- Pico/Pico 2 で自作もできる

---

# Debug Probe

- プログラムを流し込むだけならなくてもいい。
- が、これがないとプリントデバッグすらできないので、開発体験の差が激しい。実質必須。

---

# 必要なツール・フレームワーク

- [`probe-rs`](https://probe.rs/): デバッグ
- [`rp235x-hal`](https://crates.io/crates/rp235x-hal): HAL
- [`rp235x-project-template`](https://github.com/rp-rs/rp235x-project-template): テンプレート
- [Embassy](https://embassy.dev/): 組み込み Rust で非同期処理を行うためのフレームワーク

---

# `probe-rs`

- 

---

# `rp235x-hal`

- 

---

# `rp235x-project-template`

- 

---

# Embassy

- 
