# Gillenia genome assembly

## Background

Gillenia is the extant, dry-fruit bearing genus of the putative ancestral lineage of apple, pear and ~1000 pome-bearing species within the Maleae tribe. The genome of Gillenia contains 18 chromosomes (2n = 18) with the size of 330 Mb from flow cytometry. A contruction of Gillenia genome offers a potential tool for cross-family comparative studies. Here is an experience of using nanopore reads for scaffolding 10x Chromium assembly.

## Programs

**[LINKS](https://github.com/bcgsc/LINKS)**: it can be used to scaffold high-quality draft genome assemblies with any long sequences. 
It extract kmer pairs from both assembly scaffolds/contigs and nanopore raw reads, and then scaffolding the two contigs when a kmer pair from reads matches a pair generated from the two scaffolds/contigs. Three parameters required for optimisation: kmer length, k (distance for moving to the next kmer pair) and d (distance between a kmer pair).

<p align="center">
  <img width="460" height="300" src="https://github.com/christinawu2008/methods-and-musings/blob/master/genome-assembly_ont/Gillenia/links_method.png">
</p>


**[SLR](https://github.com/luojunwei/SLR)**: this is an alignment-based scaffolding tool using long reads.

## Methods

The initial scaffolds were generated from 10X Chromium reads with effective coverage of 45X using Supernova (v1.2.2), which were then subjected to a scaffolding step using ONT reads (length > 30kb). LINKS (v1.8.5) was used to perform 6 iterations (a 5k step increase of d from 5k to 50k; other parameters: k21, t10) of scaffolding on the initial scaffolds. The resulted super-scaffolds were then gap-filled with all the ONT reads using TGS-GapCloser (v1.1.1). The bacteria-like sequences identified from Kraken2 (searching database: RefSeq93) were removed from the assembly. (SLR scaffolds were compared with LINKS_i6's, which shows worse assembly statistics.)

## Results

A comparison among 10x Chromium, LINKS_i6 and SLR assembly is shown below (Scaffolding using LINKS after round 4 were only marginly increased sequence contigurity).

|                                              | 10x scaffolds | scaffolds\_links\_i6 | scaffolds\_slr |
| -------------------------------------------- | ------------- | -------------------- | -------------- |
| Number of scaffolds                          | 7108          | 4159                 | 5415           |
| Total size of scaffolds                      | 268,717,626     | 277,641,578            | 294,376,493      |
| Longest scaffold                             | 10,638,484      | 15,940,843             | 14,872,106       |
| Shortest scaffold                            | 2000          | 1959                 | 1959           |
| Number of scaffolds > 10K nt                 | 1249          | 789                  | 1116           |
| Number of scaffolds > 100K nt                | 308           | 108                  | 264            |
| Number of scaffolds > 1M nt                  | 66            | 58                   | 85             |
| Number of scaffolds > 10M nt                 | 1             | 3                    | 2              |
| N50 scaffold length                          | 847,924        | 4,034,243              | 1,339,010        |
| L50 scaffold count                           | 81            | 20                   | 56             |
| scaffold %C                                  | 18.29         | 17.84                | 17.63          |
| scaffold %N                                  | 4.12          | 6.52                 | 7.57           |
| scaffold %non-ACGTN                          | 0             | 0                    | 0              |
| Percentage of assembly in scaffolded contigs | 83.90%        | 93.30%               | 91.00%         |
| BUSCO Complete                               | 97.60%        | 98.00%               | 97.70%         |
| BUSCO Single                                 | 94.80%        | 95.10%               | 91.30%         |
| BUSCO Duplicate                              | 2.80%         | 2.90%                | 6.40%          |
| BUSCO Partial                                | 0.90%         | 0.70%                | 0.90%          |
| BUSCO Missing                                | 1.50%         | 1.30%                | 1.40%          |
| BUSCO Total genes                            | 1375          | 1375                 | 1375           |

The sequence length distribution graph showed below also indicates the effectiveness of scaffolding for relatively short sequences. 

<p align="center">
  <img width="460" height="300" src="https://github.com/christinawu2008/methods-and-musings/blob/master/genome-assembly_ont/Gillenia/seq_len_dis.png">
</p>

## Tips for running LINKS (Rene Warren: rwarren@bcgsc.ca)

* ~700-900 GB RAM is the sweet spot for LINKS. The memory panic error is a PERL out-of-memory error, unfortunately. PERL is not efficient with memory allocation and management. This will occur even if you have a 2TB machine. 
* only way to curb memory usage is to increase the spacing between kmer pair (with the skip parameter -t). It is best to choose values of -t that will place your peak memory usage to 700-900GB zone (machine permitting). This is not an ideal solution, as it makes troubleshooting challenging and decreases available kmer support. That said, you have ~125X ONT coverage of your genome. That's an overkill for LINKS and increasing -t isn't going to discard too many kmer pairs (since the redundancy in the data should capture additional supporting linkages in other reads, which is a better type of scaffolding support). 
* Alternatively, you could decrease the read coverage to 50X and decrease -t. LINKS will run faster on a smaller ONT data set, but if runtime isn't an issue, you should just keep the whole ONT set and increase -t. 
* Also, If you increase -d at each round, then you can decrease -t by a similar factor and stay within the RAM
* "-t 10" worked for this project, while "-t 5" didn't.
