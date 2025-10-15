# BINDS発現・機能解析インシリコ解析融合ユニット講習会

## 1. 概要
### 1.1 対象のデータ
- [Schneider, Kai Markus et al. Cell, Volume 186, Issue 13, 2823-2838.e20](https://doi.org/10.1016/j.cell.2023.05.001)
- 心理的ストレスを与えるために 7 日間拘束したマウスの結腸組織におけるバルク RNA-seq データです。control 群と stress 群がそれぞれ n=5 ずつ存在します。
### 1.2 講習内容
- 環境構築
- リファレンスファイルのダウンロード
- nf-core/rnaseq による定量
- nf-core/differentialabundance による発現変動解析
### 1.3 検証環境
- MacBook Air (M4, 13-inch, 2024)
  - CPU : 10コア
  - メモリ : 16GB
  - ストレージ : 256GB
  - OS : macOS Tahoe 26.0.1
- `/Users/workshop`で解析を行います。

---

## 2. 環境構築
### 2.1 Docker のインストール
[公式サイト](https://www.docker.com)から公式ドキュメントに従って Docker Desktop をインストールします。
### 2.2 Homebrew のインストール
Homebrew は macOS 用のパッケージマネージャです。
[公式サイト](https://brew.sh)に記述されているコマンドを実行後、パスを通します。
  ```zsh
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/ユーザ名/.zprofile
  eval "$(/opt/homebrew/bin/brew shellenv)"
  ```
### 2.3 Nextflow のインストール
Homebrew を使って Java と Nextflow をインストールします。
  ```zsh
  brew install openjdk@11
  brew install nextflow
  ```
---

## 3. リファレンスファイルのダウンロード
### 3.1 各データについて
#### 3.1.1 FASTQ ファイル
シーケンサーから出力される塩基配列とそのクオリティスコアが記述されたファイルです。
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
#### 3.1.2 FASTA ファイル
対象生物の転写産物が記述されたファイルです。
```
>ENSMUST00000193812.2|ENSMUSG00000102693.2|OTTMUSG00000049935.1|OTTMUST00000127109.1|4933401J01Rik-201|4933401J01Rik|1070|TEC|
AAGGAAAGAGGATAACACTTGAAATGTAAATAAAGAAAATACCTAATAAAAATAAATAAA
AACATGCTTTCAAAGGAAATAAAAAGTTGGATTCAAAAATTTAACTTTTGCTCATTTGGT
ATAATCAAGGAAAAGACCTTTGCATATAAAATATATTTTGAATAAAATTCAGTGGAAGAA
TGGAATAGAAATATAAGTTTAATGCTAAGTATAAGTACCAGTAAAAGAATAATAAAAAGA
AATATAAGTTGGGTATACAGTTATTTGCCAGCACAAAGCCTTGGGTATGGTTCTTAGCAC
```
#### 3.1.3 GTF ファイル
遺伝子アノテーション情報が記述されたファイルです。
```
chr1	HAVANA	gene	3143476	3144545	.	+	.	gene_id "ENSMUSG00000102693.2"; gene_type "TEC"; gene_name "4933401J01Rik"; level 2; mgi_id "MGI:1918292"; havana_gene "OTTMUSG00000049935.1";
chr1	HAVANA	transcript	3143476	3144545	.	+	.	gene_id "ENSMUSG00000102693.2"; transcript_id "ENSMUST00000193812.2"; gene_type "TEC"; gene_name "4933401J01Rik"; transcript_type "TEC"; transcript_name "4933401J01Rik-201"; level 2; transcript_support_level "NA"; mgi_id "MGI:1918292"; tag "basic"; tag "Ensembl_canonical"; tag "GENCODE_Primary"; havana_gene "OTTMUSG00000049935.1"; havana_transcript "OTTMUST00000127109.1";
chr1	HAVANA	exon	3143476	3144545	.	+	.	gene_id "ENSMUSG00000102693.2"; transcript_id "ENSMUST00000193812.2"; gene_type "TEC"; gene_name "4933401J01Rik"; transcript_type "TEC"; transcript_name "4933401J01Rik-201"; exon_number 1; exon_id "ENSMUSE00001343744.2"; level 2; transcript_support_level "NA"; mgi_id "MGI:1918292"; tag "basic"; tag "Ensembl_canonical"; tag "GENCODE_Primary"; havana_gene "OTTMUSG00000049935.1"; havana_transcript "OTTMUST00000127109.1";
chr1	ENSEMBL	gene	3172239	3172348	.	+	.	gene_id "ENSMUSG00000064842.3"; gene_type "snRNA"; gene_name "Gm26206"; level 3; mgi_id "MGI:5455983";
chr1	ENSEMBL	transcript	3172239	3172348	.	+	.	gene_id "ENSMUSG00000064842.3"; transcript_id "ENSMUST00000082908.3"; gene_type "snRNA"; gene_name "Gm26206"; transcript_type "snRNA"; transcript_name "Gm26206-201"; level 3; transcript_support_level "NA"; mgi_id "MGI:5455983"; tag "basic"; tag "Ensembl_canonical"; tag "GENCODE_Primary";
```
### 3.2 各データのダウンロード
#### 3.2.1 FASTQ ファイルのダウンロード
  - [PRJNA963162](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=PRJNA963162)から`/Users/worksho/fastq`に FASTQ ファイルをダウンロードする。
#### 3.2.2 リファレンスファイル（GRCm39, ReleaseM38）を [GENCODE](https://www.gencodegenes.org/mouse/) からダウンロード
  - GTF ファイル : Comprehensive gene annotation (All) を`/Users/workshop/ref`にダウンロードします。
  - FASTA ファイル : Transcript sequences	(ALL) を`/Users/workshop/ref`にダウンロードします。

---

## 4. nf-core/rnaseq による定量
### 4.1 サンプルシートの作成
サンプル名と FASTQ ファイルへのパスの対応を記述し、`meta/samplesheet_rnaseq.csv`として保存します。
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
  withLabel: process_low { cpus = 2; memory = 4.GB; maxForks = 3 }
  withLabel: process_medium { cpus = 4; memory = 6.GB; maxForks = 2 }
  withLabel: process_high { cpus = 8; memory = 12.GB; maxForks = 1 }
}
NF
```

メモリが 8 GB 以下の場合は下記のファイルを作成します。
```nf
cat > cap.nf <<'NF'
process {
  withLabel: process_low { cpus = 1; memory = 3.GB; maxForks = 2 }
  withLabel: process_medium { cpus = 2; memory = 3.GB; maxForks = 2 }
  withLabel: process_high { cpus = 7; memory = 6.GB; maxForks = 1 }
}
NF
```

### 4.3 Salmon による定量
Salmon によって遺伝子産物の定量を行います。
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

## 5. nf-core/differentialabundance による発現変動解析
### 5.1 サンプルシートの作成
サンプル名と条件の対応を記述し、`meta/samplesheet_da.csv`として保存します。
```
sample,condition
control1,control
control2,control
control3,control
control4,control
control5,control
stress1,stress
stress2,stress
stress3,stress
stress4,stress
stress5,stress
```
### 5.2 コントラストファイルの作成
比較のレイアウトを記述し、`meta/contrasts.csv`として保存します。
```
id,variable,reference,target,blocking
stress_vs_control,condition,control,stress
```

### 5.3 nf-core/differentialabundance による発現変動解析
```
nextflow run nf-core/differentialabundance \
     -r 1.5.0 \
     -profile docker \
     --input meta/samplesheet_da.csv \
     --contrasts meta/contrasts.csv \
     --matrix results/salmon/salmon.merged.gene_counts.tsv \
     --length results/salmon/salmon.merged.gene_lengths.tsv \
     --gtf ref/gencode.vM38.chr_patch_hapl_scaff.annotation.gtf.gz \
     --outdir DEG
```

---
