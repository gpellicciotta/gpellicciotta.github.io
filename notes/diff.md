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

## Patience Diff Algorithm

1. Match the first lines of both if they're identical, then match the second, third, etc. until a pair doesn't match.
2. Match the last lines of both if they're identical, then match the next to last, second to last, etc. until a pair doesn't match.
3. Find all lines which occur exactly once on both sides, then do longest common subsequence on those lines, matching them up.
4. Do steps 1-2 on each section between matched lines

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

### Plan

- [x] Implement Myers greedy algorithm in Java
- [x] Implement Myers linear-space algorithm in Java
- [x] Implement Histogram diff in Java
- [x] Implement Patience sort in Java
- [~] Implement difftool in Java (for files)
      - With ignore-lines
      - With selectable algorithm
      - With unified diff SES output
      - With JSON diff SES output
      - With JSON diff LCS output
      - Streaming with configurable block-size (and sync on longest match sequence in last 15% of block)


### Relative Performance
Performance results when comparing 999 randomly generated (but reusing the same for each algorithm) character sequences of lengths between 0-1000:
```
Executed 999 diffs with total time: 0:00:02.889 (  2889ms), avg. time:      2ms, total-lcs:  93498, total-ses: 826794, algorithm: myers linear-space diff
Executed 999 diffs with total time: 0:00:02.593 (  2593ms), avg. time:      2ms, total-lcs:  92945, total-ses: 827900, algorithm: patience diff with myers linear-space diff
Executed 999 diffs with total time: 0:00:00.176 (   176ms), avg. time:      0ms, total-lcs:  42443, total-ses: 928904, algorithm: patience diff with histogram diff
Executed 999 diffs with total time: 0:00:00.109 (   109ms), avg. time:      0ms, total-lcs:  42370, total-ses: 929050, algorithm: histogram diff
Executed 999 diffs with total time: 0:13:50.124 (830124ms), avg. time:    830ms, total-lcs:  93498, total-ses: 826794, algorithm: myers greedy diff
Executed 999 diffs with total time: 0:16:30.160 (990160ms), avg. time:    991ms, total-lcs:  93498, total-ses: 826794, algorithm: myers reverse greedy diff
```

So only the Myers algorithms give a truly minimal LCS and the linear-space version is vastly superior to the others.
Patience diff also seems to give good results, probably because it falls back on Myers for most of its work (since the sequences are randomly generated and hence might not have many unique elements).
Histogram diff is blazingly fast and assumed to be optimal for code comparison.

Another comparison, now with short sequences of lengths 0-10 (and hence a much higher chance of there being unique items):
```
Executed 999 diffs with total time: 0:00:00.062 (    62ms), avg. time:      0ms, total-lcs:    338, total-ses:   9242, algorithm: myers linear-space diff
Executed 999 diffs with total time: 0:00:00.041 (    41ms), avg. time:      0ms, total-lcs:    337, total-ses:   9244, algorithm: patience diff with myers linear-space diff
Executed 999 diffs with total time: 0:00:00.012 (    12ms), avg. time:      0ms, total-lcs:    337, total-ses:   9244, algorithm: patience diff with histogram diff
Executed 999 diffs with total time: 0:00:00.002 (     2ms), avg. time:      0ms, total-lcs:    337, total-ses:   9244, algorithm: histogram diff
Executed 999 diffs with total time: 0:00:00.323 (   323ms), avg. time:      0ms, total-lcs:    338, total-ses:   9242, algorithm: myers greedy diff
Executed 999 diffs with total time: 0:00:00.296 (   296ms), avg. time:      0ms, total-lcs:    338, total-ses:   9242, algorithm: myers reverse greedy diff
```

Another comparison, now with medium sequences of lengths 0-100:
```
Executed 999 diffs with total time: 0:00:00.240 (   240ms), avg. time:      0ms, total-lcs:   8067, total-ses:  86504, algorithm: myers linear-space diff
Executed 999 diffs with total time: 0:00:00.092 (    92ms), avg. time:      0ms, total-lcs:   6316, total-ses:  90006, algorithm: patience diff with myers linear-space diff
Executed 999 diffs with total time: 0:00:00.052 (    52ms), avg. time:      0ms, total-lcs:   6276, total-ses:  90086, algorithm: patience diff with histogram diff
Executed 999 diffs with total time: 0:00:00.027 (    27ms), avg. time:      0ms, total-lcs:   5844, total-ses:  90950, algorithm: histogram diff
Executed 999 diffs with total time: 0:00:10.013 ( 10013ms), avg. time:     10ms, total-lcs:   8067, total-ses:  86504, algorithm: myers greedy diff
Executed 999 diffs with total time: 0:00:11.468 ( 11468ms), avg. time:     11ms, total-lcs:   8067, total-ses:  86504, algorithm: myers reverse greedy diff
```
