Frequent Itemset Generator
==========================

Introduction
------------

Finds statistically significant sets of frequent items in large datasets. For
example, let's say we have a database of supermarket transactions, or baskets,
each containing a number of grocery items. If we want to find the frequent
grocery items in this database, we're finding the grocery items which are most
frequently purchased together: the "frequent itemsets."

Generally we want the sets of items to occur in
at least m transactions. We'll call this the minimum support number. The minimum
support number allows us to specify how frequent an itemset must be to be
included in the final output - obviously, a set of items that does not occur in
any transactions, or even a small number of transactions, is insignificant to
the user of the system. The minimum support generally must be tuned for each
specific dataset to determine what will give the user plenty of data, but not
so much data that the output becomes unusable.

To this end, we also include the maximum support number, which caps the number
of possible transactions any set in the final output can have. Imagine that at
a specific supermarket, nearly every customer buys milk when they buy anything
else. Then, it is not very significant to include milk in the final output,
since the item is so commonly purchased. Thus we can use the maximum support
to filter out items like this. As with the min support, this number will also
vary wildly between datasets. If you end up using the max support metric (less
than 100% of course) for a dataset,
this indicates that you should probably use the FPGrowth algorithm, as this
algorithm runs much quicker on datasets with a large number of similar items
between transactions.

Use
---

This service was built to find the genetic transcription factors (proteins)
which most frequently occur together in one gene. However, you can add your own
dataset if you want to use it for a different purpose.

Customizability
---------------

 - class BasketIterator: Implement BasketIterator to create your own dataset.
 - class ItemsetGenerator: Implement ItemsetGenerator to create your own 
 algorithm.
 - file lib/passwd.groovy: Password file - holds database configuration info.

API Reference
-------------

 - Get request: `get [minSup:decimal:0-1] [maxSup:decimal:0-1]`
 Returns: `{
    'res': "(SUCCESS | FAILURE)":string,
    'item-cnt': "integer indicating the number of unique items":integer,
    'reason': "Reason for failure if FAILURE, request type if SUCCESS.":string,
    'message': "Explanation of failure iff 'res' == 'FAILURE'.":string,
    'time': "integer indicating the time it took to find itemsets":integer,
    'data': "map with the itemsets and their frequency counts; ex: [[one, two]: 138]":map
 }'
 Returns a list of the itemsets between the min and max support values.
 'time' is always 0 on FAILURE.
 `maxSup` must be less than `minSup`, obviously. Both are decimal values
 indicating the percent support an itemset must have to be included.

 - Set request: `set [BasketIterator] [ItemsetGenerator]`
 Returns: same as Get request except with no 'data' attribute; 'time'
 will always be 0
 If the request is successful, 'res' will be set to SUCCESS and 'reason'
 will be "SET". Any subsequent queries by client will use the
 BasketIterator and ItemsetGenerator specified.

 Both Set values must be fully-qualified class names.


Two algorithms are available; selectable at runtime:
----------------------------------------------------
 - Apriori algorithm: Iterates through every transaction (supermarket basket)
 and candidate to find the most frequent sets of items. A candidate is any one
 subset of items in the entire set of items (in our example, every item in the
 supermarket). So we start with candidates with one item (Bread, milk, cheese),
 then we look for baskets with two items ([Bread, milk], [milk, cheese],
 [cheese, bread]), and so on until we've searched every subset. At worst case,
 this algorithm is exponential, as there are 2^n subsets for n items. But the
 trick is that we drop from consideration all larger subsets containing an
 infrequent item. For example, if we know that bread is in m - 1 baskets (one
 less than our minimum support) then we know bread is _never_ a frequent item,
 so we can drop [Bread, milk], [cheese, Bread], and [bread, milk, cheese] from
 our consideration. This optimization significantly speeds up the algorithm for
 reasonable minimum support values (if you set the minimum support to 0, every
 item set will be frequent and you'll have to iterate through every set).

 - FPGrowth algorithm: Based on the FPTree data structure. This algorithm builds
 a tree representing your dataset. The more items each transaction has in
 common, the smaller the tree will be and thus the faster this algorithm will
 procese all frequent itemsets.
 
License
=======

   Copyright 2014 Therin Irwin

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
