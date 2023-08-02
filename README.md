# Syntax Regex Matcher

## Description

`S`yntax `R`egex `M`atcher (SRM for short) is a package for applying regular expressions to spaCy-generated parse trees to look for syntactic constructions in English sentences. 

Given a [spaCy](https://spacy.io/usage/spacy-101) Doc object that has been dependency parsed, SRM converts the parse tree into a string while preserving the dependency structure. Then, a collection of regex are run on each sentence and a list of which patterns were detected is returned. 

## Motivation

This package was initially created to work with another one I made called `gram2vec`, a package for embedding text documents based off stylometric choices authors make pertaining to grammar. I decided to develop SRM as a separate software because I believe it can be useful in other facets of NLP/Comp Ling as well.

As far as I know, there's not many python packages for searching sentences for specific syntactic constructions. Additionally, I think generative LLMs are not yet consistantly reliable enough to be used to identify English sentence patterns. 

## Setup

In your working directory, create an environment by running (I think any version > 3.9 should work, not 100% sure though):
```bash
python3.11 -m venv venv/
source venv/bin/activate
```
which will create a directory called `venv/` to store all the dependencies. 

Next, run:
```bash
pip install git+https://github.com/eric-sclafani/syntax-regex-matcher
```
which will install SRM into your environment, as well as its dependencies.

If you'd like to play around with the source code, you can instead clone the repo directly with the `-e` flag:
```bash
pip install -e syntax-regex-matcher/
```

## Usage

After installing the package, import the `SyntaxRegexMatcher` class:
```python
from srm import SyntaxRegexMatcher

matcher = SyntaxRegexMatcher()
```
The regex matcher processes spaCy generated documents, so a spaCy language model is required. 

**`IMPORTANT`**: I recommend using spaCy's large language model as it results in the best dependency parses
```python
import spacy

nlp = spacy.load("en_core_web_lg")
```

## Adding & testing patterns

## SRM (Syntax Regex Matcher):
- SRM will eventually be ported to its own repo and thus be its own module
- For now, the focus is on getting a working system for linear dependency tree matching. Code simplification and refinement can be focused on at a later time
- Have an option for each match to tell you the exact sentence index span that the match found in the original sentence (not the linear tree). This will require a lot more work, as the regex matching happens on the _linear tree_, not the original sentence 
- Documentation (including the regex pattern testing process)
- The \<word\> part of the linear tree schema (\<word\>-\<lemma\>-\<pos\>-\<deplink\>) may not be needed. Could be removed, which would make the regex slightly more readable (and possibly optimize it a bit?). I say this because it has not played a part in any of the regex I've implemented yet

### Known issues:

- `General`
    - In sentences where the _same pattern_ occurs more than once, the regex counts it as **one** occurence, _not_ two. I'm not sure if this is an issue with the regex itself, or my python implementation. This is a priority \#1 issue.

- `It-clefts`:
    - Temporal it-clefts are not parsed correctly. 

- `Passives`:
    - In the following two sentences, "book" is being mislabeled as "nsubj" when it should be "nsubjpass":
        - "The book which was given to me by Mary was really interesting."
        - "The book given to me by Mary was really interesting"


## Funding
This software was developed under the [IARPA HIATUS](https://www.iarpa.gov/research-programs/hiatus) program