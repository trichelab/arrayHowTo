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

# force the 450k manifests to be loaded
# this may also be necessary for EPIC :-(
library(IlluminaHumanMethylation450kmanifest)
library(IlluminaHumanMethylation450kanno.ilmn12.hg19) 

library(FlowSorted.Blood.450k) 
data(FlowSorted.Blood.450k) 

# drop probes with > 10% missing values using naFrac
grSet <- sesamize(FlowSorted.Blood.450k, naFrac=0.1) 
```
Sesamizing WB_105...

Or if you're really impatient, like me:

```R
library(parallel) 
options(mc.cores=detectCores())

# drop probes with > 10% missing values, run > 10 IDATs at a time
grSet <- sesamize(FlowSorted.Blood.450k, naFrac=0.1, parallel=TRUE) 
```
Sesamizing WB_105...    
Sesamizing WB_218...    
Sesamizing WB_261...    
Sesamizing PBMC_105...    
Sesamizing PBMC_218...    
Sesamizing PBMC_261...    
Sesamizing Gran_105...    
Sesamizing Gran_218...    
Sesamizing Gran_261...    
Sesamizing CD4+_105...    

Either way, you get back a GenomicRatioSet (```?GenomicRatioSet``` for more):

```R
show(grSet)
```

Once you have a GenomicRatioSet you can feed it to Conumee, for example (much more on that topic in the conumee package vignette), or call DMRs with, say, DMRcate.

## A/B compartments

If you would like to get some of the information that Hi-C provides, without necessarily burning entire flow cells on each sample, you can use the ```compartments()``` function in ```minfi``` to look at A/B compartment structure:

```
# CD14 is a cell surface marker on classical monocytes
Monocytes <- grSet[, grep("CD14", colnames(grSet))] 

# We drop the correlation matrix in this example (keep=FALSE)
comps.mono.chr22 <- compartments(Monocytes, chr="chr22", keep=FALSE) 

# CD56 is a cell surface marker on classical natural killer (NK) cells
NKcells <- grSet[, grep("CD56", colnames(grSet))] 

# We drop the correlation matrix in this example (keep=FALSE) too
comps.NK.chr22 <- compartments(NKcells, chr="chr22", keep=FALSE)
```

This can take a while; once it's done, try poking around at the comps data structures, and see whether it's obvious how to dump this to a bigWig file using ```rtracklayer``` (and by extension whether differences of eigenvalues, in $pc, make sense).
The compartments function returns a GenomicRanges object (```?GRanges``` for more information) which maps 1:1 onto a bigWig.

```R
show(comps.NK.chr22)

# we do need to change some names, though!
names(mcols(comps.mono.chr22))[1] <- "score"
names(mcols(comps.NK.chr22))[1] <- "score"
```

Now we can dump the result and compare in, say, IGV (or you could split by compartment and run various tests in R):

```R
library(rtracklayer)
export(comps.NK.chr22, "NKcells.compartments.chr22.hg19.bw")
export(comps.NK.chr22, "Monocytes.compartments.chr22.hg19.bw")
```

If it's not obvious how to use this information, I can add another section here.
(Or maybe the IDAT files for the TARGET AML NUP98-NSD1 vs. NUP98-KDM5A comparison)

