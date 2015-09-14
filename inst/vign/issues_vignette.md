<!--
%\VignetteEngine{knitr::knitr}
%\VignetteIndexEntry{Cleaning data using GBIF issues}
%\VignetteEncoding{UTF-8}
-->



Cleaning data using GBIF issues
======

`rgbif` now has the ability to clean data retrieved from GBIF based on GBIF issues. These issues are returned in data retrieved from GBIF, e.g., through the `occ_search()` function. Inspired by `magrittr`, we've setup a workflow for cleaning data based on using the operator `%>%`. You don't have to use it, but as we show below, it can make the process quite easy.

Note that you can also query based on issues, e.g., `occ_search(taxonKey=1, issue='DEPTH_UNLIKELY')`. However, we imagine it's more likely that you want to search for occurrences based on a taxonomic name, or geographic area, not based on issues, so it makes sense to pull data down, then clean as needed using the below workflow with `occ_issues()`.

Note that `occ_issues()` only affects the data element in the gbif class that is returned from a call to `occ_search()`. Maybe in a future version we will remove the associated records from the hierarchy and media elements as they are remove from the data element.

You also get issues data back with `occ_get()`, but `occ_issues()` doesn't yet support working with data from `occ_get()`.

## Get rgbif

Install from CRAN


```r
install.packages("rgbif")
```

Or install the development version from GitHub


```r
devtools::install_github("ropensci/rgbif")
```

Load rgbif


```r
library('rgbif')
```

## Get some data

Get taxon key for _Helianthus annuus_


```r
(key <- name_suggest(q='Helianthus annuus', rank='species')$key[1])
#> [1] 3119195
```

Then pass to `occ_search()`


```r
(res <- occ_search(taxonKey=key, limit=100))
#> Records found [20650] 
#> Records returned [100] 
#> No. unique hierarchies [1] 
#> No. media records [36] 
#> Args [taxonKey=3119195, limit=100, offset=0, fields=all] 
#> First 10 rows of data
#> 
#>                 name        key decimalLatitude decimalLongitude
#> 1  Helianthus annuus 1095851641         0.00000          0.00000
#> 2  Helianthus annuus 1088900309        33.95239       -117.32011
#> 3  Helianthus annuus 1088933055        25.66564       -100.30348
#> 4  Helianthus annuus 1088909392        24.72449        -99.54020
#> 5  Helianthus annuus 1088937716        25.81691       -100.05940
#> 6  Helianthus annuus 1088944416        26.20518        -98.26725
#> 7  Helianthus annuus 1090389390        59.96150         17.71060
#> 8  Helianthus annuus 1092889365        32.71840       -114.75603
#> 9  Helianthus annuus 1092889645         1.27617        103.79136
#> 10 Helianthus annuus 1098903927        29.17958       -102.99551
#> ..               ...        ...             ...              ...
#> Variables not shown: issues (chr), datasetKey (chr), publishingOrgKey
#>      (chr), publishingCountry (chr), protocol (chr), lastCrawled (chr),
#>      lastParsed (chr), extensions (chr), basisOfRecord (chr), taxonKey
#>      (int), kingdomKey (int), phylumKey (int), classKey (int), orderKey
#>      (int), familyKey (int), genusKey (int), speciesKey (int),
#>      scientificName (chr), kingdom (chr), phylum (chr), order (chr),
#>      family (chr), genus (chr), species (chr), genericName (chr),
#>      specificEpithet (chr), taxonRank (chr), dateIdentified (chr),
#>      elevation (dbl), elevationAccuracy (dbl), stateProvince (chr), year
#>      (int), month (int), day (int), eventDate (chr), lastInterpreted
#>      (chr), identifiers (chr), facts (chr), relations (chr), geodeticDatum
#>      (chr), class (chr), countryCode (chr), country (chr), recordNumber
#>      (chr), rights (chr), municipality (chr), rightsHolder (chr),
#>      ownerInstitutionCode (chr), type (chr), occurrenceID (chr),
#>      collectionCode (chr), identifiedBy (chr), gbifID (chr),
#>      occurrenceRemarks (chr), institutionCode (chr), datasetName (chr),
#>      catalogNumber (chr), recordedBy (chr), locality (chr), language
#>      (chr), identifier (chr), modified (chr), references (chr),
#>      verbatimEventDate (chr), verbatimLocality (chr),
#>      http...unknown.org.occurrenceDetails (chr), taxonID (chr), eventTime
#>      (chr), identificationID (chr), coordinateAccuracy (dbl), depth (dbl),
#>      depthAccuracy (dbl), county (chr), informationWithheld (chr),
#>      verbatimElevation (chr), georeferenceSources (chr), habitat (chr),
#>      fieldNotes (chr)
```

## Examine issues

The dataset `gbifissues` can be retrieved using the function `gbif_issues()`. The dataset's first column `code` is a code that is used by default in the results from `occ_search()`, while the second column `issue` is the full issue name given by GBIF. The third column is a full description of the issue.


```r
head(gbif_issues())
#>    code                              issue
#> 1   bri            BASIS_OF_RECORD_INVALID
#> 2   ccm         CONTINENT_COUNTRY_MISMATCH
#> 3   cdc CONTINENT_DERIVED_FROM_COORDINATES
#> 4 conti                  CONTINENT_INVALID
#> 5  cdiv                 COORDINATE_INVALID
#> 6 cdout            COORDINATE_OUT_OF_RANGE
#>                                                                                                    description
#> 1 The given basis of record is impossible to interpret or seriously different from the recommended vocabulary.
#> 2                                                       The interpreted continent and country do not match up.
#> 3                  The interpreted continent is based on the coordinates, not the verbatim string information.
#> 4                                                                      Uninterpretable continent values found.
#> 5                                      Coordinate value given in some form but GBIF is unable to interpret it.
#> 6                                        Coordinate has invalid lat/lon values out of their decimal max range.
```

You can query to get certain issues


```r
gbif_issues()[ gbif_issues()$code %in% c('cdround','cudc','gass84','txmathi'), ]
#>       code                            issue
#> 10 cdround               COORDINATE_ROUNDED
#> 12    cudc COUNTRY_DERIVED_FROM_COORDINATES
#> 23  gass84     GEODETIC_DATUM_ASSUMED_WGS84
#> 39 txmathi           TAXON_MATCH_HIGHERRANK
#>                                                                                                                                 description
#> 10                                                                                  Original coordinate modified by rounding to 5 decimals.
#> 12                                                The interpreted country is based on the coordinates, not the verbatim string information.
#> 23 Indicating that the interpreted coordinates assume they are based on WGS84 datum as the datum was either not indicated or interpretable.
#> 39                                        Matching to the taxonomic backbone can only be done on a higher rank and not the scientific name.
```

The code `cdround` represents the GBIF issue `COORDINATE_ROUNDED`, which means that

> Original coordinate modified by rounding to 5 decimals.

The content for this information comes from [http://gbif.github.io/gbif-api/apidocs/org/gbif/api/vocabulary/OccurrenceIssue.html](http://gbif.github.io/gbif-api/apidocs/org/gbif/api/vocabulary/OccurrenceIssue.html).

## Parse data based on issues

Now that we know a bit about GBIF issues, you can parse your data based on issues. Using the data generated above, and using the function `%>%` imported from `magrittr`, we can get only data with the issue `gass84`, or `GEODETIC_DATUM_ASSUMED_WGS84` (Note how the records returned goes down to 98 instead of the initial 100).


```r
res %>%
  occ_issues(gass84)
#> Records found [20650] 
#> Records returned [100] 
#> No. unique hierarchies [1] 
#> No. media records [36] 
#> Args [taxonKey=3119195, limit=100, offset=0, fields=all] 
#> First 10 rows of data
#> 
#>                 name        key decimalLatitude decimalLongitude
#> 1  Helianthus annuus 1095851641         0.00000          0.00000
#> 2  Helianthus annuus 1088900309        33.95239       -117.32011
#> 3  Helianthus annuus 1088933055        25.66564       -100.30348
#> 4  Helianthus annuus 1088909392        24.72449        -99.54020
#> 5  Helianthus annuus 1088937716        25.81691       -100.05940
#> 6  Helianthus annuus 1088944416        26.20518        -98.26725
#> 7  Helianthus annuus 1090389390        59.96150         17.71060
#> 8  Helianthus annuus 1092889365        32.71840       -114.75603
#> 9  Helianthus annuus 1092889645         1.27617        103.79136
#> 10 Helianthus annuus 1098903927        29.17958       -102.99551
#> ..               ...        ...             ...              ...
#> Variables not shown: issues (chr), datasetKey (chr), publishingOrgKey
#>      (chr), publishingCountry (chr), protocol (chr), lastCrawled (chr),
#>      lastParsed (chr), extensions (chr), basisOfRecord (chr), taxonKey
#>      (int), kingdomKey (int), phylumKey (int), classKey (int), orderKey
#>      (int), familyKey (int), genusKey (int), speciesKey (int),
#>      scientificName (chr), kingdom (chr), phylum (chr), order (chr),
#>      family (chr), genus (chr), species (chr), genericName (chr),
#>      specificEpithet (chr), taxonRank (chr), dateIdentified (chr),
#>      elevation (dbl), elevationAccuracy (dbl), stateProvince (chr), year
#>      (int), month (int), day (int), eventDate (chr), lastInterpreted
#>      (chr), identifiers (chr), facts (chr), relations (chr), geodeticDatum
#>      (chr), class (chr), countryCode (chr), country (chr), recordNumber
#>      (chr), rights (chr), municipality (chr), rightsHolder (chr),
#>      ownerInstitutionCode (chr), type (chr), occurrenceID (chr),
#>      collectionCode (chr), identifiedBy (chr), gbifID (chr),
#>      occurrenceRemarks (chr), institutionCode (chr), datasetName (chr),
#>      catalogNumber (chr), recordedBy (chr), locality (chr), language
#>      (chr), identifier (chr), modified (chr), references (chr),
#>      verbatimEventDate (chr), verbatimLocality (chr),
#>      http...unknown.org.occurrenceDetails (chr), taxonID (chr), eventTime
#>      (chr), identificationID (chr), coordinateAccuracy (dbl), depth (dbl),
#>      depthAccuracy (dbl), county (chr), informationWithheld (chr),
#>      verbatimElevation (chr), georeferenceSources (chr), habitat (chr),
#>      fieldNotes (chr)
```

Note also that we've set up `occ_issues()` so that you can pass in issue names without having to quote them, thereby speeding up data cleaning.

Next, we can remove data with certain issues just as easily by using a `-` sign in front of the variable, like this, removing data with issues `depunl` and `mdatunl`.


```r
res %>%
  occ_issues(-depunl, -mdatunl)
#> Records found [20650] 
#> Records returned [53] 
#> No. unique hierarchies [1] 
#> No. media records [36] 
#> Args [taxonKey=3119195, limit=100, offset=0, fields=all] 
#> First 10 rows of data
#> 
#>                 name        key decimalLatitude decimalLongitude
#> 1  Helianthus annuus 1095851641         0.00000          0.00000
#> 2  Helianthus annuus 1088900309        33.95239       -117.32011
#> 3  Helianthus annuus 1088933055        25.66564       -100.30348
#> 4  Helianthus annuus 1088909392        24.72449        -99.54020
#> 5  Helianthus annuus 1088937716        25.81691       -100.05940
#> 6  Helianthus annuus 1088944416        26.20518        -98.26725
#> 8  Helianthus annuus 1092889365        32.71840       -114.75603
#> 9  Helianthus annuus 1092889645         1.27617        103.79136
#> 10 Helianthus annuus 1098903927        29.17958       -102.99551
#> 11 Helianthus annuus 1092901911        30.27181        -97.83409
#> ..               ...        ...             ...              ...
#> Variables not shown: issues (chr), datasetKey (chr), publishingOrgKey
#>      (chr), publishingCountry (chr), protocol (chr), lastCrawled (chr),
#>      lastParsed (chr), extensions (chr), basisOfRecord (chr), taxonKey
#>      (int), kingdomKey (int), phylumKey (int), classKey (int), orderKey
#>      (int), familyKey (int), genusKey (int), speciesKey (int),
#>      scientificName (chr), kingdom (chr), phylum (chr), order (chr),
#>      family (chr), genus (chr), species (chr), genericName (chr),
#>      specificEpithet (chr), taxonRank (chr), dateIdentified (chr),
#>      elevation (dbl), elevationAccuracy (dbl), stateProvince (chr), year
#>      (int), month (int), day (int), eventDate (chr), lastInterpreted
#>      (chr), identifiers (chr), facts (chr), relations (chr), geodeticDatum
#>      (chr), class (chr), countryCode (chr), country (chr), recordNumber
#>      (chr), rights (chr), municipality (chr), rightsHolder (chr),
#>      ownerInstitutionCode (chr), type (chr), occurrenceID (chr),
#>      collectionCode (chr), identifiedBy (chr), gbifID (chr),
#>      occurrenceRemarks (chr), institutionCode (chr), datasetName (chr),
#>      catalogNumber (chr), recordedBy (chr), locality (chr), language
#>      (chr), identifier (chr), modified (chr), references (chr),
#>      verbatimEventDate (chr), verbatimLocality (chr),
#>      http...unknown.org.occurrenceDetails (chr), taxonID (chr), eventTime
#>      (chr), identificationID (chr), coordinateAccuracy (dbl), depth (dbl),
#>      depthAccuracy (dbl), county (chr), informationWithheld (chr),
#>      verbatimElevation (chr), georeferenceSources (chr), habitat (chr),
#>      fieldNotes (chr)
```

## Expand issue codes to full names

Another thing we can do with `occ_issues()` is go from issue codes to full issue names in case you want those in your dataset (here, showing only a few columns to see the data better for this demo):


```r
out <- res %>% occ_issues(mutate = "expand")
head(out$data[,c(1,5)])
#>                name
#> 1 Helianthus annuus
#> 2 Helianthus annuus
#> 3 Helianthus annuus
#> 4 Helianthus annuus
#> 5 Helianthus annuus
#> 6 Helianthus annuus
#>                                                                             issues
#> 1         COUNTRY_COORDINATE_MISMATCH,GEODETIC_DATUM_ASSUMED_WGS84,ZERO_COORDINATE
#> 2                    COUNTRY_DERIVED_FROM_COORDINATES,GEODETIC_DATUM_ASSUMED_WGS84
#> 3 COORDINATE_ROUNDED,COUNTRY_DERIVED_FROM_COORDINATES,GEODETIC_DATUM_ASSUMED_WGS84
#> 4 COORDINATE_ROUNDED,COUNTRY_DERIVED_FROM_COORDINATES,GEODETIC_DATUM_ASSUMED_WGS84
#> 5 COORDINATE_ROUNDED,COUNTRY_DERIVED_FROM_COORDINATES,GEODETIC_DATUM_ASSUMED_WGS84
#> 6 COORDINATE_ROUNDED,COUNTRY_DERIVED_FROM_COORDINATES,GEODETIC_DATUM_ASSUMED_WGS84
```


## Add columns

Sometimes you may want to have each type of issue as a separate column.

Split out each issue type into a separate column, with number of columns equal to number of issue types


```r
out <- res %>% occ_issues(mutate = "split")
head(out$data[,c(1,5:10)])
#>                name cucdmis gass84 zerocd cudc cdround depunl
#> 1 Helianthus annuus       y      y      y    n       n      n
#> 2 Helianthus annuus       n      y      n    y       n      n
#> 3 Helianthus annuus       n      y      n    y       y      n
#> 4 Helianthus annuus       n      y      n    y       y      n
#> 5 Helianthus annuus       n      y      n    y       y      n
#> 6 Helianthus annuus       n      y      n    y       y      n
```

## Expand and add columns

Or you can expand each issue type into its full name, and split each issue into a separate column.


```r
out <- res %>% occ_issues(mutate = "split_expand")
head(out$data[,c(1,5:10)])
#>                name COUNTRY_COORDINATE_MISMATCH
#> 1 Helianthus annuus                           y
#> 2 Helianthus annuus                           n
#> 3 Helianthus annuus                           n
#> 4 Helianthus annuus                           n
#> 5 Helianthus annuus                           n
#> 6 Helianthus annuus                           n
#>   GEODETIC_DATUM_ASSUMED_WGS84 ZERO_COORDINATE
#> 1                            y               y
#> 2                            y               n
#> 3                            y               n
#> 4                            y               n
#> 5                            y               n
#> 6                            y               n
#>   COUNTRY_DERIVED_FROM_COORDINATES COORDINATE_ROUNDED DEPTH_UNLIKELY
#> 1                                n                  n              n
#> 2                                y                  n              n
#> 3                                y                  y              n
#> 4                                y                  y              n
#> 5                                y                  y              n
#> 6                                y                  y              n
```

## Wrap up

We hope this helps users get just the data they want, and nothing more. Let us know if you have feedback on data cleaning functionality in `rgbif` at _info@ropensci.org_ or at [https://github.com/ropensci/rgbif/issues](https://github.com/ropensci/rgbif/issues).