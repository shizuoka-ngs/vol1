# 生命科学ガチ初心者でもできるRNA-seqデータ解析 - kallistoでトランスクリプトを発現定量してみよう

## 1. 端末の環境設定（Mac）

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

今回はkallistoのインストールでminicondaを使います。

[Condaの公式サイト](https://conda.io/miniconda.html)からMac OSX 64-bit版のbashインストーラーをダウンロードできます。
スクリプトをダウンロードしたら下記のようにインストーラを実行します。
```
$ sh DLしたファイルパス/Miniconda3-latest-MacOSX-x86_64.sh
```
pyenvを使っている様な人は初心者では無いので各自調べてインストールしてください。

### sratoolkitをインストール
SRAファイルをfastq-dumpするために使います。fastq-dumpはsratool kitに含まれているため。
sratoolkitをHomebrewでインストールしておきます。

```
$ brew install sratoolkit
```


### kallistoをインストール

トランスクリプトームのリファレンス配列のインデックス作成と、発現定量にkallistoを利用します。
kallistoは [condaコマンドでインストールすることができます](https://bioconda.github.io/recipes/kallisto/README.html)。

```
$ conda config --add channels defaults
$ conda config --add channels conda-forge
$ conda config --add channels bioconda
$ conda install kallisto
```

### jupyterをインストールする
発現量のプロットにjupyterを使います。

jupyterはminicondaがインストールされている状態でcondaを使ってインストールできます。
jupyterのインストールのついでに今回、散布図を描画するために使うpandas, matplotlib, plotlyをインストールします。
```markdown
$ conda update conda
$ conda install jupyter
$ conda install pandas
$ conda install matplotlib
$ conda install plotly
```


### その他インストールしておく方が良いツール

実際の作業では数GBを超える大きめのファイルをダウンロードすることが多いため
 Chrome(ブラウザ)やbyobuが入っていると便利かもしれません。
 
## 2. ファイルをダウンロードする

今回のkallistoを用いた発現定量では、ヒトのトランスクリプトームのfastaファイル（cDNA由来の塩基配列データ）のインデックスを作り
そこに対象となるサンプルのfastqファイル（品質情報を伴ったNGS由来の塩基配列データ）をあてて発現定量していくために、ヒトのcDNA配列とサンプルのfastqファイルが必要となります。

ただし、ファイルのダウンロードは時間がかかるため、今回は必要なファイルについて**あらかじめダウンロードして置いてHDに入れて配布する**ことにします。

以下、どのファイルを用いるか参考として記述しておきます。

### ヒトのcDNA配列
[Ensembl](https://asia.ensembl.org/info/data/ftp/index.html) からヒトのfasta形式のcDNA配列を入手します。

Ensemblの[ヒトcDNA配列のftpサイト](ftp://ftp.ensembl.org/pub/release-92/fasta/homo_sapiens/cdna/)には
*.abinitio.fa.gzと *.all.fa.gzがありますが、今回は
- [Homo_sapiens.GRCh38.cdna.all.fa.gz](ftp://ftp.ensembl.org/pub/release-92/fasta/homo_sapiens/cdna/Homo_sapiens.GRCh38.cdna.all.fa.gz)

を利用します。

### fastqファイルの取得
kallistoの解析には今回ペアエンドのfastqファイルを利用します。ただしfastqファイルは非常に多くダウンロードに時間がかかるため、
今回は圧縮されファイルサイズの小さいsraファイルをダウンロードして、これをfastq-dumpを使ってペアエンドのfastqを自分で生成しています。

解析するサンプルはMCF7の低酸素状態の細胞と、コントロールの細胞から配列データをSRAファイルで入手します。

サンプルデータについての詳細は[ArrayExress:E-MTAB-4264 - Tuning the transcriptional response to hypoxia through HIF prolyl- and asparaginyl-hydroxylase inhibition:Sequencing Data](https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-4264/samples/) をご覧ください。

実際に入手したファイル（低酸素状態・コントロール）は下記の通りです。

- ERR1551404.sra: ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/sralite/ByExp/litesra/ERX/ERX162/ERX1622160/ERR1551404/ERR1551404.sra
- ERR1551408.sra: ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/sralite/ByExp/litesra/ERX/ERX162/ERX1622164/ERR1551408/ERR1551408.sra

## 3. 発現解析

### SRAファイルからfastqファイルを生成する

ダウンロードしたSRAファイルをsratoolkitを使ってペアエンドのfastqファイルに変換します。

```
$ fastq-dump --split-files 取得したSRAファイルのパス
```
配布したSRAを使うのならば、以下のような二つのファイルができるはず。
```
$ fastq-dump --split-files ERR1551404.sra 
$ ls
ERR1551404_1.fastq    ERR1551404_2.fastq
```
コントロールのサンプルについてもfastqを生成しておきます。
```
$ fastq-dump --split-files ERR1551408.sra
```
### kallisto indexでヒトのトランスクリプトームのインデックスを作成する 
kallistoではindexファイルを配布するより、kallisto indexで構築した方が速いということで、
[Ensembl](https://www.ensembl.org/info/data/ftp/index.html)からfastaファイルをダウンロードして
自分で作ることを推奨しているようです。今回ハードディスクにヒトのcDNA（FASTA）が含まれています。
このファイルを使って（gzipのままでOK）下記のようにインデックスを作成します。

```
$ kallisto index -i filename_anything_you_like Homo_sapiens.GRCh38.cdna.all.fa.gz
```

### 定量する
kallisto quantoで発現量を定量します。ペアエンドの定量なので、変換したfastqファイルの
_1.fastqと_2.fastをセットで使います。

```
$ kallisto quant -i index_file_name -o output_dir_name ERR1551404_1.fastq ERR1551404_2.fastq
```
kallistoはthread数をオプションで指定できるので、例えばコア数2のMacBook Airでは下記のような例となります。
```
$ kallisto quant -t 2 -i GRCh38_kallisto_index -o . ERR1551404_1.fastq ERR1551404_2.fastq
```
kallistoの使い方の詳細については[こちらのブログ](https://bonohu.wordpress.com/2017/11/15/kallisto/)が参考になると思います。

### 確認
kallisto quantoが終了すると、指定したoutput dirにいくつかのファイルができているはずです。
このうち、abundance.tsvを見てみます
```
$ sort -k 5 -rn abundance.tsv | less
```

上記の例では、5番目のカラム、Transcripts Per Million（TPM）を発現量の多い順にソートして表示しています。


## 4. データ可視化

