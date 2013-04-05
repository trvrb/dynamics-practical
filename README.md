# Inferring spatiotemporal dynamics of H3N2 influenza using sequence data

## Introduction

[Influenza A/H3N2](http://en.wikipedia.org/wiki/Influenza_A_virus_subtype_H3N2) has continually circulated in the human population since it's introduction in 1968.
Temperate regions have strong annual epidemics that occur each winter, while incidence in tropical regions is less periodic.
Here, we will attempt to determine global patterns of virus transmission from available sequence data.

This practical is designed to recapitulate some of the main results of the following two papers:

* [Rambaut et al. 2008. The genomic and epidemiological dynamics of human influenza A virus. Nature 453: 615-619.](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC2441973/)
* [Bedford et al. 2010. Global migration dynamics underlie evolution and persistence of human influenza A (H3N2). PLoS Pathogens 6: e1000918.](http://www.plospathogens.org/article/info%3Adoi%2F10.1371%2Fjournal.ppat.1000918)

## Required software

* [BEAST](http://beast.bio.ed.ac.uk/) is used to infer evolutionary dynamics from sequence data.
* [Tracer](http://tree.bio.ed.ac.uk/software/tracer/) is used to analyze paramater estimates from BEAST.
* [FigTree](http://tree.bio.ed.ac.uk/software/figtree/) is used to analyze phylogeny estimates from BEAST.

## Compiling sequence data

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

* The first contains ~50 samples per year from the USA and a total of 507 sequences.
* The second contains ~50 samples per year from China and a total of 499 sequences.
* The third has ~5 samples per year per region across the world giving a distribution of NorthAmerica (65), JapanKorea (60), China (58), Europe (56), SoutheastAsia (50), CentralAsia (48), Oceania (47), Africa (44), SouthAmerica (44) and a total of 472 sequences.

BEAST accepts either [NEXUS](http://en.wikipedia.org/wiki/Nexus_file) or [FASTA](http://en.wikipedia.org/wiki/FASTA_format) format for sequence data.
Metadata about location and date of sampling can be kept as separate tab-delimited files, however, I find it easier to keep sequence names tagged with metadata, i.e. `A/Malaysia/1755590/2007_CY118946_SoutheastAsia_2007-03-30` where underscores are used to separate fields.
As an aside, IRD supplies dates in USA mm/dd/yyyy format.
I find it much preferable to use yyyy/mm/dd format and have changed the date format of these sequences accordingly.
This format sidesteps any confusion that can arise from something like 05/03/2008 (is this 5 March or 3 May?).
Also, a simple sort of dates in yyyy/mm/dd format has the advantage of giving a chronological answer.

I then used [MUSCLE](http://www.drive5.com/muscle/) to align each set of sequences.
At the end of these operations I was left with three FASTA files: 

* `data/h3_usa.fasta`
* `data/h3_china.fasta`
* `data/h3_world.fasta`

## Preparing XML control files

The program BEAST takes an XML control file that specifies sequence data, metadata and also details the analysis to be run.
All program parameters lie in this control file.
However, to make things easier, BEAST is supplied with the companion program BEAUti that assists in generating XML control files.

```
Open BEAUti
```

This will show a window detailing data and analyses with the 'Partitions' panel open.
We first need to load the sequence data.

```
Click the '+' or choose 'Import Data...' from the File menu and select 'h3_china.fasta'.
```

![beauti_partitions](images/beauti_partitions.png)

This will load a data partition of 499 taxa and 1763 nucleotide sites.
Double-clicking the partition will open a window showing the sequence alignment.
It's good to check to make sure the alignment is in order.

We next label each taxon with its sampling date.

```
Click on the 'Tips' panel, select 'Use tip dates' and click on 'Guess Dates'.
```

![beauti_tips](images/beauti_tips.png)

We need to tell BEAUti where to find the tip dates in the taxon names.
Here each taxon name ends with its date of sampling separated by an underscore.

```
Select 'Defined by a prefix and its order'.
Select 'Order' equals 'last' and input '_' for 'Prefix'.
Select 'Parse as calendar date'.
```

This will result in the 'Date' and 'Height' columns filing the the date forward from the past and the height of each taxon relative to the most recent taxon.

```
Click on 'Date' to resort rows.
```

It will be helpful for later to record that the most recent tip has a date of '2012.17'.

Next, we need to specify a model of the process by which nucleotide sites evolve.

```
Click on the 'Sites' panel.
```

We default to a very simple model of evolution.
This shows that we are using an 'HKY' model to parameterize evolution between nucleotides.
This model includes a single 'kappa' parameter that specifies the rate multiplier on transitions ('A' to 'G') vs transversions ('A' to 'T').
We are estimating base frequencies and specifying no heterogeneity across nucleotide sites in the alignment.
Generally speaking, if internal branches on the tree are long then a more complex evolutionary model will be needed to capture the real branch lengths, while if internal branches are short, then inferences will be fairly robust to model choice.

Next, we need to specify a molecular clock to convert between sequence substitutions and time.

```
Click on the 'Clocks' panel.
```

We default to a strict molecular clock in which sequence substitution occur at some estimated rate per year.

Next, we need to specify a model that describes phylogenetic structure based on some underlying demographic processs.

```
Click on the 'Trees' panel.
```





Most recent China: 2012.17
Most recent USA: 2012.97
Most recent World: 2012.92


