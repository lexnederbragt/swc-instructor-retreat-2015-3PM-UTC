#Assembly using velvet

Heavily based on [material](https://github.com/lexnederbragt/INF-BIOx121_fall2014_de_novo_assembly) developed for the *de novo* assembly part of the INF-BIOx121 course "High Throughput Sequencing technologies and bioinformatics analysis" at Univ. of Oslo Fall 2014. License: CC0.


###*De novo* assembly of Illumina reads using velvet

###Installing Velvet

This material starts from an Amazon EC2 instance. You can also install velvet on your own server, or even on your unix or Mac laptop. Velvet is also available from Homebrew.

###Installing Velvet on Amazon EC2

Start an EC instance at (<https://aws.amazon.com>), make sure to choose the **m4.xlarge** image with ubuntu 14.04. If you have never done this you could follow the instructions here: <http://angus.readthedocs.org/en/2015/amazon/start-up-an-ec2-instance.html>. You will need to set up a key pair and be able to log into the the instance through ssh.

```
ssh -i Amazon.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com
```

Update the base software and install some needed software:

```
sudo apt-get update && sudo apt-get install -y g++ gcc make git zlib1g-dev python
```

Download the source code and unpack it:

```
wget https://www.ebi.ac.uk/~zerbino/velvet/velvet_1.2.10.tgz
tar -xvzf velvet_1.2.10.tgz
```

We'll compile velvet allowing for kmers up to 127 and add it to our `PATH` variable:

```
cd velvet_1.2.10
make 'MAXKMERLENGTH=127'
export PATH=${PWD}:$PATH
```
Check that you actually can run velvet by typing

```
cd
velveth
```

You should see the velvet help text.

###Downloading reads

```
cd
mkdir data
cd data
wget https://www.dropbox.com/s/kopguhd9z2ffbf6/MiSeq_Ecoli_MG1655_50x_R1.fastq
wget https://www.dropbox.com/s/i99h7dnaq61hrrc/MiSeq_Ecoli_MG1655_50x_R2.fastq
```
This should take seconds on amazon. File sizes are 250 MB.

###Assembling short-reads with Velvet

We will use Velvet to assemble Illumina reads on their own. Velvet uses the *de Bruijn graph* approach. 

We will assemble *E. coli K12* strain MG1655 which was sequenced on an Illumina MiSeq. The instrument read 150 bases from each direction.

We will first use paired end reads only: 

###Building the Velvet Index File

Velvet requires an index file to be built before the assembly takes place. We must choose a *k-* mer value for building the index. Longer *k-* mers result in a more stringent assembly, at the expense of coverage. There is no definitive value of *k* for any given project. However, there are several absolute rules:

* *k* must be less than the read length
* it should be an odd number. 

We are going to run Velvet in single-end mode, *ignoring the pairing information*. Later on we will incorporate this information.

Create the assembly folder in a suitable location:

```
cd 
mkdir velvet
cd velvet
```

Find a value of *k* (between 21 and 127) to start with, and record your choice in this google spreadsheet: <https://bit.ly/SWDCretreatVelvet>. Run `velveth` to build the hash index (see below).

Program|Options|Explanation
-------|-------|-------------
velveth||Build the Velvet index file|
||foldername|use this name for the results folder
||value\_of\_k|use k-mers of this size
||-short|short reads (as opposed to long, Sanger-like reads)
||-separate|read1 and read2 are in separate files
||-fastq|read type is fastq

Build the index as follows:

```
velveth ASM_NAME VALUE_OF_K \  
-short -separate -fastq \  
/home/ubuntu/data/MiSeq_Ecoli_MG1655_50x_R1.fastq \  
/home/ubuntu/data/MiSeq_Ecoli_MG1655_50x_R2.fastq  
```
**NOTES** 

* Change `ASM_NAME` to something else of your choosing
* Change `VALUE_OF_K` to the value you have picked
* The command is split over several lines by adding a space, and a `\` (backslash) to each line. This trick makes long commands more readable. If you want, you can write the whole command on one line instead.

After `velveth` is finished, look in the new folder that has the name you chose. You should see the following files:

```
Log
Roadmaps
Sequences
```


The '`Log`' file has a useful reminder of what commands you typed to get this assembly result, for reproducing results later on. '`Sequences`' contains the sequences we put in, and '`Roadmaps`' contains the index you just created.

Now we will run the assembly with default parameters:

```
velvetg ASM_NAME
```

Velvet will end with a text like this:

`Final graph has ... nodes and n50 of ..., max ..., total ..., using .../... reads`

The number of nodes represents the number of nodes in the graph, which (more or less) is the number of contigs. Velvet reports its N50 (as well as everything else) in 'kmer' space. The conversion to 'basespace' is as simple as adding k-1 to the reported length.

Look again at the folder `asm_name`, you should see the following extra files:

`contigs.fa`  
`Graph`  
`LastGraph`  
`PreGraph`  
`stats.txt`

The important files are:

`contigs.fa` - the assembly itself  
`Graph` - a textual representation of the contig graph  
`stats.txt` - a file containing statistics on each contig

Log your results in this google spreadsheet: <https://bit.ly/SWDCretreatVelvet>


**We will discuss the results together and determine *the optimal* k-mer for this dataset.**


----------
Part of the assembly module developed for the course "High Throughput Sequencing technologies and bioinformatics analysis" at the University of Oslo. See the [2015 course material](http://inf-biox121.readthedocs.org/en/2015/Assembly/).

Source of the reads: [http://www.illumina.com/science/data_library.ilmn](http://www.illumina.com/science/data_library.ilmn), random subsampling using seqtk [https://github.com/lh3/seqtk](https://github.com/lh3/seqtk)

