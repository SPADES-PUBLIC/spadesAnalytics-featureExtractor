# spadesAnalytics-featureExtractor
A window based feature extractor written by researchers at MIT that computes feature vectors from annotated accelerometer data. The generated features include for each axis the mean, the standard deviation, the variance, the minimum, the maximum, the kurtosis, the skewness, the 50th percentile, the sum, the sum of squares, the population variance, the top five frequencies excluding the dc component and their magnitudes, and the correlation between different sensor axes.

Usage
-----
```ShellSession
java -cp [Local directory to jar file] [com.qmedic.spades.task.SpadesSubmit] [-cluster] [MapReduce] [-mode] [Once] [-speed] [32x] [-filesystem] [s3n] [-jar] [s3n directory to jar file] [-classpath] [class name in jar file of s3n] [-input] [data input path] [-key] [key value assigned by AWS] [-secret] [secret value assigned by AWS] [-p] [parameters to be passed to runnable class]
```

* [Local directory to jar file]: The local directory for jar file 
  * Example:  output/SpadesAnalytics.jar (here SpadesAnalytics.jar is a runnable jar file in the local computer)
* [com.qmedic.spades.task.SpadesSubmit]: the class to submit the job to EMR 
* [s3n directory to jar file]: The cloud directory for jar file
  * Example: spades-data/development/user/SpadesAnalytics.jar (SpadesAnalytics.jar is stored in the directory “spades-data/development/user” of s3)
* [class name in jar file of s3n]: The runnable class in the jar file in S3
  * Example: com.qmedic.spades.task.examples.mapreduce.FeatureExtractor.FeatureExtractor
* [data input path]: Cloud directory for source data
  * Note: Its format will be discussed in the section “Data Input Format”
  * Example: spades-data/development/stanford/2/StanfordStudyYouth2/1/MasterSynced/*
* [key value assigned by AWS]: Key value assigned by AWS
  * Note: 20  alphabeta
  * Example: "yourAccessKey"
* [secret value assigned by AWS]: secret value assigned by AWS
  * Note: 40 char
  * Example: "yourSecretKey"
* [parameters to be passed to runnable class]: parameters to be passed to runnable class
  * Note: Its format will be discussed in the section “Parameter Input Format”

Input
-----
In our framework, the data input format for processing data is restricted to the following, a future work item would be to extended and allow for a variety of flexible formats.

* Input Path Folder Format :
  *Must be terminated with a wild card /*
  * The path consists of the following required entries:
    * Bucket Name: (alphanumeric, ‘-’)... e.g. spade-data
    * Release Type: development, production
    * Institution: Stanford
    * Study ID: numeric value for the user within the institution
    * Study Name:  StanfordStudyYouth2
    * Participant ID: acceptable values include (can also be specified as a wild card *)
    * Data source: Alphanumeric + ‘-’...e.g. MasterSynced
  * Optional components of the path that follow include:
    * Year: 4 character representation
    * Month: 2 character representation that goes from 01 to 12
    * Day: 2 character representation that goes from 01 to 31
    * Hour: 2 character representation that goes from 00 to 23
  * Allowed paths include the following formats: spades-data/development/stanford/2/StanfordStudyYouth2/1/MasterSynced/*

Parameter Input
---------------
The parameter input format for passing parameters to runnable class is restricted to the following, a future work item would be extended and allow for a variety of flexible formats.

General parameter input format:
```ShellSession
[Features separated by comma] : [Annotation Input]
```

Example parameter input format:
```ShellSession
mean,std,var,min,max,kurt,per50,skw,sum,sumsq,pvar,fftfreq,corr:WithAll
```

[Features separated by comma] 
* [mean]: output mean value
* [std]: output standard deviation value
* [var]: output variance value
* [min]: output minimum value
* [max]: output maximum value
* [kurt]: output kurtosis value
* [per50]: output the 50th percentile value
* [skw]: output skewness value
* [sum]: output sum value
* [sumsq]: output sum of squares value
* [pvar]: output population variance
* [fftfreq]: output the top five frequencies excluding the dc component and their magnitudes
* [corr]: output the correlation between different sensor axes

[Annotation Input]
* [WithAll]: output all annotations (such as walking-natural, moving-hip-only, unknown)
* [ANNOACTIVITY]: output only motion related annotations, excluding unknown annotations
* [WITHOUTANNO]: no annotation output

Command Line Example
--------------------
```ShellSession
java -cp output/SpadesAnalytics.jar com.qmedic.spades.task.SpadesSubmit -cluster MapReduce -mode Once -speed 32x -filesystem s3n -jar spades-data/development/user/SpadesAnalytics.jar -classpath com.qmedic.spades.task.examples.mapreduce.FeatureExtractor.FeatureExtractor -input spades-data/development/stanford/2/StanfordStudyYouth2/1/MasterSynced/*   -key "yourAccessKey" -secret "yourSecretKey" -duration 4 -cost 20 -p mean,std,var,min,max,kurt,per50,skw,sum,sumsq,pvar,fftfreq,corr:[WithAll/AnnoActivity/WithoutAnno]
```

Output File Name
----------------------------------------
Typically the feature data file will be stored in the metadata directories and will include processed data that can be recomputed from the raw data. The structure of the filename is: 
```ShellSession
[ALGORITHM].[AlgorithmID].[YYYY]-[MM]-[DD]-[hh]-[mm]-[ss]-[mmm]-[P/M][hhmm].csv.gz 
```
* ALGORITHM: Prefix of the file name
* [FEATURE NAME]: Feature name
  * Example: MetaData-FeatureExtractor-2015-10-29-10-50-21-021

Output Content
--------------
For example, the first two lines in the FeatureExtractor.csv file could look like this:
```ShellSession
HEADER_TIME_STAMP, MEAN_SensorID_0, STD_SensorID_0,VAR_SensorID_0, MIN_SensorID_0,MAX_SensorID_0, KURT_SensorID_0,PER50_SensorID_0, SKW_SensorID_0,SUM_SensorID_0, SUMSQ_SensorID_0, PVAR_SensorID_0,FFTFREQ_SensorID_0_1, CORR_SensorID_0_1, ANNOTATION	 	
2011/12/14 9:01:00.000,-0.44,0,0,-0.45,-0.43,NaN,-0.44,NaN,-66.5,29.48,0,43,NaN,Unknown
```
	 	
* HEADER_TIME_STAMP: The time to generate this output
* [MEAN_SensorID_0]: mean value for the first sensor 
* [STD_SensorID_0]: standard deviation value for the first sensor 
* [VAR_SensorID_0]: variance value for the first sensor 
* [MIN_SensorID_0]: minimum value for the first sensor 
* [MAX_SensorID_0]: maximum value for the first sensor 
* [KURT_SensorID_0]: kurtosis value for the first sensor 
* [PER50_SensorID_0]: the 50th percentile value for the first sensor 
* [SKW_SensorID_0]: skewness value for the first sensor 
* [SUM_SensorID_0]: sum value for the first sensor 
* [SUMSQ_SensorID_0]: sum of square value for the first sensor 
* [PVAR_SensorID_0]: population variance for the first sensor 
* [FFTFREQ_SensorID_0_1]: the second frequency component of the first sensor 
* [CORR_SensorID_0_1]: correlation of the first and second sensors
* [ANNOTATION]: annotation
