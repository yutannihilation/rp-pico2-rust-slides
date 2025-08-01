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

# Raspberry Pi Pico 2 を<br/>Rust で動かす。

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

- 株式会社 MIERUNE で GIS エンジニア見習いをしています。

---

# Raspberry Pi Pico とは

- Raspberry Pi 財団が出しているマイコンボード
- Raspberry Pi と違って Linux が動いたりはしない
- 安い（発売当初は 500 円くらいだった。円安...）
- C 以外に MicroPython や Rust でも動かせる
- Programmable I/O というユニークな機能がある（後述）

---

# Raspberry Pi Pico 2 とは

- Raspberry Pi Pico の上位機種
- 2024年8月に発売
- 公式にサポートされているのは C/C++ での開発だが、そろそろ Rust まわりも整ってきた
  - 発売前から Rust 開発者にチップが提供されていた

---

# Pico → Pico 2 の違い

## メリット

- 性能強化
- セキュリティ機能の強化

## デメリット

- 価格が少し高い
- [GPIO のプルダウン入力モードがうまく動かない](https://aloseed.com/it/pico-vs-pico2/)という困ったバグがある

<v-clicks>

→ 正直、初心者は Pico で十分！

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
- 自作も簡単だが、初心者はとりあえず1つは買ったほうが気が楽。
  - 動かない時の切り分け対象が減る

---

# 必要なツール・フレームワーク

- [probe-rs](https://probe.rs/): デバッグ
- [rp235x-hal](https://crates.io/crates/rp235x-hal): HAL
- [rp235x-project-template](https://github.com/rp-rs/rp235x-project-template): テンプレート
- [Embassy](https://embassy.dev/): 組み込み Rust で非同期処理を行うためのフレームワーク


---

# `probe-rs`

- ARM・RISC-V のチップにはデバッグ用のインターフェースがあり、`probe-rs` はそれと通信するためのツール
- RP235x のデバッグ用のインターフェース（ARM Debug Interface v6）にも今年2月に対応した
- VS Code 拡張もあって快適

---

# `rp235x-hal`

- HAL（ハードウェアへのアクセスを抽象化してくれるやつ）

---

# `rp235x-project-template`

- 必要な設定ファイルなどをあらかじめ配置したテンプレート
- 以下のコマンドを実行するだけ...かと思いきや、

```sh
cargo generate --git \
  https://github.com/rp-rs/rp235x-project-template
```

---

# `rp235x-project-template`

```
error: unsafe attribute used without unsafe
  --> src\main.rs:77:3
   |
77 | #[link_section = ".bi_entries"]
   |   ^^^^^^^^^^^^ usage of unsafe attribute
   |
help: wrap the attribute in `unsafe(...)`
   |
77 | #[unsafe(link_section = ".bi_entries")]
   |   +++++++                            +

error: could not compile `rp235x-project-template` (bin "rp235x-project-template") due to 1 previous error
```

---

# `rp235x-project-template`

- (Rust 2024 対応できてないだけで、だいたい動くはず...)


---

# Embassy

- 組み込み Rust で非同期処理をやるためのフレームワーク
- HAL も独自の crate を使っていて、rp-hal 系に（直接は）依存していない
- 活発に開発されているので信頼感がある

---

# Embassy vs RTIC

<Transform :scale="0.8">

![](/rtic_vs_embassy.jpg)

</Transform>

---

# 非同期処理、必要？

- 出力だけ、入力だけなら必要ないことが多そう
- 入力も出力もあるときはたぶん使いたくなる（例：ツマミのひねり具合によって LED の光らせ方を変えたい）

---

# Programmable I/O（PIO）

- CPU から独立して動くステートマシン
- GPIO ピンを操作できる
- 独自言語でプログラミングする
- 単純な命令しか実行できないが、工夫すれば通信プロトコルなども PIO だけで捌けたりする

---
layout: image-right
image: "/encoder.jpg"
---

# 例：ロータリーエンコーダー

- どっち方向にも無限に回るツマミ
- 複数の極が出ていて、その位相の違いで回転の方向を把握する

---

# ロータリーエンコーダーのデータシート

- C.W. = 時計回り、C.C.W. = 反時計回り

<Transform :scale="0.8">

![](/datasheet.png)

</Transform>

---

# PIO コード

- ピンを2つ使う

```
wait 1 pin 1
wait 0 pin 1
in pins, 2
push
```

---

# PIO コード

- ピン1が HIGH → LOW になるのを待つ  

``` {1,2}
wait 1 pin 1
wait 0 pin 1
in pins, 2
push
```

<v-clicks>

- つまり、立ち下がりを検出している
- このとき、ピン2が HIGH か LOW かが回転の方向によって違う

</v-clicks>


---

# PIO コード

- ピンの値を 2 つ Input Shift Registerに書き込む

``` {3}
wait 1 pin 1
wait 0 pin 1
in pins, 2
push
```

<v-clicks>

- 知りたいのはピン2だけだが、`in` 命令は一気に読むしかできないので、 ピン1も読んでいる

</v-clicks>

---

# PIO コード

- ISR の値を RX FIFO に書き込む

``` {4}
wait 1 pin 1
wait 0 pin 1
in pins, 2
push
```

---

# Rust コード

- ピン1の値は必ず0（なぜなら、ピン1がLOWになったタイミングで読んでいるので）
- なので、`(ピン1、ピン2)` の値は `00` か `01` かどちらか

```rs
match self.sm.rx().wait_pull().await {
    0 => return Direction::CounterClockwise,
    1 => return Direction::Clockwise,
    _ => {}
}
```

---

# もっと詳しく知りたい人はこれを

- <https://zenn.dev/yutannihilation/articles/365eb6e3648f13>

<Transform :scale="0.8">

![](/zenn.png)

</Transform>

---

# PIO のいいところ

- I/O をいっぱい扱えるようになる
- 得体の知れない言語なので「ハードウェアをやってる感」がある

---

# PIO を使ってやろうとしてること

- 7セグLEDを100個くらい同時に制御したい
- 単にオンオフじゃなくて、セグメントごとに PWM で輝度を変えたい

---
layout: section
---

# こんな感じ

---

<SlidevVideo autoplay controls>
  <source src="/7seg.mp4" type="video/mp4" />
</SlidevVideo>

---

# 何をやってるか

- 1つの7セグLEDを動かすのにピンが7～8本必要
- Pico 2 の GPIO は 26 個しかないので足りない

<v-clicks>

- シフトレジスタ IC をうまく使ってIOを増やしたい
- いっぱいつなぐと遅くなるので、PIO で高速化
- とりあえず1個は動いたのでいっぱい数珠つなぎにしていくぞ！というところで2年間くらい放置している

</v-clicks>

---
layout: section
---

# 最高の夏！


