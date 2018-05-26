# 生命科学ガチ初心者でもできるRNA-seqデータ解析 - kallistoでトランスクリプトの発現定量してみよう

## 端末の環境設定（Mac）


### Xcodeをインストールする
App storeからインストールします。

Macの各種ライブラリのインストールに必要なのはxcodeの"Command Line Tools"なのですが、
現在は
[Command Lione Toolsだけをインストールすることもできるようです](https://qiita.com/iwaseasahi/items/eb820762600c815ab100)。

この方法は自分では試していませんが、iOSの開発はしないからXcodeは必要無いという人はこちらでも良いかもしれません。


### Homebrewをインストールする

macOS用のパッケージマネージャーです。すでにインストールされている人が多いと思われますが、まだであれば
[Homebrewの公式サイト](https://brew.sh/index_ja.html)を見てインストールしてください。

※公式サイトを見て、、なのですがやることは下記のスクリプトをターミナルに貼り付けて実行するだけなので、
公式ページを開くのが面倒であればここからコピペしてもインストールできるはずです。
```markdown
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### gitをインストールする

gitも無ければインストールしてください。今回のワークショップでは、必須ではないかもしれませんが、
このレポジトリにあるドキュメントを手元のマシンにgit cloneしたいなどの需要があれば必要です。

インストールする手段は色々ある様で、例えばCommand line toolsがインストールされていると
gitコマンドを実行しようとするだけでインストールされる様ですが（未確認）、
Homebrew で下記の様にあらかじめインストールしておくのが何も考えないで済み楽な気がします。

```
$ brew install git
```

### minicondaをインストールする

今回はkallistのインストールでminicondaを使います。

[Condaの公式サイト](https://conda.io/miniconda.html)からMac OSX 64-bit版のbashインストーラーをダウンロードできます。
スクリプトをダウンロードしたら下記のようにインストーラを実行します。
```
$ sh DLしたファイルパス/Miniconda3-latest-MacOSX-x86_64.sh
```
pyenvを使っている様な人は初心者では無いので各自調べてインストールしてください。

