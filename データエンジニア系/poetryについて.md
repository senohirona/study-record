# what
pythonのサードパーティツールPoetryについて調べたこと等を記載する

# why
- コスト算出作業をpythonを利用してスクリプト化することになった
- pythonしばらく触ってなかったので昨今の開発環境

# Poetryとは
- pythonのパッケージマネージャーの1つ
    - 公式サイト: [Poetry](https://python-poetry.org/)
- pipと同じようにパッケージをpypi等からダウンロードしてきてインストールすることができる
    - jcで言うnpmやyarnと同じようなもの、らしい


# 触ってみる
公式でのチュートリアル?やQiitaに試しに触ってみる記事等があるので、まずはセットアップしてみる

### 今回の利用の流れ
- pythonのセットアップ
- プロジェクトのセットアップ
- 仮想環境のセットアップ
- 依存パッケージの追加
- 仮想環境で実行

### 使用環境
- mac
- pyenv: 2.1.0

## Poetryのインストール
- brewで入れられないかな?と思ったらあったので入れてみる

```
brew install poetry
```


## pythonのセットアップ
今回はpythonバージョンを3.8.12に設定してやってみる(3.8に関して特に深い意味はないです)

```zsh
# インストール可能なバージョンの確認
$ pyenv versison --list

# 3.8.12のインストール
$ pyencv install 3.8.12

# 今回は指定ディレクトリ内(Poetryお試し用ディレクトリ作った)でのみ3.8.12を指定する
$ pyenv local 3.8.12
```

## プロジェクトのセットアップ
次にPoetryのプロジェクト(パッケージ)をセットアップする、
これには「新規に立ち上げる場合」と「既にあるプロジェクトをpoetry管理化に置く場合」の2通りある
今回は「新規に立ち上げる場合」でやってみる

新しくプロジェクトを作る場合は、`poetry new`を使う

```
poerty new <project-name>
```

```zsh
$ poetry new poetry-tutorial
Created package poetry_tutorial in poetry-tutorial
```

最初に作ったプロジェクトは以下のような階層構造になっている
```
poetry-tutorial
├── README.rst
├── poetry-tutorial
│   └── __init__.py
├── pyproject.toml
└── tests
    ├── __init__.py
    └── test_poetry-tutorial.py
```

 - README.rst
     - プロジェクトの概要を記述するファイル。GitHubで言うREADME.mdみたいなもの
     - 拡張子は`.rst`になっていて reStructuredText というMarkdownが使われる前からpythonで使われていたらしい
     - rstファイルだが、.mdに書き換えてしまっても問題ない
 - poetry-tutorial(プロジェクト名のファイル)
     - このパッケージのpythonソースコードを格納するディレクトリで、その元締めとして`__init__.py`が作られている。ここではバージョンの定義だけがされている
 - pyproject.toml
     - poetryプロジェクトに関するメタデータや依存関係を記述するためのファイル。
     - TOMLという形式で書かれている
 - tests
     - ユニットテストを格納するディレクトリで、`__init__.py`とバージョン番号を確認する簡単なテストが`test_project_abc.py`に書かれている


## 仮想環境のセットアップ
**既存のプロジェクトを初期化する際に利用するコマンドで`poerty new`で作ったばかりのプロジェクトではやらなくてOK**

次にプロジェクト用の仮想環境を立ち上げるために`poetry init`をする
特に何も指定しない状態だと`pyproject.toml`に記載された依存関係(mainとdevelopment)のパッケージをインストールする

※ pyproject.tomlの`authors`が自分のユーザー名とメールアドレスがデフォで記載されるので、メルアドの部分は適当なものに変更した


```zsh
# ディレクトリの中に移動
$cd poetry-tutorial

$ poetry init

~~中略~~
at /usr/local/Cellar/poetry/1.1.11/libexec/vendor/lib/python3.9/site-packages/clikit/args/default_args_parser.py:300 in _add_long_option
      296│     def _add_long_option(
      297│         self, name, value, tokens, fmt, lenient
      298│     ):  # type: (str, Optional[str], List[str], ArgsFormat, bool) -> None
      299│         if not fmt.has_option(name):
    → 300│             raise NoSuchOptionException(name)
      301│
      302│         option = fmt.get_option(name)
      303│
      304│         if value is False:
```

## 依存パッケージの追加
`poetry new`した場合や`poetry init`で依存関係の追加をスキップした場合は、必要なパッケージの追加をする必要がある。
パッケージ追加には`poetry add`を使う

```
poetry add <package-name>
```

pip同様に指定したパッケージだけではなく、それが依存しているパッケージも合わせてインストールしてくれる


とりあえずpandasを試しにインストールする

```zsh
$ poetry add pandas
Creating virtualenv poetry-tutorial-cuUQfqIi-py3.8 in /Users/<user名>/Library/Caches/pypoetry/virtualenvs
Using version ^1.3.4 for pandas

Updating dependencies
Resolving dependencies... (46.4s)

Writing lock file

Package operations: 13 installs, 0 updates, 0 removals

  • Installing pyparsing (3.0.1)
  • Installing six (1.16.0)
  • Installing attrs (21.2.0)
  • Installing more-itertools (8.10.0)
  • Installing numpy (1.21.3)
  • Installing packaging (21.0)
  • Installing pluggy (0.13.1)
  • Installing py (1.10.0)
  • Installing python-dateutil (2.8.2)
  • Installing pytz (2021.3)
  • Installing wcwidth (0.2.5)
  • Installing pandas (1.3.4)
  • Installing pytest (5.4.3)
```

##  仮想環境で実行
必要なパッケージのインストールができたらスクリプトを書いて実行する

今回はpandasをつかってCSVの読み込みと取得したデータに行列を追加するスクリプトを作成

```py
import pandas as pd

# CSV読み込みと中身の表示
df = pd.read_csv('sample.csv')
print(df)

print('~~~~~~~')

# 行列に行列を追加する
df2 = df.append(df)
print(df2)
```

仮想環境での実行では`poetry run python`で実行する

```
poetry run python <script-name>
```

実際にやってみる

```zsh
$ poetry run python pandas-test.py
     name  id color  age
0    taro  11  blue   10
1  hanako  12   red   20
2     ken  13  blue   30
~~~~~~~
     name  id color  age
0    taro  11  blue   10
1  hanako  12   red   20
2     ken  13  blue   30
0    taro  11  blue   10
1  hanako  12   red   20
2     ken  13  blue   30
```


# 参考資料
- [Basic usage](https://python-poetry.org/)
- [Poetryをサクッと使い始めてみる](https://qiita.com/ksato9700/items/b893cf1db83605898d8a)
