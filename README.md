My notes on the what, how, and why of bioregionalisation. I'm almost certainly missing something.

I've put it on Github mainly to make it easier to share it with people, but also in case people want to contribute. Feel free to raise an issue or make a pull request or anything if you think there's something wrong or missing. I've almost certainly missed something.

This is mainly drawn from the [Vilhena and Antonelli Nature Comms paper](https://www.nature.com/articles/ncomms7848), this [Ebach and Parenti J. Biogeog. paper](https://onlinelibrary.wiley.com/doi/full/10.1111/jbi.12558), and this [Rueda et al. J. Biogeog. paper](https://onlinelibrary.wiley.com/doi/full/10.1111/jbi.12214).

- [What](#what)
- [How](#how)
  - [Distance based clustering](#distance-based-clustering)
  - [Network based clustering](#network-based-clustering)
- [Why bother?](#why-bother)
  - [Some uses](#some-uses)
  - [Some problems/questions](#some-problemsquestions)

# What

Bioregionalisation is the process of dividing the world up in to different biogeographical regions to study groups of species together.

One of the first widely accepted divisions was a continental-based system by [Wallace of zoogeographical regions](http://darwin-online.org.uk/converted/Ancillary/1876_GeographicalDistribution_S718.1/1876_GeographicalDistribution_S718.1.html). This was based on a mix of quantitative reasoning and (semi-)subjective judgments.

After that there were a number of refinements/different systems proposed using similar reasoning/methodology. Then it started going out of favour, until over the last decade(ish) there has been renewed interest.

Most of the new papers on bioregionalisation use clustering based methods on species occurrence/range data to delimit bioregions without the (semi-)subjective judgments.

Have a number of different uses, but all essentially use the bioregions as a unit for making analysis easier.

# How

Wallace came up with this by setting out criteria that defined suitable bioregions:

* Regions should be of a reasonable number.
* They should be roughly equal in size.
* The boundaries should reflect major geographical divisions (easily defined and remembered).
* Regions should be taxonomically distinct.
* Shouldn't just be based on distinctiveness and endemism, but also richness and diversity.

Then he defined hierarchical regions based on numbers of genera and families of mammals. So quantitatively based regions, but with (semi-)subjective delimitation.

Since Wallace there have been shifting fashions in how to define biogeographical regions, with people also proposing regions based on environmental variables like climate, or ecosystem responses like vegetation productivity that can be measured using remote sensing. [This paper by Mackey, Berry, and Brown](https://doi.org/10.1111/j.1365-2699.2007.01822.x) gives a (brief) overview of the different methods and tries to develop a general framework that all bioregionalisations can be done under. The [ecoregions proposed by Olsen et al in 2001](https://academic.oup.com/bioscience/article/51/11/933/227116) are a more modern example of using the presence/absence of taxa as the basis of biogeographical regions, but also pre-existing regionalisations based on habitats and vegetation types and other things.

Most new bioregionalisations follow the general process of:

1. Get species occurrence or range data for taxonomic group of interest.
2. Split that data up into species presences at sites (grid the data).
3. Determine the relationship between sites (distance or bipartite graph).
4. Use the relationships between sites to cluster them into bioregions.

## Distance based clustering

These all use some measure of distance between sites to cluster them. For bioregions, these are usually some sort of beta-diversity measure like:

* **Sorensen distance**

    <img src="https://latex.codecogs.com/svg.latex?\beta_\textrm{sor}&space;=&space;1&space;-&space;\frac{2&space;\left|X&space;\cap&space;Y\right|}{\left|X\right|&space;&plus;&space;\left|Y\right|}" title="\beta_\textrm{sor} = 1 - \frac{2 \left|X \cap Y\right|}{\left|X\right| + \left|Y\right|}" />

    Twice the number of the taxa in common between site *X* and site *Y*, divided by the sum of the total number of taxa at site *X* and the total number of taxa at site *Y*.

* **Simpson distance**

    <img src="https://latex.codecogs.com/svg.latex?\beta_\textrm{sim}&space;=&space;1&space;-&space;\frac{\left|X&space;\cap&space;Y\right|}{\min{(\left|X\right|,\left|Y\right|)}}" title="\beta_\textrm{sim} = 1 - \frac{\left|X \cap Y\right|}{\min{(\left|X\right|,\left|Y\right|)}}" />

    The number of taxa in common between site *X* and site *Y*, divided by the total number of taxa at the site with the least taxa. Usually chosen as it removes the influence of nested sites, so is a better measure of turnover than the Sorensen distance.

The distances between sites can then be put into a clustering algorithm such as:

* **K-means** - minimises the within cluster distances by:

    1. Pick random data points as the starting centroids for *K* clusters.
    2. Calculate the distance of all points to each centroid, and assign points to closest one.
    3. Move the centroids to the mean position within their cluster.
    4. Repeat steps 2 and 3 until there's no significant reduction in within cluster distance.

    Not usually used (not sure I found an example of it) because it needs distances to be Euclidean, 
    which the beta-diversity metrics aren't. Also probably need to ordinate sites first so the centroids can be assigned. Also need to choose the best number of clusters (either arbitrarily or with some quantitative metric).

* **Hierarchical clustering** - usually some sort of agglomerative clustering like UPGMA which constructs a tree linking all sites by:

    1. Starts by linking the two sites with the smallest distance. The branch length to the node linking them is half the distance between them.
    2. Updates the distance matrix by calculating the average distance of the sites in the first cluster to all other sites.
    3. Links the closest sites/clusters together. Could be adding a site to the first cluster or adding two sites together to make a new cluster.
    4. Update the distance matrix again, if a new site has been added to the cluster use proportional averaging to get the distance between the cluster and other sites.
    5. Repeat steps 3 and 4 until all sites are linked by the tree.

    If this is confusing the [Wikipedia site has a worked explanation](https://en.wikipedia.org/wiki/UPGMA#Working_example).

    This is much more commonly used. It has the advantage of working with whatever distance metric you use. Also has a hierarchical structure, which is something that bioregions are apparently meant to have. But you still need to decide how many clusters you want.

* **Affinity propogation** - sends messages between points to determine attractiveness between points, and the responsibility of each point to act as an examplar for a cluster. Could put in an explanation for the algorithm but its not particularly interesting. [This is a good step through it though](https://towardsdatascience.com/unsupervised-machine-learning-affinity-propagation-algorithm-explained-d1fef85f22c8). 

  Not used as much as hierarchical clustering, but has the advantage of determining the optimal number of clusters for you.

There are various other clustering algorithms that could be used, but aren't really. UPGMA tends to be used the most, and has the advantage of being hierarchical. [This is a good comparison of different distance based clustering algorithms and when to use them](https://scikit-learn.org/stable/modules/clustering.html)

**Advantages:**

  * Based on well known and widely used metrics
  * Can be extended fairly easily to take phylogenetics into account
  * Wide variety of easy to implement and use clustering algorithms, with well understood upsides and downsides

**Disadvantages:**

  * beta-diversity naturally increases with geographical distance from a point, so hard to decouple geographical distance and differences in species assemblages.
  * Can fluctuate significantly at small scales and so overestimate differences, due to competitive exclusion, spatial clustering, and environmental gradients.
  * Heavily influenced by sampling of taxa.

**Papers:**

  * [Kreft and Jetz 2010](https://onlinelibrary.wiley.com/doi/full/10.1111/j.1365-2699.2010.02375.x) provide their view of a systematic workflow for bioregionalisation. They use Simpson distance and UPGMA clustering, but they have a nice review of other peoples attempts listing what type of metric and clustering they used. This is a really informative overview paper.
  * [Heikinheimo, Fortelius, and Mannila 2007](https://onlinelibrary.wiley.com/doi/10.1111/j.1365-2699.2006.01664.x), listed in the Kreft and Jetz paper, use k-means! But it looks like they do it directly on the species presence/absence matrix, which I can't say I recommend - too many dimensions and yes/no data aren't that great for k-means, it would be better to do NMDS ordination using a beta-diversity metric and then transform that so the distances are Euclidean and then do the k-means. But it appears to have worked for them.
  * [Leprieur and Oikonomou 2013](https://doi.org/10.1111/jbi.12258) is a correspondence about how it's important to use a diversity metric that is decoupled from species richness, like the Simpson diversity.
  * [Holt et al. 2013](https://science.sciencemag.org/content/339/6115/74) updated Wallace's zoogeographical regions for the world using a phylogenetic beta-diversity metric (from their description of it, it sounds like PhyloSor) and UPGMA clustering. Lots of details in the supplementary materials.

## Network based clustering

Network-based clustering first puts the species and sites into a bipartite graph. The species and sites form two sets of nodes with no direct edges between nodes of the same site - no site-to-site edges and no species-to-species edges. So sites nodes are linked through the species that they share.

This has some nice properties, for instance if you take the square of the network's adjacency matrix you get the species co-occurrence matrix and a matrix with the number of shared species between sites.

But for clustering the main method that has been used is the Infomap method that uses [the map equation](https://www.mapequation.org/assets/publications/EurPhysJ2010Rosvall.pdf).

The map equation is a measure of the amount of information needed to encode flow on a network given a partition of the network into modules (clusters). Flow is modelled by a random walker that moves between linked nodes, and the map equation is a sum that balances the complexity of the model (the information needed to encode movements _between_ modules) with the complexity of the data given the model (the information needed to encode movements _within_ modules).

Infomap is an algorithm that searches for the modules that minimises the map equation. These modules capture areas where the random walker tend to spend a relatively long time before leaving. In this case, it means areas with lots of shared species.

**Advantages:**

  * Not coupled to geographical distance
  * Gives more weight to species that only occur at one or a few sites
  * Aligns closely with opinion-based bioregions
  * [Nice web app](https://bioregions.mapequation.org/)

**Disadvantages:**

  * No obvious way to incorporate phylogenetics
  * I think this is still influenced by sampling of taxa? Although maybe not as heavily as distance based methods?

**Papers:**

  * [Vilhena and Antonelli 2015](https://www.nature.com/articles/ncomms7848) gives an overview of the Infomap method with a discussion of its advantages.
  * [Edler et al. 2017](https://academic.oup.com/sysbio/article/66/2/197/2670349) more in-depth demonstration of Infomap, for global amphibians and mammals.
  * [Colli-Silva et al. 2019](https://onlinelibrary.wiley.com/doi/full/10.1111/jbi.13585) justifying the delimitation of campo rupestre in Eastern Brazil into two sections.
  * [Droissart et al. 2018](https://onlinelibrary.wiley.com/doi/full/10.1111/jbi.13190) regionalisation of tropical Africa using RAINBIO data.
  * [Leroy et al. 2019](https://doi.org/10.1111/jbi.13674) use Infomap to get global biogeographic regions for freshwater fish.
  

# Why bother?

Most of the reason people seem to be doing this now is for the uses of the bioregions that you get out, rather than for the actual bioregions themselves. But other people are also raising some issues with the methods used and also the general approach.

## Some uses

* Modelling the drivers of their formation
  * [Ficetola 2017](https://www.nature.com/articles/s41559-017-0089), used an already published set of regions and modelled probability of a cell belonging to each region to test the influence of climate and geography on their delimitation.
  * [Yusefi et al](https://doi.org/10.1111/jbi.13694) do some bioregionalisation for mammals in Iran using distance and network based clustering, and then model bioregion membership using climatic variables.
* Conservation
  * [Olsen 2001](https://academic.oup.com/bioscience/article/51/11/933/227116), ecoregions gap analysis of how much of each ecoregion/biome is covered by protected areas.
  * [Bernardo-Madrid 2019](https://onlinelibrary.wiley.com/doi/abs/10.1111/ele.13321), how biogeographical regions change with extinction and invasion. Looking at how the regions become more/less homogenised.
  * [Yusefi et al](https://doi.org/10.1111/jbi.13694) also do a gap analysis of how much each of their bioregions is covered by protected areas.
* Evaluating niche conservatism
  * [Crisp 2009](https://www.nature.com/articles/nature07764), assigned species to different biomes and evaluated whether evolutionary divergences were associated with transitions between biomes.
* Ancestral state reconstruction
  * [McDonald-Spicer et al. 2018](https://doi.org/10.1111/jbi.13496) use distance-based clustering and network-based clustering, but used modularity to cluster their network rather than Infomap. Use their bioregions to infer the ancestral area of Asteraceae.
* Affect of climate change on different regions

## Some problems/questions

* Are they natural?
  * [Ebach and Parenti 2015](https://onlinelibrary.wiley.com/doi/full/10.1111/jbi.12558) propose monophyly as a criteria for judging the naturalness of a region.
* Is corresponding with opinion-based regions a good thing?
* What taxonomic level should they be done at? Wallace said genus-level.
  * [Rueda et al. 2013](https://doi.org/10.1111/jbi.12214) argue that we should only be going down to genus-level.
* How do we incorporate phylogeny? Should we even incorporate phylogeny?
  * [Holt et al. 2013](https://science.sciencemag.org/content/339/6115/74) use a phylogenetic distance measure for their clustering.
  * [Vilhena and Antonelli 2015](https://doi.org/10.1038/ncomms7848) make the point that depending on the end use, using a phylogenetic measure could be circular.
* Is data quality/coverage important? How important is it?
* Could species distribution modelling help?
* How do you incorporate an indication of support/uncertainty for a particular regionalisation?


