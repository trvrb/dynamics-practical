# Inferring spatiotemporal dynamics of H3N2 influenza using sequence data

** Trevor Bedford **
** Institute of Evolutionary Biology **
** University of Edinburgh **

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
This tagging results in the following distribution of sequence counts:

- NorthAmerica	2759
- JapanKorea	855
- China	791
- SoutheastAsia	539
- Oceania	338
- CentralAsia	331
- Europe	326
- Africa	205
- SouthAmerica	115

Unfortunately, these sequences are strongly biased towards particular regions and towards the present.
Because we're interested in spatiotemporal patterns, we want a more equitable distribution in space and time, and so I subsampled the sequences.