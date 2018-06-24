# 生命科学ガチ初心者でもできるRNA-seqデータ解析 - kallistoでトランスクリプトを発現定量してみよう

- 資料：[https://github.com/shizuoka-ngs/vol1](hhttps://github.com/shizuoka-ngs/vol1/blob/master/RNA-Seq-Data-Analysis-for-Beginners.md)
- 概要：ヒトのCell line RCC4で低酸素ストレス状態のサンプルと、VHLをレスキューしたサンプルをkallistoで定量し発現解析を行います。

## 1. 端末の環境設定（Mac）

### Xcodeをインストールする
App storeからインストールします。

Macの各種ライブラリのインストールに必要なのはxcodeの"Command Line Tools"なのですが、
現在は
[Command Line Toolsだけをインストールすることもできるようです](https://qiita.com/iwaseasahi/items/eb820762600c815ab100)。

iOSの開発はしないからXcodeは必要無いという人はこちらでも良いかもしれません。


### Homebrewをインストールする

macOS用のパッケージマネージャーです。すでにインストールされている人が多いと思われますが、まだであれば
[Homebrewの公式サイト](https://brew.sh/index_ja.html)を見てインストールしてください。

※公式サイトを見て、、なのですがやることは下記のスクリプトをターミナルに貼り付けて実行するだけなので、
公式ページを開くのが面倒であればここからコピペしてもインストールできるはずです。
```markdown
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

※ 今回は必要ありませんが、Bioinfomatics関係のライブラリをインストールする際は、フォーミュラをbrew tapでリポジトリに取り込んで置くと
インストールをスムーズに行うことができる場合があります。
```
$ brew tap brewsci/science
$ brew tap brewscie/bio

```
sratoolkitをcondaでは無く（今回のハンズオンではconda installします）brewでインストールするときなどはtapが必要です。

### gitをインストールする

gitも無ければインストールしてください。今回のワークショップでは、必須ではないかもしれませんが、
このレポジトリにあるドキュメントを手元のマシンにgit cloneしたいなどの需要があれば必要です。

インストールする手段は色々ある様で、例えばCommand line toolsがインストールされていると
gitコマンドを実行しようとするだけでインストールされる様ですが（未確認）、
Homebrew で下記の様にあらかじめインストールしておくのが何も考えないで済み楽な気がします。

```
$ brew install git
```
※ 一台のMacに複数のユーザを設定して使っている場合、homebrewを使うとpermission errorとなるケースがあるようです。
homebrewは現在sudoすることができないため、この場合/usr/local/Cellarをchownする必要があります。詳細は[こちら](https://qiita.com/HIROSHI-1403/items/c90699c1a3f3bd9d63f9)を参考に。

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
※ どうしてもbrew install sratoolkitできない時は（homebrewを使い続けていると古いライブラリが残っていたりして、モジュールのバッディングを上手く解決できない時があります）直接NCBIからダウンロードした方が早い場合もあるかもしれません。「brew installできなそう」と感じた時は[こちら](http://www.sthda.com/english/wiki/install-sra-toolkit)をご参考に。

### kallistoをインストール

トランスクリプトームのリファレンス配列のインデックス作成と、発現定量にkallistoを利用します。
kallistoは [condaコマンドでインストールすることができます](https://bioconda.github.io/recipes/kallisto/README.html)。

```
$ conda config --add channels defaults
$ conda config --add channels conda-forge
$ conda config --add channels bioconda
$ conda install kallisto
```

※ kallistoはHomebrewでもインストールすることはできますが、brewsci/scieneをtapして置く必要があります。
詳細は[こちら](https://qiita.com/epsilonminder/items/e3b1fc00edb63cb3a32b)で。


### jupyter, pandas etcをインストールする
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

ただし、ファイルのダウンロードは時間がかかるため、今回は必要なファイルについて**あらかじめダウンロードして置いてHDに入れて配布します**。

以下、どのファイルを用いるか参考として記述しておきます。

※ 別のサンプルで試したい場合はあらかじめSRAファイルを自分の環境にダウンロードしておいてください。

### ヒトのcDNA配列
[Ensembl](https://asia.ensembl.org/info/data/ftp/index.html) からヒトのfasta形式のcDNA配列を入手します。

Ensemblの[ヒトcDNA配列のftpサイト](ftp://ftp.ensembl.org/pub/release-92/fasta/homo_sapiens/cdna/)には
*.abinitio.fa.gzと *.all.fa.gzがありますが、今回は
- [Homo_sapiens.GRCh38.cdna.all.fa.gz](ftp://ftp.ensembl.org/pub/release-92/fasta/homo_sapiens/cdna/Homo_sapiens.GRCh38.cdna.all.fa.gz)

を利用します。

### fastqファイルの取得
kallistoの解析には今回ペアエンドのfastqファイルを利用します。ただしfastqファイルは非常に多くダウンロードに時間がかかるため、
今回は圧縮されファイルサイズの小さいsraファイルをダウンロードして、これをfastq-dumpを使ってペアエンドのfastqを自分で生成しています。

解析するサンプルは、低酸素状態のCell line（RCC4-EV, DRR100656）と、低酸素状態をレスキューしたRCC4-VHL（DRR100657）です。

詳細は[scientific reports](http://www.nature.com/articles/s41598-018-27220-8) をご覧ください。

実際に入手したファイル（低酸素状態・コントロール）は下記の通りです。

- DRR100657.sra: ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/sra/ByExp/sra/DRX/DRX094/DRX094090/DRR100657/DRR100657.sra
- DRR100656.sra: ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/sra/ByExp/sra/DRX/DRX094/DRX094089/DRR100656/DRR100656.sra

## 3. 発現解析

### SRAファイルからfastqファイルを生成する

ダウンロードしたSRAファイルをsratoolkitを使ってペアエンドのfastqファイルに変換します。

```
$ fastq-dump --split-files 取得したSRAファイルのパス
```
配布したSRAを使うのならば、以下のような二つのファイルができるはず。
```
$ fastq-dump --split-files DRR100656.sra 
$ ls
DRR100656_1.fastq    DRR100656_2.fastq
```
コントロールのサンプルについてもfastqを生成しておきます。
```
$ fastq-dump --split-files  DRR100657.sra
```


※　fastq-dumpより速いpfastq-dumpの紹介

fastq-dumpはそれなりに時間のかかる処理ですが、お使いの端末のSSDに余裕があるようでしたら、
より処理速度の速い並列版の __pfastq-dump__ をおすすめします。

インストールはgithubから。
```
$ git clone https://github.com/inutano/pfastq-dump
$ cd pfastq-dump
$ chmod a+x bin/pfastq-dump
```
そして下記例のように実行できます。
```
$ pfastq-dump –threads 4 –outdir fq/ DRR1551404.sra
```

詳細は下記リンクをご参考にしてください。
[並列版 fastq-dump](https://bonohu.wordpress.com/2017/06/20/parallel-fastq-dump/)



### kallisto indexでヒトのトランスクリプトームのインデックスを作成する 
kallistoではindexファイルを配布するより、kallisto indexで構築した方が速いということで、
[Ensembl](https://www.ensembl.org/info/data/ftp/index.html)からfastaファイルをダウンロードして
自分で作ることを推奨しているようです。今回ハードディスクにヒトのcDNA（FASTA）が含まれています。
このファイルを使って（gzipのままでOK）下記のようにインデックスを作成します。

```
$ kallisto index -i ex_GRCh38_index Homo_sapiens.GRCh38.cdna.all.fa.gz
```

### 定量する
kallisto quantoで発現量を定量します。ペアエンドの定量なので、変換したfastqファイルの
_1.fastqと_2.fastをセットで使います。

```
$ kallisto quant -i index_file_name -o output_dir_name fastq_file_1 fastq_file_2
```
kallistoはthread数をオプションで指定できるので、例えばコア数2のMacBook Airでは下記のような例となります。
```
$ kallisto quant -t 2 -i GRCh38_index -o DRR100656_result/ DRR100656_1.fastq DRR100656_2.fastq 
```
※ お使いの端末に合わせて-tオプションの値は変更して下さい。上記では2としていますが、MacBook Airでは4でも処理できました（貸出用端末で検証）。

kallistoの使い方の詳細については[こちらのブログ](https://bonohu.wordpress.com/2017/11/15/kallisto/)が参考になると思います。

### 確認
kallisto quantoが終了すると、指定したoutput dirにいくつかのファイルができているはずです。
このうち、abundance.tsvを見てみます
```
$ sort -k 5 -rn abundance.tsv | less
```

上記の例では、5番目のカラム、Transcripts Per Million（TPM）を発現量の多い順にソートして表示しています。


## 4. 散布図を描いてみる

### Jupyterを使って散布図をプロット
二つのサンプルでそれぞれ発現量を計算したら、Jupyter Noteboookを使って、
転写産物ごとのそれぞれの発現量を散布図にプロットして確認します。

- Jupyter Notebookを起動します。
```
$ jupyter notebook
```
以下Jupyter Notebookのコードセルに記述する内容です。

- ライブラリをインポート
```python
from pandas import Series, DataFrame
import pandas as pd
import csv
import matplotlib.pyplot as plt
import numpy as np
```

- 出力したabundance.tsvをpandasに読み込む
```markdown
e1 = pd.read_table(DRR100656のabunndance.tsv)
e1 = e1.drop(columns=['length', 'eff_length', 'est_counts'])
e1.columns = ['target_id', 'TPM_DRR100656']

e2 = pd.read_table(DRR100657のabunndance.tsv)
e2 = e2.drop(columns=['length', 'eff_length', 'est_counts'])
e2.columns = ['target_id', 'TPM_DRR100657']

e = pd.merge(e1, e2, on='target_id')
```
- DataFrameに必要なフィールド（だけ）が含まれていることを確認
```python
e.head()
```
- そのままmatplotlibで散布図を書く
```python
plt.scatter(e.TPM_DRR100656, e.TPM_DRR100657)
plt.xlabel('DRR100657')
plt.ylabel('DRR100656')
```

- TPMの対数を計算しdfに追加する
```python
e['log_DRR100656'] = np.log10(e['TPM_RR100656'] + 1)
e['log_DRR100657'] = np.log10(e['TPM_DRR100657'] + 1)
e['diff'] = abs(e['log_DRR100656'] - e['log_DRR100657'])
```

- 対数値を散布図にプロットする
```python
plt.scatter(e.log_DRR100656, e.log_DRR100657)
plt.xlabel('DRR100657')
plt.ylabel('DRR100656')
```

### plotlyで散布図を描画してみる
plotlyで同様の散布図を描画してみます。
ブラウザでレンダリングするにはかなり大きいデータですので、余裕があれば試してみてください。

```python
from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
import plotly.graph_objs as go

init_notebook_mode(connected=False)  

data = [go.Scatter(
        x = e['log_DRR100656'],
        y = e['log_DRR100657'],
        mode = 'markers'
    )]

plot(data, filename='basic-scatter') 
# plotの代わりにiplotでも良いですが、メモリが足りずNotebookが固まる可能性もあります
```
### 発現量の対数値の差でトランスクリプトをソートしファイルに書き出す
次のエンリッチメント解析のため、発現量の差でトランスクリプトをソートしてファイルに書き出します。
```
e.sort_values('diff', ascending=False)[['target_id', 'diff']].iloc[:1000].to_csv('ranking_1000.txt', index=False, sep='\t')
```


## 5. metascapeでエンリッチメント解析

- [metascape.org](http://metascape.org/gp/index.html#/main/step1) のサイトでjupyte notebookで出力したファイルをアップロードします。
- "Express Analysis"ボタンを押します。
- 数十秒後に処理が終わるので"Analysis Report Page"を押して"Metascape Gene List Analysis Report"を確認します。
