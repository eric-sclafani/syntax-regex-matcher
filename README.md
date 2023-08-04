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

If you'd like to play around with the source code, you can instead clone the repo directly and install with the `-e` flag:
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

**`IMPORTANT`**: I recommend using spaCy's **large language model** (_en_core_web_lg_) as it results in the best dependency parses. Additionally, the regex were created with the large model in mind. 
```python
import spacy

nlp = spacy.load("en_core_web_lg")
```

There are two options for document matching: `.match_document()`, which matches a single spaCy document:
```python
>>> text = "It was Jane’s car that got stolen last night. What I want is some peace and quiet!"
>>> doc = nlp(text)
>>> matches = matcher.match_document(doc)
>>> for match in matches:
>>>     print(match)

it-cleft : It was Jane’s car that got stolen last night.
subj-relcl : It was Jane’s car that got stolen last night.
pseudo-cleft : What I want is some peace and quiet!
```  

Each match is an instance of the `Match` class. The class and attributes are shown here:
```python
@dataclass
>>> class Match:
>>>     pattern_name:str
>>>     matched:str
>>>     sentence:str
    
>>>     def __repr__(self) -> str:
>>>         return f"{self.pattern_name} : {self.sentence}"

>>> first_match = matches[0]
>>> print(first_match.pattern_name)
>>> print(first_match.sentence)
>>> print(first_match.matched)
it-cleft
It was Jane’s car that got stolen last night.
(was-be-VBD-ROOT(It-it-PRP-nsubj)(car-car-NN-attr(Jane-Jane-NNP-poss(’s-’s-POS-case)(stolen-steal-VBN-relcl


```

The second method is `.match_documents()`, which matches an interable of documents. This works great when using spaCy's `nlp.pipe()` function:
```python
>>> texts = ["How she paid for her food was with her credit card. When did Sarah say she was coming over?", 
         "English is spoken all over the world. When it was sunny, I went outside, but it started raining.",
         "She is the author that I have interviewed. They might have been invited to the party."]
>>> docs = nlp.pipe(texts)
>>> matches = matcher.match_documents(docs)
>>> for match in matches:
>>>    print(match)

(pseudo-cleft : How she paid for her food was with her credit card.,)
(passive : English is spoken all over the world., pseudo-cleft : When it was sunny, I went outside, but it started raining., coordinate-clause : When it was sunny, I went outside, but it started raining.)
(obj-relcl : She is the author that I have interviewed., passive : They might have been invited to the party.)
```
Keen readers may have noticed that the sentence: _When it was sunny, I went outside, but it started raining._ is incorrectly being detected as a `psueo-cleft`. This is a great time to point out that false positives/negatives are possible and the regular expressions are not fool proof. 

From my experience in crafting these regex, its often a tradeoff between precision and recall. Some patterns are more difficult to capture than others. And the more rules you add to a regex pattern, the more likely you are to accidentally capture unwanted sentences 

Additionally, one can easily view, add, and remove patterns from an instance of `SyntaxRegexMatcher` by using the following methods:
```python
# prints all currently registered patterns
>>> matcher.print_patterns() 

it-cleft : \([^-]*-be-[^-]*-[^-]*.*\([iI]t-it-PRP-nsubj\).*\([^-]*-[^-]*-NN[^-]*-attr.*\([^-]*-[^-]*-VB[^-]*-(relcl|advcl)

pseudo-cleft : \([^-]*-be-[^-]*-[^-]*.*\([^-]*-[^-]*-(WP|WRB)-(dobj|advmod)

all-cleft : (\([^-]*-be-[^-]*-[^-]*.*\([^-]*-all-(P)?DT)|(\([^-]*-all-(P)?DT-[^-]*.*\([^-]*-be-[^-]*)

there-cleft : \([^-]*-be-[^-]*-[^-]*.*\([^-]*-there-EX-expl.*\([^-]*-[^-]*-[^-]*-attr.*\([^-]*-[^-]*-[^-]*-(relcl|acl)

if-because-cleft : \([^-]*-be-[^-]*-[^-]*.*\([^-]*-[^-]*-[^-]*-advcl\([^-*]*-if-IN-mark

passive : \([^-]*-[^-]*-(NN[^-]*|PRP|WDT)-nsubjpass.*\([^-]*-be-[^-]*-auxpass

subj-relcl : \([^-]*-[^-]*-[^-]*-relcl.*\([^-]*-[^-]*-(WP|WDT)-nsubj

obj-relcl : \([^-]*-[^-]*-NN[^-]*-(nsubj|attr).*\([^-]*-[^-]*-[^-]*-(relcl|ccomp).*\([^-]*-[^-]*-(WP|WDT|IN)-(pobj|dobj)

tag-question : \([^-]*-(do|be|could|can|have)-[^-]*-ROOT.*\(\?-\?-\.-punct

coordinate-clause : \([^-]*-[^-]*-CC-cc\).*\([^-]*-[^-]*-(VB[^-]*|JJ)-conj.*\([^-]*-[^-]*-[^-]*-nsubj
```

```python
# add a dictionary of pattern name : regex mappings
matcher.add_patterns(
    {
        "pattern_name_1" : r"regex_1",
        "pattern_name_2" : r"regex_2"
    }
)

# removes patterns by name
>>> matcher.remove_patterns([
    "it-cleft",
    "passive",
    "obj-relcl"
])

>>> matcher.print_patterns()

pseudo-cleft : \([^-]*-be-[^-]*-[^-]*.*\([^-]*-[^-]*-(WP|WRB)-(dobj|advmod)

all-cleft : (\([^-]*-be-[^-]*-[^-]*.*\([^-]*-all-(P)?DT)|(\([^-]*-all-(P)?DT-[^-]*.*\([^-]*-be-[^-]*)

there-cleft : \([^-]*-be-[^-]*-[^-]*.*\([^-]*-there-EX-expl.*\([^-]*-[^-]*-[^-]*-attr.*\([^-]*-[^-]*-[^-]*-(relcl|acl)

if-because-cleft : \([^-]*-be-[^-]*-[^-]*.*\([^-]*-[^-]*-[^-]*-advcl\([^-*]*-if-IN-mark

tag-question : \([^-]*-(do|be|could|can|have)-[^-]*-ROOT.*\(\?-\?-\.-punct
```




## Pattern creation

## Contribution

## Known issues:

- `General`
    - In sentences where the _same pattern_ occurs more than once, the regex counts it as **one** occurence, _not_ two. I'm not sure if this is an issue with the regex itself, or my python implementation.

    - SRM relies solely on spaCy parses. The largest downside to this is that if spaCy makes parse errors, there's nothing that can be done about that. Parse errors can lead to false positives and false negatives. 
        - This is much more likely to happen to sentences with more complex constructions (i.e., sentences without much representation in the dependency parser's pre-training data)
        - The only solution I could think of is fine-tuning the spaCy dependency parser on more sentences


- `It-clefts`:
    - Temporal it-clefts are not parsed correctly. 

- `Passives`:
    - In the following two sentences, "book" is being mislabeled as "nsubj" when it should be "nsubjpass":
        - "The book which was given to me by Mary was really interesting."
        - "The book given to me by Mary was really interesting"


## Funding
This software was developed under the [IARPA HIATUS](https://www.iarpa.gov/research-programs/hiatus) program