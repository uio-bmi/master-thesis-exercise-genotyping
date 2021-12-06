# Master thesis on genotyping

This is a small programming exercise connected to [this master thesis](https://www.mn.uio.no/ifi/studier/masteroppgaver/bmi/understanding-genetic-variation-with-genome-graphs.html). This exercise is meant for students working on this master thesis to get a feel of the data and problems. 

Note: Parts of these exercises may be a bit tricky, and/or require special knowledge of some Python features. Feel free to reach out if you are stuck!


## Exercise 1: Representing and reading DNA
We have seqeuenced the DNA of an individual, and are left with a [fasta file](https://en.wikipedia.org/wiki/FASTA_format) containing some hundred thousand reads. Each read consists of 150 bases (A, C, T or G) and is represented by two lines, a header line and a line with the read:

Download [this fasta](reads.fa) file and have a look at it visually (e.g. by using the unix `head` command) so that you get familiar with the format.

**Exercise**: Write a Python class `Reads` that takes a file name as argument to the constructor. Make a method `get_all()` in this class that returns a list of all the sequences (not the headers), so that you can run the code like this:

```python
reads = Reads("reads.fa")
reads_as_list = reads.get_all()
print("There are %d reads in this file" % len(reads))
```


## Exercise 2: Counting kmers (subsequences)
We want to count how many times subsequences exist in the read dataset. We use the word **kmer** when talking about such subsequences, where **k** means a sequence of length k. For instance, AAAA is a 4-mer and ATG is a 3-mer. 

If we have the read `AAACCCTAA` then all possible 3-mers will be:
```
AAA
AAC
ACC
CCC
CCT
CTA
TAA
```

**Exercise:** Write code that makes a "Kmer Index" that lets you efficiently query how many times a certain kmer can be found in the whole read data set. Use a dictionary as the underlying datastructure (keys are kmers and values are counts). Your code may spend some time preprocessing the reads (e.g. count kmers and create the count dict), but after preprocessing we want to have a datastructure that lets us efficiently query how many time a certain kmer exists in the read data set.

Note 1: There shouldn't be any difference between uppercase and lowercase, so `AcT` is the same kmer as `act`.

Note 2: We can let a kmer index always be for a specific predetermined **k** (e.g. k=10). So when the user wants to create a kmer index, he/she will need to specify k. Thus, k should be an argument when creating the index, and we can only ask for the frequency of kmers with length matching this k.

It is up to you exactly how you implement a solution to this problem. One option is to make a class `KmerIndex` which has a dictionary that holds the kmer counts, and e.g. a method like `get_kmer_count(kmer)` for getting the count of a kmer. This class could then take a Reads-object and a **k** as arguments in the constructor, so that the index is always created from a Reads-object. 


## Exercise 3: Avoid reading all the reads into a list
You may have noticed that reading all the reads into a list takes some time. This solution also means that all reads will be in memory. Since our KmerIndex doesn't need all the reads at the same time, it is better memory-wise to read one read, get the kmers from that read, count them, update the index, and then proceed to read the next read.

You can do this using the `yield`-keyword in Python (google this if you have not used yield before). Implement a new method `get_reads()` in the Reads class. This method should open the fasta file, and yield one line at a time. 

You can then use this method like this when you count kmers:
```python
reads = Reads("...")
for read in reads.get_reads():
    # get kmers from this read, update the index
    
```

This way, Python will only have a single read in memory at once.


## Exercise 4: Implement a way of saving/loading this index to/from file
Since preprocessing the reads and getting kmer counts is slow for a large file, we want to be able to save our index to file, so that we only need to make it once. The next time we want to use the index, we can just read it from file (assuming this will be quick).

**Exercise** Try making a way of saving this index to file using the Pickle-functionality in Python (you may need to google Pickle). Implement a `to_file`-method in the KmerIndex class. This method takes a filename, and saves the dictionary to file. 

Also implement a `from_file`-method. A good idea is to make this method a class method (google if you are new to class methods). That way, the method doesn't need to be called on a KmerIndex-object, but will instead create a KmerIndex-object, so that we can create an object like this:

```python
kmers = KmerIndex.from_file("mykmers")
# kmers will now be a KmerIndex object
```

A way of creating a class-method may look like this:
```python
@classmethod
def from_file(cls, file_name):
    # read the dict from file
    index_dict = ...
    # return a new object 
    # you need to make sure can create an object without specifying a Reads-object
    object = cls()
    object.set_internal_dict(index_dict)
```

## Exercise 5: Test everything
Make a small test in a separate test-file that reads a few reads from a fasta file (not the big reads file). Create a KmerIndex, ask for the count  of a specific kmer, and assert that the count is correct.

Also test that writing the KmerIndex to file and reading it again gives you back the same index (with same counts).

## Exercise 6: Make the index faster and more memory-efficient
Using a dict as the underlying structure in our index is not optimal, since dicts require a lot of memory in Python. Also, even though dicts are supposed to enable quick lookup of keys, this is not true when dicts grow large (not O(1) anymore).

Assuming we are instead able to represent each kmer as an integer (hash), we could use a numpy array to represent the frequencies, where the positions in the numpy array would refer to the kmer hash, and at each such position the value would be the frequency.

Example:
 * Assume we have some rule that says that AAA is always 0, AAC is 1, AAT is 2, AAG is 3, ACA is 4, and so on (the rule is not important, only that every kmer maps to a unique int).
 * Then the numpy array with elements `[0, 1, 0, 4, ...]` would mean that AAA has frequency 0, AAC has frequency 1, AAT has frequency 0 and so on.

Looking up numbers in an index like this is extremely efficient since looking up values in a numpy array is efficient. The only downside is that we need to hash each kmer to an int-value.

### Exercise 6a: Hashing of kmers
Make a function `hash_kmer` that takes a kmer as argument. The function should return an int which is the hash for this kmer. The hash function should use this rule:
* Every base has a value. We let A=0, C=1, T=2 and G=3 (should be case insensitive).
* The hash should then be the sum of every base's value multiplied by 4 to the power of (k-i-1) where is the position of this base in the read.

Example:
* We have the kmer ATG meaning k=3
* Numerically, the bases will be `0,2,3`
* We take the first base (0) and multiply it with 4 to the power of k minus base position minus 1, and get `0 * 4**(3-0-1) = 16`
* We take the second base (2) and multiply it similary with 4 to the power of k-1-1 and get 2 * 4**1 = 8
* The third base (3) gives us only 3 (since 4 to the power of 0 is 1).
* The hash is the sum of all these numbers: 16+8+3 = 27

Implement `hash_kmer` so that it uses this rule. Note: This rule will only work for **k** where the resulting sum is not too big (not a higher number than we can represent with uint64). You can assume from now that k is always pretty small, e.g. < 15. It is a good idea.

### Exercise 6b:  Make a new KmerIndex with numpy as backend
Using the hash-function, write a new class `NumpyKmerIndex` that instead of a dict uses a numpy array as "backend". 

When creating the index, you will first need to make an empty numpy array (using np.zeros) that is big enough so that it can keep the count of any kmer of length k (compute the maximum int that any such kmer can hash to).

The `to_file` and `from_file` methods will now be much faster, since you can use np.save and np.load with the numpy array. 

This new index could have two method for getting counts of kmers, one taking a sequence as argument and one taking a hashed kmer (integer) as input (assuming the kmer has been pre-hashed).

### Exercise 6c
Benchmark the two KmerIndexes. Make a small test-case where you ask for the count of all kmers from e.g. the first 1000 lines of the reads.fa file.

How much time does it take to only ask for the counts using the two indexes (not including hashing of kmers)? How long time does it take to hash the kmers before querying the index?

### Bonus-exercise
You probably notice that hashing the kmers takes a substantial amount of time.

Try to see if you can come up with a solution using numpy to get the hashes faster.

Hints: 
- You can convert a sequence to a numeric array (where every base has been replace with an integer) using np.where on a numpy char array.
- If you have a numeric array representing the sequence, you can compute all the kmer hashes using np.convolve. This is not straightforward, but is possible if you understand how np.convolve works.

If you manage to write code for getting all the kmers from a read fast, you could then make a method in NumpyKmerIndex that gives you the counts of multiple kmers (having a numpy array as argument and returning a numpy array). This will give us a way of efficiently query the frequency for multiple kmers.
