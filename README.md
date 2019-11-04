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

MAKE A LIST OF OLD BIOREGIONS STUFF. ECOREGIONS?

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

* K-means - minimises the within cluster distances by:

    1. Pick random data points as the starting centroids for *K* clusters.
    2. Calculate the distance of all points to each centroid, and assign points to closest one.
    3. Move the centroids to the mean position within their cluster.
    4. Repeat steps 2 and 3 until there's no significant reduction in within cluster distance.

    Not usually used (not sure I found an example of it) because it needs distances to be Euclidean, 
    which the beta-diversity metrics aren't. Also probably need to ordinate sites first so the centroids can be assigned. Also need to choose the best number of clusters (either arbitrarily or with some quantitative metric).

* Hierarchical clustering - usually some sort of agglomerative clustering like UPGMA which constructs a tree linking all sites by:

    1. Starts by linking the two sites with the smallest distance. The branch length to the node linking them is half the distance between them.
    2. Updates the distance matrix by calculating the average distance of the sites in the first cluster to all other sites.
    3. Links the closest sites/clusters together. Could be adding a site to the first cluster or adding two sites together to make a new cluster.
    4. Update the distance matrix again, if a new site has been added to the cluster use proportional averaging to get the distance between the cluster and other sites.
    5. Repeat steps 3 and 4 until all sites are linked by the tree.

    If this is confusing the [Wikipedia site has a worked explanation](https://en.wikipedia.org/wiki/UPGMA#Working_example).

    This is much more commonly used. It has the advantage of working with whatever distance metric you use. Also has a hierarchical structure, which is something that bioregions are apparently meant to have. But you still need to decide how many clusters you want.

    * [Kreft and Jetz 2010](https://onlinelibrary.wiley.com/doi/full/10.1111/j.1365-2699.2010.02375.x) uses UPGMA, fair amount of regionalisation done since sites this method.

* Affinity propogation - sends messages between points to determine attractiveness between points, and the responsibility of each point to act as an examplar for a cluster. Could put in an explanation for the algorithm but its not particularly interesting. [This is a good step through it though](https://towardsdatascience.com/unsupervised-machine-learning-affinity-propagation-algorithm-explained-d1fef85f22c8). 

    Not used as much as hierarchical clustering, but has the advantage of determining the optimal number of clusters for you.

There are various other clustering algorithms that could be used, but aren't really. UPGMA tends to be used the most, and has the advantage of being hierarchical. [This is a good comparison of different distance based clustering algorithms and when to use them](https://scikit-learn.org/stable/modules/clustering.html)

Advantages of distance-based regionalisation:

  * Based on well known and widely used metrics
  * Can be extended fairly easily to take phylogenetics into account
  * Wide variety of easy to implement and use clustering algorithms, with well understood upsides and downsides

Disadvantages:

  * beta-diversity naturally increases with geographical distance from a point, so hard to decouple geographical distance and differences in species assemblages.
  * Can fluctuate significantly at small scales and so overestimate differences, due to competitive exclusion, spatial clustering, and environmental gradients.
  * Heavily influenced by sampling of taxa.

## Network based clustering

Network-based clustering first puts the species and sites into a bipartite graph. The species and sites form two sets of nodes with no direct edges between nodes of the same site - no site-to-site edges and no species-to-species edges. So sites nodes are linked through the species that they share.

This has some nice properties, for instance if you take the square of the network's adjacency matrix you get the species co-occurrence matrix and a matrix with number number of shared species between sites.

But for clustering the main method that has been used is the Infomaps method that uses [the map equation](https://www.mapequation.org/assets/publications/EurPhysJ2010Rosvall.pdf).

The map equation is a measure of how optimally a network is encoded. By grouping the nodes in a network into modules (clusters), you can decrease the detail needed to encode the nodes within each module. However, as you increase the number of modules you need a more complex code to describe moving between the modules. So it is a balance between the within-module and between-module complexity.

The Infomap algorithm is a fast way of sampling how much time you would spend in each area of a network if you were taking a random walk through it. Areas where you'd spend a lot of time are areas where the sites have lots of shared species (lots of links). These are grouped in modules (clusters), as a way of minimising the map equation.

Advantages:

  * Not coupled to geographical distance so much?
  * Gives more weight to species that only occur at one or a few sites
  * Aligns closely with opinion-based bioregions
  * [Nice web app](https://bioregions.mapequation.org/)

Disadvantages:

  * No obvious way to incorporate phylogenetics
  * I think this is still influenced by sampling of taxa? Although maybe not as heavily as distance based methods?

Used in:
  * [Colli-Silva et al. 2019](https://onlinelibrary.wiley.com/doi/full/10.1111/jbi.13585) justifying the delimitation of campo rupestre in Eastern Brazil into two sections.
  * [Droissart et al. 2018](https://onlinelibrary.wiley.com/doi/full/10.1111/jbi.13190) regionalisation of tropical Africa using RAINBIO data.
  * [Elder et al. 2017](https://academic.oup.com/sysbio/article/66/2/197/2670349) demonstrating Infomaps for global amphibians and mammals.

# Why bother?

Most of the reason people seem to be doing this now is for the uses of the bioregions that you get out, rather than for the actual bioregions themselves. But other people are also raising some issues with the methods used and also the general approach.

## Some uses

* Modelling the drivers of their own formation
  * [Ficetola 2017](https://www.nature.com/articles/s41559-017-0089), used an already published set of regions and modelled probability of a cell belonging to each region to test the influence of climate and geography on their delimitation.
* Conservation
  * [Olsen 2001](https://academic.oup.com/bioscience/article/51/11/933/227116), ecoregions gap analysis of how much of each ecoregion/biome is covered by protected areas.
  * [Bernardo-Madrid 2019](https://onlinelibrary.wiley.com/doi/abs/10.1111/ele.13321), how biogeographical regions change with extinction and invasion. Homogenisation.
* Evaluating niche conservatism
  * [Crisp 2009](https://www.nature.com/articles/nature07764), assigned species to different biomes and evaluated whether evolutionary divergences were associated with transitions between biomes.
* Ancestral state reconstruction
* Affect of climate change on different regions

## Some problems/questions

* Are they natural?
* Is corresponding with opinion-based regions a good thing?
* What taxonomic level should they be done at? Wallace said genus-level.
* How do we incorporate phylogeny? Should we even incorporate phylogeny?
* Is data quality/coverage important? How important is it?
* Could species distribution modelling help?


