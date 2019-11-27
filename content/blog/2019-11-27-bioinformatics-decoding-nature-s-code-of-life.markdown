---
title: 'Bioinformatics: Decoding Nature''s Code of Life'
author: Nabiilah Ardini Fauziyyah
github: https://github.com/NabiilahArdini
date: '2019-11-27'
slug: biocode
categories:
  - R
tags:
  - Bioinformatics
  - Bioconductor
  - msa
  - ape
description: ''
featured: ''
featuredalt: ''
featuredpath: ''
linktitle: ''
type: post
---


It is inarguable that Data Science gives a tremendous impact on today's industry. Furthermore, it also accelerates the development of basic science research including *Biology*. Biology harbors some of the most intriguing ideas we may find today; from finding cures for genetic diseases, to something far as breeding mutants! This article will guide you through the wondrous journey of when Data Science meets Biology and how it can impact our life. 

<img src="/img/biointro-venn.png" style="display: block; margin: auto;" />

The specialized Data Science area which combines *the use of computer science and statistics and the knowledge of Biology to understand biological data* is called **Bioinformatics**. Bioinformatics has given the world so much knowledge and improvement without us knowing. Having only a few articles that combine both the scientific and practical side of this topic, making it somewhat unexposed clearly to the public. Therefore, throughout this article we will be talking about:

* Misconception around Bioinformatics
* Biological Data for Bioinformatics
* Databases & Tools in R
* Case Study: Bionanocellulose for Future Packaging
* Future Prospect of Bioinformatics

## Misconception around Bioinformatics

Bioinformatics is originally a subtopic in Biology. Like biology, it has the main purpose to *understand living things and how it functions*. If we compare it with the more familiar business-related topic, which aims to optimize business practices through data utilization, it has a very little difference. Bioinformatics aims to improve the efficiency and effectivity of biological research through insights derived from **biological data**.

Many of the misconceptions revolved around Bioinformatics is related to the scope and the data it uses. For example, many of the cases most people considered as bioinformatics may come from the healthcare industry which utilizes various measurement data related to health. Some of them are predicting the status of [breast cancer](https://archive.ics.uci.edu/ml/datasets/Heart+Disease) from a digitized cell image data, diagnosing the status of [heart diseases](https://archive.ics.uci.edu/ml/datasets/Heart+Disease) from health records and laboratory test result, etc. Still, both cases use *normal tabular data* for analysis, which therefore more appropriately reported as a Data Science application in "healthcare", and *not yet* considered as Bioinformatics.  

<img src="/img/biointro-heart.png" style="display: block; margin: auto;" />

The data you encounter from these cases is a normal measurement data that may come from many specialized areas of Biology, such as health & medicine, agriculture, environment, bioenergy, biomaterials, etc. Deep knowledge about that respective field is required in order to understand, interpret, and utilize the data appropriately. Meanwhile, the biological data that is used in bioinformatics is rather different. 

Biological data that is used in bioinformatics is probably very new to you, certainly more challenging to interpret, but also more interesting to work with. When we deal with this biological data we are actually dealing with the code that resides inside ourselves; the code that encodes a living being. This *code* is very personal and unique for every living being, including you! This data is the nature's code of life, **DNA**.

With this nature's code of life, we may explore a deeper area of Biology which focuses not only to **predict** the class/statuses of disease but to **discover** what genes (code) that encodes the disease and find a way to correct them. In fact, many application of Bioinformatics helps the *discovery of potential natural resources for industrial application*, like what we will do in the later section, about finding a new material that is biodegradable enough to replace plastic for our future packaging--*talking about green living!* ;)

## Biological Data for Bioinformatics

All living things are composed of cells. A cell is the basic unit of life. A single cell can regulate itself to keep living; to eat, to interact with other cells, to defend itself from danger, to repair itself, and reproduce. For example, is a single cell bacteria named *Salmonella* sp. that may cause you diarrhea. In complex organisms such as plants, animals, and humans, a single organism consist of various types and number of cells. In humans alone, it consists of approximately 40 trillion cells. These cells arranged together forming an organized structure with very complex yet neat mechanisms, making our body function almost perfectly every single day. 

Living things may be said as the most complex and magnificent molecular machine ever happened. Scientist passionately performs research to understand what makes living things function and what makes them malfunctions (ill/diseased). Through many years of research, the answer is finally proposed in 1957 and known as **The Central Dogma of Biology**.  

<img src="/img/biointro-centraldogma.png" style="display: block; margin: auto;" />

> The ‘Central Dogma’ describes that the appearance and functional ability of all living things comes from the instructions encoded in their DNA.

<img src="/img/biointro-dna.gif" style="display: block; margin: auto;" />

That above is **DNA**. It resides within the cell of each living organism. DNA is a strand with a double helix structure that contains the genetic code of an organism. This genetic code works like instructions that tell the cell what **protein** it should make. A special region in DNA that store codes for each specific protein is called a **gene**. Therefore each protein has its specific gene. The genetic code in DNA is written as a code of 4 characters, representing the 4 types of nucleotide (the building block of DNA) called adenine ( A ), cytosine ( C ), guanine ( G ), thymine ( T ). Every time a protein needs to be made, the gene must be *transcribed* first into its single strand structure called messenger RNA (mRNA), and then *translated* into protein. Unlike DNA which only has 4 character codes, proteins have 20 character codes representing each amino acid as the building block to making protein. The cryptogram to translate mRNA into protein is presented below.

<img src="/img/biointro-crypto.png" style="display: block; margin: auto;" />

The correct genetic codes and smooth translation from DNA to protein are important so that every cell in our body has enough protein that it needs to functions properly. For example, red blood cells need hemoglobin, a functional protein that binds oxygen and helps red blood cells carry oxygen throughout the body. Another example is collagen, a structural protein which promotes skin cell's elasticity. Another type of protein also exists such as hormones, enzymes, antibodies, etc. A failure in providing the correct protein because of some *error code in the DNA* may cause serious diseases such as Alzheimer's, Sickle Cell Anemia, Down syndrome, etc.

The codes in DNA also determines an organisms appearance. Some of you might have white skins while other have exotic skin, some of you may have deep black eyes or rather more brownish, curly hair or straight, etc. Not just humans, plants and animals have it too. Have you ever seen various breeds of cats? How can Persians have thick long hair compared to Siameses which has short hair but with black pointed areas in their faces, ears, feet, and tail? How can there be a red-colored chili and the green-colored one? All of these characteristics is already encoded in their DNA. 

> "Hey, if we can decode the code of life, is it possible to make a costum organisms? Can we benefit from that?

After knowing the relation between DNA and the measured characteristic traits (phenotype) of an organism, scientists began to do research using both DNA and phenotype data to better understand living things and its mechanism (from genetic codes to its outcomes). This practice is what we called as bioinformatics analysis. The data were stored in a database and analyzed using statistical methods with the help of computers, mainly because they are extremely large in size. A full DNA of a single cell bacteria can take about over 14 mega base pairs (14000000 character codes) while a human can have around 3 billion DNA base pairs, and we haven't even combined them with the phenotype data. Biological databases are now expanding at a very high rate. More software and methods are also being developed to help analyze biological data. It is not an exaggeration to say that Bioinformatics is rapidly evolving and becoming one of the hottest topics in Biology today.

## Databases & Tools in R

Research on Bioinformatics has produced numerous amounts of data and is right now being stored and maintained in Biological Databases. The databases store different types of biological data. These are the most used databases for sequence data (genetic codes) today:

* GenBank from [NCBI](www.ncbi.nlm.nih.gov)

GenBank is an open-access sequence database, containing an annotated collection of all publicly available nucleotide sequences and their protein translations. This database is maintained by the National Center for Biotechnology Information (NCBI), a part of the National Institutes of Health in the United States. 

* [ENA](www.ebi.ac.uk/ena)

The European Nucleotide Archive (ENA) is a repository providing free and unrestricted access to annotated DNA and RNA sequences. The ENA is maintained by the European Bioinformatics Institute.

* [DDBJ](www.ddbj.nig.ac.jp)

The DNA Data Bank of Japan (DDBJ) is a biological database that collects DNA sequences. It is located at the National Institute of Genetics (NIG) in the Shizuoka prefecture, Japan. 

A joint effort to collect and disseminate databases containing DNA and RNA sequences between these three databases have been conducted through **The International Nucleotide Sequence Database Collaboration** (INSDC). It exchanges data on a daily basis thus these three databases contain the same data at any given time. Other databases related to protein structure and function are also available, such as [UNIPROT](www.uniprot.org) and [PDB](https://www.rcsb.org/). Data related to the metabolic pathway is also available in [KEGG](genome.jp/kegg/) database.

Meanwhile, there are also various tools and software used for Bioinformatics, depending on the case we are trying to solve. In R, Bioinformaticians have their own repository to search for packages related to bioinformatics analysis called [Bioconductor](https://bioconductor.org/). The installation of packages from Bioconductor is slightly different from the installation of packages from CRAN. Inside, there are approximately 1800++ packages for scientists to perform their bioinformatics analysis.

<img src="/img/biointro-site.png" style="display: block; margin: auto;" />

<br>

## Case Study: Biocellulose for Future Packaging

The value of biological research rested on the main idea and purpose of research. Those things will help scientists to design specific **workflow** that may guide them throughout many trials and errors. The following is an example of how Bioinformatics can solve a biological case, that is to *discover potential natural resources for industrial application*. 

It is 2019 already and many people are shifting towards a more eco-friendly way of living. One of them is to reduce the use of plastic. Starting from replacing the use of plastic straws to stainless steel, the use of reusable tumbler and lunchboxes, etc. Conversation about 'green packaging' has also been IN for the past few years. While plastic is dominantly being used for packaging, scientists are finding a way to replace plastic with a more friendly biomaterial which is both biodegradable and versatile for the packaging industry. Through years of research, they finally found it and it's called **Biocellulose**. 

<img src="/img/biointro-bc.jpg" style="display: block; margin: auto;" />

Bacterial cellulose or biocellulose is a nanomaterial produced by some bacteria. Biocellulose harbors some [interesting properties](https://www.prescouter.com/2018/02/nanocellulose-applications-packaging/) that makes it a potential candidate to replace plastic:

* edible
* flexible
* antimicrobial
* biodegradable
* strong and has barrier properties against oxygen and water

[Biocellulose](https://www.researchgate.net/publication/331174516_Bacterial_Cellulose_as_a_Raw_Material_for_Food_and_Food_Packaging_Applications) can be produced from various bacterial genus (*Acetobacter, Agrobacterium, Azotobacter, Komagataeibacter, Pseudomonas, Rhizobium, Sarcina*, etc.). However, the major producers of bacterial cellulose belong to the *Komagataeibacter* genus due to its higher BC yield and purity[^1]. For example, *Komagataeibacter* sp. strain SFCB 22-18 that has been [pattened](https://patents.google.com/patent/KR20140130569A/en?q=Komagataeibacter&oq=Komagataeibacter) as a biocelluloce producer in Korea and can be analyzed further for industrial-scale production. 

Knowing this fact, indonesian's packaging companies may also want to apply such technology in their packaging. Unfortunately, a bacteria that is originated from a four season region may not be fully compatible to grow in a warmer/tropical region. This is due to the different temperature, humidity, nutrition availability, etc. that affect bacterial growth. Such bacteria may need special treatments thus result in higher cost. Can we find our own biocellulose bacteria and produce our own green packaging? 

**Well, of course!** 

Let's discover local biocelullose producing bacteria using data science! We can do that by analyzing the similarity of DNA between the known biocellulose producing bacteria and our local potential bacteria using a method called **Multiple Sequence Alignment** (MSA) and **phylogenetic analysis**. 

MSA is one of the most fundamental tasks in bioinformatics. MSA is a sequence alignment of three or more biological sequences (protein, DNA, or RNA) to detect the sequence similarity between each sample which can asses for *the detection of similar characteristics*. Each alignment were calculated for its score (from a match/mismatch from aligned genetic codes). An optimal alignment was picked and later can be processed to form a phylogenetic tree. Algorithms like ClustalW, ClustalOmega, and MUSCLE are well known and widely used for MSA. 

After MSA, we can perform **phylogenetic analysis**. This analysis will perform hierarchical clustering to visualize sequence similarity from MSA in the form of phylogenetic tree (dendrogram). *Neighbor-Joining* and *UPGMA* algorithm is widely used for this analysis. The three will reveal which samples are similar to one another and which samples are different. 

The workflow for this bioinformatics analysis is below:

1. Sample local bacteria from potential places
2. Obtain DNA sequence from potential bacteria & widely known bacteria
3. Find the genetic relationship between local and widely known bacteria through MSA and phylogenetic analysis
4. Determine the potential local bacteria for biocellulose production and further development.

Imagine a young scientist in Indonesia who has already sampled a few of the potential bacteria from [Kombucha's SCOBY](https://www.dezeen.com/2018/11/13/sustainable-food-packaging-emma-sicher-peel/) and now is asking for your help. He gave you the DNA sequence from the potential bacteria he obtained and you were asked to help determine which bacteria have the highest potential to produce biocellulose, so that the young scientist can focus more to analyze that particular bacteria. These are the steps you can do to solve that problem:

### Install Packages

We will be using specific libraries for certain purposes:

* `msa`: for MSA 
* `seqinr` and `ape`: for phylogenetic analysis. 

`msa` package is stored in Bioconductor repository, therefore we need to install it first with a different syntax:


```r
# if (!requireNamespace("BiocManager", quietly=TRUE))
# install.packages("BiocManager")
# BiocManager::install("msa")
```

`BiocManager::install()` is a recommended way to install Bioconductor packages instead of using standard way in which R packages are installed via `install.packages()`. There are good reasons for that which you can read further in [here](https://bioconductor.org/install/#why-biocmanagerinstall).

### Read Data

DNA sequences data is stored in a FASTA format, a text-based format for representing either nucleotide sequences or amino acid (protein) sequences. The nucleotides or amino acids are represented using single-letter codes. Sequence names and comments can also be added after `>` one line before the sequence.
 
This below is an example:

<img src="/img/biointro-fasta.png" style="display: block; margin: auto;" />

For this article, I have prepared a dummy data containing real DNA sequence data from GenBank, and some dummy DNA sequence for analysis. This data stored in a fasta format and named `bioc.fasta`. To read the data we can use `readDNAStringSet()` function from Biostrings package that already loaded as the dependencies for msa package.


```r
library(msa)
myseq <- readDNAStringSet("data_input/bioc.fasta")
myseq
```

```
#>   A DNAStringSet instance of length 18
#>      width seq                                              names               
#>  [1]  1481 GAGTTTCATCCTGGCTCAGAGCG...CTGCGGCTGGATCACCTCCTTT Komagataeibacter ...
#>  [2]  1352 GTGGCGGACGGGTGAGTAACGCG...CACGGTCGGGTCAGCGACTGGG Komagataeibacter ...
#>  [3]  1386 GCAGTCGCACGAACCTTTCGGGG...GGTCAGCGACTGGGGTGAAGTC Komagataeibacter ...
#>  [4]  1390 ATCCTGGCTCAGAGCGAACGCTG...CGGTGAGCGAACCGCAAGGACG Komagataeibacter ...
#>  [5]  1482 GAGTTTGATCATGGCTCAGAGCG...CTGCGGCTGGATCACCTCCTTT Komagataeibacter ...
#>  ...   ... ...
#> [14]  1285 CGTGGGAATCTACCCTTTTCTAC...TACCCGAAGGTAGTGCGCTAAC Agrobacterium rhi...
#> [15]  1493 AGAGTTTGATCCTGGCTCAGAAT...CTGCGGCTGGATCACCTCCTTT Acidobacterium ca...
#> [16]  1481 GAGTTTCATCCTGGCTCAGAGCG...CTGCGGCTGGATCACCTCCTTT Sample1
#> [17]  1446 TGAGTTTGATCCTGGCTCAGAGC...GGGGTGAAGTCGTAACAAGGTA Sample2
#> [18]  1481 GAGTTTGATTATGGCTCAGAGCG...CTGCGGCTGGATCACCTCCTTT Sample3
```

The summary above shows some information:

* DNAStringSet instances length shows how many DNA sequence we have 
* width: the length of each DNA sequence
* seq: the DNA sequence
* names: the label for each DNA sequence


```r
names(myseq)
```

```
#>  [1] "Komagataeibacter xylinus strain NCIB 11664"
#>  [2] "Komagataeibacter xylinus strain LMG 1515"  
#>  [3] "Komagataeibacter xylinus strain KTH 5655"  
#>  [4] "Komagataeibacter xylinus strain ZHCJ618 "  
#>  [5] "Komagataeibacter europaeus"                
#>  [6] "Komagataeibacter intermedius"              
#>  [7] "Komagataeibacter medellinensis"            
#>  [8] "Komagataeibacter swingsii"                 
#>  [9] "Komagataeibacter oboediens"                
#> [10] "Komagataeibacter saccharivorans"           
#> [11] "Komagataeibacter kakiaceti"                
#> [12] "Agrobacterium rhizogenes LMG 9509"         
#> [13] "Agrobacterium rhizogenes LMG 152"          
#> [14] "Agrobacterium rhizogenes strain G4.1.1"    
#> [15] "Acidobacterium capsulatum ATCC 51196"      
#> [16] "Sample1"                                   
#> [17] "Sample2"                                   
#> [18] "Sample3"
```

using `names()` function we can find out the labels from our DNAStringSet data. This data contain 18 DNA sequences containing *16S rRNA gene* that is widely used for sequence similarity analysis. 4 of them are from *Komagataeibacter xylinus* (with various strains) that are being widely developed for biocellulose industrial production. The last 3 unidentified sample is the DNA of our local potential superbugs. The other DNA sample act as comparations to analyze whether our sample resides within the same cluster of *Komagataeibacter xylinus* (meaning that it may also produce high yield of biocellulose) or not.

### Perform MSA

To perform multiple sequence alignment, we can use `msa()` function. The `msa()` function will perform MSA on our DNAStringSet with the default substitution matrix with ClustalW algorithms. 


```r
bc_msa <- msa(myseq)
```

```
#> use default substitution matrix
```

```r
bc_msa
```

```
#> CLUSTAL 2.1  
#> 
#> Call:
#>    msa(myseq)
#> 
#> MsaDNAMultipleAlignment with 18 rows and 1509 columns
#>      aln                                                   names
#>  [1] -------------------------...AANN--------------------- Agrobacterium rhi...
#>  [2] -------------------------...------------------------- Agrobacterium rhi...
#>  [3] -------------------------...AACC--------------------- Agrobacterium rhi...
#>  [4] -----------------ATCCTGGC...------------------------- Komagataeibacter ...
#>  [5] ---------TGAGTTTGATCCTGGC...------------------------- Sample2
#>  [6] ----------GAGTTTGATCATGGC...AACCTGCGGCTGGATCACCTCCTTT Komagataeibacter ...
#>  [7] ----------GAGTTTGATCMTGGC...AACCTGCGGCTGGATCACCTCCTTT Komagataeibacter ...
#>  [8] ---------TGAGTTTGATCCTGGC...------------------------- Komagataeibacter ...
#>  [9] ATGAACCTGAGAGTTTGATCCTGGC...AACCTGCGGCTGGATCACCTCCTTT Komagataeibacter ...
#> [10] ----------GAGTTTCATCCTGGC...AACCTGCGGCTGGATCACCTCCTTT Komagataeibacter ...
#> [11] ----------GAGTTTCATCCTGGC...AACCTGCGGCTGGATCACCTCCTTT Sample1
#> [12] -------------------------...------------------------- Komagataeibacter ...
#> [13] -------------------------...------------------------- Komagataeibacter ...
#> [14] -------------------------...AACCTGCGGC--------------- Komagataeibacter ...
#> [15] ---------AGAGTTTGATCCTGGC...AACCTGCGGCTGGATCACCTCCTT- Komagataeibacter ...
#> [16] -------------------------...AACCTGC------------------ Komagataeibacter ...
#> [17] ----------GAGTTTGATTATGGC...AACCTGCGGCTGGATCACCTCCTTT Sample3
#> [18] ---------AGAGTTTGATCCTGGC...AACCTGCGGCTGGATCACCTCCTTT Acidobacterium ca...
#>  Con ----------GAGTTT?ATC?TGGC...AACCTGC???--------------- Consensus
```

The `msa()` function will return an `MSA DNA multiple alignments` which contains the DNA aligned sequences (notice the matched/similar codes for specific regions) and a consensus sequence, the most frequent residues found at each position in a sequence alignment. Below is the more beautiful visualization of MSA using BioEdit software that I used often back in college. With R, we can perform it just using a few line of code!

<img src="/img/biointro-msa.png" style="display: block; margin: auto;" />

### Phylogenetic Analysis

We can perform phylogenetic analysis to utilize and visualize better our MSA result. First, we need to convert `msa` object into an `alignment` object. After that, we can compute the distance matrix (the similarity/identity score between each of the DNA samples) using the `dist.alignment()` from the `seqinr` package.


```r
# convert msa to alignment file
bc_align <- msaConvert(bc_msa, type="seqinr::alignment")

library(seqinr)
#getting identity score (match) 
bc_ident <- dist.alignment(bc_align, "identity")
```

After we have the distance matrix object, we can use the object to build a phylogenetic tree. Below is an example in using the [Neighbor-Joining Algorithm](https://en.wikipedia.org/wiki/Neighbor_joining) that is widely used for phylogenetic analysis from `ape` package. The Neighbor-Joining (NJ) algorithm allows for unequal rates of evolution for each organism thus resemble what actually happened in real life. Because NJ algorithm allows unequal rates of evolution, the branch lengths are proportional to the amount of change for each organism.


```r
library(ape)
# making phylogenetic tree using identity score
bcTree <- nj(bc_ident)
plot(bcTree, main="Phylogenetic Tree of Biocellulose Producing Bacteria")
```

<img src="/blog/2019-11-27-bioinformatics-decoding-nature-s-code-of-life_files/figure-html/unnamed-chunk-15-1.png" width="960" style="display: block; margin: auto;" />

We have known that the major producers of bacterial cellulose belong to the *Komagataeibacter* genus due to its higher biocelllose yield and purity. From the phylogenetic tree above, we know that sample 1 and sample 2 is within the cluster with many *Komagataeibacter xylinus* species. Sample 1 is especially closely related to *K. xylinus* strain NCIB 1166. Meanwhile, sample 3 did not included in any of the cluster containing Komagataeibacter bacteria and therefore might have a lower chance to produce high yield and high purity biocellulose. 

From these findings, it is beter to focus on Sample 1 and 2 to for the development of biocellulose producing bacteria for future packaging in Indonesia.

## Future Prospect of Bioinformatics

The future of Bioinformatics is very promising. People nowadays are more conscious about healthy and wiser living. They care about their health, their food, and where it comes from. Sustainability issues are getting more recognition, promoting the collaboration between Biology and industrial applications. From an article shared by [AgFunderNews](https://agfundernews.com/25-startups-innovating-with-sustainable-packaging-solutions.html), there are already 25 startups from around the world that are focusing on sustainable packaging solution. One of them is [Evoware](https://www.instagram.com/evowareworld/?hl=en), a start-up from Indonesia that develops edible packaging from seaweed and algae.

There are many more things we can do in the future and the case example above is only a tiny portion of what Bioinformatics can do. The widely known Bioinformatics application in our life includes: 

* Development of personalized diet & medicine
* Drug design & discovery
* Gene therapy
* Discovery and development of novel materials
* Selective Breeding for crops/livestock improvement
* Epidemiology analysis
* Evolutionary analysis, etc.  

Once we understand the code of life, the possibility of what we can do is almost endless. With the advancement of technology, the only limit is probably our own imagination. Just make sure that what we do is for the good of the people, the living things, and our earth.

## References

[^1]: De Azeredo, H.M.C., Barud, H.D., Farinas, C.S. 2019. Bacterial Cellulose as a Raw Material for Food and Food Packaging Applications. *Frontiers in Sustainable Food Systems*, Vol. 3(7): 1-5
