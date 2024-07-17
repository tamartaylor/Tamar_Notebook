2024-07-17-Tamar\_Photosurvey\_script

## **Taxonomy group: Bryozoa**

library(ggplot2)

library(dplyr)

library(tidyr)

library(broom)

library(grid)

library(cowplot)

library(corrplot)

library(Hmisc)

library(ARTool)

\#### PHOTOSURVEY

setwd("C:/Users/taylor/Documents/Maya\_class") # set working directory

\# To load the metadata- NOTE: we work at the transect level.

photo\_metadata=read.csv("Photosurvey\_metadata.csv", header=TRUE, row.names=1, stringsAsFactors = TRUE)

str(photo\_metadata)

\# To load the processed and filtered coverage data by taxonomic group

photosurvey\_coverage=read.csv("Photosurvey\_processed\_data.csv", header=TRUE, row.names=1, stringsAsFactors = TRUE)

str(photosurvey\_coverage)

\# To make sure the order of trnasect\_IDs is the same in the data and the metadata

ord=match(row.names(photosurvey\_coverage), row.names(photo\_metadata))

photo\_metadata=photo\_metadata[ord,]

\# To merge metadata and coverage values

Photosurvey=cbind(photo\_metadata, photosurvey\_coverage)

str(Photosurvey)

\# 1. test homogeneity of variances:

bartlett.test(Photosurvey$Bryozoa~Photosurvey$site)

\# 2. ArtAnova test:

\### now let's test for all factors together- non-parametric anova

library(ARTool)

m=art(Bryozoa~site+as.factor(depth)+season+site\*as.factor(depth)\*season, data=Photosurvey)

anova(m)

\# Plot by season:

ggplot(Photosurvey, aes(x=season, y=Bryozoa, fill=site))+

`  `geom\_boxplot(outlier.shape = NA)+

`  `geom\_point(position=position\_jitterdodge(jitter.width = 0.2), size=1)+

`  `theme\_light()+

`  `xlab("Season") + ylab("Bryozoa %") +

`  `theme(axis.title.x = element\_text(colour="Black", size=14),

`        `axis.title.y = element\_text(colour="Black", size=14),

`        `axis.text.x = element\_text(size=12),

`        `axis.text.y = element\_text(size=12),

`        `legend.title = element\_text(size=14), 

`        `legend.text = element\_text(size=12),

`        `plot.caption = element\_text(size=12, hjust = 0)) +

`  `facet\_grid(.~depth, labeller = labeller(depth = c(

`    `'10' = '10 meter',

`    `'25' = '25 meter',

`    `'45' = '45 meter'

`  `)))  +

`  `labs(caption = "Figure 1 Bryozoa percentage at fall and at spring in Achziv and Sdot Yam at three depth points: 10, 25, 45 meters.")

\#### To check the relatedness between coverage by different groups? (correlations)

library(Hmisc)

str(Photosurvey)

\# Variables 10 to 22 have the measurements

corrs=rcorr(as.matrix(Photosurvey[,10:22]), type="spearman")

\# A function that will take the square matrix and turn it to long format.

flattenCorrMatrix <- function(cormat, pmat) {

`  `ut <- upper.tri(cormat)

`  `data.frame(

`    `row = rownames(cormat)[row(cormat)[ut]],

`    `column = rownames(cormat)[col(cormat)[ut]],

`    `cor  =(cormat)[ut],

`    `p = pmat[ut]

`  `)

}

\# Apply this function to our data

corrs\_flat=flattenCorrMatrix(corrs$r, corrs$P)

\# Add p value adjustment using the Benjamini-Hochber method (BH)

corrs\_flat$p.adj=p.adjust(corrs\_flat$p, "BH")

write.csv(corrs\_flat, "Taxonomic\_groups\_spearman.csv")

\# Plot correlation for Bryozoa with Cnidaria:

ggplot(Photosurvey, aes(x=Bryozoa, y=Cnidaria))+

`  `geom\_point()+

`  `geom\_smooth(method='lm')+

`  `theme\_light() +

xlab("Bryozoa %") + ylab("Cnidaria %") +

`  `theme(axis.title.x = element\_text(colour="Black", size=14),

`        `axis.title.y = element\_text(colour="Black", size=14),

`        `axis.text.x = element\_text(size=12),

`        `axis.text.y = element\_text(size=12),

`        `plot.caption = element\_text(size=12, hjust = 0)) +

`    `labs(caption = "Figure 2 Correlation between Broyzoa percentage (X axis) and Cnidaria percentage (Y axis).")

\# Plot correlation for Bryozoa with with Algae:

ggplot(Photosurvey, aes(x=Bryozoa, y=Algae))+

`  `geom\_point()+

`  `geom\_smooth(method='lm')+

`  `theme\_light() +

`  `xlab("Bryozoa %") + ylab("Algae %") +

`  `theme(axis.title.x = element\_text(colour="Black", size=14),

`        `axis.title.y = element\_text(colour="Black", size=14),

`        `axis.text.x = element\_text(size=12),

`        `axis.text.y = element\_text(size=12),

`        `plot.caption = element\_text(size=12, hjust = 0)) +

`  `labs(caption = "Figure 3 Correlation between Broyzoa percentage (X axis) and Algae percentage (Y axis).")

