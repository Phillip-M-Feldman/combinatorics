# combinatorics

# Dr. Phillip M. Feldman

<h1>1. Introduction</h1>

<p>This Python module supplement Python's <code>itertools</code> module.  &nbsp;<code>combinatorics</code> fills in gaps in the
following areas of basic combinatorics:

<ol type="A">
<li>ordered and unordered m-way combinations,
<li>generalizations of the four basic occupancy problems (aka <em>balls in boxes</em>), and
<li>constrained permutations, otherwise known as the <em>off-by-m problem</em>.
</ol>

<p>Application areas include reliability engineering, quantum mechanics, and frequency-based
cryptanalysis.  (Frequency-based cryptanalysis is useless against modern encryption, but it is
fun to create simple substitution cyphers and experiment with cracking them).

<br>


<h1>2. Elementary Combinatorics</h1>

<h2>2.1 <code>prod</code></h2>

<h3>Overview</h3>

This function, like NumPy's <code>prod</code> function, calculates the product of a sequence
of integers (typically specified via a list or tuple). Because the NumPy function uses 64-bit
integer arithmetic with silent handling of overflows, results are wrong if the correct answer
would exceed the limits of a signed 64-bit integer. When operating on a sequence of integers,
the <code>prod</code> function defined in this module uses large integer arithmetic and thus
always gives correct results.

<h3>Example (Interactive ipython I/O)</h3>

<pre>
   In [1]: prod(range(1,11))
   Out[1]: np.int(3628800)

   In [2]: prod(range(1,31))
   Out[2]: np.int64(-8764578968847253504)

   In [3]: from combinatorics import prod

   In [4]: prod(range(1,11))
   Out[4]: 3628800

   In [5]: prod(range(1,31))
   Out[5]: 265252859812191058636308480000000L
</pre>


<h2>2.2 <code>n_choose_m</code></h2>

<p>The function <code>n_choose_m</code> calculates $\left( \begin{array}{c}
n \\ m \end{array} \right)$, defined as the number of ways in which one can
select m of n distinct objects without regard for order.  The function uses only
integer operations:

<ol>
<li>If $m > n-m$, we replace $m$ by $n-m$.  This depends on the fact that
$\left( \begin{array}{c} n \\ m \end{array} \right)
= \left( \begin{array}{c} n \\ n-m \end{array} \right)$.

<li><p>We calculate the answer by evaluating
<pre>
      prod(range(n-m+1,n+1)) / prod(range(2,m+1)),
</pre>

which is equivalent to $n! / m! / (n-m)!$.
</ol>

Note: Python can handle integers of arbitrary size, but the algorithm tends to
bog down for very large values of n, partly because of the number of operations
being performed and partly because of the memory requirements.  For values of n
above about 10000, use <code>m_choose_n_ln</code> instead of
<code>m_choose_n</code>.


<h2>2.3 <code>n_choose_m_ln</code></h2>

<p>This function calculates the natural logarithm of $\left( \begin{array}{c}
n \\ m \end{array} \right)$.  The calculation is done using SciPy's
<code>gammaln</code> function.  For large $n$, especially for $n > 10000$,
this function is much faster than <code>n_choose_m</code> (computational and
memory requirements are both much lower).

<p>Note: To obtain a value for $\left( \begin{array}{c} n \\ m \end{array}
\right)$, apply the <code>math.exp</code> or <code>numpy.exp</code> function
to the result returned by this function.


<h1>3. M-Way Combinations<h1>

<h2>3.1 <code>m_way_ordered_combinations</code></h2>

<h3>Signature</h3>

<pre>
   m_way_ordered_combinations(items, ks)
</pre>


<h3>Overview</h3>

This function returns a generator that produces all m-way ordered combinations
(multinomial combinations) from the specified collection of items, with
<code>ks[i]</code>items in the ith group, i= 0, 1, 2, ..., m-1, where
<code>m= len(ks)</code> is the number of groups.  <em>Ordered combinations</em>
means that the relative order of equal-size groups is important.  The order of
the items within any group is not important.  The total number of combinations
generated is given by the multinomial coefficient formula (see below).


<h3>Inputs</h3>

<code>items</code> must be (A) a list, tuple, or other iterable, or (B) a
positive integer.  If <code>items</code> is an integer, it is replaced by
<code>range(items)</code>.

<code>ks</code> should be either a list or tuple containing non-negative
integers, where the sum of these integers does not exceed the length of
<code>items</code>.


<h3>Example</h3>

Let <code>items=[0,1,2,3,4,5]</code> and <code>ks=[2,2,2]</code>.  The output
includes a total of 90 combinations.  Two of these are the following:

<pre>
   ((0, 1), (2, 3), (4, 5))
   ((2, 3), (0, 1), (4, 5))
</pre>

These are distinct because the order of the groups, which differs, is
significant.


<h3>Notes</h3>

The total number of combinations generated is given by the following multinomial
coefficient:

<p>\[ \frac{n!}{k_0! \cdot k_1! \cdot \cdots \cdot k_{m-1}!} \]

<p>where n is the number of items, m is the number of groups, and k_i is the
number of items in the ith group.


<h2>3.2 <code>m_way_unordered_combinations</code></h2>

<h3>Signature</h3>

<pre>
   m_way_unordered_combinations(items, ks)
</pre>


<h3>Overview</h3>

This function returns a generator that produces all m-way unordered combinations
from the specified collection of items, with <code>ks[i]</code> items in the ith
group, i= 0, 1, 2, ..., m-1, where <code>m= len(ks)</code> is the number of
groups.  <i>Unordered combinations</i> means that the relative order of
equal-size groups is not important.  The order of the items within any group is
also unimportant.


<h3>Inputs</h3>

<code>items</code> must be (A) a list, tuple, or other iterable, or (B) a
positive integer.  If <code>items</code> is an integer, it is replaced by
<code>range(items)</code>.

<code>ks</code> should be either a list or tuple containing non-negative
integers, where the sum of these integers does not exceed the length of
<code>items</code>.


<h3>Example</h3>

Consider the following reliability problem: Six similar but non-identical units
must be grouped into three pairs&mdash;primary and redundant&mdash;except that
we don't care which is which.  Identify all the ways in which this can be done.

<pre class="literal-block">
from combinatorics import *
list(m_way_unordered_combinations(6, [2,2,2]))
</pre>

The output consists of the 15 combinations listed below:

<pre>
(0, 1), (2, 3), (4, 5)
(0, 1), (2, 4), (3, 5)
(0, 1), (2, 5), (3, 4)
(0, 2), (1, 3), (4, 5)
(0, 2), (1, 4), (3, 5)
(0, 2), (1, 5), (3, 4)
(0, 3), (1, 2), (4, 5)
(0, 3), (1, 4), (2, 5)
(0, 3), (1, 5), (2, 4)
(0, 4), (1, 2), (3, 5)
(0, 4), (1, 3), (2, 5)
(0, 4), (1, 5), (2, 3)
(0, 5), (1, 2), (3, 4)
(0, 5), (1, 3), (2, 4)
(0, 5), (1, 4), (2, 3)
</pre>


<h3>Notes</h3>

When all group sizes are unequal, the total number of combinations generated is
given by the multinomial coefficient (see above).  When two or more groups have
equal sizes, the number of combinations is less than the multinomial coefficient
because combinations that differ only in the relative order of equal-size groups
are excluded.


<h1>4. Balls in Boxes</h1>

<h2>4.1 Introduction</h2>

Many combinatorial problems are equivalent to the problem of distributing balls
into boxes. The balls may be either identical (unlabeled) or distinguishable
(labeled), and the boxes may also be either identical or distinguishable; we
thus have four classes of balls-in-boxes problems. Although typically the boxes
do not have capacity limits, i.e., one may choose to put all of the balls in any
single box, the most general formulation of the problem allows for capacity
limits.


<h2>4.2 <code>unlabeled_balls_in_labeled_boxes</code></h2>

<h3>Signature</h3>

<pre>
   unlabeled_balls_in_labeled_boxes(n, box_sizes)
</pre>


<h3>Overview</h3>

This function returns a generator that produces all distinct distributions of
indistinguishable balls among labeled boxes with specified box sizes
(capacities). This is a generalization of the most common formulation of the
problem, where each box is sufficiently large to accommodate all of the balls,
and is an important example of a class of combinatorics problems called <em>
weak composition</em> problems.


<h3>Inputs</h3>

<p><code>n</code> is the number of balls.

<p><code>box_sizes</code> is a list of length 1 or greater. The length of the
list corresponds to the number of boxes. <code>box_sizes[i]</code> is a positive
integer that specifies the maximum capacity of the ith box. If
<code>box_sizes[i]</code> equals <code>n</code> (or greater), the ith box can
accommodate all <code>n</code> balls and thus effectively has unlimited
capacity.


<h3>Acknowledgment</h3>

I'd like to thank Chris Rebert for helping me to convert my prototype
class-based code into a generator function.


<h2>4.3 <code>unlabeled_balls_in_unlabeled_boxes</code></h2>

<h3>Signature</h3>

<pre>
   unlabeled_balls_in_unlabeled_boxes(n, box_sizes)
</pre>


<h3>Overview</h3>

This function returns a generator that produces all distinct distributions of
indistinguishable balls among indistinguishable boxes, with specified box sizes
(capacities).  This is a generalization of the most common formulation of the
problem, where each box is sufficiently large to accommodate all of the balls.
It might be asked, <em>In what sense are the boxes indistinguishable if they
have different capacities?</em>  The answer is that the box capacities must be
considered when distributing the balls, but once the balls have been
distributed, the identities of the boxes no longer matter.


<h3>Example (Interactive ipython I/O)</h3>

<pre>
   In [1]: from combinatorics import *

   In [2]: list(unlabeled_balls_in_unlabeled_boxes(10, [5,4,3,2,1]))
   Out[2]:
   [(5, 4, 1, 0, 0),
    (5, 3, 2, 0, 0),
    (5, 3, 1, 1, 0),
    (5, 2, 2, 1, 0),
    (5, 2, 1, 1, 1),
    (4, 4, 2, 0, 0),
    (4, 4, 1, 1, 0),
    (4, 3, 3, 0, 0),
    (4, 3, 2, 1, 0),
    (4, 3, 1, 1, 1),
    (4, 2, 2, 2, 0),
    (4, 2, 2, 1, 1),
    (3, 3, 3, 1, 0),
    (3, 3, 2, 2, 0),
    (3, 3, 2, 1, 1),
    (3, 2, 2, 2, 1)]
</pre>


<h2>4.4 <code>labeled_balls_in_unlabeled_boxes</code></h2>

<h3>Signature</h3>

<pre>
   labeled_balls_in_unlabeled_boxes(n, box_sizes)
</pre>


<h3>Overview</h3>

This function returns a generator that produces all distinct distributions of
distinguishable balls among indistinguishable boxes, with specified box sizes
(capacities). This is a generalization of the most common formulation of the
problem, where each box is sufficiently large to accommodate all of the balls.

<p><code>labeled_balls_in_labeled_boxes(balls, box_sizes)</code>: This function
returns a generator that produces all distinct distributions of distinguishable
balls among distinguishable boxes, with specified box sizes (capacities).  This
is a generalization of the most common formulation of the problem, where each
box is sufficiently large to accommodate all of the balls.


<h3>Example (Interactive ipython I/O)</h3>

<pre>
In [1]: from combinatorics import *

In [2]: list(labeled_balls_in_labeled_boxes(3, [2,2]))
Out[2]:
[((0, 1), (2,)),
 ((0, 2), (1,)),
 ((1, 2), (0,)),
 ((0,), (1, 2)),
 ((1,), (0, 2)),
 ((2,), (0, 1))]
</pre>


<h1>5. Partitions</h1>

<h2>5.1 Introduction</h2>

<em>In number theory and combinatorics, a partition of a positive integer n,
also called an integer partition, is a way of writing n as a sum of positive
integers. Two sums that differ only in the order of their summands are
considered to be the same partition.</em> (The quote is from
<a href="http://en.wikipedia.org/wiki/Partition_(number_theory">
http://en.wikipedia.org/wiki/Partition_(number_theory</a>).


<h2>5.2 <code>partitions</code></h2>

<h3>Signature</h3>

<pre>
   partitions(n)
</pre>


<h3>Overview</h3>

The function <code>partitions</code> generates partitions of the specified
integer using <code>unlabeled_balls_in_unlabeled_boxes</code>.
<span style="font-weight:bold">Note that this is an inefficient method for
generating partitions.</span>  <code>partitions2</code> (see the next section
of this document) is more efficient.  The code for <code>partitions</code>,
which is quite small, follows:

<pre>
def partitions(n):
   _partitions= unlabeled_balls_in_unlabeled_boxes(n, n*[n])

   for _partition in _partitions:
      yield tuple([p for p in _partition if p])
</pre>


<h3>Example (Interactive ipython I/O)</h3>

In this example, I generate a table listing the number of partitions versus $n$.

<pre>
   In [1]: for i in range(1, 16):
      ...:     print('%2d: %d' % (i, len(list(partitions(i)))))
      ...:
    1: 1
    2: 2
    3: 3
    4: 5
    5: 7
    6: 11
    7: 15
    8: 22
    9: 30
   10: 42
   11: 56
   12: 77
   13: 101
   14: 135
   15: 176
</pre>

Running the above example took more than a minute of CPU time on my computer.


<h2>5.3 <code>partitions2</code></h2>

<h3>Signature</h3>

<pre>
   partitions(n)
</pre>


<h3>Overview</h3>

The function <code>partitions2</code>, which was written by David Eppstein,
efficiently generates partitions of the specified integer. Generating a table of
the number of partitions of the first 15 integers requires a small fraction of a
second.


<h3>Example (Interactive ipython I/O)</h3>

In this example, I generate a table listing the number of partitions of the
number 6.

<pre>
   In [1]: from combinatorics import *

   In [2]: p= partitions2(6)

   In [3]: for i, partition in enumerate(p):
      ...:     print('%2d: %s' % (i, partition))
      ...:
    0: [1, 1, 1, 1, 1, 1]
    1: [1, 1, 1, 1, 2]
    2: [1, 1, 2, 2]
    3: [2, 2, 2]
    4: [1, 1, 1, 3]
    5: [1, 2, 3]
    6: [3, 3]
    7: [1, 1, 4]
    8: [2, 4]
    9: [1, 5]
   10: [6]
</pre>


<h1>6. Constrained Permutations</h1>

<p><code>off_by_one(n)</code>: This function returns a generator that enumerates
all possible solutions of the <i>off-by-one</i> problem.  This problem can be
stated as follows:</p>

<Blockquote> <b> <i> A list of n items is provided.  Each item is in its correct
position, or one before or one after its correct position.  Enumerate all
possibilities for the correct ordering. </b> </i> </Blockquote>

<p>(These are essentially constrained permutations). With two lines of code, one
can generate a table showing the number of possible permutations versus n:</p>

<pre class="literal-block">
for i in range(2,10):
   print('%d, %d' % (i, len(list(off_by_one(i))))
</pre>
<p>The output is as follows:</p>
<pre class="literal-block">
2, 2
3, 3
4, 5
5, 8
6, 13
7, 21
8, 34
9, 55
</pre>

<p>Note that the number of possible orders follows the Fibonacci sequence.</p>

<p><code>off_by_m(n, m)</code>: This recursive generator function enumerates all
possible solutions of the 'off-by-m' problem.  This problem, which is a
generalization of the off-by-one problem, can be stated as follows:</p>

<Blockquote> <b><i> A list of n items is provided.  Each item is in its correct
position, or up to m positions before or after its correct position.  Enumerate
all possibilities for the correct ordering. </i></b> </Blockquote>
