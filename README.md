# FASTdoop

### Introduction

FASTdoop is a generic Hadoop library for the management of FASTA and FASTQ files. It includes
three input reader formats with associated record readers. These readers are optimized to
read data efficiently from FASTA/FASTQ files in a variety of settings. They are:

- FASTAshortInputFileFormat: optimized to read a collection of short sequences from a FASTA file.
- FASTAlongInputFileFormat: optimized to read a very large sequence (even gigabytes long) from a FASTA file.
- FASTQInputFileFormat: optimized to read a collection of short sequences from a FASTQ file.


### Using FASTdoop on Spark

Starting from version X, FASTdoop can be used also on _Spark_. This can be achieved using the _newAPIHadoopFile_ method of the _JavaSparkContext_ class. This method returns a new RDD whose content is loaded from an input specified file or collection of files. It requires the specification of the input path, of the class to be used for handling that particular _InputFileFormat_, of the class used to represent the key of the new RDD, of the class used to represent the value of the new RDD and of a _Configuration_ object.

This is an example where a file containing one long sequence encoded in FASTA format is loaded using the FASTdoop class _FASTAlongInputFileFormat_:

```java
SparkSession spark = SparkSession.builder().master("local[*]").appName("FASTdoop Test Long").getOrCreate();	
SparkContext sc = spark.sparkContext();
JavaSparkContext jsc = new JavaSparkContext(sc);

Configuration inputConf = jsc.hadoopConfiguration();

String inputPath = "data/big.fasta";

JavaPairRDD<Text, PartialSequence> dSequences2 = jsc.newAPIHadoopFile(inputPath, 
		FASTAlongInputFileFormat.class, Text.class, PartialSequence.class, inputConf);

/* We drop the keys of the new RDD since they are not used */
JavaRDD<PartialSequence> dSequences = dSequences2.values();

for (PartialSequence sequence : dSequences.collect()) {
	System.out.println("ID: " + sequence.getKey());
	System.out.println("Sequence: " + sequence.getValue());
}
```

Instead, this is an example where a file containing one short sequence encoded in FASTA format is loaded using the FASTdoop class _FASTAshortInputFileFormat_:

```java
SparkSession spark = SparkSession.builder().master("local[*]").appName("FASTdoop Test Short").getOrCreate();	
SparkContext sc = spark.sparkContext();
JavaSparkContext jsc = new JavaSparkContext(sc);

Configuration inputConf = jsc.hadoopConfiguration();

String inputPath = "data/short.fasta";

JavaPairRDD<Text, Record> dSequences2 = jsc.newAPIHadoopFile(inputPath, 
		FASTAshortInputFileFormat.class, Text.class, Record.class, inputConf);

/* We drop the keys of the new RDD since they are not used */
JavaRDD<Record> dSequences = dSequences2.values();

for (Record sequence : dSequences.collect()) {
	System.out.println("ID: " + sequence.getKey());
	System.out.println("Sequence: " + sequence.getValue());
}
```

### Building FASTdoop

(Using Maven)
As an alternative to using the provided jar, it is possible to build FASTdoop from scratch
starting from the source files. In this case, the building of FASTdoop would not require the installation of Hadoop,
as it can be managed using the provided Maven project. In this case, it is only required to clone the repository and load the project inside Eclipse or another IDE. Then, the FASTdoop jar could be created by issuing the Maven install procedure (w.g., clicking on the ```Run As > Maven install``` option if using Eclipse).

The Maven dependecies are:
- Apache Hadoop Common 2.7.0
- Apache Hadoop MapReduce Core 2.7.0
- Apache Spark Core 2.3.0
- Apache Spark SQL 2.3.0

The building process can also be issued via terminal, by moving in the FASTdoop main directory and running the following command-line:

```console
mvn install
```

(Using Ant)
Finally, you can build FASTdoop from scratch using the Hadoop libraries installed on your own computer. 
The compilation process uses the __ant__ software (see http://ant.apache.org). Be also sure to have
the ```$HADOOP_HOME``` environment variable set at the Hadoop installation path and, then,
run the following commands from the shell:

```console
cd FASTdoop-1.0/
ant build clean
ant build
ant build createjar
```

At the end of the compilation, the FASTdoop-1.0.jar file will be created in the current
directory.


### Usage Examples

FASTdoop comes with three test classes that can be used to parse the content of FASTA/FASTQ
files. The source code of these classes is available in the src directory. The following 
examples assume that the java classpath environment variable is set to include the jar files
of a standard Hadoop installation (>=2.7.0).

Example 1: Print on screen all the short sequences contained in FASTdoop-1.0/data/short.fasta

```console
java -cp FASTdoop-1.0.jar fastdoop.test.TestFShort data/short.fasta
```

Example 2: Print on screen the long sequence contained in fastdoop-1.0/data/long.fasta

```console
java -cp FASTdoop-1.0.jar fastdoop.test.TestFLong data/long.fasta
```

Example 3:  Print on screen all the short sequences contained in fastdoop-1.0/data/short.fastq

```console
java -cp FASTdoop-1.0.jar fastdoop.test.TestFQ data/short.fastq
```
