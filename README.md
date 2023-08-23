# Syntax Regex Matcher

## Description

`S`yntax `R`egex `M`atcher (SRM for short) is a package for applying regular expressions to spaCy-generated parse trees to look for syntactic constructions in English sentences. 

Given a [spaCy](https://spacy.io/usage/spacy-101) Doc object that has been dependency parsed, SRM converts the parse tree into a string while preserving the dependency structure. Then, a collection of regex are run on each sentence and a list of which patterns were detected is returned. 

## Motivation

This package was initially created to work with another one I made called `gram2vec`, a package for embedding text documents based off stylometric choices authors make pertaining to grammar. SRM was developed as a separate software because I believe it can be useful in other facets of NLP/Comp Ling too. As far as I know, there's not many python packages for searching sentences for specific syntactic constructions. 

One might also think what the point of this package is if LLMs are getting better at "understanding" language. This is a valid thought, but the following points hopefully clear this up:

- LLMs, as powerful as they are, can be inconsistent with their answers due to their stochastic nature. Using a rule-based approach guarantees deterministic outputs, which can be desirable for certain applications.

- LLMs don't learn and understand language like humans do. They are simply just really, really good at repeating things they've seen in the pre-training stage. That's why GPT works so well, since it was basically trained on all of the internet prior to 2021. Again, I prefer rule-based methods because that's how language itself works (`in my opinion`). Linguistics involves the study of rules pertaining to specific languages (phonological, syntactic, phonetic, etc..) and how, as humans, we learn and understand these rules. 

- SRM is very lightweight, only requiring the use of spaCy for sentence parsing.

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

### Matcher
Import the `SyntaxRegexMatcher` class:
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
>>> @dataclass
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
Keen readers may have noticed that the sentence: _When it was sunny, I went outside, but it started raining._ is incorrectly being detected as a `pseudo-cleft`. This is a great time to point out that false positives/negatives are possible and the regular expressions are not fool proof. 

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

### Linear trees

Under the hood, given a spaCy-generated parse tree, SRM converts each dependency node into a string using the following format:
```
(<word>-<lemma>-<pos>-<deplink>) 
```
It also preserves dependencies by using nested parenthesis:
```
The chicken, who was busy laying eggs, sat happily.

(sat-sit-VBD-ROOT(chicken-chicken-NN-nsubj(The-the-DT-det)(,-,-,-punct)(was-be-VBD-relcl(who-who-WP-nsubj)(busy-busy-JJ-acomp(laying-lay-VBG-xcomp(eggs-egg-NNS-dobj)(,-,-,-punct)(happily-happily-RB-advmod)(.-.-.-punct))))))
```

A little unreadable, but this format allows for regex matching. The tree linearizing function is available those who just want to use that functionality:
```python
>>> from srm import linearize_tree
>>> import spacy

>>> nlp = spacy.load("en_core_web_lg")
>>> doc = nlp("All the dog in the tree knew was that the bone was on the grass.")
>>> sentences = list(doc.sents)
>>> linearize_tree(sentences[0])

(knew-know-VBD-ROOT(dog-dog-NN-nsubj(All-all-PDT-predet)(the-the-DT-det)(in-in-IN-prep(tree-tree-NN-pobj(the-the-DT-det)(was-be-VBD-ccomp(was-be-VBD-ccomp(that-that-IN-mark)(bone-bone-NN-nsubj(the-the-DT-det)(on-on-IN-prep(grass-grass-NN-pobj(the-the-DT-det)(.-.-.-punct))))))))))
```

## Pattern creation

The pattern creation process is simple (but sometimes not easy!). It just involves generating a collection of `linear trees` from example sentences and manually observing capturable substrings that each sentence shares. Here is my process:

1. First think about what syntactic phenomena that would be interesting to capture. For me, this involved consulting with fellow linguists, along with some Googling here and there.

2. Add a JSON object to the [pattern_tests.json](pattern_testing/pattern_tests.json). It should have `name` and `tests` fields. The `name` is the name of the syntactic pattern, and each `test` is an array containing the truth value of whether that sentence is an example of that pattern, and the sentence itself. Having both `true` and `false` examples helps refine the regex:
```json
{
        "name" : "psuedo-cleft",
        "tests" : [
            ["TRUE", "What I want is some peace and quiet!"],
            ["TRUE", "What you need to do is to rest for a while."],
            ["TRUE", "Where I want to go is a place so far away from here."],
            ["TRUE", "How she paid for her food was with her credit card."],
            ["TRUE", "Some peace and quiet is what I want."],
            ["TRUE", "A place so far away from here is where I want to go."],
            ["FALSE", "I want a hamburger."],
            ["FALSE", "Where did I put that potato?"],
            ["FALSE", "I like having peace and quiet"],
            ["FALSE", "What I need is none of your business."]
        ]
    },
```
3. Assuming the added JSON object is formatted correctly, run the [test_patterns.py](pattern_testing/test_patterns.py) script to generate linear strings. They will be saved in the [pattern_test_outputs/](pattern_testing/pattern_test_outputs) directory.

4. Navigate to the newly created pattern file in [pattern_test_outputs/](pattern_testing/pattern_test_outputs)your_pattern_name.txt and copy+paste its contents into [https://regex101.com/](https://regex101.com/). This website makes it very simple to test whether regex patterns work because it updates the matching in real time. Best of all, patterns and test strings can be saved to the website.

5. Finally, some patterns are easier to create than others. Often times, its a balance between false positives and false negatives. After you're happy with the pattern, add it to your `SyntaxRegexMatcher` instance as outlined in the previous section.

## Contributions

All patterns are not fool proof, so if you come across false positives or negatives, feel free to submit an issue. This helps immensely when fine tuning the patterns.

If you'd like to submit a PR to permanently add a pattern to SRM, please follow the steps in the previous section to ensure your pattern works. Make sure to include the pattern tesing .txt file so I can verify the pattern.

Feel free to suggest other modifications as well. You can view the TODO.md for things on the bucket list.

All contributions are greatly appreciated.
 

## Known issues (as of 08/07/2023):

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


## Acknowledgements

This research is supported in part by the Office of the Director of National Intelligence (ODNI), Intelligence Advanced Research Projects Activity (IARPA), via the HIATUS Program contract #2022-22072200005. The views and conclusions contained herein are those of the authors and should not be interpreted as necessarily representing the official policies, either expressed or implied, of ODNI, IARPA, or the U.S. Government. The U.S. Government is authorized to reproduce and distribute reprints for governmental purposes notwithstanding any copyright annotation therein.