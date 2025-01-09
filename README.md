# buildSynReadData

This software package provides tools to build synthetic read sequences to test the AAVengeR integration site pipeline and evaluate its performance. 
Synthetic reads are created with the buildSynReadData.py script:

```
usage: buildSynReadData.py [-h] [-o] [-m] [-n] [-f] [-p] [-d] [-s] [-r] [-g] [-i] [-t] [-x] [-y] [-z] [-e] [-c]

options:
  -h, --help                  Show this help message and exit
  -o, --outputDir            Output directory path. Default (./output).
  -m, --mode                 Mode (integrase or AAV). Default (integrase).
  -n, --nSites               Number of sites to generate. Default (100).
  -f, --nFrags               Number of fragments for each site (limit 100). Default (5).
  -p, --nReadsPerFrag        Number of reads per fragment. Default (25).
  -d, --distBetweenFragEnds  Distance (NT) between fragment break points. Default (25).
  -s, --seed                 Random seed. Default (1).
  -r, --refGenomePath        Path to 2bit reference genome. Default (~/AAVengeR/data/referenceGenomes/blat/hg38.2bit).
  -g, --refGenomeID          Reference genome ID for sample data table. Default (hg38).
  -i, --integraseHMM         Name of AAVengeR HMM to use (integrase mode only). Default (validation.hmm).
  -t, --anchorReadStartSeq   Anchor read start sequence filter (AAV mode only). Default (TCTGCGCGCT).
  -x, --R1_length            Total length of R1 reads. Default (150).
  -y, --R2_length            Total length of R2 reads. Default (150).
  -z, --I1_length            Total length of I1 reads. Default (12).
  -e, --percentGenomicError  Percent gDNA error (0.0 - 1.0) to simulate in R1 and R2 reads. Default (0).
  -c, --positionChatterSD    Standard deviation of Gaussian centered on expected fragment ends used to simulate position chatter. (Default 0.50).
```
<br>
The script can simulate reads for both retroviral vectors (mode: integrase) and AAV vectors (mode: AAV)<br>
<br> 

```
buildSynReadData.py --output integrase_dataSet --mode integrase --R1_length 150 --R2_length 150 --nSites 500 --seed 1
buildSynReadData.py --output AAV_dataSet --mode AAV --R1_length 200  --R2_length 300 --nSites 500 --seed 1
```
<br>
All synthetic reads contain bases annotated as Q30 (ASCII code '?') and include the targeted integration site and simulated fragment number in their read ids: <br>
<br>

```
%> zcat R1.fastq.gz | more
@chr14+35327409_read1_frag1
GAACGAGCACTAGTAAGCCCTTTATTTCCCGACTCCGCTTAAGGGACTGTTCAAGCGATTCTCCTGCCTCAGCCTCCCGAGTAGCTGGGATTACAGGCATGCGCCACCGTGCCCGGCTAATTTTGTATTTTTAGTAGAGATGGGGTTTCT
+
??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
@chr14+35327409_read2_frag1
GAACGAGCACTAGTAAGCCCTTTATTTCCCGACTCCGCTTAAGGGACTGTTCAAGCGATTCTCCTGCCTCAGCCTCCCGAGTAGCTGGGATTACAGGCATGCGCCACCGTGCCCGGCTAATTTTGTATTTTTAGTAGAGATGGGGTTTCT
+
??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
@chr14+35327409_read3_frag1
GAACGAGCACTAGTAAGCCCTTTATTTCCCGACTCCGCTTAAGGGACTTTCAAGCGATTCTCCTGCCTCAGCCTCCCGAGTAGCTGGGATTACAGGCATGCGCCACCGTGCCCGGCTAATTTTGTATTTTTAGTAGAGATGGGGTTTCTC
+
??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
```

<br>
In addition to the simulated reads, a truth file is provided detailing the targeted sites and expected number of recovered reads and fragments associated with each site. 
Simulated sites are spread across three subjects each with three samples.<br>
<br>

```
%> head truth.tsv
trial   subject    sample   posid           nReads  nFrags  nUMIs   leaderSeq
test    subjectA   sample1  chr14-35327409  125     5       5       TCTGCGCGCTCGCTCGCTCA
test    subjectA   sample1  chr13-20014542  125     5       5       TCTGCGCGCTCGCTCGCTCA
test    subjectA   sample1  chr12-51988724  125     5       5       TCTGCGCGCTCGCTCGCTCA
test    subjectA   sample1  chr3-176464418  125     5       5       TCTGCGCGCTCGCTCGCTCA
test    subjectA   sample1  chr18-22625755  125     5       5       TCTGCGCGCTCGCTCGCTCA
test    subjectA   sample1  chr15+78237251  125     5       5       TCTGCGCGCTCGCTCGCTCA
(...)
```
<br>
After AAVengeR has processed the synthetic data, the evalSynDataResult.R can be used to evaulate AAVengeR's performance: <br>
<br>

```
./evalSynDataResult.R --sitesFile    integrase_dataSet/output/core/sites.rds
                      --multiHitFile integrase_dataSet/output/core/multiHitClusters.rds
                      --truthFile    integrase_dataSet/truth.tsv
                      --outputDir    integrase_evaluation
```
<br>
The evaluation script returns two tables. The first table is a summary of AAVengeR's overall recovery of simualted sites. 
The last two columns, leaderSeqDistMean and leaderSeqDistSD, report the average edit distance between expected and recovered
leader sequences and the standard devitation of these distances.<br>
<br>

```
nSitesExpected  nSitesRecovered percentUniqueRecovery   percentTotalRecovery    leaderSeqDistMean    leaderSeqDistSD
500             467             93.4%                   97.8%                   0.00                 0.00
```
<br>
The second table provides a site by site reporting where the last four columns report differences between expected values and results.<br>
<br>

```
trial   subject         sample  posid           nReads  nFrags  nUMIs   leaderSeq               posDiff   readDiff  fragDiff  leaderSeqDist
test    subjectA        sample1 chr14-35327409  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr13-20014542  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr12-51988724  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr3-176464418  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr18-22625755  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr15+78237251  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr12-130407244 125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr11-13727336  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr9-10789850   125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr4-29219427   125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr18+17354356  125     5       5       TCTGCGCGCTCGCTCGCTCA    NA        NA        NA        NA
test    subjectA        sample1 chr9+92455387   125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr9-95123552   125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr14+95622356  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr19+34928047  125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr3-8782050    125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
test    subjectA        sample1 chr11-5118425   125     5       5       TCTGCGCGCTCGCTCGCTCA    0         0         0         0
(...)
```
