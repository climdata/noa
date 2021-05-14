---
title: "NOA"
author: "KMicha71"
date: "14 5 2021"
output:
  html_document: 
    keep_md: true
  pdf_document: default
---



## Nort Atlantic Oscilation

Download data from:

https://crudata.uea.ac.uk/cru/data/nao/

Jones, P.D., Jónsson, T. and Wheeler, D., 1997: Extension to the North Atlantic Oscillation using early instrumental pressure observations from Gibraltar and South-West Iceland. Int. J. Climatol. 17, 1433-1450. doi: 10.1002/(SICI)1097-0088(19971115)17:13<1433::AID-JOC203>3.0.CO;2-P




```sh
 [ -f ./download/nao_3dp.dat ] && mv -f ./download/nao_3dp.dat ./download/nao_3dp.dat.bck
 wget -q -P download https://crudata.uea.ac.uk/cru/data/nao/nao_3dp.dat
 sed -i '1s/^/year,m1,m2,m3,m4,m5,m6,m7,m8,m9,m10,m11,m12,sum\n/' ./download/nao_3dp.dat
 sed -i 's/^ //g' ./download/nao_3dp.dat 
 sed -i 's/   / /g' ./download/nao_3dp.dat 
 sed -i 's/  / /g' ./download/nao_3dp.dat 
 sed -i 's/ /,/g' ./download/nao_3dp.dat 
 #sed -i '/^"/d' ./download/monthly_in_situ_co2_mlo.csv
 #sed -i '2d;3d' ./download/monthly_in_situ_co2_mlo.csv
```

## Convert to standard format


```r
library(reshape2)
noa <- read.csv("./download/nao_3dp.dat", sep=",")
noaNew <- melt(noa, id.vars = c("year"))
noaNew <- subset(noaNew, -99 < noaNew$value)
names(noaNew)[names(noaNew) == "variable"] <- "month"
noaNew <- subset(noaNew, 'sum' != noaNew$month)
names(noaNew)[names(noaNew) == "value"] <- "noa1"
noaNew$month <- strtoi(gsub("m", "", noaNew$month))

noaNew$ts <- signif(noaNew$year + (noaNew$month-0.5)/12, digits=6)
noaNew$time <- paste(noaNew$year,noaNew$month, '15 00:00:00', sep='-')

noaNew$noa2 <- round(noaNew$noa1)
noaNew$noa2[noaNew$noa2 < -4] <- -4
noaNew$noa2[noaNew$noa2 > 4] <- 4

noaNew <- noaNew[order(noaNew$time),]

write.table(noaNew, file = "./csv/monthly_noa.csv", append = FALSE, quote = TRUE, sep = ",",
            eol = "\n", na = "NA", dec = ".", row.names = FALSE,
            col.names = TRUE, qmethod = "escape", fileEncoding = "UTF-8")
```

## Plot diagramm


```r
require("ggplot2")
```

```
## Loading required package: ggplot2
```

```r
noa <- read.csv("./csv/monthly_noa.csv", sep=",")
mp <- ggplot() +
      geom_line(aes(y=noa$noa2, x=noa$ts), color="blue") +
      xlab("Year") + ylab("NOA")
mp
```

![](README_files/figure-html/plot-1.png)<!-- -->
