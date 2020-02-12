### Read mapping

In order to bin MAGs, we need to generate coverage profiles to augment the binning process. Given our data set, we have two ways we could do this:

1. Map the ONT reads to the assemblies, for a single coverage value per contig
1. Map out Illumina reads from the triplicate samples, in order to provide differential coverage profiles

We will do both...

----

#### Preprocessing

For the following command examples, I have taken the assemblies from each tool and renamed them into the form:

```bash
[Correction]_[Sample]_[Assembly tool].fna
```

So that they are easier to handle in `bash` loops. Before mapping, I will use `seqmagick` to remove some short contigs (defined here as under 5 kbp) in order to remove some shrapnel from the binning process.

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        seqmagick convert --min-length 5000 NoCorr_${i}_${asm}.fna NoCorr_${i}_${asm}.m5000.fna
        seqmagick convert --min-length 5000 FMLRC_${i}_${asm}.fna FMLRC_${i}_${asm}.m5000.fna
    done
done
```

This is not a standard part of most workflows, but I have found that using a tool like `EukRep` to split apart the prokaryotic and eukaryotic reads before binning. In my experience (admittedly, anecdotal) this yields a greater number bins, and a higher average quality of bin than binning with all the reads together.

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        EukRep --tie prok -i NoCorr_${i}_${asm}.m5000.fna \
                          -o NoCorr_${i}_${asm}.m5000.euk.fna \
                          --prokarya NoCorr_${i}_${asm}.m5000.prok.fna

        EukRep --tie prok -i FMLRC_${i}_${asm}.m5000.fna \
                          -o FMLRC_${i}_${asm}.m5000.euk.fna \
                          --prokarya FMLRC_${i}_${asm}.m5000.prok.fna
    done
done
```

----

#### ONT mapping

A recent manuscript by [Sevin *et al.* (2019)](https://doi.org/10.6084/m9.figshare.10260740) used `bwa` with the `-x ont2d` parameter for mapping, which is what I will use here. Apply the steps:

1. Index with `bwa` v0.7.17
1. Map with `bwa mem`
1. Sort and compress the *sam* file with `samtools` v1.8

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        # Index, map, and sort for uncorrected reads
        bwa index -p NoCorr_${i}_${asm}.m5000 NoCorr_${i}_${asm}.m5000.prok.fna
        bwa mem -t 30 -x ont2d NoCorr_${i}_${asm}.m5000 ${i}.nanofilt.fastq.gz > NoCorr_${i}_${asm}.m5000.sam
        samtools sort -@ 30 -o NoCorr_${i}_${asm}.m5000.bam NoCorr_${i}_${asm}.m5000.sam
                 
        # Index, map, and sort for FMLRC corrected reads
        bwa index -p FMLRC_${i}_${asm}.m5000 FMLRC_${i}_${asm}.m5000.prok.fna
        bwa mem -t 30 -x ont2d FMLRC_${i}_${asm}.m5000 ${i}.nanofilt.fastq.gz > FMLRC_${i}_${asm}.m5000.sam
        samtools sort -@ 30 -o FMLRC_${i}_${asm}.m5000.bam FMLRC_${i}_${asm}.m5000.sam    
    done
done
```

----

#### Illumina mapping

This is the same workflow as above, but using `bowtie2` to map the short reads from each replicate to the assembly.

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        # Uncorrected reads...
        bowtie2-build --threads 30 NoCorr_${i}_${asm}.m5000.prok.fna NoCorr_${i}_${asm}.m5000

        SAMPLE=$(echo ${i} | cut --characters=1-2)
        for REP in S1 S2 S3;
        do
            bowtie2 --sensitive --threads 30 --minins 0 --maxins 900 -x NoCorr_${i}_${asm}.m5000 \
                    -1 ${SAMPLE}${REP}_R1.fastq.gz -2 ${SAMPLE}${REP}_R2.fastq.gz \
                    -S NoCorr_${SAMPLE}${REP}_${asm}.sam
            samtools sort -@ 30 -o NoCorr_${SAMPLE}${REP}_${asm}.bam NoCorr_${SAMPLE}${REP}_${asm}.sam
        done

        # FMLRC reads...
        bowtie2-build --threads 30 FMLRC_${i}_${asm}.m5000.prok.fna FMLRC_${i}_${asm}.m5000

        SAMPLE=$(echo ${i} | cut --characters=1-2)
        for REP in S1 S2 S3;
        do
            bowtie2 --sensitive --threads 30 --minins 0 --maxins 900 -x FMLRC_${i}_${asm}.m5000 \
                    -1 ${SAMPLE}${REP}_R1.fastq.gz -2 ${SAMPLE}${REP}_R2.fastq.gz \
                    -S FMLRC_${SAMPLE}${REP}_${asm}.sam
            samtools sort -@ 30 -o FMLRC_${SAMPLE}${REP}_${asm}.bam FMLRC_${SAMPLE}${REP}_${asm}.sam
        done
    done
done
```

----

Next up - [Binning](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/5_binning.md)
