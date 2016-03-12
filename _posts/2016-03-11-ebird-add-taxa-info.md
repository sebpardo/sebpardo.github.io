---
layout: post
title: Adding Order and Family fields to eBird's taxonomic checklist
date: "March 11, 2016"
modified: 2016-03-11
---

As a bird-nerd, I spend a lot of my time playing with bird-related data. Lately I've been tinkering with code to interface with [eBird](http://www.ebird.org), a global citizen science initiative that records bird sightings throughout the world, and even helps you maintain your own lists (for those of you who are listers, that is...).

The [eBird API interface](https://confluence.cornell.edu/display/CLOISAPI/eBird+API+1.1) allows for downloading their most up-to-date latest checklist, and can be accessed in R using the `ebirdtaxonomy` function in the [rebird](https://github.com/ropensci/rebird) package. Unfortunately, this checklist doesn't include any high-level taxonomic information, such as order and family, for each species.

I was looking for a way to compare my eBird sightings across taxa, so after a while of searching for different taxonomic checklists and ways of merging them, I've come up with a way that's only *slightly* convoluted to include higher taxonomic info into any eBird checklist.

``` r
library(dplyr)
library(tidyr)
library(lazyeval)
library(rebird)
library(tidyr)
library(xml2)
```

First, we load eBird's taxonomic checklist and separate the scientific name into two columns, `genus` and `species`

``` r
ebirdtax <- ebirdtaxonomy(cat = "species") %>% 
  separate(sciName, c("genus", "species"), sep = " ", remove = FALSE)
ebirdtax %>% select(genus, species, everything())
```

    ## Source: local data frame [10,473 x 11]
    ## 
    ##          genus       species speciesCode category                comName
    ##          (chr)         (chr)       (chr)    (chr)                  (chr)
    ## 1     Struthio       camelus     ostric2  species         Common Ostrich
    ## 2     Struthio molybdophanes     ostric3  species         Somali Ostrich
    ## 3         Rhea     americana     grerhe1  species           Greater Rhea
    ## 4         Rhea       pennata     lesrhe2  species            Lesser Rhea
    ## 5  Nothocercus        julius     tabtin1  species Tawny-breasted Tinamou
    ## 6  Nothocercus    bonapartei     higtin1  species       Highland Tinamou
    ## 7  Nothocercus nigrocapillus     hootin1  species         Hooded Tinamou
    ## 8      Tinamus           tao     grytin1  species           Gray Tinamou
    ## 9      Tinamus    solitarius     soltin1  species       Solitary Tinamou
    ## 10     Tinamus       osgoodi     blatin1  species          Black Tinamou
    ## ..         ...           ...         ...      ...                    ...
    ## Variables not shown: sciNameCodes (chr), sciName (chr), taxonID (chr),
    ##   taxonOrder (dbl), comNameCodes (chr), bandingCodes (chr)

This checklist has 10473 species and, as already mentioned, doesn't contain any columns with order or family information. We need to obtain that information somewhere else and match it to this checklist. For ease of comparison throughout this post, we'll just focus on distinct genera

``` r
ebirdgen <- select(ebirdtax, genus) %>%
  distinct(genus)
ebirdgen
```

    ## Source: local data frame [2,226 x 1]
    ## 
    ##           genus
    ##           (chr)
    ## 1      Struthio
    ## 2          Rhea
    ## 3   Nothocercus
    ## 4       Tinamus
    ## 5  Crypturellus
    ## 6    Rhynchotus
    ## 7   Nothoprocta
    ## 8       Nothura
    ## 9     Taoniscus
    ## 10     Eudromia
    ## ..          ...

This leaves us with 2226 unique genera.

There are other organizations that also maintain their own complete bird species checklists, but do include information on order and families for each species. One of these is [BirdLife International](http://www.birdlife.org/datazone/info/taxonomy). We need to download the latest version of this checklist; for simplicity I've already turned this list into a .csv file which you can download [here](http://www.asd.com/) (you can also download the original .zip file with the complete information [here](http://www.birdlife.org/datazone/userfiles/file/Species/Taxonomy/BirdLife_Checklist_Version_80.zip)).

A little bit of wrangling is required to make this checklist comparable with eBird's taxonomy

``` r
birdlifetax <- read.csv("~/Projects/stats-ebird/BirdLife-v8.csv", 
                        stringsAsFactors = FALSE) %>% tbl_df %>%
  select(Order, Family_name, Scientific.name, everything())
birdlifetax
```

    ## Source: local data frame [11,862 x 14]
    ## 
    ##               Order   Family_name          Scientific.name    Family
    ##               (chr)         (chr)                    (chr)     (chr)
    ## 1  STRUTHIONIFORMES Struthionidae         Struthio camelus Ostriches
    ## 2  STRUTHIONIFORMES Struthionidae   Struthio molybdophanes Ostriches
    ## 3  STRUTHIONIFORMES Struthionidae         Struthio camelus Ostriches
    ## 4  STRUTHIONIFORMES       Rheidae Pterocnemia tarapacensis     Rheas
    ## 5  STRUTHIONIFORMES       Rheidae           Rhea americana     Rheas
    ## 6  STRUTHIONIFORMES       Rheidae             Rhea pennata     Rheas
    ## 7  STRUTHIONIFORMES       Rheidae        Rhea tarapacensis     Rheas
    ## 8  STRUTHIONIFORMES       Rheidae                Rhea nana     Rheas
    ## 9  STRUTHIONIFORMES       Rheidae             Rhea pennata     Rheas
    ## 10 STRUTHIONIFORMES     Tinamidae       Nothocercus julius  Tinamous
    ## ..              ...           ...                      ...       ...
    ## Variables not shown: Common.name (chr), Authority (chr),
    ##   BirdLife.taxonomic.treatment (chr), IUCN_RL_2015 (chr), Synonyms (chr),
    ##   Alternative.common.names (chr), Taxonomic.notes (chr),
    ##   Taxonomic.sources. (chr), SISRecID (int), SpcRecID (int)

As you can see, this second checklist contains more species than eBird (11862). We'll split the scientific name into Genus and species, make orders lowecase except for the first letter, and select all unique genera with their respective order and family columns

``` r
birdlifegen <- birdlifetax %>%
  separate(Scientific.name, c("genus", "species"), sep = " ", remove = FALSE) %>%
  mutate(order = paste0(toupper(substring(Order, 1, 1)), tolower(substring(Order, 2)))) %>%
  select(order, family = Family_name, genus) %>%
  distinct(genus)
birdlifegen
```

    ## Source: local data frame [2,219 x 3]
    ## 
    ##               order        family        genus
    ##               (chr)         (chr)        (chr)
    ## 1  Struthioniformes Struthionidae     Struthio
    ## 2  Struthioniformes       Rheidae  Pterocnemia
    ## 3  Struthioniformes       Rheidae         Rhea
    ## 4  Struthioniformes     Tinamidae  Nothocercus
    ## 5  Struthioniformes     Tinamidae      Tinamus
    ## 6  Struthioniformes     Tinamidae Crypturellus
    ## 7  Struthioniformes     Tinamidae   Rhynchotus
    ## 8  Struthioniformes     Tinamidae  Nothoprocta
    ## 9  Struthioniformes     Tinamidae      Nothura
    ## 10 Struthioniformes     Tinamidae    Taoniscus
    ## ..              ...           ...          ...

The BirdLife checklist also contains more unique genera than eBird's checklist. What is interesting to see is which ones (and how many) genera are different between these checklists.

``` r
anti_join(ebirdgen, birdlifegen, by = "genus") %>%
  select(genus) %>% distinct(genus)
```

    ## Source: local data frame [173 x 1]
    ## 
    ##            genus
    ##            (chr)
    ## 1        Euodice
    ## 2     Spermestes
    ## 3    Odontospiza
    ## 4   Paludipasser
    ## 5  Sporaeginthus
    ## 6      Granatina
    ## 7     Coccopygia
    ## 8   Pachyphantes
    ## 9     Carpospiza
    ## 10        Alario
    ## ..           ...

There are 173 genera in the eBird checklist that are not in the BirdLife checklist! It's important to know that the difference from `anti_join` (and also `setdiff`) is asymmetrical, therefore the order in which we write the arguments matters. Just out of curiosity, we can have a look at the symmetric difference between these sets:

``` r
sym_diff <- function(a,b) unique(c(setdiff(a,b), setdiff(b,a)))

ebirdbl.diff <- sym_diff(ebirdgen$genus, birdlifegen$genus)
data.frame(genus = ebirdbl.diff) %>% tbl_df
```

    ## Source: local data frame [339 x 1]
    ## 
    ##              genus
    ##             (fctr)
    ## 1             Chen
    ## 2       Oressochen
    ## 3      Oceanodroma
    ## 4     Ichthyophaga
    ## 5        Crecopsis
    ## 6      Anurolimnas
    ## 7  Aenigmatolimnas
    ## 8      Porphyriops
    ## 9    Mustelirallus
    ## 10 Chroicocephalus
    ## ..             ...

So between the eBird and BirdLife checklists, there are genera that are are in one but not the other!

There are too many genera missing to enter by hand, so we need to find another data source to fill in the gaps. The [IOC World Bird List](http://www.worldbirdnames.org/) is another well-maintained bird species checklist. Their spreadsheet file is way too messy to work with, however we can parse the [XML file](http://www.worldbirdnames.org/master_ioc-names_xml.xml) they provide. I have very little experience with XML code but I am very grateful that [Scott Chamberlain](https://twitter.com/sckottie), who works at [ROpenSci](http://ropensci.org/), conjured some black magic and came up with code to turn the XML into a data frame (you can see his gist [here](https://gist.github.com/sckott/c0437a71a889793e30d5)).

``` r
# sourcing the gist creates a data frame named df
devtools::source_gist("https://gist.github.com/sckott/c0437a71a889793e30d5")
```

``` r
iocgen <- mutate(df, order = paste0(toupper(substring(order, 1, 1)), 
                                        tolower(substring(order, 2))))
iocgen
```

    ## Source: local data frame [2,284 x 3]
    ## 
    ##               order        family        genus
    ##               (chr)         (chr)        (chr)
    ## 1      Tinamiformes     Tinamidae      Tinamus
    ## 2      Tinamiformes     Tinamidae  Nothocercus
    ## 3      Tinamiformes     Tinamidae Crypturellus
    ## 4      Tinamiformes     Tinamidae   Rhynchotus
    ## 5      Tinamiformes     Tinamidae  Nothoprocta
    ## 6      Tinamiformes     Tinamidae      Nothura
    ## 7      Tinamiformes     Tinamidae    Taoniscus
    ## 8      Tinamiformes     Tinamidae     Eudromia
    ## 9      Tinamiformes     Tinamidae    Tinamotis
    ## 10 Struthioniformes Struthionidae     Struthio
    ## ..              ...           ...          ...

The IOC checklist has more genera than either of the checklists examined previously. Now we combine the IOC and BirdLife genera and compare it with the eBird checklist:

``` r
allgenera <- full_join(iocgen, birdlifegen, by = c("order", "family", "genus"))
allgenera %>% arrange(genus)
```

    ## Source: local data frame [2,833 x 3]
    ## 
    ##               order          family        genus
    ##               (chr)           (chr)        (chr)
    ## 1       Apodiformes     Trochilidae     Abeillia
    ## 2  Caprimulgiformes     Trochilidae     Abeillia
    ## 3     Passeriformes       Cettiidae   Abroscopus
    ## 4     Passeriformes       Sylviidae   Abroscopus
    ## 5       Galliformes        Cracidae      Aburria
    ## 6     Passeriformes    Meliphagidae Acanthagenys
    ## 7     Passeriformes      Thraupidae  Acanthidops
    ## 8     Passeriformes     Emberizidae  Acanthidops
    ## 9     Passeriformes    Fringillidae     Acanthis
    ## 10    Passeriformes Acanthisittidae Acanthisitta
    ## ..              ...             ...          ...

By sorting the data frame by Genus, we can see that in certain cases (like for the Genus *Abeillia*) the order and family information does not match between these two checklists. Here I made a judgement call and after a quick scan through Wikipedia I decided that the IOC checklist had the most updated taxonomic placement. For example, the family Trochilidae are currently placed in the order Apodiformes rather than Caprimulgiformes, therefore, the first instance of this Genus is the correct one. As `full_join` places rows from the data frame specified first (`iocgen` in this case) before those from the other data frame, we can use `distinct` to keep the first instance of each genus:

``` r
allgenera <- allgenera %>% distinct(genus) 
allgenera %>% arrange(genus)
```

    ## Source: local data frame [2,406 x 3]
    ## 
    ##            order          family           genus
    ##            (chr)           (chr)           (chr)
    ## 1    Apodiformes     Trochilidae        Abeillia
    ## 2  Passeriformes       Cettiidae      Abroscopus
    ## 3    Galliformes        Cracidae         Aburria
    ## 4  Passeriformes    Meliphagidae    Acanthagenys
    ## 5  Passeriformes      Thraupidae     Acanthidops
    ## 6  Passeriformes    Fringillidae        Acanthis
    ## 7  Passeriformes Acanthisittidae    Acanthisitta
    ## 8  Passeriformes    Acanthizidae       Acanthiza
    ## 9  Passeriformes    Meliphagidae Acanthorhynchus
    ## 10 Passeriformes    Acanthizidae     Acanthornis
    ## ..           ...             ...             ...

So, we kept the first instance of *Abeillia*, and so on. Now we a have a comprehensive list of genera, with their respective family and order, from two separate checklists. We can now match it to the eBird checklist:

``` r
ebirdorders <- left_join(ebirdgen, allgenera, by = "genus") %>%
  select(order, family, genus)

ebirdmissing <- ebirdorders %>% filter(is.na(order))
ebirdmissing
```

    ## Source: local data frame [39 x 3]
    ## 
    ##    order family          genus
    ##    (chr)  (chr)          (chr)
    ## 1     NA     NA     Oressochen
    ## 2     NA     NA   Ichthyophaga
    ## 3     NA     NA      Crecopsis
    ## 4     NA     NA  Mustelirallus
    ## 5     NA     NA  Rhamphomantis
    ## 6     NA     NA      Damophila
    ## 7     NA     NA   Calorhamphus
    ## 8     NA     NA       Guarouba
    ## 9     NA     NA    Euchrepomis
    ## 10    NA     NA Cercomacroides
    ## ..   ...    ...            ...

There are still 39 genera for which we do not have order or family info. Given that the eBird checklist is ordered taxonomically, one way around this is by checking the order and family values around each `NA` value: if they are the same, then we can use this value to fill in the blanks:

``` r
NAs <- which(ebirdorders$genus %in% ebirdmissing$genus)
NAs
```

    ##  [1]   30  304  342  351  526  721  806  946  966  997 1234 1241 1248 1303
    ## [15] 1317 1403 1405 1406 1407 1408 1427 1428 1540 1733 1737 1741 1745 1754
    ## [29] 1758 1853 1878 1909 1976 2162 2163 2186 2202 2209 2221

``` r
print(ebirdorders[(NAs[1]-1):(NAs[1]+1),])
```

    ## Source: local data frame [3 x 3]
    ## 
    ##          order   family      genus
    ##          (chr)    (chr)      (chr)
    ## 1 Anseriformes Anatidae Pteronetta
    ## 2           NA       NA Oressochen
    ## 3 Anseriformes Anatidae Chloephaga

There are some cases where there are more than one `NA` in a row, and even sometimes these gaps are in between different families:

``` r
print(ebirdorders[2161:2164,])
```

    ## Source: local data frame [4 x 3]
    ## 
    ##           order       family             genus
    ##           (chr)        (chr)             (chr)
    ## 1 Passeriformes Fringillidae          Linurgus
    ## 2            NA           NA Pseudochloroptila
    ## 3            NA           NA            Alario
    ## 4 Passeriformes Fringillidae           Serinus

``` r
print(ebirdorders[1402:1409,])
```

    ## Source: local data frame [8 x 3]
    ## 
    ##           order        family          genus
    ##           (chr)         (chr)          (chr)
    ## 1 Passeriformes Campephagidae    Campochaera
    ## 2            NA            NA    Malindangia
    ## 3 Passeriformes Campephagidae         Lalage
    ## 4            NA            NA      Celebesia
    ## 5            NA            NA Cyanograucalus
    ## 6            NA            NA      Analisoma
    ## 7            NA            NA      Edolisoma
    ## 8 Passeriformes   Neosittidae  Daphoenositta

We can make an ugly `for` loop to deal with this situation, in which it checks the adjacent non-`NA` values, and if they are identical, uses them to replace the `NA`s in between:

``` r
for (i in 1:nrow(ebirdorders)) {
  if (is.na(ebirdorders$family[i])) {
    nextval <- which(!is.na(ebirdorders$family[i+1:(i+9)]))[1]
    nextval2 <-which(!is.na(ebirdorders$order[i+1:(i+9)]))[1]
    if (identical(ebirdorders$family[i-1], ebirdorders$family[i+nextval])) {
      ebirdorders$family[i] <- ebirdorders$family[i-1]
    }
    if (identical(ebirdorders$order[i-1], ebirdorders$order[i+nextval2])) {
      ebirdorders$order[i] <- ebirdorders$order[i-1]
    }
  }
}

ebirdorders %>% filter(is.na(order))
```

    ## Source: local data frame [0 x 3]
    ## 
    ## Variables not shown: order (chr), family (chr), genus (chr)

``` r
ebirdorders %>% filter(is.na(family))
```

    ## Source: local data frame [8 x 3]
    ## 
    ##           order family          genus
    ##           (chr)  (chr)          (chr)
    ## 1    Piciformes     NA   Calorhamphus
    ## 2 Passeriformes     NA    Euchrepomis
    ## 3 Passeriformes     NA    Ceratopipra
    ## 4 Passeriformes     NA      Celebesia
    ## 5 Passeriformes     NA Cyanograucalus
    ## 6 Passeriformes     NA      Analisoma
    ## 7 Passeriformes     NA      Edolisoma
    ## 8 Passeriformes     NA  Grammatoptila

There are now only 8 genera with missing family values, but none that are missing their respective order as we managed to fill in all of those. The remaining gaps are from `NA`s between different families, hence we can't infer the family based on their position in the checklist. We'll have to add these by hand:

``` r
for (i in 1:nrow(ebirdorders)) {
  if (is.na(ebirdorders$family[i])) {
    if (ebirdorders$genus[i] %in% c("Calorhamphus")) {
      ebirdorders$family[i] <- "Megalamidae"
      }
    if (ebirdorders$genus[i] %in% c("Ceratopipra")) {
      ebirdorders$family[i] <- "Pipridae"
      }
    if (ebirdorders$genus[i] %in% c("Celebesia","Cyanograucalus",
                                      "Analisoma", "Edolisoma")) {
      ebirdorders$family[i] <- "Campephagidae"
      }
    if (ebirdorders$genus[i] %in% c("Grammatoptila")) {
      ebirdorders$family[i] <- "Leiothrichidae"
      }
    if (ebirdorders$genus[i] %in% c("Euchrepomis")) {
      ebirdorders$family[i] <- "Thamnophilidae"
      }
  }
}

anyNA(ebirdorders$family)
```

    ## [1] FALSE

Voila! No more missing values! We can join this list of genera back to the full taxonomic checklist with all species, or to your own checklist of sightings downloaded from eBird's [website](http://ebird.org/ebird/downloadMyData):

``` r
ebirdfull <- left_join(ebirdtax, ebirdorders, by = "genus")
anyNA(ebirdfull$family)
```

    ## [1] FALSE

So, there it is! A *slightly* complicated way to add order and family information to the eBird taxonomic checklist. If there's and eBird developers reading this, please feel free to add it to the eBird-1.1-SpeciesReference results fields! ;)
