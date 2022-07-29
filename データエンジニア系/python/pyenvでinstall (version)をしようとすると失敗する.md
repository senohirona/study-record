# what
- pyenvにpython3.8以降のバージョンをインストールしようと思ったらコケたのでその解消ログ

### 基本情報
- mac
- pyenvのバージョン: 2.1.0(brewで入れた)


# 事象
 pyenvを利用してバージョン3.8.0をインストールしようとしたところ、以下エラーが出てコケた

```
$ pyenv install 3.8.0
python-build: use openssl@1.1 from homebrew
python-build: use readline from homebrew
Downloading Python-3.8.0.tar.xz...
-> https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tar.xz
Installing Python-3.8.0...
python-build: use tcl-tk from homebrew
python-build: use readline from homebrew
python-build: use zlib from xcode sdk

BUILD FAILED (OS X 10.15.7 using python-build 20180424)

Inspect or clean up the working tree at /var/folders/q_/jfd5qg1d6_jdrnhxbw_qhd6r0000gn/T/python-build.20211025153933.75644
Results logged to /var/folders/q_/jfd5qg1d6_jdrnhxbw_qhd6r0000gn/T/python-build.20211025153933.75644.log

Last 10 log lines:
config.status: creating pyconfig.h
creating Modules/Setup.local
creating Makefile


If you want a release build with all stable optimizations active (PGO, etc),
please run ./configure --enable-optimizations


Makefile:208: *** missing separator.  Stop.
```

# 解決方法
## zlibを入れる
これでは解決せず

```
$ brew install zlib
==> Downloading https://ghcr.io/v2/homebrew/core/zlib/manifests/1.2.11
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/zlib/blobs/sha256:8ec66cf6faa310712767efc3022fdd16568a79234439f64bf579acb628f893bc
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:8ec66cf6faa310712767efc3022fdd16568a79234439f64bf579acb628f893bc?se=2021-10-25T06%3A55%3A00Z&sig=%2By2eaiSalyp07tmAao1xdocyAdINkcr2kFFTT2m9eNQ%3D&sp=r&spr=https&sr=b&sv=2019-12-12
######################################################################## 100.0%
==> Pouring zlib--1.2.11.catalina.bottle.tar.gz
==> Caveats
zlib is keg-only, which means it was not symlinked into /usr/local,
because macOS already provides this software and installing another version in
parallel can cause all kinds of trouble.

For compilers to find zlib you may need to set:
  export LDFLAGS="-L/usr/local/opt/zlib/lib"
  export CPPFLAGS="-I/usr/local/opt/zlib/include"

For pkg-config to find zlib you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/zlib/lib/pkgconfig"

==> Summary
🍺  /usr/local/Cellar/zlib/1.2.11: 12 files, 376.6KB
```

## bzip2

```
$ brew install bzip2
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> Updated Formulae
Updated 16 formulae.

==> Downloading https://ghcr.io/v2/homebrew/core/bzip2/manifests/1.0.8-1
Already downloaded: /Users/'user名'/Library/Caches/Homebrew/downloads/a863192981a3c995f4e1a9515d97692bfb1dba5716740bf8c1dcd64ea7d5c2dd--bzip2-1.0.8-1.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/bzip2/blobs/sha256:78421d5891328cb96cce8ff6a6c20ce5930a4a74fd1b24b05ef02cd92117c5fd
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:78421d5891328cb96cce8ff6a6c20ce5930a4a74fd1b24b05ef02cd92117c5fd?se=2021-10-25T07%3A05%3A00Z&sig=XhAHRX8gCvqpdmtKmDjc%2BNuQhHbf%2FHapdedbqUio1Pc%3D&sp=r&spr=https&sr=b&sv=2019-12-12
######################################################################## 100.0%
==> Pouring bzip2--1.0.8.catalina.bottle.1.tar.gz
==> Caveats
bzip2 is keg-only, which means it was not symlinked into /usr/local,
because macOS already provides this software and installing another version in
parallel can cause all kinds of trouble.

If you need to have bzip2 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/bzip2/bin:$PATH"' >> ~/.zshrc

For compilers to find bzip2 you may need to set:
  export LDFLAGS="-L/usr/local/opt/bzip2/lib"
  export CPPFLAGS="-I/usr/local/opt/bzip2/include"

==> Summary
🍺  /usr/local/Cellar/bzip2/1.0.8: 26 files, 533.0KB
```

## ログファイルを見てみる
似たような記事を調べてみたけれどなかなか解決せず詰まった…
しかし、エラー文の中に「ログファイルはここ」というものを見つけたので見てみる

```
$ vim /var/folders/q_/jfd5qg1d6_jdrnhxbw_qhd6r0000gn/T/python-build.20211025164843.93506.log
```

中身

```log
/var/folders/q_/jfd5qg1d6_jdrnhxbw_qhd6r0000gn/T/python-build.20211025164843.93506 ~
/var/folders/q_/jfd5qg1d6_jdrnhxbw_qhd6r0000gn/T/python-build.20211025164843.93506/Python-3.8.0 /var/folders/q_/jfd5qg1d6_jdrnhxbw_qhd6r0000gn/T/python-build.20211025164843.93506 ~
checking build system type... x86_64-apple-darwin19.6.0
checking host system type... x86_64-apple-darwin19.6.0
〜〜中略〜〜

If you want a release build with all stable optimizations active (PGO, etc),
please run ./configure --enable-optimizations


Makefile:208: *** missing separator.  Stop.
```

最後の `please run ./configure --enable-optimizations` とは...?
一体何の意味なのか調べてみる

Googleで「pyenv pyenv  mac please run ./configure --enable-optimizations」とググってみる

すると、以下の記事にたどり着いた

[pyenv install of Python errors with missing separators](https://stackoverflow.com/questions/69496719/pyenv-install-of-python-errors-with-missing-separators)

自分と同じ「pyenvでバージョンをインストールしようとすると失敗する」という内容で、ログファイルの内容も自分のところで出たものとほぼ同じ模様

これに対する回答を見てみる

> I hit the same problem and filed https://github.com/pyenv/pyenv/issues/2102

> This was resolved by running rm -rf /usr/local/Cellar/tcl-tk/8.6.11_1.reinstall and installing from pyenv. It appears there were duplicate tcl-tk installs.

 1つ目の回答のgithubURLに行ってみた

[pyenv install 3.10.0 fails with Makefile:222: *** missing separator](https://github.com/pyenv/pyenv/issues/2102)

どうやらCellarにあるpyenvのファイルが複数あるとインストールが失敗することがあるらしい

### pyenvのファイルを消してみる

まずは、pyenvのファイルが複数あるのかを調べてみる

```
# ファイルのある場所を確認
$ brew --cellar tcl-tk
/usr/local/Cellar/tcl-tk

# ファイルが複数あるかを調べる
$ ls -l "$(brew --cellar tcl-tk)"
total 0
drwxr-xr-x  12 'user名'  admin  384 Oct 22 19:22 8.6.11_1
drwxr-xr-x  11 'user名'  admin  352 Jan 28  2018 8.6.8
```

Githubの質問者と同様、複数存在していた
次に、古い方(今回は8.6.8)のファイルを移動させる
(削除でも良かったが、消すのが不安だったため移動)

```
# 8.6.8の方をpyenv-cellarというフォルダ(適当に作った)に移動
$ mv /usr/local/Cellar/tcl-tk/8.6.8 ~/pyenv-cellar

# 再度ファイル数を確認(1つになっているのを確認)
$ ls -l "$(brew --cellar tcl-tk)"
total 0
drwxr-xr-x  12 'user名'  admin  384 Oct 22 19:22 8.6.11_1
```

次にpyenvを再インストール

```
brew reinstall pyenv
```

これでもう一度バージョンをインストールしてみる

```
$ pyenv install 3.8.0
python-build: use openssl@1.1 from homebrew
python-build: use readline from homebrew
Downloading Python-3.8.0.tar.xz...
-> https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tar.xz
Installing Python-3.8.0...
python-build: use tcl-tk from homebrew
python-build: use readline from homebrew
python-build: use zlib from xcode sdk
Installed Python-3.8.0 to /Users/'user名'/.pyenv/versions/3.8.0
```

無事にできた

# 参考資料
- https://qiita.com/kzrashi/items/80ff82d4330d51a398dc
- https://it-jog.com/py/intro/pyenvsetting
- https://stackoverflow.com/questions/69496719/pyenv-install-of-python-errors-with-missing-separators
- https://github.com/pyenv/pyenv/issues/2102
