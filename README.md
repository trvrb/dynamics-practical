## Introduction

In 2009, a [variant of influenza H1N1](http://en.wikipedia.org/wiki/Pandemic_H1N1/09_virus) emerged through reassortment of bird, swine and human flu viruses, with surface proteins HA and NA of swine origin.
Owing to a lack of previous immunity to these proteins, this virus spread rapidly through the human population, causing a worldwide [pandemic](http://en.wikipedia.org/wiki/2009_flu_pandemic).
Here, we will investigate the growth of the virus population and its global spread using publicly available sequence data.
This tutorial walks through how to use [BEAST](http://beast.bio.ed.ac.uk/) and associated software to infer spatiotemporal dynamics from viral sequence data.
Accompanying slides on the theory behind this analysis are available in [theory/slides.pdf](theory/slides.pdf).

## Required software

* [BEAST](http://beast.bio.ed.ac.uk/) is used to infer evolutionary dynamics from sequence data.
* [BEAGLE](http://beast.bio.ed.ac.uk/BEAGLE) is a helper library that allows faster and more advanced functions to be run in BEAST. For this practical, it is not necessary to install CUDA drivers (step 2 in the BEAGLE installation).
* [Tracer](http://tree.bio.ed.ac.uk/software/tracer/) is used to analyze parameter estimates from BEAST.
* [FigTree](http://tree.bio.ed.ac.uk/software/figtree/) is used to analyze phylogeny estimates from BEAST.
* [Google Earth](http://www.google.com/earth/) is used to display phylogeographic reconstructions.

## Contents

1. [Compile sequence data](compile-sequence-data.md)
2. [Prepare a skyline analysis](prepare-a-skyline-analysis.md)
3. [Run the skyline analysis](run-the-skyline-analysis.md)
4. [Examine the skyline output](examine-the-skyline-output.md)
5. [Examine the skyline tree](examine-the-skyline-tree.md)
6. [Prepare a logistic growth analysis](prepare-a-logistic-growth-analysis.md)
7. [Run the logistic growth analysis](run-the-logistic-growth-analysis.md)
8. [Examine the logistic growth output](examine-the-logistic-growth-output.md)
9. [Prepare a phylogeographic analysis](prepare-a-phylogeographic-analysis.md)
10. [Run the phylogeographic analysis](run-the-phylogeographic-analysis.md)
11. [Examine the phylogeographic output](examine-the-phylogeographic-output.md)

### Next section

* [Compile sequence data](compile-sequence-data.md)