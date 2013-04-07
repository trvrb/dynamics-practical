# Inferring spatiotemporal dynamics of the H1N1 influenza pandemic from sequence data

## Introduction

In 2009, a [variant of influenza H1N1](http://en.wikipedia.org/wiki/Pandemic_H1N1/09_virus) emerged from a reassortant of bird, swine and human flu viruses, with surface proteins HA and NA of swine origin.
Owing to a lack of previous immunity to these proteins in the human population, this virus spread rapidly through the human population, causing a worldwide [pandemic](http://en.wikipedia.org/wiki/2009_flu_pandemic).
Here, we will investigate the growth of the virus population and its global spread using publicly available sequence data.

## Required software

* [BEAST](http://beast.bio.ed.ac.uk/) is used to infer evolutionary dynamics from sequence data.
* [BEAGLE](http://beast.bio.ed.ac.uk/BEAGLE) is a helper library that allows faster and more advanced functions to be run in BEAST. For this practical, it is not necessary to install CUDA drivers (step 2 in the BEAGLE installation).
* [Tracer](http://tree.bio.ed.ac.uk/software/tracer/) is used to analyze paramater estimates from BEAST.
* [FigTree](http://tree.bio.ed.ac.uk/software/figtree/) is used to analyze phylogeny estimates from BEAST.

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
Metadata about location and date of sampling can be kept as separate tab-delimited files, however, I find it easier to keep sequence names tagged with metadata, i.e. `A/Shenzhen/40/2009_China_2009-06-09` where underscores are used to separate fields.
As an aside, IRD supplies dates in USA mm/dd/yyyy format.
I find it much preferable to use yyyy-mm-dd format and so changed the date format of these sequences accordingly.
This format sidesteps any confusion that can arise from something like 05/03/2009 (is this 5 March or 3 May?).
Also, a simple sort of dates in yyyy-mm-dd format has the advantage of giving a chronological answer.

I then used [MUSCLE](http://www.drive5.com/muscle/) to align each set of sequences and trimmed the ends of alignments to remove uncertain sites.
The final aligned dataset is found is `data/pandemic.fasta`.

## Prepare a skyline analysis

The program BEAST takes an XML control file that specifies sequence data, metadata and also details the analysis to be run.
All program parameters lie in this control file.
However, to make things easier, BEAST is distributed with the companion program BEAUti that assists in generating XML control files.
Here, we will generate an XML that specifies a 'skyline' analysis, in which we estimate changes virus population size through time.

**Open BEAUti.**

This will show a window detailing data and analyses with the 'Partitions' panel open.
We first need to load the sequence data.

**Click on the '+' or choose 'Import Data...' from the File menu and select 'h3_china.fasta'.**

This will load a data partition of 499 taxa and 1763 nucleotide sites.

![beauti_partitions](images/beauti_partitions.png)

Double-clicking the partition will open a window showing the sequence alignment.
It's good to check to make sure the alignment is in order.

We next label each taxon with its sampling date.

**Select the 'Tips' panel, select 'Use tip dates' and click on 'Guess Dates'.**

We need to tell BEAUti where to find the tip dates in the taxon names.
Here each taxon name ends with its date of sampling separated by an underscore.

**Select 'Defined by a prefix and its order'.**

**Select 'Order' equals 'last' and input '_' for 'Prefix'.**

**Select 'Parse as calendar date'.**

![beauti_tips_guess_dates](images/beauti_tips_guess_dates.png)

This will result in the 'Date' and 'Height' columns filing the the date forward from the past and the height of each taxon relative to the most recent taxon.

![beauti_tips](images/beauti_tips.png)

**Click on 'Date' to resort rows.**

It will be helpful for later to record that the most recent tip has a date of '2009.75'.

Next, we need to specify a model of the process by which nucleotide sites evolve.

**Select the 'Sites' panel.**

We default to a very simple model of evolution.
This shows that we are using an 'HKY' model to parameterize evolution between nucleotides.
This model includes a single 'kappa' parameter that specifies the rate multiplier on transitions ('A' to 'G') vs transversions ('A' to 'T').
We are estimating base frequencies and specifying no heterogeneity across nucleotide sites in the alignment.
Generally speaking, if internal branches on the tree are long then a more complex evolutionary model will be needed to capture the real branch lengths, while if internal branches are short, then inferences will be fairly robust to model choice.

![beauti_sites](images/beauti_sites.png)

Next, we need to specify a molecular clock to convert between sequence substitutions and time.

**Select the 'Clocks' panel.**

We default to a strict molecular clock in which sequence substitution occur at some estimated rate per year.

![beauti_clocks](images/beauti_clocks.png)

Next, we need to specify a model that describes phylogenetic structure based on some underlying demographic processs.

**Select the 'Trees' panel.**

Here, we will choose a model that describes how the virus population size changes through time.
There are parametric models that assume some basic function (like exponential growth) and there are non-parametric models that don't make any strong assumptions about the pattern of change.
We begin by choosing a simple non-parametric model.

**Select 'Coalescent: Bayesian Skyline' from the 'Tree Prior' dropdown.**

This model assumes a fixed number of windows, where within each effective population size is constant and there is some weak autocorrelation assumed between windows to smooth the estimates.
We begin with the default 10 windows ('Number of groups'), and start with a random initial tree.

![beauti_trees_skyline](images/beauti_trees_skyline.png)

Generally, these non-parametric skyline (and skyride) models offer flexibility for the data to say what it wants to say.
However, these complex models suffer from the [bias-variance tradeoff](http://scott.fortmann-roe.com/docs/BiasVariance.html) and often give wide bounds of uncertainty to the resulting estimates.
We will return to this in a moment.

Next, we need to specify priors for each parameter in the model.

**Select the 'Priors' panel.**

For the most part, BEAST has very sensible default priors.
In this case, we can leave most of the parameters at their default values.

![beauti_priors](images/beauti_priors.png)

However, we are forced to choose a prior for evolutionary rate.

**Click on the prior for 'clock.rate' (currently highlighted in red).**

Although including improper priors will not harm parameter estimates, it will adversely impact the ability to do model comparison, and so it is not recommended.
Here, we still keep a unformative prior on 'clock.rate', choosing uniform between 0 and 1.

**Select 'Uniform' from the 'Prior Distribution' dropdown.**

**Enter 0.0 as a lower-bound and 1.0 as an upper bound.**

We actually have a good expectation from knowledge of RNA virus mutation rates that 'clock.rate' should be around 0.005.
We include this an initial value to aid convergence.

**Enter 0.005 as an 'Initial value'.**

![beauti_priors_uniform](images/beauti_priors_uniform.png)

After setting this, the 'clock.rate' prior no longer shows as red.

Next, we need to specify operators (or proposals) for the [MCMC](http://en.wikipedia.org/wiki/Markov_chain_Monte_Carlo) sampler.

**Select the 'Operators' panel.**

The exact choice of MCMC proposals will have no effect on eventual outcomes.
However, good proposals will make the MCMC more efficient and poor proposals will lead to MCMC inefficiency and longer run times.
In this case, we can stick with the default list of operators.

![beauti_operators](images/beauti_operators.png)

Next, we need to specify how often and where MCMC samples are logged to.

**Select the 'MCMC' panel.**

Generally, larger datasets will require longer chains and less frequent sampling.
I usually aim for 2000 samples, planning to throw out the first 500 or 1000 as [burn-in](http://en.wikipedia.org/wiki/Burn-in).

**Enter 50000000 (50 million) for 'Length of chain'.**

**Enter 25000 for 'Log parameters'.**

**Enter pandemic_skyline for 'File name stem'.**

This will result in 2000 samples logged to the files `pandemic_skyline.log` and `pandemic_skyline.trees`.

And that's it.  We just need to save the XML control file.

**Click on 'Generate BEAST File...' **

**Select Continue when shown the priors.**

**Save the XML as 'pandemic_skyline.xml'.**

For the most part, BEAUti does a good job creating an XML optimized for the analysis at hand.
However, there are often small details that need to be cleaned up by hand.
In this case, BEAUti defaults to writing out evolutionary rates to every branch in the phylogeny.
However, with a strict clock model, every branch will have the same rate at sampled MCMC step and this rate will already by written to the log file.
Writing out these rates to the trees file will just result in an unnecessarily larger file.

**Open 'pandemic_skyline.xml' in a text editor.**

XML is structured in a hierarchical fashion with logical blocks of markup surrounding by open (`<block>`) and close (`</block>`) tags.

**Find the XML block that specifies tree output called 'logTree'.**

```
	<logTree id="treeFileLog" logEvery="25000" nexusFormat="true" fileName="pandemic_skyline.trees" sortTranslationTable="true">
		<treeModel idref="treeModel"/>
		<trait name="rate" tag="rate">
			<strictClockBranchRates idref="branchRates"/>
		</trait>
		<posterior idref="posterior"/>
	</logTree>
```

**Delete the 'trait' block that contains 'strictClockBranchRates'.**

```
	<logTree id="treeFileLog" logEvery="25000" nexusFormat="true" fileName="pandemic_skyline.trees" sortTranslationTable="true">
		<treeModel idref="treeModel"/>
		<posterior idref="posterior"/>
	</logTree>
```

This fine-tuning of the XML can be quite helpful and there are many, more advanced, analyses that require editing the XML rather than relying on BEAUti output.

I've included this XML with the practical as `xml/pandemic_skyline.xml`.

## Run the skyline analysis

This XML file contains all the information that BEAST requires.

**Open BEAST.**

**Click on 'Choose File...' and select 'pandemic_skyline.xml'.**

By default, BEAST will have 'Allow overwriting of log files' turned off, so that you can't accidently overwrite a previous run's output.
However, you'll often need to check this box to allow overwriting when there are previous log files in the same directory that should be overwritten.

![beast_skyline](images/beast_skyline.png)

Also, BEAGLE is turned off by default.
BEAGLE is an additional Java library that contains high-performance code to compute evolutionary likelihoods on phylogenetic trees.
Most of the BEAST analyses will work on the built-in BEAST engine.
However, some of the newer analyses will require BEAGLE.
In this case, we can leave it unchecked or checked as desired.

**Click on 'Run'.**

This will start the BEAST run and the initial setup and the MCMC progress will be logged to a window.
The TMRCA ('rootHeight') and the per-year substitution rate ('clock.rate') will be logged to this window.
These two parameters often give a good idea of whether the MCMC appears to be behaving properly and help for monitoring convergence.
Additionally, the files `pandemic_skyline.log` and `pandemic_skyline.trees` will start to fill with MCMC samples.

Unfortunately, BEAST runs can take quite a long time, and it's not always clear how long they need to run.
Because of these time requirements, I almost never run BEAST analyses locally, prefering instead to get the XML up and running and tested locally and then running BEAST on a cluster node to perform the full analysis.
In this case, it took a single cluster node 16 hours to compute the 50 million MCMC steps, resulting in 2001 samples being logged and a 48 MB trees file.

I've included the resulting log file and tree file in the practical as `output/pandemic_skyline.log` and `output/pandemic_skyline.trees`.

## Examine the skyline output

Here, we begin by looking at estimated parameter values from the skyline analysis.

**Open Tracer.**

Tracer is designed to take a tab-delimited file specifying parameter values with each line representing a separate MCMC sample.

**Click '+' and select 'pandemic_skyline.log' from the resulting dialog.**

This displays each parameter as a separate element in the 'Traces' list.
First off, we need to assess whether the MCMC has converged to its stationary distribution.
The simplest way to do this is to look at MCMC state through time.

**Select 'posterior' from the list of 'Traces' and select the 'Trace' panel on the right.**

## Prepare a logistic growth analysis

We saw that the skyline analysis suggested that virus population size started off very small near the beginning of 2009 and increased throughout the year, though slowing down closer to Sep 2009.
A parametric model of [logistic growth](http://en.wikipedia.org/wiki/Logistic_function#In_ecology:_modeling_population_growth) fits this pattern nicely.
In this model, exponential growth occurs with rate *r*, but is damped as the population size *N* approaches carrying-capicity *K*.
The instantaneous rate of change in population size is:

![logistic_rate](images/logistic_rate.png)

If we start with some initial population size *N*<sub>0</sub> then we can solve for *N* at an arbitrary timepoint *t*:

![logistic_n0](images/logistic_n0.png)

However, because TMRCA is constantly varying over the course of the MCMC, BEAST parameterizes logistic growth not in terms of *N*<sub>0</sub>, but instead in terms of *t*<sub>50</sub>, that is the time back from the "present" (in this case 2009.75) when the population is at half its carrying capacity:

![logistic_t50](images/logistic_t50.png)

Thus, to model logistic growth, we need to estimate *r*, *K* and *t*<sub>50</sub>.

To prepare this XML, follow the exact steps from [preparing the skyline analysis](#prepare-a-skyline-analysis).
Here, start by importing sequence data in the 'Partitions' panel, adding tip dates in the 'Tips' panel.

**Select the 'Trees' panel and choose 'Logistic Growth' from the 'Tree Prior' dropdown.**

This will setup a demographic function based on the above outlined logistic growth model.

![beauti_trees_logistic](images/beauti_trees_logistic.png)

**Select the 'Priors' panel and choose a Uniform(0,1) prior for 'clock.rate'.**

![beauti_priors_logistic](images/beauti_priors_logistic.png)

The use of a Gamma prior for 'logistic.t50' is rather strange and exists as a holdover from a previous parameterization using a shape parameter to define timescale rather than 'logistic.t50'.
We'll fix this by editing the raw XML later.

**Select the 'MCMC' panel, set the length of the chain to 50000000, set logging of parameters every 25000 steps and set the file name stem to pandemic_logistic.**

This will log parameters and trees to `pandemic_logistic.log` and `pandemic_logistic.trees`.

![beauti_mcmc_logistic](images/beauti_mcmc_logistic.png)

**Save the XML file as pandemic_logistic.xml.**

**Open the resulting file in text editor and delete the 'trait' block from the the 'logTree' block as before.**

We will also manually set the prior on 'logistic.t50' by replacing the 'gammaPrior' with:

```
	<uniformPrior lower="0.0" upper="1.0">
		<parameter idref="logistic.t50"/>
	</uniformPrior>	
```

With this completed, we are ready to run this analysis in BEAST.

## Run the logistic growth analysis

**Open BEAST, select the file pandemic_logistic.xml and choose 'Run'.**

This will produce the output files `pandemic_logistic.log` and `pandemic_logistic.trees`, again taking ~16 hours to complete on a single cluster node.

I've included these files with the practical as `output/pandemic_logistic.log` and `output/pandemic_logistic.trees`.

## Examine the logistic growth output

**Open pandemic_logistic.log in Tracer.**

Here, we see that each trace is behaving well, but that we need to set the burn-in higher than the default 5 million.

**Set 'Burn-in' to 10000000 (10 million).**

Here, we see we get the following parameter estimates and 95% credible intervals for the logistic growth model:

                        | Lower | Mean  | Upper
---                     | ---   | ---   | ----
*K* (popSize)           | 9.49  | 48.38 | 140.49
*r* (growthRate)        | 6.65  | 10.64 | 14.47
*t*<sub>50</sub> (t50)  | 0.02  | 0.17  | 0.28

Tracer can also be used to reconstruct population size through time according to the logistic growth model.

**Choose 'Demographic Reconstruction...' from the 'Analysis' menu.**

In the resulting dialog box, the demographic function chosen needs to be matched to the one actually used.

**Select 'Logistic Growth (Growth Rate)' from the 'Demographic Model' dropdown.**

The traces used for each of the three parameters are selected automatically.
However, overall timescale still needs to be set.

**Enter 2009.75 for 'Age of youngest tip'.**

![tracer_logistic_dialog](images/tracer_logistic_dialog.png)

**Click on 'OK' to perform the analysis.**

The resulting window shows the median estimate of population size through time, as well as its 95% credible interval.

![tracer_logistic_history](images/tracer_logistic_history.png)

Here, we see that population size has grown exponentially until around July or August when it's begun to saturate.
However, although the rate of initial exponential increase appears fairly well defined, there appears to be little certainty to the degree of recent saturation.

The equation log(2) / *r* can be used to calculcate the doubling time in years of virus population.
The doubling time in days can be calculcated by multiplying this by 365.
Additionally, with knowledge of the serial interval between infections we can convert the growth rate in terms of years into a growth rate in terms of serial intervals.


                     | Lower | Mean  | Upper
---                  | ---   | ---   | ----
*r* (per year)       | 6.65  | 10.64 | 14.47
Doubling time (days) | 17.48 | 23.78 | 38.04
