## Approach one - assembly of uncorrected reads

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

This yielded assemblies of 79 Mbp - 184 Mbp, which I then used to manually set the genome size for `metaFlye` as suggested [here](https://github.com/fenderglass/Flye/blob/flye/docs/FAQ.md). These values were also used for `Flye`.

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
         --out-dir ${SAMPLES[$i]}_flye/
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

Next up - [Assembly (part 2)](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/3_error_correction.md).
