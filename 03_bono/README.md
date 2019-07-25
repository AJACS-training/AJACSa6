# 遺伝子発現データ解析の実際（ハンズオン）

## Macの環境設定

### (bio)condaのインストール

[Biocondaの公式インストール手順](https://bioconda.github.io/user/install.html)を元に進める。

minicondaをまずインストール

```
# インストーラーを取得
% curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh
# インストーラー実行
% sh Miniconda3-latest-MacOSX-x86_64.sh
```

biocondaを使えるようにchannel追加

```
# condaでchannel追加
% conda config --add channels defaults
% conda config --add channels bioconda
% conda config --add channels conda-forge
```

## 遺伝子発現データ解析

### データ取得

公共DBから計算に必要なデータを取得する必要があります。

Reference transcriptome配列は今回、[Gencodeプロジェクト](https://www.gencodegenes.org)で維持されている配列セットを利用します。

またfastqファイルはさらに大きく、ダウンロードに時間がかかるため、 普通は圧縮されファイルサイズの小さいsraファイルをダウンロードして、これを`fasterq-dump`を使ってペアエンドのfastqを自分で生成します。

しかしながら、これらのファイルのダウンロード操作は大変時間がかかるため、今回の講習では取得済みのReference transcriptome配列とsraファイルを入れたハードディスクドライブを配布して利用してもらいます。

解析するサンプルは、低酸素状態のCell line（RCC4-EV, DRR100656）と、低酸素状態をレスキューしたRCC4-VHL（DRR100657）です。
詳細は原著論文([doi:10.1038/s41598-018-27220-8](https://doi.org/10.1038/s41598-018-27220-8))をご覧ください。
実際に入手したファイル（低酸素状態・コントロール）は下記の通りです。

- Reference transcriptome配列: `gencode.v31.transcripts.fa.gz`
- sraファイル
  - `DRR100657.sra` 入手元URL `ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/sra/ByExp/sra/DRX/DRX094/DRX094090/DRR100657/DRR100657.sra`
  - `DRR100656.sra` 入手元URL `ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/sra/ByExp/sra/DRX/DRX094/DRX094089/DRR100656/DRR100656.sra`

### FASTQファイルの生成

sra-toolsをインストールすると、同時にfasterq-dumpがインストールされる。
このfasterq-dumpコマンドでsraをfastqに展開することができる。
生成されたfastqはサイズが非常に大きいので要注意。

```
# condaを使ってsra-toolsをインストール。fasterq-dumpがインストールされる。
% conda install sra-tools
# sraファイルをfastqに変換
% fasterq-dump DRR100656.sra
% fasterq-dump DRR100657.sra
```

そして、生成されたファイルを確認する。

```
# 生成されたファイルをlsで確認
% ls -l
-rw-r--r-- 1 bono staff 8396394240  7  3 21:14 DRR100656.sra_1.fastq
-rw-r--r-- 1 bono staff 8396394240  7  3 21:14 DRR100656.sra_2.fastq
-rw-r--r-- 1 bono staff 8215871450  7  3 21:01 DRR100657.sra_1.fastq
-rw-r--r-- 1 bono staff 8215871450  7  3 21:01 DRR100657.sra_2.fastq
# wcでそれぞれのファイルの行数を数える
% wc -l *.fastq
92772772 DRR100656.sra_1.fastq
92772772 DRR100656.sra_2.fastq
90782932 DRR100657.sra_1.fastq
90782932 DRR100657.sra_2.fastq
367111408 total
```

ペアエンドのため、1つのエントリあたり2つのファイルができ、それぞれは同じ行数となっている。
この例では、ヘッダ行に関しても全く同じ文字数だったため、行数のみならず、バイト数も同一となっている。

### FASTQファイルのトリミング

塩基配列解読の結果には、品質の悪い領域も含んでいる。
それを除いた方が結果が良くなるので、それらを除く処理（トリミング）を通常行う。
ここではそのトリミングと品質管理(Quality Control:QC)を同時に実行してくれる`trim_galore`を用いた実例を紹介する。

```
# trim_galoreでトリミングとQC
% conda install trim-galore
# trim_galoreの実行
% trim_galore --fastqc --trim1 --gzip --paired DRR100656.sra_1.fastq DRR100656.sra_2.fastq
# 生成されたファイルの確認
% ls -l
-rw-r--r-- 1 bono staff 1926035002  7  3 17:15 DRR100656.sra
-rw-r--r-- 1 bono staff 8396394240  7  3 21:14 DRR100656.sra_1.fastq
-rw-r--r-- 1 bono staff       4016  7  3 22:33 DRR100656.sra_1.fastq_trimming_report.txt
-rw-r--r-- 1 bono staff 1424837751  7  3 22:49 DRR100656.sra_1_val_1.fq.gz
-rw-r--r-- 1 bono staff     666141  7  3 22:51 DRR100656.sra_1_val_1_fastqc.html
-rw-r--r-- 1 bono staff     434721  7  3 22:51 DRR100656.sra_1_val_1_fastqc.zip
-rw-r--r-- 1 bono staff 8396394240  7  3 21:14 DRR100656.sra_2.fastq
-rw-r--r-- 1 bono staff       4185  7  3 22:49 DRR100656.sra_2.fastq_trimming_report.txt
-rw-r--r-- 1 bono staff 1436903935  7  3 22:49 DRR100656.sra_2_val_2.fq.gz
-rw-r--r-- 1 bono staff     662392  7  3 22:54 DRR100656.sra_2_val_2_fastqc.html
-rw-r--r-- 1 bono staff     422191  7  3 22:54 DRR100656.sra_2_val_2_fastqc.zip
# trim_galoreの実行(DRR100657)
% trim_galore --fastqc --trim1 --gzip --paired DRR100657.sra_1.fastq DRR100657.sra_2.fastq
# 生成されたファイルの確認
% ls -l DRR100657*
(略)
```

この例の場合（リード数：約23Mx2）、dual-coreのMacBookProで、`fasterq-dump`の実行はそれぞれ2分弱、`trim_galore`の実行はそれぞれ30分弱かかった（作業ディレクトリは外付けHDではなく、内臓のSSD）。
2セットあるので、**ダウンロードしたあとも約1時間は前処理にかかる**、ということである。

上述の処理では`trim_galore`がFASTQファイルをgzip圧縮してくれるが(`--gzip`オプション)、普通はコマンドラインで自ら実行する必要がある。
その場合、並列版gzipコマンドである`pigz`を使うことが多い。
利用可能なCPUを全て使ってファイル圧縮をするため、体感時間が劇的に早くなる。

```
# condaを使って並列化gzipであるpigzをインストール
% conda install pigz
```

### salmonのインストール

遺伝子発現定量化ツールのsalmonをインストール

```
# condaでsalmonをインストール
% conda install salmon
```

### index作成

salmon検索用にindex作成する。
この操作は基本1回目だけでよく、異なる生物種やトランスクリプトーム配列セットに対して検索する場合には新規に作成し直す必要がある。

```
# salmon index を実行
% salmon index -p 2 -t gencode.v31.transcripts.fa.gz -i gencode_v31
```

dual-coreのMacBookProで、4分弱で実行終了した。

### 発現量の定量

salmonによる発現量の定量を実行する。

```
# salmon quant を実行
% salmon quant -p 2 -i gencode_v31 -l A --validateMappings \
-1 DRR100656.sra_1_val_1.fq.gz  -2 DRR100656.sra_2_val_2.fq.gz \
-o salmon_output_DRR100656
% salmon quant -p 2 -i gencode_v31 -l A --validateMappings \
-1 DRR100657.sra_1_val_1.fq.gz  -2 DRR100657.sra_2_val_2.fq.gz \
-o salmon_output_DRR100657
```

dual-coreのMacBookProで、それぞれ8分弱で実行終了した。

### 発現定量結果の確認

```
# ls で出力先ディレクトリの中身をそれぞれ確認
% ls salmon_output_DRR100656
aux_info/  cmd_info.json  libParams/  lib_format_counts.json  logs/  quant.sf
% ls salmon_output_DRR100656
aux_info/  cmd_info.json  libParams/  lib_format_counts.json  logs/  quant.sf
# それぞれのディレクトリのquant.sfが発現定量結果
% less salmon_output_DRR100656/quant.sf
% less salmon_output_DRR100657/quant.sf
# 左から4カラム目がTPM値なので、高いもの順に並べて見てみる。'q'を押すと終了する
% sort -k 4 -rn salmon_output_DRR100656/quant.sf | less
% sort -k 4 -rn salmon_output_DRR100657/quant.sf | less
```

## 解析結果の可視化

発現値は定量化されましたが、これだけでは特徴がわからないので、二つの実験状態をx軸とy軸にもつ散布図を書いて可視化する。

### jupyter, 可視化用のライブラリをインストール

```
# condaでjupyter他をインストール
% conda install jupyter
% conda install pandas
% conda install matplotlib
```


### Jupyter Notebookを起動

```
% jupyter notebook
```

### 出力したabundance.tsvをpandasに読み込む

```
import pandas as pd
e1 = pd.read_table('salmon_output_DRR100656/quant.sf')
e1 = e1.drop(columns=['Length', 'EffectiveLength', 'NumReads'])
e1.columns = ['Name', 'TPM_DRR100656']
e2 = pd.read_table('salmon_output_DRR100657/quant.sf')
e2 = e2.drop(columns=['Length', 'EffectiveLength', 'NumReads'])
e2.columns = ['Name', 'TPM_DRR100657']

# 二つのデータを'target_id'で結合
e = pd.merge(e1, e2, on='Name')

# DataFrameに必要なフィールドが含まれていることを確認
e.head()

```

### matplotlibで散布図を描く

```
plt.scatter(e.TPM_DRR100656, e.TPM_DRR100657)
plt.xlabel("DRR100656")
plt.ylabel("DRR100657")
```

### seabornが好きなひとはこちら

```
%matplotlib inline
import seaborn as sns
ax = sns.scatterplot(x="TPM_DRR100656", y="TPM_DRR100657", data=e)

# jointplot
sns.jointplot(e[("TPM_DRR100656")], e[("TPM_DRR100657")])

```

## 【発展】CWLを使ってsalmonを実行する

*注意：以下はハンズオンのメインコンテンツではありません*

上記のハンズオンが簡単にできてしまう人向けに。
Common Workflow Language (CWL)を使って実行してみよう。
下記のブログを参考に。

- [Running salmon via CWL](https://bonohu.github.io/running-salmon-via-cwl.html)
- [Common Workflow Language(CWL)を使ってkallistoを実行する](http://bonohu.jp/blog/running-kallisto-via-cwl.html)

### Dockerのインストール

[公式のサイトのDLリンク](https://docs.docker.com/docker-for-mac/install/)から
Docker Desktop for MacのインストーラーをDLして、表示される手順通りにインストールを進めます
（dockerhubへのサインインが必要です）。

### cwltoolのインストール

[@manabuishiirbのブログ](https://qiita.com/manabuishiirb/items/d866af4a5b1032eba374)ではpipで入れる方法が紹介されているが、現在ではcondaで入れる方がよい（まぜるな危険）。

```
# condaでcwltoolをインストール
% conda install cwltool
```

### pitagora-cwlをgit clone

```
# pitagora-cwlをgit clone
% git clone https://github.com/pitagora-network/pitagora-cwl
```

### 発展課題

1. CWLを使って、上述のReference transcriptome配列とFASTQ配列を使って
  A. salmonで発現定量せよ
  B. kallistoで発現定量せよ
2. 上述のPythonコードを再利用して、salmonとkallistoでの発現定量値を比較する散布図を作成せよ
3. その結果から言えるプログラムの特性を考察せよ
4. さらに時間がある場合には、[HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml)による発現定量も同じFASTQ配列に対して行い、salmonで計算した結果と比較解析せよ

参考：Shizuoka-ngs/appendix [発現定量化ツールsalmonによる定量とkallistoとの比較](https://github.com/shizuoka-ngs/appendix/blob/master/kallisto_and_salmon.md)
