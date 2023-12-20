# Grain Microbiome Workshop

# Introduction
‘Sourdough’ breads, made using flours milled from grains including wheat and rye, are leavened with a natural starter. The starter microbiome originates from microorganisms found in grain and flour. Its composition changes markedly on incubation with regular feedings. Leavening results from yeast fermentation of the low concentration of simple sugars found in flour, and souring is due to acidifying bacteria such as lactobacilli. 
	To initiate a long-term project, we analyzed rye and wheat berry microbiomes by amplification and sequencing of colonies obtained from plate cultures. Genera were identified using a 16S rRNA primer pair for bacteria and an Internal Transcribed Spacer (ITS) pair for fungi. Remarkably, dominant fungal and  bacterial genera characteristic of a mature starter were not observed among genera identified in the grains.
 
<img width="3164" alt="image" src="https://github.com/vetrabindra01/York-College-Grain-Mcirobiome-Workshop/assets/97687143/e549a724-0020-4d78-ba48-23940c33d588">

# Primers for 16S rRNA gene sequencing
Primers used: (V3/V4)

341f: CCTACGGGNGGCWGCAG
 
806r: GACTACNVGGGTMTCTAATCC

# commands
The mcirobiome analysis was performed using [QIIME 2 command-line interface](https://docs.qiime2.org/2023.9/interfaces/q2cli/).

1) Activate qiime2 using conda.
```
conda activate qiime2-amplicon-2023.9
```
2) Import using input format Laneless.
```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path Laneless \ 
  --input-format CasavaOneEightLanelessPerSampleDirFmt \
  --output-path demux-paried-end.qza
```
3) Trim adapters.
```
qiime cutadapt trim-paired \
  --i-demultiplexed-sequences demux-paried-end.qza \
  --p-front-f CCTACGGGNGGCWGCAG \
  --p-front-r GACTACNVGGGTMTCTAATCC \
  --o-trimmed-sequences trimmed-demux-paried-end.qza \
  --p-discard-untrimmed --p-match-read-wildcards    
```
4) copy demultiplexed file.
```
cp trimmed-demux-paried-end.qza demux.qza 
```
5) Generate a summary of demultiplexing results.
```
qiime demux summarize \
  --i-data demux.qza \
  --o-visualization demux.qzv
```
6) View demux file
```
qiime tools view demux.qzv
```
6) Sequence quality control and feature table construction using DADA2.
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left-f 0 \
  --p-trim-left-r 0 \
  --p-trunc-len-f 280 \
  --p-trunc-len-r 180 \
  --o-representative-sequences rep-seqs-dada2.qza \
  --o-table table-dada2.qza \
  --o-denoising-stats stats-dada2.qza
```


