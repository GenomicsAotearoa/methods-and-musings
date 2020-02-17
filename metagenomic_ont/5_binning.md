### Binning and dereplication

When binning metagenomes, our favoured workflow is to bin using the tools `MetaBAT`, `MaxBin`, and `CONCOCT` and then dereplicate the outputs using `DAS_Tool`. This is the approach we will apply here.

Given how bulky this code gets, I am only providing snippets for the uncorrected (`UnCorr`) data. The exact same commands were run for all of the samples corrected with `FMLRC`.

----

#### ONT mapping

#### MetaBAT v2.13

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        jgi_summarize_bam_contig_depths --outputDepth NoCorr_${i}_${asm}.metabat.txt NoCorr_${i}_${asm}.m5000.bam
        metabat2 -t 10 -s 50000 -a NoCorr_${i}_${asm}.metabat.txt \
                                -i NoCorr_${i}_${asm}.m5000.prok.fna \
                                -o NoCorr_${i}_${asm}.metabat_ont/NoCorr_${i}_${asm}.metabat_ont
    done
done
```

#### MaxBin v2.2.6

For `MaxBin`, we will just recycle the coverage table from `MetaBAT`.

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        cut -f1,4 NoCorr_${i}_${asm}.metabat.txt > NoCorr_${i}_${asm}.maxbin.txt
        mkdir NoCorr_${i}_${asm}.maxbin_ont/
        run_MaxBin.pl -thread 10 -contig NoCorr_${i}_${asm}.m5000.prok.fna \
                                 -abund NoCorr_${i}_${asm}.maxbin.txt \
                                 -out NoCorr_${i}_${asm}.maxbin_ont/NoCorr_${i}_${asm}.maxbin_ont
    done
done
```

#### CONCOCT v1.0.0

The `CONCOCT` pipeline requires us to chop up our contigs into shorter sub-contigs, which are used for clustering into bins. In order to simplify the loops run here, we'll split this step out first.

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        samtools index NoCorr_${i}_${asm}.m5000.bam
        cut_up_fasta.py -c 10000 -o 0 --merge_last -b NoCorr_${i}_${asm}.m5000.prok.10k.bed \
                        NoCorr_${i}_${asm}.m5000.prok.fna > NoCorr_${i}_${asm}.m5000.prok.10k.fna
        concoct_coverage_table.py NoCorr_${i}_${asm}.m5000.prok.10k.bed \
                                  NoCorr_${i}_${asm}.m5000.bam > \
                                  NoCorr_${i}_${asm}.concoct.txt
    done
done
```

Now perform the binning protocol...

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        concoct -t 10 --composition_file NoCorr_${i}_${asm}.m5000.prok.10k.fna \
                      --coverage_file NoCorr_${i}_${asm}.concoct.txt \
                      -b NoCorr_${i}_${asm}.concoct_ont/

        merge_cutup_clustering.py NoCorr_${i}_${asm}.concoct_ont/clustering_gt1000.csv > \    
                                  NoCorr_${i}_${asm}.concoct_ont/clustering_merged.csv
                      
        mkdir NoCorr_${i}_${asm}.concoct_ont/fasta_bins/
        extract_fasta_bins.py NoCorr_${i}_${asm}.m5000.prok.fna \
                              NoCorr_${i}_${asm}.concoct_ont/clustering_merged.csv \
                              --output_path NoCorr_${i}_${asm}.concoct_ont/fasta_bins/

        for fasta_file in `ls NoCorr_${i}_${asm}.concoct_ont/fasta_bins/`;
        do
            mv NoCorr_${i}_${asm}.concoct_ont/fasta_bins/${fasta_file} \
               NoCorr_${i}_${asm}.concoct_ont/NoCorr_${i}_${asm}.concoct_ont.${fasta_file}
        done
    done
done
```


----

#### Illumina mapping

This is very similar ot the above examples, but this timne we have three samples to provide contrast in the coverage profiles. This means that I need to use the same `cut` trick from before to differentiate the assembled sample from the replicates.

#### MetaBAT v2.13

```bash
for i in S4R3 S5R2 S6R1;
do
    SAMPLE=$(echo ${i} | cut --characters=1-2)
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        jgi_summarize_bam_contig_depths --outputDepth NoCorr_${i}_${asm}.metabat_hiseq.txt \
                                        NoCorr_${SAMPLE}R1_${asm}.m5000.bam \
                                        NoCorr_${SAMPLE}R2_${asm}.m5000.bam \
                                        NoCorr_${SAMPLE}R3_${asm}.m5000.bam
        metabat2 -t 10 -s 50000 -a NoCorr_${i}_${asm}.metabat_hiseq.txt \
                                -i NoCorr_${i}_${asm}.m5000.prok.fna \
                                -o NoCorr_${i}_${asm}.metabat_hiseq/NoCorr_${i}_${asm}.metabat_hiseq
    done
done
```

#### MaxBin v2.2.6

This time I need to `cut` four olumns from the `MetaBAT` table - the contig names, and then 1 sample for each of the sequencing replicates. These occur in columns 4, 6, and 8.

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        cut -f1,4,6,8 NoCorr_${i}_${asm}.metabat_hiseq.txt > NoCorr_${i}_${asm}.maxbin_hiseq.txt
        mkdir NoCorr_${i}_${asm}.maxbin_hiseq/
        run_MaxBin.pl -thread 10 -contig NoCorr_${i}_${asm}.m5000.prok.fna \
                                 -abund NoCorr_${i}_${asm}.maxbin_hiseq.txt \
                                 -out NoCorr_${i}_${asm}.maxbin_hiseq/NoCorr_${i}_${asm}.maxbin_hiseq
    done
done
```

#### CONCOCT v1.0.0

Step 1 - creating the coverage profiles across the 10 kbp contig fragments.

```bash
for i in S4R3 S5R2 S6R1;
do
    SAMPLE=$(echo ${i} | cut --characters=1-2)
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        samtools index NoCorr_${SAMPLE}R1_${asm}.m5000.bam
        samtools index NoCorr_${SAMPLE}R2_${asm}.m5000.bam
        samtools index NoCorr_${SAMPLE}R3_${asm}.m5000.bam

        cut_up_fasta.py -c 10000 -o 0 --merge_last -b NoCorr_${i}_${asm}.m5000.prok.10k.bed \
                        NoCorr_${i}_${asm}.m5000.prok.fna > NoCorr_${i}_${asm}.m5000.prok.10k.fna

        concoct_coverage_table.py NoCorr_${i}_${asm}.m5000.prok.10k.bed \
                                  NoCorr_${SAMPLE}R1_${asm}.m5000.bam \
                                  NoCorr_${SAMPLE}R2_${asm}.m5000.bam \
                                  NoCorr_${SAMPLE}R3_${asm}.m5000.bam > \
                                  NoCorr_${i}_${asm}.concoct_hiseq.txt
    done
done
```

Step 2 - binning and reconstruction.

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        concoct -t 10 --composition_file NoCorr_${i}_${asm}.m5000.prok.10k.fna \
                      --coverage_file NoCorr_${i}_${asm}.concoct_hiseq.txt \
                      -b NoCorr_${i}_${asm}.concoct_hiseq/

        merge_cutup_clustering.py NoCorr_${i}_${asm}.concoct_hiseq/clustering_gt1000.csv > \    
                                  NoCorr_${i}_${asm}.concoct_hiseq/clustering_merged.csv
                      
        mkdir NoCorr_${i}_${asm}.concoct_ont/fasta_bins/
        extract_fasta_bins.py NoCorr_${i}_${asm}.m5000.prok.fna \
                              NoCorr_${i}_${asm}.concoct_hiseq/clustering_merged.csv \
                              --output_path NoCorr_${i}_${asm}.concoct_hiseq/fasta_bins/

        for fasta_file in `ls NoCorr_${i}_${asm}.concoct_hiseq/fasta_bins/`;
        do
            mv NoCorr_${i}_${asm}.concoct_hiseq/fasta_bins/${fasta_file} \
               NoCorr_${i}_${asm}.concoct_hiseq/NoCorr_${i}_${asm}.concoct_hiseq.${fasta_file}
        done
    done
done
```

----

Next up - [Comparison amd evaluation](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/6_evaluation.md)
