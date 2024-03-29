# Diff Algorithms and Tools

Algorithms to determine the difference between two sequences (i.e. two files, token-streams, DNA sequences) typically try to determine:
- A minimum edit script to transform sequence A into sequence B
- A maximal common subsequence (from which a minimal edit script can be derived)

The best known (and implemented) algorithms are:
1. [An O(ND) Difference Algorithm and Its Variations by Eugen W. Myers](http://www.xmailserver.org/diff2.pdf)  
   This is the algorithm implemented in the *NIX diff tool and also Github's diff

2. [Histogram Diff]  
   This is an algorithm derived from `Patience Diff` and also implemented in Github's diff.

3. [Patience Diff by Bram Cohen](https://bramcohen.livejournal.com/73318.html)  
   Do a LCS on unique lines, since they are the best predictor of commonality (i.e. same origin)

## Myers Diff Algorithm

1. Do a breadth-first search to find the possible end-points reached (with increasing allowed d=edit distance)
2. Do this both from the start of both sequences and from the end, to determine (with linear space) the 
   midpoint (or points if they happen to be match points)
3. Split the original sequences  in a part before and a part after the midpoint(s)
4. Repeat steps 1-3 recursively on each part, making sure to take care of the edge case where 1 or both sequences are empty
   and also (as optimization) ensuring to already discard matching items at start and at end of both sequences

## Histogram Diff Algorithm

1. Find common lines/items between the two sequence and count their occurrences in both sequences
2. Take common lines/items with lowest occurrence count: take the first occurrence in each sequence and consider
   this the match pair used to split the original sequences
3. Split the original sequences in a part before and part after the match pair
4. Apply steps 1-3 recursively on each part

## Patience Diff Algorithm

1. Match the first lines of both if they're identical, then match the second, third, etc. until a pair doesn't match.
2. Match the last lines of both if they're identical, then match the next to last, second to last, etc. until a pair doesn't match.
3. Find all lines which occur exactly once on both sides, then do longest common subsequence on those lines, matching them up.
4. Do steps 1-2 on each section between matched lines

## Patience Sort Algorithm

Computing the longest increasing subsequence.

The algorithm:
0. Start with zero piles
1. Take each number in turn and:
2. Put it on the left-most pile whose top number is larger than the current number (while also recording the top number of the previous pile)
3. or start a new pile if needed
4. Once all numbers are handled: take the top number of the last pile and
   follow it's back-link, recursively
5. The list of numbers obtained in this way is the reverse longest common subsequence


Why does it work>
Because it basically is an implementation of a breadth first search.

Example:
```
  Seq: 7 1 5 4 8 2 10

                    Breadth-first search:                            Patience sort piles:                                                                                             
    Round #1:       7                                                7                                                                          
                                                                                                
    Round #2:       7                                                7                                          
                    1                                                1                                          
                                                                                                    
    Round #3:       7                                                7    5                                                                    
                    1 -> 5                                           1                                                          
                                                                                                    
    Round #4:       7                                                7    5                                          
                    1 -> 5                                           1    4                                           
                    1 -> 4                                                                                          
                    
    Round #5:       7 -> 8                                           7    5    8                                               
                    1 -> 5 -> 8                                      1    4                                                    
                    1 -> 4 -> 8                                                                                          
                                                                                                              
    Round #6:       7 -> 8                                           7    5    8                                               
                    1 -> 5 -> 8                                      1    4                                                    
                    1 -> 4 -> 8                                           2                                                             
                    1 -> 2                                                                                          
                                                                                                              
    Round #7:       7 -> 8 -> 10                                     7    5    8   10                                                     
                    1 -> 5 -> 8 -> 10                                1    4                                                      
                    1 -> 4 -> 8 -> 10                                     2                                                                  
                    1 -> 2 -> 10                                                                                          
```

Some more intuition about the equivalence of both representations: 
  - Starting a new pile is effectively saying: the current number can 'increase' the longest currently established subsequence
  - Putting a number on top of a file is saying: this number, when used in place of the old top number, is a better starting point (because lower) for the rest of the subsequence


## References

- [An O(ND) Difference Algorithm and Its Variations by Eugen W. Myers](http://www.xmailserver.org/diff2.pdf)
- [Patience Diff by Bram Cohen](https://bramcohen.livejournal.com/73318.html)
- [Tutorial + interactive test program for Myers Diff](http://simplygenius.net/Article/DiffTutorial1)
- [A very good tutorial for implementing the various diff algorithms](https://blog.jcoglan.com/2017/02/12/the-myers-diff-algorithm-part-1/)
- [Site explaining various diff algorithms](https://tiarkrompf.github.io/notes/)
- [A full study on the different git diff algorithms](https://link.springer.com/article/10.1007/s10664-019-09772-z)
- [Practical implementation tips for Myers in Python](https://blog.robertelder.org/diff-algorithm/)
- [Wiki on Diff Algorithms](https://wiki.c2.com/?DiffAlgorithm)
- [Java Implementation](https://bmsi.com/java/Diff.java)
- [Apache Commons Java Implementation](https://commons.apache.org/sandbox/commons-text/jacoco/org.apache.commons.text.diff/StringsComparator.java.html)
- [Python Implementation of Myers Greedy Algorithm](https://gist.github.com/adamnew123456/37923cf53f51d6b9af32a539cdfa7cc4)
- [Git Implementation of Myers Diff, in C](https://github.com/git/git/blob/b06d3643105c8758ed019125a4399cb7efdcce2c/xdiff/xdiffi.c#L347)
- [UNIX Diff Utility implementing Myers Diff, in C](https://github.com/Distrotech/diffutils/blob/9e70e1ce7aaeff0f9c428d1abc9821589ea054f1/src/analyze.c#L559)
- [Patience Diff Explained](https://blog.jcoglan.com/2017/09/19/the-patience-diff-algorithm/)
- [Histogram Diff Explained](https://tiarkrompf.github.io/notes/?/diff-algorithm/aside3)


## Implementation Notes

See https://github.com/gpellicciotta/hinolugi-support.java, the `com.hinolugi.support.diff` package

From an implementation point of view, the following have proven to be most challenging in implementing Myers diff with linear-space:
- Turning the 1-based used of x and y indexes into 0-based ones
- Understanding what to return as 'middle snake': actual implementation seem to differ in what they return 
  (and hence in how to handle the splitting)
- Understanding that the algorithm can work also with areas that are empty or half-empty


### Plan

- [x] Implement Myers greedy algorithm in Java
- [x] Implement Myers linear-space algorithm in Java
- [x] Implement Histogram diff in Java
- [x] Implement Patience sort in Java
- [~] Implement difftool in Java (for files)
      - [~] With ignore-lines
      - [x] With selectable algorithm
      - [x] With unified diff SES output
      - [x] With JSON diff SES output
      - [x] With JSON diff LCS output
      - [x] Streaming with configurable block-size (and sync on longest match sequence in last 15% of block)


### Relative Performance
Performance results when comparing 999 randomly generated (but reusing the same for each algorithm) character sequences of lengths between 0-1000:
```
myers linear-space diff                     total time: 0:00:02.889 (  2889ms), avg. time:      2ms, total-lcs:  93498, total-ses: 826794
patience diff with myers linear-space diff  total time: 0:00:02.593 (  2593ms), avg. time:      2ms, total-lcs:  92945, total-ses: 827900
patience diff with histogram diff           total time: 0:00:00.176 (   176ms), avg. time:      0ms, total-lcs:  42443, total-ses: 928904
histogram diff                              total time: 0:00:00.109 (   109ms), avg. time:      0ms, total-lcs:  42370, total-ses: 929050
myers greedy diff                           total time: 0:13:50.124 (830124ms), avg. time:    830ms, total-lcs:  93498, total-ses: 826794
myers reverse greedy diff                   total time: 0:16:30.160 (990160ms), avg. time:    991ms, total-lcs:  93498, total-ses: 826794
```

So only the Myers algorithms give a truly minimal LCS and the linear-space version is vastly superior to the others.
Patience diff also seems to give good results, probably because it falls back on Myers for most of its work (since the sequences are randomly generated and hence might not have many unique elements).
Histogram diff is blazingly fast and assumed to be optimal for code comparison.

Another comparison, now with short sequences of lengths 0-10 (and hence a much higher chance of there being unique items):
```
myers linear-space diff                     total time: 0:00:00.062 (    62ms), avg. time:      0ms, total-lcs:    338, total-ses:   9242
patience diff with myers linear-space diff  total time: 0:00:00.041 (    41ms), avg. time:      0ms, total-lcs:    337, total-ses:   9244
patience diff with histogram diff           total time: 0:00:00.012 (    12ms), avg. time:      0ms, total-lcs:    337, total-ses:   9244
histogram diff                              total time: 0:00:00.002 (     2ms), avg. time:      0ms, total-lcs:    337, total-ses:   9244
myers greedy diff                           total time: 0:00:00.323 (   323ms), avg. time:      0ms, total-lcs:    338, total-ses:   9242
myers reverse greedy diff                   total time: 0:00:00.296 (   296ms), avg. time:      0ms, total-lcs:    338, total-ses:   9242
```

Another comparison, now with medium sequences of lengths 0-100:
```
myers linear-space diff                     total time: 0:00:00.240 (   240ms), avg. time:      0ms, total-lcs:   8067, total-ses:  86504
patience diff with myers linear-space diff  total time: 0:00:00.092 (    92ms), avg. time:      0ms, total-lcs:   6316, total-ses:  90006
patience diff with histogram diff           total time: 0:00:00.052 (    52ms), avg. time:      0ms, total-lcs:   6276, total-ses:  90086
histogram diff                              total time: 0:00:00.027 (    27ms), avg. time:      0ms, total-lcs:   5844, total-ses:  90950
myers greedy diff                           total time: 0:00:10.013 ( 10013ms), avg. time:     10ms, total-lcs:   8067, total-ses:  86504
myers reverse greedy diff                   total time: 0:00:11.468 ( 11468ms), avg. time:     11ms, total-lcs:   8067, total-ses:  86504
```
