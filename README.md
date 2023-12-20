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
7) Sequence quality control and feature table construction using DADA2.
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
8) Visualize dada2 stats.
```
qiime metadata tabulate \
  --m-input-file stats-dada2.qza \
  --o-visualization stats-dada2.qzv
```
9) Rename files for easy to use commands.
```
mv rep-seqs-dada2.qza rep-seqs.qza
mv table-dada2.qza table.qza
```
10) Summarize feature table.
```
qiime feature-table summarize \
  --i-table table.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file sample-metadata.tsv
```
11) Visualize representative sequences.
```
qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \
  --o-visualization rep-seqs.qzv
```
12) Make tree files for phylogenetic analysis.
```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
```

13) Extract reference reads for taxonomic classification.
```
qiime feature-classifier extract-reads \
  --i-sequences silva-138-99-seqs.qza \
  --p-f-primer CCTACGGGNGGCWGCAG \
  --p-r-primer GACTACNVGGGTMTCTAATCC \
  --p-min-length 100 \
  --p-max-length 600 \
  --o-reads ref-seqs-341f-806r.qza
```
14) Build taxonomic classifier.
```
qiime feature-classifier fit-classifier-naive-bayes \
  --i-reference-reads ref-seqs-341f-806r.qza \ 
  --i-reference-taxonomy silva-138-99-tax.qza \ 
  --o-classifier classifier-silva-341f-806r.qza
```
15) Taxonomic assignments to representative sequences.
```
qiime feature-classifier classify-sklearn \
  --i-classifier classifier-silva-341f-806r.qza \      
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza
```
16) Visualize taxonomy.
```
qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
```
17) Make bar plots.
```
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
```
18) Collapse taxonomy at genus level.
```
qiime taxa collapse \
  --i-table table.qza \ 
  --i-taxonomy taxonomy.qza \
  --p-level 6 \
  --o-collapsed-table table-level-6.qza
```
19) Export qza file.
```
qiime tools export --input-path table-level-6.qza --output-path exported-Table-level-6

cd exported-Table-level-6 
```
20) Conver biom file to text file.
```
biom convert -i feature-table.biom -o feature-table.txt --to-tsv
```

21) Delete top two cantaminating plastid DNA. And save as another name.

22) Convert .txt file back to biom file.
```
biom convert -i feature-table-filtered.txt -o feature-table-filtered.biom --to-hdf5 --table-type="OTU table"

biom normalize-table -r -i feature-table-filtered.biom -o rel-abun-feature-table-filtered.biom

biom convert -i rel-abun-feature-table-filtered.biom -o rel-abun-feature-table-filtered.txt --to-tsv
```


<img width="438" alt="Screenshot 2023-12-20 at 4 47 20 PM" src="https://github.com/vetrabindra01/York-College-Grain-Mcirobiome-Workshop/assets/97687143/b86e82e8-5dcc-49d2-83d0-6039225bff4d">


