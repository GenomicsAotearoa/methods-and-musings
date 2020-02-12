## Experimental information - the study site and available sequencing data

To begin with, we already had a data set of nine sites taken from an estuary. As part of our ongoing study, we had obtained triplicate metagenomes from the sediment at each site, sequenced on the Illumina HiSeq. On analysis these data, we identified four sites that were rich in sequences belonging to the phylum Thaumarchaeota, which is important in the research questions we are asking of our data.

From three of these sites, we were able to extract sufficient high molecular weight DNA for sequencing on the Oxford PromethION platform. From each of these samples, we obtained between 1.2 and 2.1 million sequences, which were quality filtered with `porechop` and trimmed with `nanofilt`. The basic statistics for our samples were:

|Sample|Raw sequences|Post-*porechop*|Post-*nanofilt*|
|:---|:---:|:---:|:---:|
|S4R3|2,088,722|2,087,200|1,954,049|
|S5R2|1,577,002|1,574,819|1,407,863|
|S6R1|1,236,066|1,234,600|1,065,611|

----

Next up - [Assembly (part 1)](https://github.com/GenomicsAotearoa/methods-and-musings/blob/master/metagenomic_ont/2_uncorrected_assembly.md).
