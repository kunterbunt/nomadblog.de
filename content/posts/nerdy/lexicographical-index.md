+++
draft = false
hasMath = true
tags = [
  "tech", "math"
]
date = "2017-03-28T14:21:39+02:00"
title = "Lexicographical index calculation"
summary = "Finding the index in a container that holds the frequency of elements in lexicographical order. Includes C++ source code for word frequency vectors (that makes use of this index calculation)."
logo = ""
showLogo = false
+++

Building upon the idea of [word frequency vectors]({{< relref "/posts/nerdy/word-frequency-vectors.md" >}}), when implementing the idea in a language of your choice, you might encounter the problem of instantiating a container that is going to store the observed frequencies. In an efficient implementation you could instantiate such a container that holds the right number of items, which is \\(\left| \sum\_{language} \right| ^n\\), e.g. for DNA the alphabet is \\(\sum_{DNA}=\\{A,C,T,G\\}\\). \\(n\\) is the word length for which you're finding the frequency vector.

So, in `C++`, the container could look like this:

```c++
std::vector<unsigned long> mFrequencyVector(getDimension(), 0UL);
```

`mFrequencyVector` holds one counter each for every word of length `n`, and you can easily assume it holds these elements in lexicographical order.
Now you're counting the occurrences of the words and want to update the respective counter in `mFrequencyVector`, but which one is it?  For `n=2` you'll find:

```
0 AA
1 AC
2 AG
3 AT
4 CA
5 CC
6 CG
7 CT
8 GA
...
15  TT
```

Consider your `alphabet` a vector `['A', 'C', 'G', 'T']`. Each contained character has an index corresponding to `[0, 1, 2, 3]`.
For arbitrary `n` you can compute the index of a `word` as

$$ \left| \sum \right| ^{n-1} \cdot index(word.at(0)) + \left| \sum \right| ^{n-2} \cdot index(word.at(1))\; + $$
$$ \ldots + \left| \sum \right| ^{0} \cdot index(word.at(n-1)) $$

For example, `ACT` has the index
```
4^2 * Index(A) + 4^1 * Index(C) + 4^0 * Index(T)
<=>
16 * 0 + 4 * 1 + 1 * 3 = 7
```

A full implementation of a `word frequency vector class` in `C++` is this:

```c++
#include <vector>
#include <string>
#include <cmath>
#include <exception>

class WordFrequencyVector {
  public:
    static WordFrequencyVector get(const unsigned int& wordLength, const std::vector<char>& alphabet, const string& message) {
      WordFrequencyVector vec(wordLength, alphabet);
      string copy = message;
      std::transform(copy.begin(), copy.end(), copy.begin(), ::toupper);

      vec.find(copy);
      return vec;
    }

    unsigned long at(const size_t& position) const {
      return mFrequencyVector.at(position);
    }

    unsigned long at(const std::string& word) const {
      return mFrequencyVector.at(getIndex(word));
    }

    size_t size() const {
      return mFrequencyVector.size();
    }

    double getDistance(const WordFrequencyVector& other) const {
      return WordFrequencyVector::euclideanDistance(mFrequencyVector, other.mFrequencyVector);
    }

    /**
     * @param v1
     * @param v2
     * @return The euclidean distance between v1 and v2.
     */
    static double euclideanDistance(std::vector<unsigned long> v1, std::vector<unsigned long> v2) {
      double sum = 0;
      for (size_t i = 0; i < min(v1.size(), v2.size()); i++)
        sum += pow(((double) v1.at(i)) - ((double) v2.at(i)), 2);
      return sqrt(sum);
    }

  protected:
    WordFrequencyVector(const unsigned int& wordLength, const std::vector<char>& alphabet)
        : mWordLength(wordLength), mAlphabet(alphabet), mFrequencyVector(std::vector<unsigned long>(getDimension(), 0UL)) {
      // Init-list does all the work.
    }

    void find(const std::string& message) {
      for (size_t i = 0; i + (mWordLength-1) < message.length(); i++) {
        std::string word = message.substr(i, mWordLength);
        mFrequencyVector.at(getIndex(word))++;
      }
    }

  private:
    unsigned int mWordLength;
    std::vector<char> mAlphabet;
    std::vector<unsigned long> mFrequencyVector;

    /**
     * @param word
     * @return The position of 'word' in the frequency vector.
     */
    size_t getIndex(const std::string& word) const {
      size_t alphabetSize = mAlphabet.size(), wordLength = word.length();
      size_t index = 0;
      for (size_t i = 0; i < wordLength; i++)
        index += pow(alphabetSize, wordLength - (i+1)) * getIndex(word.at(i));
      return index;
    }

    /**
     * @param character
     * @return The position of 'character' in the alphabet.
     */
    size_t getIndex(const char& character) const {
      vector<char>::const_iterator i = std::find(mAlphabet.begin(), mAlphabet.end(), character);
      if (i == mAlphabet.end())
        throw invalid_argument(std::string("WordFrequencyVector::getIndex(char) called but char couldn't be found for \'" + std::to_string(character) + "\'!"));
      return (size_t) std::distance(mAlphabet.begin(), i);
    }

    /**
     * @return size(alphabet)^n
     */
    unsigned int getDimension() const {
      return (unsigned int) pow((double) mAlphabet.size(), (double) mWordLength);
    }
};
```
