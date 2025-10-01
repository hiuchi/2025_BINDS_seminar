# BINDS発現・機能解析インシリコ解析融合ユニット講習会

## 1. 概要
### 1.1 対象のデータ
-  [Schneider, Kai Markus et al. Cell, Volume 186, Issue 13, 2823-2838.e20](https://doi.org/10.1016/j.cell.2023.05.001)
-  心理的ストレスを与えるため7日間拘束したマウスにおける結腸全組織のバルク RNA-seq データ、control 群と stress 群それぞれ各 n=5
### 1.2 解析設計
- 生データ（FASTQ）を nf-core/rnaseq で定量
- 定量結果を nf-core/differentialabundance へ入力し、control vs stress の2群発現変動解析を実施
### 1.3 検証環境
- MacBook Air (M4, 13-inch, 2024)
  - CPU : 10コア
  - メモリ : 16GB
  - ストレージ : 256GB

---

## 2. 環境構築
### 2.1 Docker のインストール
[公式サイト](https://www.docker.com)から公式ドキュメントに従って Docker Desktop をインストールする。
### 2.2 Homebrew のインストール
[公式サイト](https://brew.sh)に記述されているコマンドを実行後、パスを通す。
  ```zsh
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/ユーザ名/.zprofile
  eval "$(/opt/homebrew/bin/brew shellenv)”
  ```
### 2.3 Nextflow のインストール
Homebrew を使って JAVA と Nextflow をインストールする。
  ```zsh
  brew install openjdk@11
  brew install nextflow
  ```
---

## 3. リファレンスファイルとFASTQファイルのダウンロード
### 3.1 各データについて
#### 3.1.1 FASTQ ファイル
シーケンサーから出力された塩基配列とそのクオリティが記述されているファイル
```
@SRR24350711.1 NB551353:21:HYMVTBGX9:1:11101:19311:1044 length=76
AACAANTCCAGCCCCCACTGCGTGTGGCGTTCCAGCACCTCAAACTGATCCCACAACTCGGTACCCCAATCCATGC
+SRR24350711.1 NB551353:21:HYMVTBGX9:1:11101:19311:1044 length=76
AAAA6#/EAE/EAEEA/EE/EEE/EEE<E<//A/E</EAEE///A/EE//EA//<//EEE<</<E<<<</6<AAAA
@SRR24350711.2 NB551353:21:HYMVTBGX9:1:11101:13087:1052 length=76
CTGTANAACACCTTCACCTTTAACATCTAGACATTCGCTTTTCTTCTGTGTTCTCCAGTGTTTACTGTAATCTCCC
+SRR24350711.2 NB551353:21:HYMVTBGX9:1:11101:13087:1052 length=76
AAAAA#EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEAEEEEEEEEEEEEEEEEEEAEEE<EAEEAEEEEEEAEA<<
```
#### 3.1.2 ゲノムファイル
対象生物のゲノム配列が記述されたファイル
```
>ENSMUST00000193812.2|ENSMUSG00000102693.2|OTTMUSG00000049935.1|OTTMUST00000127109.1|4933401J01Rik-201|4933401J01Rik|1070|TEC|
AAGGAAAGAGGATAACACTTGAAATGTAAATAAAGAAAATACCTAATAAAAATAAATAAA
AACATGCTTTCAAAGGAAATAAAAAGTTGGATTCAAAAATTTAACTTTTGCTCATTTGGT
ATAATCAAGGAAAAGACCTTTGCATATAAAATATATTTTGAATAAAATTCAGTGGAAGAA
TGGAATAGAAATATAAGTTTAATGCTAAGTATAAGTACCAGTAAAAGAATAATAAAAAGA
AATATAAGTTGGGTATACAGTTATTTGCCAGCACAAAGCCTTGGGTATGGTTCTTAGCAC
```
#### 3.1.3 GTF ファイル
遺伝子アノテーションが記述されたファイル
```
chr1	HAVANA	gene	3143476	3144545	.	+	.	gene_id "ENSMUSG00000102693.2"; gene_type "TEC"; gene_name "4933401J01Rik"; level 2; mgi_id "MGI:1918292"; havana_gene "OTTMUSG00000049935.1";
chr1	HAVANA	transcript	3143476	3144545	.	+	.	gene_id "ENSMUSG00000102693.2"; transcript_id "ENSMUST00000193812.2"; gene_type "TEC"; gene_name "4933401J01Rik"; transcript_type "TEC"; transcript_name "4933401J01Rik-201"; level 2; transcript_support_level "NA"; mgi_id "MGI:1918292"; tag "basic"; tag "Ensembl_canonical"; tag "GENCODE_Primary"; havana_gene "OTTMUSG00000049935.1"; havana_transcript "OTTMUST00000127109.1";
chr1	HAVANA	exon	3143476	3144545	.	+	.	gene_id "ENSMUSG00000102693.2"; transcript_id "ENSMUST00000193812.2"; gene_type "TEC"; gene_name "4933401J01Rik"; transcript_type "TEC"; transcript_name "4933401J01Rik-201"; exon_number 1; exon_id "ENSMUSE00001343744.2"; level 2; transcript_support_level "NA"; mgi_id "MGI:1918292"; tag "basic"; tag "Ensembl_canonical"; tag "GENCODE_Primary"; havana_gene "OTTMUSG00000049935.1"; havana_transcript "OTTMUST00000127109.1";
chr1	ENSEMBL	gene	3172239	3172348	.	+	.	gene_id "ENSMUSG00000064842.3"; gene_type "snRNA"; gene_name "Gm26206"; level 3; mgi_id "MGI:5455983";
chr1	ENSEMBL	transcript	3172239	3172348	.	+	.	gene_id "ENSMUSG00000064842.3"; transcript_id "ENSMUST00000082908.3"; gene_type "snRNA"; gene_name "Gm26206"; transcript_type "snRNA"; transcript_name "Gm26206-201"; level 3; transcript_support_level "NA"; mgi_id "MGI:5455983"; tag "basic"; tag "Ensembl_canonical"; tag "GENCODE_Primary";
```
### 3.2 各データのダウンロード
#### 3.2.1 FASTQファイルのダウンロード
  - [PRJNA963162](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=PRJNA963162)
#### 3.2.2 リファレンスファイル（GRCm39, ReleaseM38）を [GENCODE](https://www.gencodegenes.org/mouse/) からダウンロード**
  - GTF ファイル : Comprehensive gene annotation (All) をダウンロード
  - ゲノムファイル : Transcript sequences	(ALL) をダウンロード

---

## 4. nf-core/rnaseq による定量
### 4.1 サンプルシートの作成
サンプル名と FASTQ ファイルへのパスの対応を記述し、`meta/samplesheet_rnaseq.csv`として保存する。
```csv
sample,fastq_1,strandedness
stress1,/Users/ユーザ名/workshop/fastq/SRR24350711.fastq.gz,auto
stress2,/Users/ユーザ名/workshop/fastq/SRR24350712.fastq.gz,auto
stress3,/Users/ユーザ名/workshop/fastq/SRR24350713.fastq.gz,auto
stress4,/Users/ユーザ名/workshop/fastq/SRR24350714.fastq.gz,auto
stress5,/Users/ユーザ名/workshop/fastq/SRR24350715.fastq.gz,auto
control1,/Users/ユーザ名/workshop/fastq/SRR24350716.fastq.gz,auto
control2,/Users/ユーザ名/workshop/fastq/SRR24350717.fastq.gz,auto
control3,/Users/ユーザ名/workshop/fastq/SRR24350718.fastq.gz,auto
control4,/Users/ユーザ名/workshop/fastq/SRR24350719.fastq.gz,auto
control5,/Users/ユーザ名/workshop/fastq/SRR24350720.fastq.gz,auto
```
### 4.2 キャップ指定ファイルの作成
今回は廉価なマシンを使用しているため、メモリが足りなくなりエラーが出ることがあります。`cap.nf`というファイルを作成し Nextflow に読み込ませることで、メモリを使いすぎないようにします。
```nf
cat > cap.nf <<'NF'
process {
  withLabel: process_low { cpus = 1; memory = 3.GB }
  withLabel: process_medium { cpus = 2; memory = 6.GB }
  withLabel: process_high { cpus = 4; memory = 8.GB }
}
NF
```
### 4.3 salmon による定量
salmon によって遺伝子産物の定量を行います。
```
nextflow run nf-core/rnaseq \
  -r 3.21.0 \
  -profile docker \
  -c cap.nf \
  --input meta/samplesheet_rnaseq.csv \
  --transcript_fasta ref/gencode.vM38.transcripts.fa.gz \
  --gtf ref/gencode.vM38.chr_patch_hapl_scaff.annotation.gtf.gz \
  --gencode \
  --skip_trimming \
  --skip_alignment \
  --pseudo_aligner salmon \
  --outdir results
```
---

## 6. サンプルシート

### 6.1 `nf-core/rnaseq` 用（4 列固定）

`meta/samplesheet_rnaseq.csv`
```csv
sample,fastq_1,fastq_2,strandedness
S1,fastq/SRRXXXX_1.fastq.gz,fastq/SRRXXXX_2.fastq.gz,auto
S2,fastq/SRR...._1.fastq.gz,fastq/SRR...._2.fastq.gz,auto
...
S10,fastq/SRR...._1.fastq.gz,fastq/SRR...._2.fastq.gz,auto
```

### 6.2 `nf-core/differentialabundance` 用（観測表 + コントラスト）

`meta/samplesheet_da.csv`
```csv
sample,condition,replicate
S1,control,1
S2,control,2
S3,control,3
S4,control,4
S5,control,5
S6,stress,1
S7,stress,2
S8,stress,3
S9,stress,4
S10,stress,5
```

`meta/contrasts.csv`
```csv
id,variable,reference,target,blocking
stress_vs_control,condition,control,stress,
```

## 7. 定量：`nf-core/rnaseq`（Salmon, 省リソース設定）

```bash
cd ~/proj_stress_colitis

nextflow run nf-core/rnaseq -r 3.21.0 \
  -profile conda \
  --input meta/samplesheet_rnaseq.csv \
  --fasta ref/Mus_musculus.GRCm39.cdna.fa.gz \
  --gtf   ref/gencode.vM36.annotation.gtf.gz \
  --aligner salmon \
  --max_cpus 6 \
  --max_memory '12.GB' \
  --save_reference \
  --outdir results/rnaseq
```

## 8. 差次的発現：`nf-core/differentialabundance`

Salmon の gene-level **カウント行列**と**トランスクリプト長行列**を併用して、長さバイアスを補正しつつ DE を実行します。

```bash
nextflow run nf-core/differentialabundance -r 1.5.0 \
  -profile conda \
  --study_name GSE229320_stress_colon \
  --study_type rnaseq \
  --matrix results/rnaseq/**/salmon.merged.gene_counts.tsv \
  --transcript_length_matrix results/rnaseq/**/salmon.merged.gene_lengths.tsv \
  --input  meta/samplesheet_da.csv \
  --contrasts meta/contrasts.csv \
  --gtf ref/gencode.vM36.annotation.gtf.gz \
  --features_gtf_feature_type gene \
  --max_cpus 6 \
  --max_memory '12.GB' \
  --outdir results/da
