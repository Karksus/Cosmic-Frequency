COSMIC FREQUENCY
================
Pedro Serio
2023-03-09

In this tutorial we will learn how to get tissue-specific mutated sample
frequency with data from COSMIC, as explained in their portal
(<https://cancer.sanger.ac.uk/cosmic/help/faq>).

# Firt steps:

1 - Go to the COSMIC download page
(<https://cancer.sanger.ac.uk/cosmic/download>);  
2 - Make sure that the portal is showing your desired Human Genome
Version (Genome Version button at the top of the page);

Obs: You will need to log in (or create an account) your COSMIC account
to be able do download data.

3 - Choose your desired tissue (e.g. breast cancer) and download the
following files:

-CosmicCompleteTargetedScreensMutantExport.tsv.gz;  
-CosmicGenomeScreensMutantExport.tsv.gz;  
-CosmicSample.tsv.gz

Obs: In this example, I am using files from the **Release 88**, and
**Human Genome 38**.

Set your working directory and load the needed packages.

``` r
library(dplyr)
library(readr)
library(data.table)
```

Some of those files are pretty big. I recommend that you read only the
useful columns of the files, as showed next:

Peek at a small portion of the data with the fread function from the
“data.table” package.

``` r
genome<-fread("V88_38_GENOMESCREENMUTANT.csv",nrows = 10)
target<-fread("V88_38_TARGETEDSCREENMUTANT.csv",nrows = 10)
samples<-fread("V88_38_SAMPLE.csv",nrows = 10)
```

# Prepare the data

Again with fread, choose the columns that you want (or need) to read.

Firstly, we will read the Targeted file, which lists samples with
mutations (positives) and samples with no mutation (negatives) from
targeted screens. This is the biggest file between the three that we
will use,thus, we will readas few columns as possible.

``` r
target<-fread("V88_38_TARGETEDSCREENMUTANT.csv", select = c("GENE_NAME",
                                                            "ACCESSION_NUMBER",
                                                            "SAMPLE_NAME",
                                                            "MUTATION_AA"))
```

Next, the genome screen file lists samples with mutations (positives)
from whole-genome screens.

``` r
genome<-fread("V88_38_GENOMESCREENMUTANT.csv", select = c("GENE_NAME",
                                                        "ACCESSION_NUMBER",
                                                        "SAMPLE_NAME",
                                                        "SAMPLE_SOURCE"))
```

Obs:samples without mutations (negatives) are not included and must be
extracted from the Samples file, by selecting rows where the ‘whole
genome screen’ column is equal to ‘y’ (see below).

Read the two needed columns.

``` r
samples<-fread("V88_38_SAMPLE.csv", select = c("SAMPLE_NAME","WHOLE_GENOME_SCREEN"))
#filter
samples_wgs<-unique(filter(samples,WHOLE_GENOME_SCREEN=="y"))
```

Now we can begin to do the calculation but first you need to know a
small but important detail: COSMIC calculates its frequencies separating
the **transcripts** from each gene. Although I will not do it in this
tutorial, if you do not want to consider each gene transcript as a
different “gene” you must remove the transcript ID suffixes from the
GENE_NAME column, as exemplified below:

``` r
genome$GENE_NAME<-gsub(pattern = "_ENST.*",replacement = "",genome$GENE_NAME)
```

# Make the function

Now, lets make a function to loop it later

``` r
cosmic_gene_freq<-function(x=character()){

# CALCULE OPTION 1:
#MUTANT FREQUENCY FROM TARGETED SCREENS
#extract samples with mutations (positives) and samples with no mutation (negatives)
target_gene<-filter(target,GENE_NAME==x)

target_pos<-filter(target_gene,MUTATION_AA!="null")
target_pos_samples<-nrow(unique(target_pos[,3]))

target_neg<-filter(target_gene,MUTATION_AA=="null")
target_neg_samples<-nrow(unique(target_neg[,3]))

freq_target<-target_pos_samples/(target_pos_samples+target_neg_samples)*100

#CALCULE OPTION 2:
#MUTANT FREQUENCY FROM GENOME-WIDE SCREENS
selected_genes_genome<-filter(genome,GENE_NAME==x)

#Now that we selected the gene we just need to count samples and apply the calc
freq_genome<-nrow(unique(selected_genes_genome))/nrow(samples_wgs)*100

#CALCULE OPTION 3:
#MUTANT FREQUENCY FROM TOTAL DATASET (TARGET+GENOME)
#This is the calcule used in the values you can visualized at the "Tissue distribution"
#tab in the COSMIC gene pages.

freq_all<-(target_pos_samples+nrow(unique(selected_genes_genome)))/
  (target_neg_samples+target_pos_samples+nrow(samples_wgs))*100

return(freq_all)#we will return this one used by COSMIC, but you can do the same with
#the others (or all at the same time).
}
```

# Make a for loop for multiple genes

Make a list of gene symbols. You can also use a vector, if you want to
take a list from a big table, for example:

``` r
my_genes<- c("TP53","GATA3","PIK3CA")

#make an empty df
foo = data.frame()

#make a for loop with your function
for(i in 1:length(my_genes)){ 
  foo[i,1] = cosmic_gene_freq(my_genes[[i]])}

#finish with a table
genes_df<-cbind(my_genes,foo)
```

Done!

| my_genes |        V1 |
|:---------|----------:|
| TP53     | 26.511007 |
| GATA3    |  9.557325 |
| PIK3CA   | 27.045322 |
