#Data quality

This is an informatics study that focuses on data quality (rather than a clinical question).

#How to execute the study (old)

##Step 1
Execute the latest version of Achilles

##Step 2.
Install the package DataQuality. The --no-multiarch eliminates errors on some Windows computers (it is not always necessar). 

```R
install.packages("devtools")
library(devtools)
install_github("ohdsi/StudyProtocolSandbox/DataQuality",args="--no-multiarch")
library(DataQuality)

```

##Step 3. 
Execute the following code:

```R
#use your previous connectionDetails object with username and psw for database
#or get it from an external file 
source('c:/r/conn.R')  #

library(DataQuality)


workFolder <- 'c:/temp/DQ'  #folder must exist (use forward slashes)

#populate database parameters
cdmDatabaseSchema <-'ccae'
resultsDatabaseSchema <-'ccae' #at most sites this likely will not be the same as cdmDatabaseSchema


executeDQ(connectionDetails = connectionDetails,cdmDatabaseSchema = cdmDatabaseSchema,workFolder = workFolder)

packageResults(connectionDetails,cdmDatabaseSchema,workFolder)

#to see what is being used, inspect the resulting zip file (this small data subset is being submitted to the study team)
```

##Step 4
This step is optional. ZIP file can also be emailed.
To use OHDSI mechanism for data submission, ask the study PI (Vojtech Huser) via email to provide you studyKey and studySecret keys to allow you to upload the data to a studyBucket.

To submit results, use the following command (study is using OHDSI bucket mechanism (Amazon S3 technology) 

```R
submitResults(exportFolder =file.path(workFolder,'export'),
              studyBucketName = 'ohdsi-study-dataquality',
              key=studyKey,
              secret =studySecret
              )


```


#Use of output data

If any site requires a formal Data Use Agreement between the your site and the Data Quality Study Principal Investigator please fill in the  Data Use Agreement template (see  the extras folder) and email it to the DQ study PI (for second signature for data recipient).

We plan to compare several sites on how they use Achilles, however the final manuscript or any of its apendices will not expose publically any details about any given site.

If you share your site's data with the DataQuality study principal investigator or the study team, it will be only for the purpose of comparison. All compared sites will be refered to under meaningless site ID. All results will be pooled together so that any site or dataset will be hidden in a crowd of several sites/datasets.

This principle was used in the initial study of Achilles Heel output. (precursor to this study)


#Additional tools
The tool relies on new computations done via Achilles. Using Achilles version >1.2 is required


# ARCHIVED OLD  CONTENT - How to execute the study (old)

The study only needs current  Achilles package and only extracts from those results are used in the DQ study.

Step 1 is to execute latest version of Achilles. Step 2 is to execute the R code below:

```R
#use your previous connectionDetails object with username and psw for database

# use this for single dataset Iris ZIP file generation
 dataLinks=c('mdcr_v5');resultsLinks=c('nih')


#use this for multiple datasets ZIP file process (modify the strings)
 dataLinks=c('ccae_v5','mdcr_v5','mdcd_v5')
 resultsLinks=c('ccae_v5_results','mdcr_v5_results','mdcd_v5_results')
      #ignore this line resultsLinks=c('nih','nih','nih')



#make sure you have the lastest Achilles
library(Achilles)

for (i in seq_along(dataLinks)){
 print(dataLinks[i])

 cdmDatabaseSchema=dataLinks[i];resultsDatabaseSchema=resultsLinks[i]
 
 
  #get Heel output 
  heelRes<-Achilles:::fetchAchillesHeelResults(connectionDetails,resultsDatabaseSchema)

  #get Derived Measures  (this function will be rewritten to not use Iris package at all)
  heelDerivedMeasuresTable<-Iris::fetchResultsTable(connectionDetails,resultsDatabaseSchema,'achilles_results_derived')
  
  #optionaly include Heel Derived measures output
  write.csv(heelDerivedMeasuresTable,paste0(connectionDetails$schema,'-iris_part-',99,'.csv'),na='',row.names=F)


  #optionaly include Heel output
  write.csv(heelRes,paste0(connectionDetails$schema,'-iris_part-',0,'.csv'),na='',row.names=F)


}

zip('iris-export.zip',files='*iris_part*.csv')
#inspect the zip file to see what is being exported

```
