# TextbookMath

This is TextbookMath, a suite of test problems for higher-order automated theorem provers.  TextbookMath is intended to assess prover performance in verifying individual proof steps in textbook proofs.  Currently the suite contains 962 higher-order problems, which correspond to the steps in 151 different proofs.

At the moment, all problems in TextbookMath are derived from the excellent textbook Number Systems and the Foundations of Analysis (Elliott Mendelson, 1973), which develops the number systems ℕ, ℤ, ℚ and ℝ starting from the Peano postulates.  Currently the suite includes only parts of the development of ℕ and ℤ.  I may extend the suite in the future with more problems from the Mendelson textbook, or from other undergraduate-level textbooks.

The suite is constructed using the [Natty](https://github.com/medovina/natty) natural-language proof assistant.  It contains a set of source files written in N, which is Natty's controlled natural language for writing mathematics.  I have transcribed the mathematics from the Mendelson text fairly closely into these source files, though as described below the translation is not always completely literal.  Natty can transform these N files into a series of files in the standard THF format for representing problems in higher-order logic.

The source files include

* `set.n`: a few definitions about sets and functions (Mendelson, sections 1.6-1.8, 1.19-20)
* `nat.n`: definition of ℕ, addition, ordering, multiplication on ℕ (2.1, 2.3-2.5)
* `int.n`: definition of ℤ, addition, multiplication, ordering on ℤ (3.1-3.4)
* `div.n`: division, greatest common divisors (3.5)
* `prime.n`: relative primality, prime numbers (3.5)
* `card.n`: a few definitions about cardinality (appendix D)

TextbookMath comes in two variants: a monomorphic variant found in the nums_mono directory, an a polymorphic variant in the nums directory.  The monomorphic variant exists because not all higher-order provers support polymorphic types; in particular, E does not.  

## Building TextbookMath

You will first need to build and install [Natty](https://github.com/medovina/natty).  To build the polymorphic variant:

```
$ cd nums
$ natty -x -r prime.n
```

This will produce a directory `thf` with subdirectories `card`, `div` and so on, one for each module.

To build the monomorphic variant, use `cd nums_mono` instead.

## Evaluating provers

Natty includes a script `eval.py` that can evaluate automatic provers on a series of THF files.  To evaluate [E](https://www.eprover.org/), [Vampire](https://vprover.github.io/), [Zipperposition](https://github.com/sneeuwballen/zipperposition) and Natty's own automatic prover on the TextbookMath problems, first make sure that you have installed Natty and the other provers.  Then run

```
$ python3 /path/to/eval.py -a -d thf
```

That will produce a series of CSV files, one for each module in the suite, plus a file `summary.csv` summarizing the provers' performance on all modules.  By default the evaluation will use a time limit of 5 seconds per problem; you can change this with the `-t` option.

You may wish to edit the dictionary `all_provers` at the top of `eval.py` to modify the set of provers that will be tested and/or the arguments to be passed to each prover.

## Evaluation results

In February 2026 I evaluated the following automatic provers on the monomorphic variant of TextbookMath:

* Natty (built from tag 0.0.3, commit `818c9aea`)
* [E](https://www.eprover.org/) (version 3.3.1-ho, built with the `–enable-ho` option from commit `8f999d07`, run with the `-auto` option)
* [Vampire](https://vprover.github.io/) 4.8 (built from the `hol` branch from commit `7b44b1c2f`
* [Zipperposition](https://github.com/sneeuwballen/zipperposition) (built from git master at commit `050072e0`, run with `-mode best`)

I used a 5-second time limit for each problem.  Here are the results.

| module | | Natty + by | Natty | E | Vampire | Zipperposition |
| --- | --- | --- | --- | --- | --- | --- |
| div | proved (of 141) <br> avg time | 115 (82%) <br> 0.13 sec | 113 (80%) <br> 0.14 sec | 114 (81%) <br> 0.28 sec | 111 (79%) <br> 0.47 sec | 101 (72%) <br> 0.69 sec |
| int | proved (of 483)<br>avg time | 457 (95%)<br>0.05 sec | 451 (93%)<br>0.04 sec | 373 (77%)<br>0.10 sec | 390 (81%) <br> 0.50 sec | 384 (80%) <br> 0.32 sec |
| nat | proved (of 301)<br>avg time | 299 (99%)<br>0.02 sec | 290 (96%)<br>0.02 sec | 285 (95%)<br>0.01 sec | 280 (93%)<br>0.16 sec | 261 (87%)<br>0.14 sec |
| prime | proved (of 37)<br>avg time | 23 (62%)<br>0.07 sec | 23 (62%)<br>0.07 sec | 37 (100%)<br>0.68 sec | 37 (100%)<br>0.08 sec | 36 (97%)<br>1.24 sec |
| TOTAL | proved (of 962)<br>avg time | 894 (93%)<br>0.05 sec | 877 (91%)<br>0.05 sec | 809 (84%)<br>0.12 sec | 818 (85%)<br>0.36 sec | 782 (81%)<br>0.35 sec |

In the table above, "Natty + by" shows Natty's performance when it is given the reasons with which each proof step has been annotated  (e.g. "therefore x = y + 1 **by Theorem 3.2.1**").  This increases its success rate slightly.  (The other provers do not have the ability to follow reasons, so there is no such column for them.)

It is evident that although higher-provers can verify many proof steps automatically in a short time, their overall success rate is far from 100%.

Note that Natty's automatic prover has been designed and tuned for proving relatively easy proof steps, such as the ones in this test suite.  I do not believe that Natty's performance will be competitive with other automatic provers on more general sets of harder problems, such as those in the broader [TPTP](https://www.tptp.org/) test suite.

I intend to publish a paper this year explaining Natty's prover and this evaluation in more detail.

## Notes on the translation from the Mendelson text

I attempted to translate theorems and proofs as literally as possible from the Mendelson text into the N source files in this suite, though some changes were necessary for various practical and foundational reasons. For example, Mendelson’s foundation is ZFC set theory, which is fundamentally untyped: every value is a set. However N is based on higher-order logic, in which every variable has a type. In particular, Mendelson defines ℤ to be a set of equivalence classes of ℕ × ℕ. ℕ and ℤ should be distinct types in N, so I translated Mendelson’s definition into a set of axioms that declare that ℤ is a type that is isomorphic to a set of equivalence classes of ℕ × ℕ. Additionally, Mendelson proves many theorems about ℕ and ℤ with a level of generality that goes beyond these particular number systems (e.g. on commutative rings in general). In the translation into N I have asserted these theorems only on the particular number systems ℕ and ℤ.

Despite these differences, I believe that the translation is basically faithful to the Mendelson text in the sense that individual proof steps have almost always been preserved. Furthermore, in the translation I have annotated proof steps with reasons to be used in proving them exactly whenever such reasons appeared in the original textbook.

---

[Adam Dingle](https://ksvi.mff.cuni.cz/~dingle/)<br>
KSVI, Faculty of Mathematics and Physics, Charles University
