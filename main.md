# BINDS発現・機能解析インシリコ解析融合ユニット講習会

## 1. 概要
- **対象のデータ**：
  -  [Schneider, Kai Markus et al. Cell, Volume 186, Issue 13, 2823-2838.e20](https://doi.org/10.1016/j.cell.2023.05.001)
  -  心理的ストレスを与えるため7日間拘束したマウスにおける結腸全組織のバルク RNA-seq データ、Control 群と Stress 群それぞれ各 n=5
- **解析設計**：
  - 生データ（FASTQ）を nf-core/rnaseq で定量
  - 定量結果を nf-core/differentialabundance へ入力し、control vs stress の2群発現変動解析を実施
- **検証環境**：MacBook Air (M4, 13-inch, 2024)
  - CPU : 10コア
  - メモリ : 16GB
  - ストレージ 256GB

---

## 2. 環境構築
- **Docker のインストール**
  - [公式サイト](https://www.docker.com)から Docker Desktop をインストール
- **Homebrew のインストール**
  ```zsh
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/ユーザ名/.zprofile
  eval "$(/opt/homebrew/bin/brew shellenv)”
  ```
- **Nextflow のインストール**
  ```zsh
  brew install openjdk@11
  brew install nextflow
  ```
---

## 3. リファレンスファイルとFASTQファイルのダウンロード
- **FASTQファイルのダウンロード**
  - [PRJNA963162](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=PRJNA963162)
- **SRA Run Selector（FASTQ 直リンク一覧）**  
  https://www.ncbi.nlm.nih.gov/Traces/study/?acc=PRJNA963162  
  - テーブルの **FASTQ files（FTP/HTTPS）** 列から各 SRR の `.fastq.gz` を保存し、`~/proj_stress_colitis/fastq/` に配置します。
- **Processed（時間短縮の保険：行列一括）**  
  https://www.ncbi.nlm.nih.gov/geo/download/?acc=GSE229320&format=file  
  - `GSE229320_RAW.tar` に処理済み行列が含まれます（デモ時間が限られる場合のバックアップ）。

---

## 5. 参照データ（GRCm39, 同一リリースで揃える）

- **GENCODE（例：release M36, GRCm39）**  
  https://www.gencodegenes.org/mouse/release_M36.html  
  - 取得ファイル：  
    - GTF（Comprehensive annotation）  
    - Transcript FASTA（cDNA sequences）  
  - 保存例：`ref/gencode.vM36.annotation.gtf.gz`、`ref/Mus_musculus.GRCm39.cdna.fa.gz`

> `--aligner salmon` を用いるため、**transcriptome FASTA + GTF** で十分です（ゲノム全体の STAR インデックスは不要）。

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
