# Inferring spatiotemporal dynamics of H3N2 influenza using sequence data

## Introduction

[Influenza A/H3N2](http://en.wikipedia.org/wiki/Influenza_A_virus_subtype_H3N2) has continually circulated in the human population since it's introduction in 1968.
Temperate regions have strong annual epidemics that occur each winter, while incidence in tropical regions is less periodic.
Here, we will attempt to determine global patterns of virus transmission from available sequence data.

## Data

I started by downloading a full set of sequences of the HA segment of H3N2 influenza from the [Influenza Research Database](http://www.fludb.org) dated between 2000 and 2012.  
This gave me 11,448 sequences.
I went through and cleaned these sequences by keeping only sequences with complete dates that were at least 900 bases in length.
I also eliminated sequences with duplicate strain names, keeping the longest sequence of this set.
These operations left 7870 sequences.

IRD labels sequences with country of sampling.
However, we need things more coarsely grained than this, and so I've tagged each sequence based on 9 geographic regions.
This tagging results in the following distribution of sequence counts: NorthAmerica (2767), China (1185), JapanKorea (1178), Oceania (845), SoutheastAsia (725), Europe (480), CentralAsia (339), Africa (215), SouthAmerica (117).

It's clear that these counts are biased between regions (and also biased towards the present).
Because we're interested in spatiotemporal patterns, we want a more equitable distribution in space and time.

Consequently, I've prepared three datasets by subsampling this full dataset:

- `h3_usa.fasta` contains
- `h3_world.fasta` has 5 samples per year per region giving a distribution of NorthAmerica (65), JapanKorea (60), China (58), Europe (56), SoutheastAsia (50), CentralAsia (48), Oceania (47), Africa (44), SouthAmerica (44).

