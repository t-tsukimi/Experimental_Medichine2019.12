# QIIME2: Wet研究者も使える微生物叢解析ツール

本ページは実験医学2019年12月号（羊土社）「クローズアップ実験法」の補足ページとなります。  


### コマンド集


QIIME 2のダウンロード・インストール
```sh
curl -OL  https://data.qiime2.org/distro/core/qiime2-2019.7-py36-osx-conda.yml
conda env create -n qiime2-2019.7 --file qiime2-2019.7-py36-osx-conda.yml
```
作業ディレクトリの作成・シーケンスデータのダウンロード
```sh
mkdir qiime2
cd qiime2 #以降qiime2ディレクトリで作業する

curl -OL http://www.mothur.org/w/images/d/d6/MiSeqSOPData.zip #約37MBのファイルがダウンロードさ
unzip MiSeqSOPData.zip #解凍

mkdir input inputフォルダの作成
mv MiSeq_SOP/*.fastq input #使用するデータのみinputフォルダに移動
gzip input/* # fastq.gzファイルに圧縮
```
manifestファイル、metadataファイルのダウンロード
```sh
#manifestファイルのダウンロード
curl -O https://raw.githubusercontent.com/t-tsukimi/Experimental_Medichine2019.12/master/qiime2/manifest.txt

#metadataファイルのダウンロード
curl -O https://raw.githubusercontent.com/t-tsukimi/Experimental_Medichine2019.12/master/qiime2/sample-metadata.txt
```


シーケンスデータのインポート
```sh
conda activate qiime2-2019.7 #QIIME 2の起動
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \ #インポート後のデータ形式
  --input-path manifest.txt \ #manifestファイル名
  --output-path demux.qza \ #インポート後のファイル名
  --input-format PairedEndFastqManifestPhred33V2 #読み込むデータ形式

qiime demux summarize \ #シーケンスデータの可視化
  --i-data demux.qza \ #可視化したいqzaファイル名
  --o-visualization demux.qzv #出力するqzvファイル名
```
クオリティーコントロール・アセンブリ
```sh
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux.qza \ #インプットファイル名
  --p-trim-left-f 0 \ #Foward readの5′末端から削る塩基数
  --p-trim-left-r 0 \ #Reverse readの5′末端から削る塩基数
  --p-trunc-len-f 240 \ #Foward readの3′末端から残す塩基数
  --p-trunc-len-r 160 \ #Reverse readの3′末端から残す塩基数
  --o-table table.qza \ #出力するtableファイル名
  --o-representative-sequences rep-seqs.qza \#出力する代表配列ファイル名
  --o-denoising-stats denoising-stats.qza \ #出力するクオリティーコントロール結果ファイル名
  --p-n-threads 0 #解析に使用するスレッド数

# 以降結果の可視化
qiime feature-table summarize \
  --i-table table.qza \ #入力するtableファイル名
  --o-visualization table.qzv \#出力するqzvファイル名
  --m-sample-metadata-file sample-metadata.txt #metadataファイル名

qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \ #入力する代表配列ファイル名
  --o-visualization rep-seqs.qzv #出力するqzvファイル名

qiime metadata tabulate \
  --m-input-file denoising-stats.qza \#入力するクオリティーコントロール結果ファイル名
  --o-visualization denoising-stats.qzv #出力するqzvファイル名
```
代表配列の系統樹作成
```sh
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \ #入力する代表配列ファイル名
  --o-alignment phylogeny/aligned-rep-seqs.qza \ #出力するアラインメントファイル名
  --o-masked-alignment phylogeny/masked-aligned-rep-seqs.qza \
  --o-tree phylogeny/unrooted-tree.qza \ #出力する無根系統樹ファイル名
  --o-rooted-tree phylogeny/rooted-tree.qza \ #出力する有根系統樹ファイル名
  --output-dir phylogeny/ #出力ディレクトリ名
```

α多様性β多様性指数の算出
```sh
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny phylogeny/rooted-tree.qza \ #入力する有根系統樹ファイル名
  --i-table table.qza \ #入力するtableファイル名
  --p-sampling-depth 2518 \ #算出に用いるリード数
  --m-metadata-file sample-metadata.txt \ #metadataファイル名
  --output-dir core-metrics-results #出力ディレクトリ名

qiime metadata tabulate \ #α多様性（faith_pdの可視化）
  --m-input-file core-metrics-results/faith_pd_vector.qza \ #入力するqzaファイル名
  --o-visualization core-metrics-results/faith_pd_vector.qzv #入力するqzaファイル名
```

Rarefaction curveの算出
```sh
qiime diversity alpha-rarefaction \
  --i-table table.qza \ #入力するtableファイル名
  --i-phylogeny phylogeny/rooted-tree.qza \ #入力する有根系統樹ファイル名
  --p-max-depth 2518 \ #算出に用いるリード数
  --m-metadata-file sample-metadata.txt \ #metadataファイル名
  --o-visualization alpha-rarefaction.qzv #出力ファイル名
```
細菌種同定のための分類器作成
```sh
curl -OL ftp://greengenes.microbio.me/greengenes_release/gg_13_5/gg_13_8_otus.tar.gz #データベースのダウンロード
mkdir classifier
mv gg_13_8_otus.tar.gz classifier 
tar zxvf classifier/gg_13_8_otus.tar.gz -C classifier #解凍

#教師データ（Green genesデータベースの読み込み）
qiime tools import \
  --type 'FeatureData[Sequence]' \ #インポート後のデータ形式
  --input-path classifier/gg_13_8_otus/rep_set/99_otus.fasta \ #インポートするファイル名     
  --output-path classifier/99_otus.qza #インポート後のファイル名

qiime tools import \
  --type 'FeatureData[Taxonomy]' \
  --input-format HeaderlessTSVTaxonomyFormat \ #インポート後のデータ形式
  --input-path classifier/gg_13_8_otus/taxonomy/99_otu_taxonomy.txt \ #インポートするファイル名
  --output-path classifier/ref-taxonomy.qza #インポート後のファイル名

#V4領域の抽出
qiime feature-classifier extract-reads \ 
  --i-sequences classifier/99_otus.qza \ #教師データとなるファイル名
  --p-f-primer GTGCCAGCMGCCGCGGTAA \ #PCRに用いたFowardプライマー配列
  --p-r-primer GGACTACHVGGGTWTCTAAT \ #PCRに用いたReverseプライマー配列
  --p-min-length 100 \ #切り出し後に100bpより短い配列は削除
  --p-max-length 400 \ #切り出し後に400bpより長い配列は削除
  --o-reads classifier/ref-seqs_gg99_v4.qza #出力ファイル名

#学習
qiime feature-classifier fit-classifier-naive-bayes \ 
  --i-reference-reads classifier/ref-seqs_gg99_v4.qza \ #切り出した教師データ（配列）ファイル名
  --i-reference-taxonomy classifier/ref-taxonomy.qza \ #切り出した教師データファイル（分類）名
  --o-classifier classifier/classifier_gg99_v4.qza #出力する分類器ファイル名

```
細菌叢組成の算出
```sh
mkdir taxonomy

#細菌種の同定
qiime feature-classifier classify-sklearn \ 
  --i-classifier classifier classifier_gg99_v4.qza \ 
  --i-reads rep-seqs.qza \
  --o-classification taxonomy/taxonomy_v4.qza

#可視化
qiime metadata tabulate \
  --m-input-file taxonomy/taxonomy_v4.qza \
  --o-visualization taxonomy/taxonomy_v4.qzv


#100 %積み上げ棒グラフ作成
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy/taxonomy_v4.qza \ 
  --m-metadata-file sample-metadata.txt \
  --o-visualization taxonomy/taxa-bar-plots_v4.qzv #積み上げ棒グラフの作成
```