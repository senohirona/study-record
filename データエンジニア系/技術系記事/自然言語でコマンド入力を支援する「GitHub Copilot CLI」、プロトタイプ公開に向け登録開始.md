# 自然言語でコマンド入力を支援する「GitHub Copilot CLI」、プロトタイプ公開に向け登録開始

参考記事

[うろ覚えのシェルやGitコマンドでも大丈夫。自然言語でコマンド入力を支援する「GitHub Copilot CLI」、プロトタイプ公開に向け登録開始](https://www.publickey1.jp/blog/23/gitgithub_copilot_cli.html)

**うろ覚えのシェルやGitコマンドでも大丈夫。**

(「とてもA○ple」と思ってしまった)

GitHubの研究開発部門であるGitHub Nextは「[GitHub Copilot CLI](https://githubnext.com/projects/copilot-cli/)」のプロトタイプ公開に向け、ウェイティングリストへの[登録を開始](https://githubnext.com/projects/copilot-cli/)した。

(ちなみに、登録開始は[ツイッター](https://twitter.com/mattrothenberg/status/1625874421968666627?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E1625874421968666627%7Ctwgr%5Edc0b06c9754fbe143cc4f6da6f0665050e6a3d30%7Ctwcon%5Es1_&ref_url=https%3A%2F%2Fwww.publickey1.jp%2Fblog%2F23%2Fgitgithub_copilot_cli.html)でも告知。)

### 何ができるのか？

- 自然言語でAIと対話しコマンドライン入力を支援してくれる

例えば、Copilot CLIに対して、tsファイルのリストを出したいと質問する。

Copilot CLIはこれに対しての提案と説明をが表示。

最後に3つの選択肢として `「This looks right, thank you!」（これです、ありがとう！）` か、 `「Actually, I can be more specific. Let me clarify!」（もう少し設定を絞り込みたいです）` か、 `「Cancel」（キャンセル）` が表示され、ユーザーはこれをカーソルで選択。

もしよければコマンドが生成され、実行されるという仕組み。

### 現状公開されているものについて

3つのシェルコマンドが用意されている。

- ??
    - 任意のシェルコマンドを探すためのコマンド
    - コマンドやループを組み合わせたり、不明瞭な検索フラグを投げるなどして
- git?
    - gitコマンドに関する検索を行うコマンド
- gh?
    - GitHub CLIのコマンドとクエリのインターフェイスのパワーと、AIが複雑なフラグやjq式を生成してくれるという利便性を兼ね備えているのです　(原文翻訳)
    - …🤔

### どうやったら使えるのか

- ウェイティングリスト(今回の記事の最初に貼っているURL)に自分のアカウントを登録することで利用可能
    - 登録車が多いと思われるので、利用できるようになるには少し時間がかかるかも？

### 余談

[https://twitter.com/kis/status/1628216660590215169?s=20](https://twitter.com/kis/status/1628216660590215169?s=20)

…たしかに