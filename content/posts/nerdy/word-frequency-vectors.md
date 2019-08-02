+++
summary = "Word frequency vectors applied on evolutionary distances between DNA sequences and on bit error rates."
logo = "/imgs/logos/phylogenetic-tree.png"
showLogo = true
draft = false
hasMath = true
date = "2017-03-24T18:45:50+01:00"
title = "Word frequency vectors and their uses"
tags = [
  "tech", "math"
]
+++

During some work with a Bioinformatics group at the University of Göttingen, we would use `word frequency vectors` to determine *evolutionary distances between DNA or Protein sequences*.

A DNA sequence, when saved digitally, is nothing more than a string of characters. Each one corresponds to one of the four possible nucleotides. You could call this DNA's alphabet \\(\sum_{DNA}=\\{A,C,T,G\\}\\). A chain of these nucleotides makes up the structure of a DNA strand.

Let's say you have somehow obtained two DNA sequences:
$$S_1=AACCGT$$
$$S_2=AACGT$$

Notice that they are *not equal in length*: \\(l(S\_1)=6 \neq l(S\_2)=5\\). Three different mutations can happen to DNA sequences over time:

1. *Substitution* happens when a nucleotide turns into another one.
+ *Deletion* is when a nucleotide disappears.
+ *Insertion* is when a nucleotide appears.

The **evolutionary distance** is then a distance measure that tells us *how different* two sequences are. A naive approach could be comparing each character with the one in the same position in the other sequence. This works fine for the first three characters, but then it goes all wrong:

```
AACCGT (S1)
AACGT
✓✓✓✕✕✕
```

It sure looks like the second `C` has been *deleted* from \\(S\_1\\). And this **character-wise checking fails if characters can be removed from one of the strings**.   
We are counting 3 errors in 6 positions using this method, resulting in a distance of \\( \frac{3}{6}=0.5 \\). Really, only one character had been deleted, so we should have found a distance of \\( \frac{1}{6} \\).

Now before word frequency vectors help with this problem, let's make a quick jump over to the field of Computer Networks. Here we are often interested in how well a packet transmitted over a channel is received. Depending on the specific channel characteristics, different errors can occur to transmissions:

1. *Bit flips* happen when a transmitted `1` is flipped to a `0` or vice-versa.
+ *Bit deletions* can also happen when part of the data stream is distorted.
+ *Bit additions* might also be possible in noisy channels.

See a similarity? The two fields have exactly the same problem. In Computer Networks you would ask how high the error rate of a transmission is, given the input and the observed output.

#### How can word frequency vectors help us?
Taking \\(S\_1\\) as an example again, we can construct its word frequency vector for all words of length `3` (or any other number \\(n>0)\\). Start at the beginning of the sequence and note down the three characters you see:

```
AACCGT
^^^
```

Now shift over the sequence until you can't find another word of length `3`:

```
  AACCGT
1 ^^^
2  ^^^
3   ^^^
4    ^^^
```

You remember all occurrences of the found words:

```
AAC 1
ACC 1
CCG 1
CGT 1
```

And so you've found the `3`-word-frequency-vector \\( v \\). What do we know about this vector? It holds the number of times you observe a character triplet. So except for the four triplets we've found, all other entries in the vector are `0`.   
How many entries there? Well, as many as there are triplets. Each character \\(c\in\sum\_{DNA}\\) has exactly 4 possibilities. So that's 4 possibilities of choosing the first character. Then you again have 4 possibilties for the second one, and again for the third one. So in total we have

$$4\cdot 4\cdot 4=64=4^3=\left| \sum\_{DNA} \right| ^n$$

which is the dimensionality of our triplet frequency vector on the DNA alphabet. This is easily adjusted for longer and shorter words, or different alphabets, like Proteins or English.

#### What now?
We know how to find these vectors for any sequence now. So to compare two sequences, we find the two vectors and calculate a distance between them. To start with, try the Euclidean distance \\( d\_{euclidean}(v,w) = \sqrt{\sum\_{i=1}^n (v_i - w_i)^2} \\).

```
  AACGT (S2)
1 ^^^
2  ^^^
3   ^^^
```

so \\( w \\) in our case looks like

```
AAC 1
ACG 1
CGT 1
```

they have the triplets `AAC`, `CGT` in common, and \\( v \\) has `ACC` and `CCG`, while \\( w \\) has `ACG`. Their distance is

$$ d\_{euclidean}(v,w)=\sqrt{(1-1)^2 + (1-1)^2 + (1-0)^2 + (1-0)^2 + (0-1)^2} = 1,732 $$

Unfortunately this is not the error rate of \\( \frac{1}{6} \\) we were looking for either. However, it does enable us to compare more than 2 sequences with each other, because their pairwise distance metrics tell us how distant they are in relation to each other.   
In the world of Computer Networks we can compare different strategies that have an effect on bit error chances. Each strategy's resulting vector distance allows us to compare strategies and how they affect bit error rates.

This pairwise comparison of sequences, or strategies, wouldn't have been possible with our naive character-wise checking idea. With word frequency vector distances we can depict the similarity of species by finding distances between their genomes' DNA sequences and visualizing them in a [phylogenetic tree](https://en.wikipedia.org/wiki/Phylogenetic_tree), or try different networking techniques and protocols and see how they perform compared to each other.

An implementation in `C++` can be found [in this post]({{< relref "/posts/nerdy/lexicographical-index.md" >}}).
