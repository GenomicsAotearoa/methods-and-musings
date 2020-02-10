## Metagenomic assembly and binning with Oxford Nanopore Technology

At the time of this project, there is not a lot of literature on best practices for assembling and recovering metagenome-assembled genomes ([MAGs](https://doi.org/10.1038/nbt.3893)) from long-read sequence data. This is a (non-exhaustive) walkthrough of some of the testing I have done to attempt to recover MAGs from a set of brackish/marine sediment samples, taken from our Waiwera study site.

----

### Background reading

Based on my reading of the literature at the time, there are very few published manuscripts performing this kind of analysis. On reading, most of the studies I was able to find were either performed by mapping long metagenomic reads to a reference genome, or were performed on mock communities:

1. [Charalampous *et al.* (2019)](https://www.nature.com/articles/s41587-019-0156-5)) - MinION
1. [Cuscó *et al.* (2019)](https://www.biorxiv.org/content/10.1101/585067v1.full) [PREPRINT] - MinION
1. [Kafetzopoulou et al., 2019](https://science.sciencemag.org/content/363/6422/74.full) - MinION, removed of host DNA then assembled the remainder.
1. [Pearman *et al.* (2019)](https://www.biorxiv.org/content/10.1101/363622v2) [PREPRINT] - MinION
1. [Nicholls *et al.* (2019)](https://academic.oup.com/gigascience/article/8/5/giz043/5486468) - GridION and PromethION, analysis of mock microbial metagenomes

Manuscripts that performed a more expected metagenomic binning protocol:

1. [Beaulaurier *et al.* (2019)](https://www.biorxiv.org/content/10.1101/619684v1.full)) - GridION, aiming to recover viral genomes from microbial communities
1. [Sommerville *et al.* (2019)](https://bmcmicrobiol.biomedcentral.com/articles/10.1186/s12866-019-1500-0) - MinION and PacBio
1 [Holm *et al.* (2019)](https://www.biorxiv.org/content/biorxiv/early/2019/06/03/657197.full.pdf) [PREPRINT] - PacBio Sequel II
1. [Song *et al.* (2019)](https://doi.org/10.1016/j.margen.2019.05.002) - PacBio RS II

However, across all of these there was not a lot of consistency in methods, or re-deploying of methods to a metagenomic pipeline.

----

### Experimental information - the study site and available sequencing data

To begin with, we already had a data set of nine sites taken from an estuary. As part of our ongoing study, we had obtained triplicate metagenomes from the sediment at each site, sequenced on the Illumina HiSeq. On analysis these data, we identified four sites that were rich in sequences belonging to the phylum Thaumarchaeota, which is important in the research questions we are asking of our data.

From three of these sites, we were able to extract sufficient high molecular weight DNA for sequencing on the Oxford PromethION platform. From each of these samples, we obtained between 1.2 and 2.1 million sequences, which were quality filtered with `porechop` and trimmed with `nanofilt`. The basic statistics for our samples were:

|Sample|Raw sequences|Post-`porechop`|Post-`nanofilt`|
|:---|:---:|:---:|:---:|
|S4R3|2,088,722|2,087,200|1,954,049|
|S5R2|1,577,002|1,574,819|1,407,863|
|S6R1|1,236,066|1,234,600|1,065,611|

----

### Approach one - assembly of uncorrected reads

To begin, I performed assembly with a number of long-read assembly tools. Because of the way the sample/analysis permutations balloon out later in the analysis I did not spend any time optimising the parameters, with the exception of `metaFlye` which requires (or strongly suggests) setting a genome size for the assembly.

Actual assembly was performed on [NeSI](https://www.nesi.org.nz/) using the [slurm](https://slurm.schedmd.com/documentation.html) job management system, but some exemplars of how I ran the assemblies are below:

#### metaFlye v2.4.2

```bash
for i in S4R3 S5R2 S6R1;
do
    flye --threads 20 --meta --genome-size 100000000 \
         --nano-raw ${i}.nanofilt.fastq.gz \
         --out-dir ${i}_metaflye/
done
```

This yielded assemblies of 79 Mbp - 184 Mbp, which I then used to manually set the genome size for `metaFlye` as suggested [here](https://github.com/fenderglass/Flye/blob/flye/docs/FAQ.md). These values were also used for `Flte`.

#### metaFlye v2.4.2

```bash
SAMPLES=(S4R3 S5R2 S6R1)
SIZES=(150115155 184224069 79837143)

for i in {1..3};
do
    flye --threads 20 --meta --genome-size ${SIZES[$i]} \
         --nano-raw ${SAMPLES[$i]}.nanofilt.fastq.gz \
         --out-dir ${SAMPLES[$i]}_metaflye_retuned/
done
```

#### Flye v2.4.2

```bash
for i in {1..3};
do
    flye --threads 20 --genome-size ${SIZES[$i]} \
         --nano-raw ${SAMPLES[$i]}.nanofilt.fastq.gz \
         --out-dir ${SAMPLES[$i]}_metaflye_retuned/
done
```

#### MiniMap2 / MiniASM v0.3-r179

```bash
for i in S4R3 S5R2 S6R1;
do
    gunzip -c ${i}.nanofilt.fastq.gz > ${i}.nanofilt.fq
    minimap2 -x ava-ont -t 20 ${i}.nanofilt.fq ${i}.nanofilt.fq | gzip -1 > ${i}.nanofilt.fq_reads.paf.gz

    miniasm -f ${i}.nanofilt.fq ${i}_reads.paf.gz > ${i}_asm.gfa
done
```

#### Canu v1.8

The settings for this run were taken from the Canu [documentation](https://readthedocs.org/projects/canu/downloads/pdf/latest/). These conflict a bit with [this blog post](https://github.com/marbl/canu/issues/634), so I favoured the manual.

The median genome sizes were just calculated from our Illumina-generated MAGs.

```bash
SAMPLES=(S4R3 S5R2 S6R1)
SIZES=(3.4 3.5 3.3)

for i in {1..3};
do
    canu corMinCoverage=0 corOutCoverage=10000 corMhapSensitivity=high correctedErrorRate=0.16 \
         maxThreads=20 redMemory=32 oeaMemory=32 batMemory=90 useGrid=false \
         genomeSize=${SIZES[$i]}m \
         -nanopore-raw ${SAMPLES[$i]}.nanofilt.fastq.gz \
         -p ${SAMPLES[$i]} \
         -d ${SAMPLES[$i]}_canu/
done
```

#### Shashta v0.1.0

This tool does not support multi-line fasta files. I had to convert the reads using `seqmagick`.

```bash
for i in S4R3 S5R2 S6R1;
do
    seqmagick convert --line-wrap 0 ${i}.nanofilt.fastq.gz ${i}.shasta.fna

    shasta \
        --MarkerGraph.minCoverage 1 --MarkerGraph.maxCoverage 300 --MarkerGraph.highCoverageThreshold 300 \
        --input ${i}.shasta.fna \
        --output ${i}_shasta/
done
```

----

### Approach two - error correction and subsequent assembly

----

### Read mapping - two strategies

----

### Evaluation of MAGs

----
