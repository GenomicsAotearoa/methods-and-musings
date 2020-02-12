### Approach two - error correction and assembly

The most obvious issue with load-read sequencing at the moment is the comparatively high error rate of Nanopore and PacBio sequencing when compared with Illumina technology. To circumvent this, there are a number of tools for using high-quality, short sequences to correct the error-prone long sequences in order to improve assembly.

----

#### Error correction

When doing my research for this project, I found an excellent comparison ([Fu *et al.*, 2019](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1605-z)) which reported the following figure:

![](https://media.springernature.com/full/springer-static/image/art%3A10.1186%2Fs13059-018-1605-z/MediaObjects/13059_2018_1605_Fig6_HTML.png?as=webp)

Based on these results, I selected `FMLRC` as the tool to use, as it was generally the best-performing tool in every category than time efficiency. With all bioinformatic analysis, I take the view that quality trumps speed, so this is an acceptable cost to me.

*With that said, I timed the runtime of `FMLRC` across the correction process. Results below...*

I used the workflow suggested on the `FMLRC` (v1.0.0), except rather than use `sort` to organise the reads I used a brief piece of `python` code to drastically speed up the process.

```bash
for i in S4R3 S5R2 S6R1;
do
    gunzip -c ${i}_R1.fastq.gz | awk 'NR % 4 == 2' > S4R3.txt
    gunzip -c ${i}_R2.fastq.gz | awk 'NR % 4 == 2' >> S4R3.txt
    gunzip -c ${i}_single.fastq.gz | awk 'NR % 4 == 2' >> S4R3.txt
done
```

```python
def import_sorted_strings(sample_name):
    return sorted( [ x.strip() for x in open('{}.txt'.format(sample_name), 'r') ] )

def push_out(sample_name, strings):
    output = open('{}.sorted.txt'.format(sample_name), 'w')
    _ = [ output.write( s + '\n' ) for s in strings ]
    output.close()

for sample in ['S4R3', 'S5R2', 'S6R1']:
    sorted_seqs = import_sorted_strings(sample)
    push_out(sample, sorted_seqs)
```

```bash
for i in S4R3 S5R2 S6R1;
do
    # Preprocess the reads
    cat ${i}.sorted.txt | tr NT TN > temp_${i}.txt
    sropebwt2 -LR temp_${i}.txt | tr NT TN > map_${i}.txt
    fmlrc-convert -i map_${i}.txt ${i}.npy
    rm temp_${i}.txt map_${i}.txt

    # Perform correction - need to extract the ONT reads first
    gunzip -c ${i}.nanofilt.fastq.gz > ${i}.nanofilt.fastq
    fmlrc -p 30 ${i}.npy ${i}.nanofilt.fastq ${i}.fmlrc.fasta
done
```

Total time to process:

1. S4R3: 1 h 55 min
1. S5R2: 2 h 12 min
1. S6R1: 57 min

Which is hardly a problem. I wouldn't even consider five an a half hours a meaningful delay in this analysis.

----

#### Assembly

Assembly was performed using the same tools and settings as before, which the exception that for `flye` and the retuned `metaFlye` rounds, the genome size parameter was updated.

When assembly sample S6R1 with `Canu`, assembly failed despite multiple attempts, so this sample is not included in subsequent sections.

----

#### Hybrid assembly

Another option for using the Illumina data to improve the assembly is to use the hybrid assembly option in `metaSPAdes`. Lets add it to the mix!

```bash
for i in S4R3 S5R2 S6R1;
do
  spades.py --meta -t 30 -m 2800 -k 43,55,77,99,121 \
            --nanopore ${i}.nanofilt.fastq.gz \
            -1 ${i}_R1.fastq.gz -2 ${i}_R2.fastq.gz -s ${i}_single.fastq.gz \
            -o ${i}_hybridspades/
done
```

----

Next up - [Read mapping](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/4_read_mapping.md).
