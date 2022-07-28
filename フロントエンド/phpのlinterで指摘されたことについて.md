# what
- phpコードを修正していて、lintに指摘された事をまとめたもの
- 対応に困ったものについてはコメントなど残す

# 指摘達
## 共通している所
一部のファイルには以下のような部分があり、linterから指摘が入る
```
<?php
/**
 * Version    : 1.1.0
 * Author     : inc2734
 * Author URI : http://2inc.org
 * Created    : April 17, 2015
 * Modified   : August 30, 2015
 * License    : GPLv2 or later
 * License URI: license.txt
 */
?>
```
上記はhabakiriのコードからファイルにコピペしてきているようなので上記においての指摘は一旦無視している


## 優しい指摘
- Found: ==. Use strict comparisons (=== or !==)
    - 「合致する時は`===`で書いてね」という意味
## 対応に困った指摘
- is this commented out code?
    - コードがコメントアウトされている際に表示される現
    - 現状では該当コードを消すのはまだしないのでそのままの状態
    - 今9後、一時的にコードをコメントアウトする場合もあるので、このlintの指摘は無効化にしたほうが良いと考える
- Use Yoda Condition checks, you must.
    - ヨーダ記法での書き方に変更してくれ、という意味
        - [ヨーダ記法](https://ja.wikipedia.org/wiki/%E3%83%A8%E3%83%BC%E3%83%80%E8%A8%98%E6%B3%95)
    - 現状書き換えてはいるが、今後どうするかについては要相談
