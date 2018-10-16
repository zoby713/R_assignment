---
title: "R_assignment"
author: "Jialu Wei"
date: "October 12, 2018"
output: html_document
---
Good job of telling me what packages to install before starting. JZ
#Environment setting
```{r}
if (!require("tidyverse")) install.packages("tidyverse")
library(tidyverse)

if (!require("ggplot2")) install.packages("ggplot2") # install ggplot2 if it's not already installed
library(ggplot2)

if (!require("reshape2")) install.packages("reshape2") # install reshape2 if it's not already installed
library(reshape2)

if (!require("plyr")) install.packages("plyr") # install plyr if it's not already installed
library(plyr)

if (!require("dplyr")) install.packages("dplyr") # install dplyr if it's not already installed
library(dplyr)
```

Reading the data from the git hub repository 
```{r}
fang_et_al <- read.table("https://raw.githubusercontent.com/EEOB-BioData/BCB546X-Fall2018/master/assignments/UNIX_Assignment/fang_et_al_genotypes.txt", sep = "\t",header = T)
snp_position <- read.table("https://raw.githubusercontent.com/EEOB-BioData/BCB546X-Fall2018/master/assignments/UNIX_Assignment/snp_position.txt",sep = "\t", fill = TRUE, header = T)

```

This worked well for me. JZ
#############Part I#############

some of the results:

[1] 2782  986 
2782 rows and 986 columns;

[1] 983  15 
983 rows and 13 columns;

both files are data.frame;

[1] "TRIPS" "ZDIPL" "ZLUXR" "ZMHUE" "ZMMIL" "ZMMLR" "ZMMMR" "ZMPBA" "ZMPIL" "ZMPJA" "ZMXCH" "ZMXCP" "ZMXIL" "ZMXNO" "ZMXNT" "ZPERR"
16 levels of groups;

[1] "1"        "10"       "2"        "3"        "4"        "5"        "6"        "7"        "8"        "9"        "multiple"
[12] "unknown" 
12 levels of chromosomes
...

#Data inspection
```{r}
head(fang_et_al) # print the first few rows of the data
head(snp_position)

dim(fang_et_al)# prints number of columns and number of rows
dim(snp_position)

str(fang_et_al)# see data structure
str(snp_position)

class(fang_et_al)# ouput the type of data
class(snp_position)

levels(fang_et_al$Group)# inspect te levels of groups
levels(snp_position$Chromosome)

```

#Data processing

Good descriptions for all of your commands. JZ
```{r}
#extract the data needed 
geno_maize <- filter(fang_et_al, `Group`== c("ZMMIL","ZMMLR","ZMMMR")) %>% t() %>% as.data.frame() 
geno_teosinte <- filter(fang_et_al, `Group`==c("ZMPBA","ZMPIL","ZMPJA")) %>% t() %>% as.data.frame()
snp_infor <- select(snp_position,`SNP_ID`,`Chromosome`,`Position`) 

#merge genotype with snp together seperately for maize and teosinte
maize <- merge(snp_infor,geno_maize,by.x = "SNP_ID",by.y = "row.names",all = TRUE)
maize <- maize[-(984:986),]
teosinte <- merge(snp_infor,geno_teosinte,by.x = "SNP_ID",by.y = "row.names",all = TRUE)
teosinte <- teosinte[-(984:986),]
maize$Position <- as.numeric(as.character(maize$Position))
teosinte$Position <- as.numeric(as.character(teosinte$Position))
#generate files for maize
##position ascending
for (i in 1:10){
  maize_chr <- filter(maize, Chromosome == i ) %>% arrange(Position)
  write.table(maize_chr,paste("maize_chr",i,"_asce.txt",sep=""))
}
#position descending & -/- as missing data
for (i in 1:10){
  maize2 <- maize
  maize2[] <- lapply(maize[], as.character)
  maize2[maize2 == "?/?"] <- "-/-"
  maize2[,3] <- as.numeric(maize2[,3])
  maize_chr <- filter(maize2, Chromosome == i ) %>% arrange(desc(Position))
  write.table(maize_chr,paste("maize_chr",i,"_desc.txt",sep=""))
}

#generate files for teosinte
##position ascending
for (i in 1:10){
  teosinte_chr <- filter(teosinte, Chromosome == i ) %>% arrange(Position)
  write.table(teosinte_chr,paste("teosinte_chr",i,"_asce.txt",sep=""))
}
#position descending & -/- as missing data
for (i in 1:10){
  teosinte2 <- teosinte
  teosinte2[] <- lapply(teosinte[],as.character) 
  teosinte2[teosinte2 == "?/?"] <- "-/-"
  teosinte2[,3] <- as.numeric(teosinte2[,3])
  teosinte_chr <- filter(teosinte2, Chromosome == i ) %>% arrange(desc(Position))
  write.table(teosinte_chr,paste("teosinte_chr",i,"_desc.txt",sep=""))
}

```


#############Part II#############


#We define SNP as the ones have variation within group
#plots showing SNP numbers on each chromosome and the share contributed by each group
#the top three groups contributing most to the total number are ZMPBA, ZMMIL, ZMXCP. 
```{r}
fang_et_al2 <- fang_et_al[,-1]
chr_inf <- select(snp_position,`SNP_ID`,`Chromosome`)
library(reshape2)
#inspect SNP patterns for each group using unique() 
melt_group <- melt(fang_et_al2,id="Group")
unique_group <- unique(melt_group) 
unique_group_rmna <- subset(unique_group,value != "?/?") #remove missing data
unique_group_rmna[,3] <- 1
#get the SNP pattern numbers for each group and select those >1 (variate)
unique_group_snp <- dcast(unique_group_rmna,Group ~ variable, sum) %>% melt(id="Group") %>% subset(value > 1) 
unique_group_snp2 <- dcast(unique_group_snp,Group ~ variable) %>% t() %>% as.data.frame()
unique_group_snp2_header <- unique_group_snp2[1,]
unique_group_snp2_header[] <- lapply(unique_group_snp2_header[], as.character)
colnames(unique_group_snp2) <- unique_group_snp2_header
unique_group_snp2 <- unique_group_snp2[-1,]
unique_group_snp2 # reshaped dataframe showing how many SNP patterns for each group on each site
#merge the genotype and SNP information together or say add chromosome information 
merge1 <-merge(chr_inf,unique_group_snp2,by.x = "SNP_ID", by.y = "row.names")
melt_all <- melt(merge1, id = c("Chromosome","SNP_ID"),na.rm = TRUE)
colnames(melt_all)[3] <- "Group"
ggplot(melt_all)+ geom_bar(aes(Chromosome,fill = Group))+xlab("Chromosome") +ylab("Total Number of SNPs")
ggplot(melt_all)+ geom_bar(aes(Group))+xlab("Group") +ylab("Total Number of SNPs")


```


#plot showing missing data rate and heterozygosity rates for each species and each group
```{r}
genotype_basic <- fang_et_al[,-2]
genotype_basic[genotype_basic[] == "?/?"] = NA
genotype_melt <- melt(genotype_basic,id = c("Sample_ID","Group"))
#add a new column showing whether this coloum is homozygous heterozygous or NA
genotype_melt$status <- (genotype_melt$value=="A/A"|genotype_melt$value =="C/C"|genotype_melt$value=="G/G"|genotype_melt$value=="T/T")
#sort by Group and sample_ID
genotype_melt <- arrange(genotype_melt,Group,Sample_ID)
#get the total number of homo/heterozygous/NA for each species
genotype_counts_sample<-ddply(genotype_melt,c("Sample_ID"),summarise,total_homozygous=sum(status,na.rm=TRUE),total_heterozygous=sum(!status,na.rm = TRUE), total_NA=sum(is.na(status)))
#get the total number of homo/heterozygous/NA for each group
genotype_counts_group<-ddply(genotype_melt,c("Group"),summarise,total_homozygous=sum(status,na.rm=TRUE),total_heterozygous=sum(!status,na.rm = TRUE), total_NA=sum(is.na(status)))

genotype_melted_sample<-melt(genotype_counts_sample,measure.vars = c("total_homozygous","total_heterozygous","total_NA"))
genotype_melted_group<-melt(genotype_counts_group,measure.vars = c("total_homozygous","total_heterozygous","total_NA"))

#polts showing missing data rate and heterozygosity rates for each species and each group
ggplot(genotype_melted_sample,aes(x=Sample_ID,y=value,fill=variable))+geom_bar(stat="identity")
ggplot(genotype_melted_group,aes(x=Group,y=value,fill=variable))+geom_bar(stat="identity",position = "fill") #normalize the height 
```


#my own visualization
#similar with the last plot, we would like to see the hererosity in maize and teonsinte on each chromosome 
```{r}
maize_own <- maize[,-3]
maize_own[maize_own[] == "?/?"] = NA
maize_own_melt <- melt(maize_own,id = c("Chromosome","SNP_ID"))
maize_own_melt$status <- (maize_own_melt$value=="A/A"|maize_own_melt$value =="C/C"|maize_own_melt$value=="G/G"|maize_own_melt$value=="T/T")
maize_own_counts<-ddply(maize_own_melt,c("Chromosome"),summarise,total_homozygous=sum(status,na.rm=TRUE),total_heterozygous=sum(!status,na.rm = TRUE), total_NA=sum(is.na(status)))
maize_own_counts_melt<-melt(maize_own_counts,measure.vars = c("total_homozygous","total_heterozygous","total_NA"))
ggplot(maize_own_counts_melt,aes(x=Chromosome,y=value,fill=variable))+geom_bar(stat="identity")+ggtitle("Maize")

teosinte_own <- teosinte[,-3]
teosinte_own[teosinte_own[] == "?/?"] = NA
teosinte_own_melt <- melt(teosinte_own,id = c("Chromosome","SNP_ID"))
teosinte_own_melt$status <- (teosinte_own_melt$value=="A/A"|teosinte_own_melt$value =="C/C"|teosinte_own_melt$value=="G/G"|teosinte_own_melt$value=="T/T")
teosinte_own_counts<-ddply(teosinte_own_melt,c("Chromosome"),summarise,total_homozygous=sum(status,na.rm=TRUE),total_heterozygous=sum(!status,na.rm = TRUE), total_NA=sum(is.na(status)))
teosinte_own_counts_melt<-melt(teosinte_own_counts,measure.vars = c("total_homozygous","total_heterozygous","total_NA"))
ggplot(teosinte_own_counts_melt,aes(x=Chromosome,y=value,fill=variable))+geom_bar(stat="identity")+ggtitle("Teosinte")


```

Good Project everything ran well for me. 