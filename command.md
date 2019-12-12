# QIIME2: Wet研究者も使える細菌叢解析ツール

本ページは実験医学2019年12月号（羊土社）「クローズアップ実験法」の補足ページとなります。  


### コマンド集


#### QIIME 2のダウンロード・インストール
```sh
curl -OL  https://data.qiime2.org/distro/core/qiime2-2019.7-py36-osx-conda.yml
conda env create -n qiime2-2019.7 --file qiime2-2019.7-py36-osx-conda.yml
```
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/qiime2_mac_install1.png)

QIIME 2が正常にインストールされたか確認するためには、下記コマンドでQIIME 2を起動し、ヘルプコマンド実行すれば良い。エラーなどが表示されなければ正常にインストールされている。
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/qiime2_mac_install2.png)


#### 作業ディレクトリの作成・シーケンスデータのダウンロード
```sh
mkdir qiime2
cd qiime2 #以降qiime2ディレクトリで作業する

curl -OL http://www.mothur.org/w/images/d/d6/MiSeqSOPData.zip #約37MBのファイルがダウンロードさ
unzip MiSeqSOPData.zip #解凍

mkdir input inputフォルダの作成
mv MiSeq_SOP/*.fastq input #使用するデータのみinputフォルダに移動
gzip input/* # fastq.gzファイルに圧縮
```
#### manifestファイル、metadataファイルのダウンロード
```sh
#manifestファイルのダウンロード
curl -O https://raw.githubusercontent.com/t-tsukimi/Experimental_Medichine2019.12/master/qiime2/manifest.txt

#metadataファイルのダウンロード
curl -O https://raw.githubusercontent.com/t-tsukimi/Experimental_Medichine2019.12/master/qiime2/sample-metadata.txt
```


#### シーケンスデータの読み込み

```sh
#QIIME 2の起動
conda activate qiime2-2019.7 
```

```sh
#シーケンスデータの読み込み
qiime tools import \ 
  --type 'SampleData[PairedEndSequencesWithQuality]' \ 
  --input-path manifest.txt \ 
  --output-path demux.qza \ 
  --input-format PairedEndFastqManifestPhred33V2 
```
- --type: インポート後のデータ形式  
- --input-path: manifestファイル名
- --output-path: インポート後のファイル名
- --input-format: 読み込むデータ形式

```sh
#読み込んだデータ概要の可視化
qiime demux summarize \ 
  --i-data demux.qza \ 
  --o-visualization demux.qzv 
```
- --i-data: 可視化したいqzaファイル名
- --o-visualization: 出力するqzvファイル名  


#### クオリティーコントロール・アセンブリ
```sh
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux.qza \ 
  --p-trim-left-f 0 \ 
  --p-trim-left-r 0 \ 
  --p-trunc-len-f 240 \ 
  --p-trunc-len-r 160 \ 
  --o-table table.qza \ 
  --o-representative-sequences rep-seqs.qza \ 
  --o-denoising-stats denoising-stats.qza \ 
  --p-n-threads 0 
```
- --i-demultiplexed-seqs: インプットファイル名
- --p-trim-left-f (r): Foward (Reverse) readの5′末端から削る塩基数
- --p-trunc-len-f (r): Foward (Reverse) readの5′末端から残す塩基数
- --o-representative-sequences: 出力するtableファイル名
- --o-denoising-stats: 出力するクオリティーコントロール結果ファイル名
- --p-n-threads: 解析に使用するスレッド数 (0を指定すると現在使用できる最大スレッド数で実行）

#### クオリティーコントロール・アセンブリ結果の可視化
```sh
#tableファイルの可視化
qiime feature-table summarize \ 
  --i-table table.qza \ 
  --o-visualization table.qzv \ 
  --m-sample-metadata-file sample-metadata.txt 
```
- --i-table: 入力するtableファイル名
- --o-visualization: 出力するqzvファイル名
- --m-sample-metadata-file: metadataファイル名

```sh
#代表配列ファイルの可視化
qiime feature-table tabulate-seqs \ 
  --i-data rep-seqs.qza \ 
  --o-visualization rep-seqs.qzv 
```
--i-data: 入力する代表配列ファイル名  
--o-visualization: 出力するqzvファイル名

```sh
#クオリティーコントロール結果ファイル名
qiime metadata tabulate \ 
  --m-input-file denoising-stats.qza \ 
  --o-visualization denoising-stats.qzv 
```
- --m-input-file: 入力するクオリティーコントロール結果ファイル名
- --o-visualization: 出力するqzvファイル名


#### 代表配列の系統樹作成
```sh
qiime phylogeny align-to-tree-mafft-fasttree \ 
  --i-sequences rep-seqs.qza \ 
  --o-alignment phylogeny/aligned-rep-seqs.qza \ 
  --o-masked-alignment phylogeny/masked-aligned-rep-seqs.qza \ 
  --o-tree phylogeny/unrooted-tree.qza \ 
  --o-rooted-tree phylogeny/rooted-tree.qza \ 
  --output-dir phylogeny/ 
```
- --i-sequences: 入力する代表配列ファイル名
- --o-alignment: 出力するアラインメントファイル名
- --o-tree: 出力する無根系統樹ファイル名
- --o-rooted-tree: 出力する有根系統樹ファイル名
- --output-dir: 出力ディレクトリ名


#### α多様性・β多様性指数の算出
```sh
qiime diversity core-metrics-phylogenetic \ 
  --i-phylogeny phylogeny/rooted-tree.qza \ 
  --i-table table.qza \ 
  --p-sampling-depth 2518 \ 
  --m-metadata-file sample-metadata.txt \ 
  --output-dir core-metrics-results 
```
- --i-phylogeny: 入力する有根系統樹ファイル名
- --i-table: 入力するtableファイル名
- --p-sampling-depth: 算出に用いるリード数 (この値に満たないサンプルは自動的に除外される)
- --m-metadata-file: metadataファイル名
- --output-dir: 出力ディレクトリ名
```sh
#α多様性 (faith pd)の可視化
qiime metadata tabulate \ 
  --m-input-file core-metrics-results/faith_pd_vector.qza \ 
  --o-visualization core-metrics-results/faith_pd_vector.qzv 
```
- --m-input-file: 入力するqzaファイル名
- --o-visualization: #出力するqzvファイル名


#### Rarefaction curveの算出
```sh
qiime diversity alpha-rarefaction \ 
  --i-table table.qza \ 
  --i-phylogeny phylogeny/rooted-tree.qza \ 
  --p-max-depth 2518 \ 
  --m-metadata-file sample-metadata.txt \ 
  --o-visualization alpha-rarefaction.qzv 
```
- --i-table: 入力するtableファイル名
- --i-phylogeny: 入力する有根系統樹ファイル名
- --p-max-depth: 算出に用いるリード数
- --m-metadata-file: metadataファイル名
- --o-visualization: 出力ファイル名  


#### 細菌種同定のための分類器作成
```sh
#使用するデータベース(GreenGenes)のダウンロード及び解凍
curl -OL ftp://greengenes.microbio.me/greengenes_release/gg_13_5/gg_13_8_otus.tar.gz #データベースのダウンロード
mkdir classifier
mv gg_13_8_otus.tar.gz classifier 
tar zxvf classifier/gg_13_8_otus.tar.gz -C classifier #解凍
```

```sh
#配列ファイルの読み込み
qiime tools import \
  --type 'FeatureData[Sequence]' \ 
  --input-path classifier/gg_13_8_otus/rep_set/99_otus.fasta \      
  --output-path classifier/99_otus.qza 
```
- --type: インポート後のデータ形式
- --input-path: インポートするファイル名
- --output-path: インポート後のファイル名

```sh
#taxonomyファイルの読み込み
qiime tools import \
  --type 'FeatureData[Taxonomy]' \
  --input-format HeaderlessTSVTaxonomyFormat \ 
  --input-path classifier/gg_13_8_otus/taxonomy/99_otu_taxonomy.txt \ 
  --output-path classifier/ref-taxonomy.qza 
```
- --type: インポート後のデータ形式
- --input-format: 読み込むデータ形式
- --input-path: インポートするファイル名
- --output-path: インポート後のファイル名


```sh
#V4領域の抽出
qiime feature-classifier extract-reads \ 
  --i-sequences classifier/99_otus.qza \ 
  --p-f-primer GTGCCAGCMGCCGCGGTAA \ 
  --p-r-primer GGACTACHVGGGTWTCTAAT \ 
  --p-min-length 100 \ 
  --p-max-length 400 \ 
  --o-reads classifier/ref-seqs_gg99_v4.qza 
```
- --i-sequences: #教師データとなるファイル名 (今回はGreenGenesデータベース)
- --p-f(r)-primer: PCRに用いたFoward (Reverse) プライマー配列
- --p-min(max)-length: 切り出し後にこの値より短い（長い）配列を削除
- --o-reads: 出力ファイル名

```sh
#学習
qiime feature-classifier fit-classifier-naive-bayes \ 
  --i-reference-reads classifier/ref-seqs_gg99_v4.qza \ 
  --i-reference-taxonomy classifier/ref-taxonomy.qza \ 
  --o-classifier classifier/classifier_gg99_v4.qza 

```
- --i-reference-reads: 切り出した教師データ（配列）ファイル名
- --i-reference-taxonomy: 切り出した教師データ（taxnomy）ファイル名
- --o-classifier: 出力する分類器ファイル名

#### 細菌叢組成の算出
```sh
mkdir taxonomy

#細菌種の同定
qiime feature-classifier classify-sklearn \ 
  --i-classifier classifier/classifier_gg99_v4.qza \ 
  --i-reads rep-seqs.qza \ 
  --o-classification taxonomy/taxonomy_v4.qza
```
- --i-classifier: 分類器ファイル名
- --i-reads: 種同定する代表配列ファイル名
- --o-classification: 出力ファイル名

```sh
#同定結果の可視化
qiime metadata tabulate \ 
  --m-input-file taxonomy/taxonomy_v4.qza \ 
  --o-visualization taxonomy/taxonomy_v4.qzv
```
- --m-input-file: 同定結果ファイル名
- --o-visualization: 出力ファイル名

```sh
#100%積み上げ棒グラフの作成
qiime taxa barplot \ 
  --i-table table.qza \ 
  --i-taxonomy taxonomy/taxonomy_v4.qza \ 
  --m-metadata-file sample-metadata.txt \ 
  --o-visualization taxonomy/taxa-bar-plots_v4.qzv 
```
- --i-table: tableファイル名
- --i-taxonomy: 同定結果ファイル名
- --m-metadata-file: metadataファイル名
- --o-visualization: 出力ファイル名
