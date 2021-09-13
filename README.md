
# (1) Finding English Graphemes and Phonemes

### CMU (\~134K Words)

**Source:** [CMU Pronouncing Dictionary](http://www.speech.cs.cmu.edu/cgi-bin/cmudict?in=urge)

**Description:** Dataset containing words and their phonemes. It uses a modified ARPAbet (39 phonemes instead of 44). It does not break words down into syllables or graphemes though. Therefore, additional work is needed to find grapheme-to-phoneme mappings and phoneme-to-grapheme mappings. The dataset also contains many names, abbreviations and acronyms which I'm choosing to filter out.



# (2) Finding English Word Dictionaries

I found several text files of English dictionaries and utilize these, along with a list of proper nouns & adjectives, to filter the CMU Pronouncing Dictionary down to an English vocabulary.

### Webster - Proper Nouns & Adjectives

**Source:** [Scrapmaker Webster Dictionary](https://www.scrapmaker.com/data/wordlists/dictionaries/webster-dictionary.txt)

**Method:** Extracted all capitalized nouns and adjectives from original file (alphabetized at the top)

### Webster (\~282K words)

**Source:** [Scrapmaker Webster Dictionary](https://www.scrapmaker.com/data/wordlists/dictionaries/webster-dictionary.txt)

**Method:** Removed all proper nouns and adjectives from original file (alphabetized at the top)

### OED (\~89K words)
  **Source:** [Oxford-Dictionary-Json](https://github.com/cduica/Oxford-Dictionary-Json)
  
  **Method:** Remove all the oids's and just keep words and definitions.

### Unix (\~200K words)

  **Source:** `/usr/share/dict/words`
  
  **Method:** Unix systems come with a file of English words. Copy into file with `cat /usr/share/dict/words > ~/Desktop/words.txt`

### GradyAugmented (\~122K words)

#### R Code to produce the GradyAugmented dictionary
    require(qdapDictionaries)
    write.csv(GradyAugmented, "<PATH_TO_FILE>/GradyAugmentedR.csv",  row.names = FALSE)

# (3) Whittling CMU Down to English Words

### CMU English Words (\~36K Words)

**Method**: If a CMU Pronouncing Dictionary word is in at least 2 of the above dictionaries (after proper word removal), the word is included in the steps that follow.


# (4) Gather a List of All Possible Phoneme-to-Grapheme Mappings

### Using Researched Phoneme-Grapheme Correspondences

**Source:** [Phoneme-Grapheme Correspondences as Cues to Spelling
    Improvement (1966)](https://files.eric.ed.gov/fulltext/ED128835.pdf)

**Description:** This paper enumerated all of the phoneme-to-grapheme mappings in a list \~17K English words. The phonemes used for CMU and the phonemes for the paper were different. So, I manually went through the mappings, and translated them into a CMU-friendly format. I decided to treat silent letters differently than the paper. For instance, a silent **e** at the end of ape would be transcribed as containing the graphemes **a_e** and **p**. I, instead, treat the silent letter as being part of the nearest phoneme-producing grapheme, which in this case is **p**. The graphemes by my method would be **a** and **pe**.


# (5) Dividing Words Into Graphemes

### Infer Graphemes

**Method:** From the paper, we ostensibly have a dictionary mapping of all possible phonemes-to-graphemes. This should allow us to take the phoneme input from CMU, then cycle through all possible mapped graphemes for each phoneme. Using this procedure, we should be able to derive the correct graphemes of the original word. Due to diphonemes (mergers of two phonemes) and multiple-letter graphemes (like **OUGH** in **th-r-ough**), however, there is ambiguity and some portion of words will have multiple possible graphemic breakdowns.

# (6) Addressing Ambiguity of Graphemes

### (A) Algorithmically Aligning Graphemes and Phonemes

**Source:** [`m2m-aligner`](https://github.com/letter-to-phoneme/m2m-aligner)

**Steps to Run `m2m-aligner`**
```
git clone --depth 1     https://github.com/letter-to-phoneme/m2m-aligner.git
cd m2m-aligner
./m2m-aligner --delX --maxX 2 --maxY 2 -i cmudict.txt

```

**Description:** This `m2m-aligner` is an "algorithm that creates lexicon alignments without requiring annotated data nor linguistic knowledge". It can take the list of words and phonemes, and output aligned graphemes and phonemes. It's not always correct, but I found its output trustworthy enough to use when manually inferring graphemes produced multiple possibilities.


### (B) Manually Splitting Words Into Graphemes

**Description:**  For words where I didn't trust the `m2m-aligner`, I manually mapped \~1.5K words into their graphemes. Silent letters were tricky since sometimes they can be attributed to multiple graphemes. For instance the word **bared** could be split up as B A **RE** D or B A R **ED**. In these cases, where there is a *morpheme*, I tried to default to keeping the morpheme together (e.g. **bare** is a morpheme in bared, so I decided to use the grapheme **RE** instead of splitting up the **R** and **E**). Since I'm not a linguist, I'm not sure that's the right move and I'm pretty sure this is the step with the most mistakes.


# (7) Checking the Work
At this point, the data has aligned mappings between phonemes and graphemes. If there were any phoneme-to-grapheme mappings missing from the research paper, I could identfiy them, add them into my "known mappings", and re-run the process. This is how I ended up with \~35K mapped words with 60 phonemes (15 vowels, 24 consonants, 21 diphonemes) and 275 graphemes (33 vowels, 101 consonants, 141 mixed).
