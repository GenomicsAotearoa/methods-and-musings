# Gillenia genome assembly

## Background

Gillenia genome size is around 330Mb and initial scaffolds were produced from 10x chromium reads using Supernova.
Nanopore reads including MiNION and Promethon reads were then generated for improving scaffolds.
Long reads (> 30 bases) were chosen for post-scaffolding using nanopore reads. 

## Programs

* **[LINKS](https://github.com/bcgsc/LINKS)**: can be used to scaffold high-quality draft genome assemblies with any long sequences. It extract kmer pairs from both assembly scaffolds/contigs and nanopore raw reads, and then scaffolding the two contigs when a kmer pair from reads matches a pair generated from the two scaffolds/contigs. Three parameters required for optimisation: kmer length, k (distance for moving to the next kmer pair) and d (distance between a kmer pair). The d can be iteratively increased from each time of scaffolding the previously generated scaffolds.

![LINKS method](https://github.com/christinawu2008/methods-and-musings/blob/master/genome-assembly_ont/Gillenia/links_method.png)

* **[SLR](https://github.com/luojunwei/SLR)**: this is an alignment-based scaffolding tool using long reads.


## Results

We have done 6 round of scaffolding using LINKS and 1 round of scaffolding using SLR. Scaffolding using LINKS after round 4 were only marginly increased sequence contigurity.
All resulted scaffolds were then gapfilled using TGS-GapCloser with raw nanopore reads, then were removed with contamination sequences.

* statistics of scaffold sets produced from post-scaffolding step



|                                              | 10x scaffolds | scaffolds\_links\_i6 | scaffolds\_slr |
| -------------------------------------------- | ------------- | -------------------- | -------------- |
| Number of scaffolds                          | 7108          | 4159                 | 5415           |
| Total size of scaffolds                      | 268717626     | 277641578            | 294376493      |
| Longest scaffold                             | 10638484      | 15940843             | 14872106       |
| Shortest scaffold                            | 2000          | 1959                 | 1959           |
| Number of scaffolds > 10K nt                 | 1249          | 789                  | 1116           |
| Number of scaffolds > 100K nt                | 308           | 108                  | 264            |
| Number of scaffolds > 1M nt                  | 66            | 58                   | 85             |
| Number of scaffolds > 10M nt                 | 1             | 3                    | 2              |
| N50 scaffold length                          | 847924        | 4034243              | 1339010        |
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

* scaffold length distribution
![LINKS & SLR](https://github.com/christinawu2008/methods-and-musings/blob/master/genome-assembly_ont/Gillenia/seq_len_dis.png)

## tips for running LINKS

* ~700-900 GB RAM is the sweet spot for LINKS. The memory panic error is a PERL out-of-memory error, unfortunately. PERL is not efficient with memory allocation and management. This will occur even if you have a 2TB machine. 
* only way to curb memory usage is to increase the spacing between kmer pair (with the skip parameter -t). It is best to choose values of -t that will place your peak memory usage to 700-900GB zone (machine permitting). This is not an ideal solution, as it makes troubleshooting challenging and decreases available kmer support. That said, you have ~125X ONT coverage of your genome. That's an overkill for LINKS and increasing -t isn't going to discard too many kmer pairs (since the redundancy in the data should capture additional supporting linkages in other reads, which is a better type of scaffolding support). 
* Alternatively, you could decrease the read coverage to 50X and decrease -t. LINKS will run faster on a smaller ONT data set, but if runtime isn't an issue, you should just keep the whole ONT set and increase -t. 
* Also, If you increase -d at each round, then you can decrease -t by a similar factor and stay within the RAM
* "-t 10" worked for this project, while "-t 5" didn't.
