# SEMPRE as semantic parser

This fork is created for [CS703 Fall2020 course project](https://github.com/GindaChen/cs703-sqlizer) to replicate the semantic parser used in SQLizer.

*See the original [sempre repo](https://github.com/percyliang/sempre) and original [README.md](./README.origin.md) if this repo is not what you’re lookinig for.*



## Quick Start



### Environment

- Ubuntu 16.04 / 18.04, Cloudlab machine. 
  - If need to host a freebase, need >100GB memory.
  - Sempre is also tested on Ubuntu 12.04 and MacOS. 
- Java 8 (not 7)
- Ant 1.8.2 (or beyond, I assume - We use  `Ant 1.10`)
- Ruby 1.8.7 or 1.9
- wget
- make (for compiling fig and Virtuoso)
- zip (for unzip downloaded dependencies)

```shell
# Quick check script
java -version && ant -version && ruby -v
wget -h && make -v && zip -h
```










### Dependency if needed

- [Lucene 4.4.0](https://archive.apache.org/dist/lucene/java/4.4.0/lucene-4.4.0.zip): The parser needed for training.
- [Lucene indeices](./dependency-archive/lucene-index). This is needed to train the `freebase` index.
  - `lib/lucene/4.4/inexact`: https://nlp.stanford.edu/software/sempre/release-emnlp2013/lib/lucene/4.4/inexact.tar.bz2/
  - `lib/lucene/4.4/free917`: http://nlp.stanford.edu/software/sempre/release-emnlp2013/lib/lucene/4.4/free917.tar.bz2
  - 





## Example Commands

*All the following command assume the current working directory is the `sempre` directory.*



Verbose everything

```bash
./run @mode=simple \
# -Grammar.inPaths sqlizer/simple.grammar \ 
-languageAnalyzer corenlp.CoreNLPAnalyzer \
-Parser.verbose 3 -MergeFn.verbose 3  -TypeInference.verbose 3 -MixParser.verbose 3 -FuzzyMatchFn.verbose 3 -SelectFn.verbose 3 -JoinFn.verbose 3 -Learner.verbose 3 -SimpleLexiconFn.verbose 3
```



Open freebase client with CoreNLP. (Not very much natrual language support)

```shell
./run @mode=simple-freebase-nocache @sparqlserver=localhost:3001 @domain=free917  @pooldir=1 -languageAnalyzer corenlp.CoreNLPAnalyzer \
-Grammar.inPaths freebase/data/emnlp2013.grammar

# Try the following:
# > California
# > Find me the number of papers in OOPSL 2020
```



Open full freebase in foreground mode

```shell
virtuoso-opensource/install/bin/virtuoso-t +configfile lib/fb_data/93.exec/vdb/virtuoso.ini +wait +foreground
# Freebase database backend start in console.
```



*(Need Freebase started)* Short query over the freebase over california

```shell
./run @mode=query @sparqlserver=localhost:3001 \
-formula '(fb:location.location.containedby fb:en.california)'
```



*(Freebase)* Test on some small formulas

```shell
# The small formula file 
#			freebase/data/free917-small-formula.txt 
# is a pre-defined file that record 
# some of the target formulas inside free917.

./run @mode=query -cachePath ./query_cache.txt  \
@sparqlserver=localhost:3001 \
-SparqlExecutor.readTimeoutMs 6000000 \
-formulasPath freebase/data/free917-small-formula.txt
```





*(Need Freebase started)* Long query over the freebase over the 50th question:

>  “Where did the rolling stones 2009 concert tour take place”

```shell
./run \
@mode=query \
@sparqlserver=localhost:3001 \
-SparqlExecutor.readTimeoutMs 6000000 \
-formula '(and (fb:type.object.type fb:location.location) (and (fb:type.object.type fb:location.administrative_division) ((lambda x (fb:location.statistical_region.foreign_direct_investment_net_inflows (fb:measurement_unit.dated_money_value.valid_date (var x)))) (date 2009 -1 -1))))'
# ... 
#   (list (name fb:en.poland Poland) (name fb:en.united_kingdom_of_great_britain_and_ireland "United Kingdom") (name fb:en.united_states_of_america "United States of America") ...)
# } [5m54s]
```



## SQLizer useful commands

Simply run the rigid grammar on core nlp (with all four functional tags on)

```shell
./run @mode=simple -Grammar.inPaths sqlizer/simple.grammar -languageAnalyzer corenlp.CoreNLPAnalyzer \
-Grammar.tags compositional join bridge inject
```



Run rigid grammar only with `join` and `compositional`

```bash
./run @mode=simple -Grammar.inPaths sqlizer/simple.grammar -languageAnalyzer corenlp.CoreNLPAnalyzer \
-Grammar.tags join compositional
```



Add the following flags as verbose all:

```bash
-Parser.verbose 3 -MergeFn.verbose 3  -TypeInference.verbose 3 -MixParser.verbose 3 -FuzzyMatchFn.verbose 3 -SelectFn.verbose 3 -JoinFn.verbose 3 -Learner.verbose 3 -SimpleLexiconFn.verbose 3
```



## Knowledge in NLP

- [**NLTK Tags**](./sqlizer/README-corenlp-NLTK-tags.md). The abbreviation used in the Stanford CoreNLP that explains the property of the word.
- 



## Toolchain in Sempre

- Stanford `corenlp`: The 

- `Virtuoso`, `freebase`, `Sparql` as the knowledge base backend.
  - **Freebase**: a graph database. Originally supported by google, but deprecated 2015. Now we need to host the **full-virtuoso** (`./pull-dependency freebase-full`) in our end to maintain the entity matching and phrase matching.
  - **SparQL**: a graph database query language (like SQL). Think of it as the query language to Freebase.
  - **[Virtuoso](https://github.com/openlink/virtuoso-opensource)**: a graph database engine. In fact, it is a “universal” data engine that supports graph database. 



## Questions

What are all the modes in the master server, and how to use them to

- Only retrieve parsed derivation, instead of triggering a freebase query?

```shell
# Possible @mode=["test", "freebase", "cacheserver", "filterfreebase", "sparqlserver", "indexfreebase", "convertfree917", "query", "simple", "simple-sparql", "simple-lambdadcs", "simple-freebase", "simple-freebase-nocache", "overnight", "tables", "genovernight", "genovernight-wrapper", "geo880"]
```



What is `BridgeFn` and `JoinFn`?



When doinig freebase training, It seems the bottleneck is the database output rather than the parser.

1. How does the database actually work? Why can’t it using 40 cores?
2. How to increase its core?
3. What is the best time-out?



