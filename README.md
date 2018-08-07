# arrayHowTo

This is a step-by-step demo to accompany the Epigenomics Workshop 2018 practicum for DNA methylation microarrays.
Since the new best practices for Bioconductor include using the BiocManager package, let's begin there.
If you use Rstudio, now would be a good time to start it up. 

```R
install.packages("BiocManager")
library("BiocManager")
install(c("sesame","minfi","conumee"))
```

The BiocManager package has some neat new features.  Let's try one.

```R
available("FlowSorted")
```

# minfi

This is handy. We can automatically retrieve minfi data for the demo!

```R
install("FlowSorted.Blood.450k") 
library(FlowSorted.Blood.450k)
```

# SeSAMe

The ```sesamize()``` function is relatively new and we cannot guarantee that it has passed continuous integration testing in Bioconductor-devel (for reasons too boring to discuss).  We will take advantage of BiocManager's wonderful GitHub integration to use the bleeding edge version of the package.
```R
install("zwdzwd/sesame")
```

Predictably, there is a bit of a timing bug in assigning probe IDs in sesamize, so let's brute-force fix that:
```R
library(BiocManager)
for (i in available("IlluminaHumanMethylation450k")[-3]) {
  install(i) 
  library(i, character.only=TRUE)
} 
```

Now:

```R
library(sesame)
library(minfi) 
library(IlluminaHumanMethylation450kmanifest)
library(IlluminaHumanMethylation450kanno.ilmn12.hg19) 

data(FlowSorted.Blood.450k) 
grSet <- sesamize(FlowSorted.Blood.450k) 
show(grSet)
```

