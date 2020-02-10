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

### Overview of analysis

1. [Experimental information](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/1_experimental_setup.md)
1. [Approach one - assembly of uncorrected reads](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/2_uncorrected_assembly.md)
1. [Approach two - error correction and assembly](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/3_error_correction.md)
1. [Read mapping](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/4_read_mapping.md)
1. [Binning and dereplication](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/5_binning.md)
1. [Comparison and evaluation of MAGs](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/6_evaluation.md)

---
