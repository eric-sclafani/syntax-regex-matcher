# Syntax Regex Matcher

## Description

`S`yntax `R`egex `M`atcher (SRM for short) is a package for applying regular expressions to spaCy-generated parse trees to look for syntactic constructions in English sentences. 

Given a [spaCy](https://spacy.io/usage/spacy-101) Doc object that has been dependency parsed, SRM converts the parse tree into a string while preserving the dependency structure. Then, a collection of regex are run on each sentence and a list of which patterns were detected is returned. 

## Motivation

This package was initially created to work with another one I made called `gram2vec`, a package for embedding text documents based off stylometric choices authors make pertaining to grammar. I decided to develop SRM as a separate software because I believe it can be useful in other facets of NLP/Comp Ling as well.

As far as I know, there's not many python packages for searching sentences for specific syntactic constructions. Additionally, I think generative LLMs are not yet consistantly reliable enough to be used to identify English sentence patterns. 

## Setup

Run `pip install srm` to install the package. The only dependency is spaCy.

## Usage

## Adding & testing patterns

## Funding
This software was developed under the [IARPA HIATUS](https://www.iarpa.gov/research-programs/hiatus) program