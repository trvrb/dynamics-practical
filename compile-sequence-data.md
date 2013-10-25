## Compile sequence data

I started by downloading a set of sequences of the HA segment of pandemic H1N1 influenza from the [Influenza Research Database](http://www.fludb.org) sampled during 2009.
This gave me 8176 sequences.
I went through and cleaned these sequences by keeping only sequences with complete dates that were at least 900 bases in length.
I also eliminated sequences with duplicate strain names, keeping the longest sequence of this set.
These operations left 6862 sequences.

IRD labels sequences with country of sampling.
However, we need things more coarsely grained than this, and so I tagged each sequence based on 11 geographic regions.
This tagging results in the following distribution of sequence counts: USACanada (2211), Europe (2092), SoutheastAsia (673), China (560), JapanKorea (554), CentralAsia (309), SouthAmerica (308), Mexico (243), CentralAmerica (93), Africa (161), Oceania (93).

It's clear that these counts are biased between regions (and also biased across time).
Because we're interested in spatiotemporal patterns, I subsampled this dataset to give a more equitable distribution in space and time.
I sampled at most 10 sequences per month per region from the beginning of 2009 through September 2009.
This resulted in a total of 514 sequences with the following distribution: USACanada (91), Europe (90), SoutheastAsia (80), China (82), JapanKorea (80), CentralAsia (74), SouthAmerica (58), Mexico (61), CentralAmerica (62), Africa (54), Oceania (33).

BEAST accepts either [NEXUS](http://en.wikipedia.org/wiki/Nexus_file) or [FASTA](http://en.wikipedia.org/wiki/FASTA_format) format for sequence data.
Metadata about location and date of sampling can be kept as separate tab-delimited files, however, I find it easier to incorporate metadata into sequence names, i.e. `A/Shenzhen/40/2009_China_2009-06-09` where underscores are used to separate fields.

I then used [MUSCLE](http://www.drive5.com/muscle/) to align each set of sequences and trimmed the ends of alignments to remove uncertain sites.
I've included the final aligned dataset as `data/pandemic.fasta`.

### Next section

* [Prepare a skyline analysis](prepare-a-skyline-analysis.md)