### Binning and dereplication

When binning metagenomes, our favoured workflow is to bin using the tools `MetaBAT`, `MaxBin`, and `CONCOCT` and then dereplicate the outputs using `DAS_Tool`. This is the approach we will apply here.

----

#### ONT mapping

#### MetaBAT v2.13

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        # Uncorrected...
        jgi_summarize_bam_contig_depths --outputDepth NoCorr_${i}_${asm}.metabat.txt NoCorr_${i}_${asm}.m5000.bam
        metabat2 -t 10 -s 50000 -a NoCorr_${i}_${asm}.metabat.txt \
                                -i NoCorr_${i}_${asm}.m5000.prok.fna \
                                -o NoCorr_${i}_${asm}.metabat_ont/NoCorr_${i}_${asm}.metabat_ont

        # FMLRC...
        jgi_summarize_bam_contig_depths --outputDepth FMLRC_${i}_${asm}.metabat.txt FMLRC_${i}_${asm}.m5000.bam
        metabat2 -t 10 -s 50000 -a FMLRC_${i}_${asm}.metabat.txt \
                                -i FMLRC_${i}_${asm}.m5000.prok.fna \
                                -o FMLRC_${i}_${asm}.metabat_ont/FMLRC_${i}_${asm}.metabat_ont
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
        # Uncorrected...
        cut -f1,4 NoCorr_${i}_${asm}.metabat.txt > NoCorr_${i}_${asm}.maxbin.txt
        mkdir NoCorr_${i}_${asm}.maxbin_ont/
        run_MaxBin.pl -thread 10 -contig NoCorr_${i}_${asm}.m5000.prok.fna \
                                 -abund NoCorr_${i}_${asm}.maxbin.txt \
                                 -out NoCorr_${i}_${asm}.maxbin_ont/NoCorr_${i}_${asm}.maxbin_ont

        # FMLRC...
        cut -f1,4 FMLRC_${i}_${asm}.metabat.txt > FMLRC_${i}_${asm}.maxbin.txt
        mkdir FMLRC_${i}_${asm}.maxbin_ont/
        run_MaxBin.pl -thread 10 -contig FMLRC_${i}_${asm}.m5000.prok.fna \
                                 -abund FMLRC_${i}_${asm}.maxbin.txt \
                                 -out FMLRC_${i}_${asm}.maxbin_ont/FMLRC_${i}_${asm}.maxbin_ont
    done
done
```

#### CONCOCT v1.0.0

The `CONCOCT` pipeline requires us to chop up our contigs into shorter sub-contigs, which are used for clustering into bins. In order to simplify the loops run here, we'll split this step out first. Given how long these commands are, I'm not showing the `FMLRC` version here, but it should be obvious from the patterns above how it is occuring.

```bash
for i in S4R3 S5R2 S6R1;
do
    for asm in flye metaflye metaflye_retuned miniasm canu shasta hybridspades;
    do
        # Uncorrected...
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
        # Uncorrected...
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

To do...

----

Next up - [Comparison amd evaluation](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/6_evaluation.md)
