Ravi Sood
February 4, 2014
BIOST 578
Homework 1
========================================================

1. Use the GEOmetabd package to find all HCV gene expression data using the Illumina platform submitted by an investigator at Yale. This should be done with a single query, showing the title, the GSE accession number, the GPL accession number and the manufacturer and the description of the platform used.

First, install bioconductor and download the database:

```r
# Install bioconductor packages
source("http://bioconductor.org/biocLite.R")
biocLite()

# Download the database
getSQLiteFile()
```


Next, connect to and query the database:

```r
getwd()
```

```
## [1] "C:/Users/dDub/Documents/GitHub/HW1-rfsood"
```

```r

# Load GEOmetadb package
library(GEOmetadb)
```

```
## Loading required package: GEOquery
## Loading required package: Biobase
## Loading required package: BiocGenerics
## Loading required package: parallel
## 
## Attaching package: 'BiocGenerics'
## 
## The following objects are masked from 'package:parallel':
## 
##     clusterApply, clusterApplyLB, clusterCall, clusterEvalQ,
##     clusterExport, clusterMap, parApply, parCapply, parLapply,
##     parLapplyLB, parRapply, parSapply, parSapplyLB
## 
## The following object is masked from 'package:stats':
## 
##     xtabs
## 
## The following objects are masked from 'package:base':
## 
##     anyDuplicated, append, as.data.frame, as.vector, cbind,
##     colnames, duplicated, eval, evalq, Filter, Find, get,
##     intersect, is.unsorted, lapply, Map, mapply, match, mget,
##     order, paste, pmax, pmax.int, pmin, pmin.int, Position, rank,
##     rbind, Reduce, rep.int, rownames, sapply, setdiff, sort,
##     table, tapply, union, unique, unlist
## 
## Welcome to Bioconductor
## 
##     Vignettes contain introductory material; view with
##     'browseVignettes()'. To cite Bioconductor, see
##     'citation("Biobase")', and for packages 'citation("pkgname")'.
## 
## Setting options('download.file.method.GEOquery'='auto')
## Loading required package: RSQLite
## Loading required package: DBI
```

```r

# Connect to the database
con <- dbConnect(SQLite(), "GEOmetadb.sqlite")

# Query the database
dbGetQuery(con, "SELECT gse.title, gse.gse, gpl.gpl, gpl.manufacturer, gpl.technology, gse.contact FROM (gse JOIN gse_gpl ON gse.gse = gse_gpl.gse) JOIN gpl ON gse_gpl.gpl=gpl.gpl WHERE gse.title LIKE '%Hepatitis C%' AND gpl.manufacturer LIKE '%Illumina%' AND gse.contact LIKE '%Yale%'")
```

```
##                                                                                                              title
## 1 Impaired TLR3-mediated immune responses from macrophages of patients chronically infected with Hepatitis C virus
##        gse      gpl  manufacturer            technology
## 1 GSE40812 GPL10558 Illumina Inc. oligonucleotide beads
##                                                                                                                                                                                                                            contact
## 1 Name: Ruth,R,Montgomery;\tEmail: ruth.montgomery@yale.edu;\tDepartment: Internal Medicine;\tInstitute: Yale University School of Medicine;\tAddress: 300 Cedar St.;\tCity: New Haven;\tState: CT;\tZip/postal_code: 06520;\tCountry: USA
```


Running the above code yields the following result:

title
Impaired TLR3-mediated immune responses from macrophages of patients chronically infected with Hepatitis C virus
       
gse
GSE40812

gpl
GPL10558

manufacturer
Illumina Inc.

technology
oligonucleotide beads

contact
1 Name: Ruth,R,Montgomery;\tEmail: ruth.montgomery@yale.edu;\tDepartment: Internal Medicine;\tInstitute: Yale University School of Medicine;\tAddress: 300 Cedar St.;\tCity: New Haven;\tState: CT;\tZip/postal_code: 06520;\tCountry: USA

2. Reproduce your above query using the data.table package. Again, try to use a single line of code. [Hint: You first need to convert all db tables to data.table tables].


```r
# Install data.table
install.packages("data.table")
```

```
## Error: trying to use CRAN without setting a mirror
```

```r

# Load data.table
library(data.table)

# Convert db tables to data.table tables
gse <- as.data.table(dbGetQuery(con, "SELECT gse.title, gse.gse, gse.contact FROM gse"))
gpl <- as.data.table(dbGetQuery(con, "SELECT gpl.gpl, gpl.manufacturer, gpl.technology FROM gpl"))
gse_gpl <- as.data.table(dbGetQuery(con, "Select gse_gpl.gse, gse_gpl.gpl FROM gse_gpl"))

# Close the connection
dbDisconnect(con)
```

```
## [1] TRUE
```

```r

# Join the tables using two right-joins and return rows where title contains
# 'Hepatitis C', manufacturer contains 'Illumina', and contact contains
# 'Yale'
merge(x = merge(x = gse, y = gse_gpl, by = "gse", all.y = TRUE), y = gpl, by = "gpl", 
    all.y = TRUE)[title %like% "Hepatitis C"][contact %like% "Yale"][manufacturer %like% 
    "Illumina"]
```

```
##         gpl      gse
## 1: GPL10558 GSE40812
##                                                                                                               title
## 1: Impaired TLR3-mediated immune responses from macrophages of patients chronically infected with Hepatitis C virus
##                                                                                                                                                                                                                             contact
## 1: Name: Ruth,R,Montgomery;\tEmail: ruth.montgomery@yale.edu;\tDepartment: Internal Medicine;\tInstitute: Yale University School of Medicine;\tAddress: 300 Cedar St.;\tCity: New Haven;\tState: CT;\tZip/postal_code: 06520;\tCountry: USA
##     manufacturer            technology
## 1: Illumina Inc. oligonucleotide beads
```


Running the above code yields the same result as in part (1).
