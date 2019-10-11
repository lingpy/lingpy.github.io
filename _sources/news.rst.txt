===============================
Whats New in Version |version|?
===============================

Version |version| of LingPy serves in primary instance as a bugfix release that helps to make sure
that LingPy can be used properly. Nevertheless, there are certain things that have changed, and that
should also be mentioned in this context.

In the following, we list both the new features introduced in version 2.6.3 and 2.6.4, as well as the new
functions from the new version |version|.

Partial Cognate Detection with Customized Scorer (2.6.5)
--------------------------------------------------------

A problem of partial cognate detection with LingPy was so far, that the scorer in our
:py:class:`~lingpy.compare.partial.Partial` class was computed from
`LexStat` directly, without assuming that the data involved there should also be partial cognates.
This has been addressed now, and a function `get_partial_scorer` has been added, that consistently
computes the scoring function by only considering morphemes, never full words.

Enhanced Loading of CLDF into Wordlist (2.6.5)
----------------------------------------------

To make sure that CLDF data can be loaded from within LingPy more properly, we further enhanced the
`from_cldf` function by adding a more consistent conversion between name spaces. As of now, users
can first choose the columns they want to load from a given CLDF dataset, and then define, by
passing a list of tuples of source and target, how these should be named from within LingPy. 

As an example, try::
  
  >>> from lingpy import *
  >>> from lingpy.tests.util import test_data
  >>> wl = Wordlist.from_cldf(
  ...     test_data('cldf', 'test-metadata.json'), columns=("parameter_id", "language_id",
  ...     "segments"), namespace=(("language_id", "doculect"), ("parameter_id", "concept"),
  ...     ("segments", "tokens"))

This will load the columns `parameter_id`, `language_id`, and `segments` from the CLDF dataset and
represent them with the namespace `doculect`, `concept`, and `tokens`.


Loading CLDF Data from Wordlist Objects (2.6.4)
-----------------------------------------------

Instead of using the `from_cldf` function, you can now simply use the `Wordlist` class to load a
CLDF dataset::
  
  >>> from lingpy import *
  >>> wl = Wordlist.from_cldf('path/to/metadata/json')


Handling Ngrams (2.6.4)
-----------------------

This module offers a general class to handle Markov Models and many utility functions for the
manipulation of ngrams::

  >>> from lingpy.sequence.ngrams import *
  >>> list(get_n_ngrams('LingPy', 3))
  [('$$$', '$$$', '$$$', 'L'),
   ('$$$', '$$$', 'L', 'i'),
   ('$$$', 'L', 'i', 'n'),
   ('L', 'i', 'n', 'g'),
   ('i', 'n', 'g', 'P'),
   ('n', 'g', 'P', 'y'),
   ('g', 'P', 'y', '$$$'),
   ('P', 'y', '$$$', '$$$'),
   ('y', '$$$', '$$$', '$$$')]

So as you can see, this function allows you to retrieve as many ngrams from a given string as you
want. The new module also inherits the functions that one could find in the `sound_classes` module
before, including shortcuts for creating bigrams and trigrams.

Bugfixes in Wordlist Class (2.6.3)
----------------------------------

We fixed some problems related to the :py:class:`~lingpy.basic.wordlist.Wordlist` class in LingPy. First, if you
make the transition from a Wordlist object to a :py:class:`~lingpy.compare.lexstat.LexStat` object, the data
will be deep-copied. As a result, writing::
  
  >>> from lingpy.tests.util import test_data
  >>> from lingpy import *
  >>> wl = Wordlist(test_data('KSL.qlc'))
  >>> lex = LexStat(wl)
  >>> lex.add_entries('segments', 'tokens', lambda x: x)
  >>> 'segments' in wl.header
  False

will no longer add the same number of new entries to the Wordlist object (as it happened before).
You can now also easily access the header of a wordlist in its current order::

  >>> wl = Wordlist(test_data('KSL.qlc'))
  >>> wl.columns
  ['doculect', 'concept', 'glossid', 'orthography', 'ipa', 'tokens', 'cogid']

This is specifically convenient if you construct a Wordlist by creating a dictionary inside a Python
script.

Sanity Checks of Linguistic Data (2.6.3)
----------------------------------------

We introduce a new module, called :py:mod:`~lingpy.compare.sanity`. In this module, we provide a couple of new
functions which you can use to check the consistency of your data. One specific focus, given that
LingPy is focused on sequence comparison, is the *coverage* of words in a wordlist. Coverage is
hereby understood as the minimal number of words shared per meaning slot in each language pair.
Interestingly, you will notice, when testing certain datasets, that mutual coverage can at times be
extremely low. As a rule of thumb, if your data's mutual coverage is below 100 for moderately
divergent language samples, you should not run an analysis using the "lexstat" algorithm of the
~lingpy.compare.lexstat.LexStat class, but instead turn to the "sca" method provided by the same
class. Mutual coverage can be computed in a straightforward manner::

  >>> from lingpy.compare.sanity import mutual_coverage_check
  >>> wl = Wordlist(test_data('KSL.qlc'))
  >>> for i in range(wl.height, 1, -1):
          if mutual_coverage_check(wl, i):
              print('mutual coverage is {0}'.format(i))
              break
      200

We highly recommend all users which deal with spotty and patchy data to have a closer look at the
coverage functions offered in the :py:mod:`~lingpy.compare.sanity` module. You may also want to check out the
:py:func:`~lingpy.compare.sanity.synonymy` function which computes the number of synonyms in your dataset. Here
again, we recommend to pay specific attention to not exceed a value of maximally three words per
concept and language.

Alignments and Cognate Sets (2.6.3)
-----------------------------------

This is again a small change, but to allow for a more consistent integration with external tools, we
now allow to use the cognate set identifier "0" to account for cases where not decision has yet been
made. That means, if you assign cognate sets IDs ("COGID") to your data, and one is set to "0", this
won't be aligned, and not clustered together with other words which have identifier "0".

Random Clusters, Lumper, and Splitter (2.6.3)
---------------------------------------------

We added a new experimental module, in which we added a couple of functions that help to deal with
random clusters. The module :py:mod:`~lingpy.algorithm.cluster_util` offers ways to mutate a given cluster, to
create a random cluster, or to create all possible clusters for a given size of entities. This
module was originally prepared to allow to add random cognate sets to a wordlist in order to compare
this random output with non-random algorithms for cognate detection::

   >>> from lingpy.evaluate.acd import random_clusters
   >>> random_clusters(wl, ref="randomid")

We then figured that it would also be interesting to compare random clusters with "extreme"
clusters, namely, the clusters provided by "lumpers" and "splitters". Thus, we added another custom
function::

   >>> from lingpy.evaluate.acd import extreme_clusters
   >>> extreme_clusters(wl, ref='lumperid', bias='lumper')

If you carry out an assessment of cognate detection algorithms of your data, we recommend to always
compare those against the lumper and the splitter, as this can already tell you a lot about the
nature of your data. In some datasets, the lumper will receive evaluation scores of more than 73%,
only because the cognate density is so high in the data. This means in turn, if your algorithm does
not perform much better than 73% in these cases, it does not mean that it is performing quite well.

Adding Support to Read CLDF (2.6.3)
-----------------------------------

The cross-linguistic data formats (`CLDF <http://cldf.clld.org>`_) initiative (:evobib:`Forkel2017a`)
provides standardized formats for wordlists and cognate judgments. The `pycldf
<http://github.com/cldf/pycldf>`_ package provides support to convert LingPy-formatted data sets
into CLDF format. LingPy now also provides support to read CLDF-files and convert them into
~lingpy.basic.wordlist.Wordlist objects::

   >>> wl = Wordlist(test_data('KSL.qlc'))
   >>> from lingpy.convert.cldf import to_cldf, from_cldf
   >>> to_cldf(wl)
   >>> wl = from_cldf('cldf/Wordlist-metadata.json')


Adding Nexus Output (2.6.3)
---------------------------

We had nexus output before, but now, Simon Greenhill has helped us to provide a stable export to
both MrBayes and Beast, also including the situation where you want to calculate rates for each
concept class::

  >>> wl = Wordlist(test_data("KSL.qlc"))
  >>> from lingpy.convert.strings import write_nexus
  >>> mb = write_nexus(wl, 'mrbayes', filename='ksl-mrbayes.nex')
  >>> beast = write_nexus(wl, 'beastwords', filename='ksl-beast.nex')


Orthography Profile Creation (2.6.3)
------------------------------------

We introduced the commandline of LingPy in version 2.5, but have not really worked on it since then,
as we think that cognate detection analyses should not go on silent by just typing a command into
the terminal. Instead, we encourage users to learn what the major algorithms are, and how they
should be used. We also recommend you to have a look at an `online tutorial <https://github.com/shh-dlce/qmss-2017/tree/master/LingPy>`_ which Johann-Mattis List prepared early in 2017. However, in one particular cases, the commandline actually proved to be very useful, and this is for the creation of orthography profiles (:evobib:`Moran2017`).
Thus, if you now have a normal LingPy wordlist, you can preparse it with LingPy to create an initial
(hopefully useful) orthography profile, via the commandline::

  $ lingpy profile -i input-file.tsv --column=form --context -o output-profile.tsv

If you choose the "---context" option, this means that LingPy will add information on which language
shows which pattern, and whether this occurs in the beginning ("^") or the end ("$") of a given
phonetic sequence. More information can also be found in [this](https://zenodo.org/badge/latestdoi/109654333) tutorial on LingPy and EDICTOR.

LingPy Cookbook (2.6.3)
-----------------------

We decided that we will start introducing LingPy in bits and bytes, since the full reference may at
times overwhel new users. We started collecting some little howtos on our LingPy Cookbook, which you
can fine `here <https://github.com/lingpy/cookbook>`_.


What's Next?
------------

.. toctree::
   :maxdepth: 1
   
   intro
   examples
   tutorial/index
   docu/index
   reference
   download
