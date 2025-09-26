# Stress-induced Colitis (Mouse, Bulk RNA-seq) — 再現デモ手順（MacBook Air）

## 1. 概要
- **実験設計**：心理ストレス（7日間拘束）**Stress** vs **Control**、各 n=5、結腸全組織のバルクRNA-seq。  
- **解析設計**：  
  - 生データ（FASTQ）を **nf-core/rnaseq**（`--aligner salmon`）で定量（軽量・高速）  
  - 集約された gene-level 行列を **nf-core/differentialabundance** へ入力し、**control vs stress** の 2群 DE を実施  
- **再現性**：パイプラインのバージョンを `-r` で固定（`rnaseq 3.21.0`、`differentialabundance 1.5.0`）。

---

## 2. 必要環境（MacBook Air 向け）
- macOS（Apple Silicon, 10-core CPU / 8-core GPU / 16 GB unified memory）
- Homebrew（Nextflow の導入に使用）
- Miniforge / Mambaforge 等の **conda**（`-profile conda` 用）
- ディスク空き **50–80 GB** 目安（FASTQ・参照・結果を含む）
- JVM メモリ制限（推奨）
  ```bash
  echo "export NXF_OPTS='-Xms1g -Xmx4g'" >> ~/.zshrc
  source ~/.zshrc

---

## 3. プロジェクト準備

```bash
mkdir -p ~/proj_stress_colitis/{fastq,ref,meta,results}
brew install nextflow         # Homebrew 経由で Nextflow を導入
# conda は Miniforge/Mambaforge を GUI インストーラ等で導入しておく

---

## 4. データのダウンロード（ブラウザのみ）

- **GEO Series（設計とサンプル一覧）**  
  https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE229320
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
