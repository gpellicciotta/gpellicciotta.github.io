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
- [Site explaining various diff algorithms](https://tiarkrompf.github.io/notes/)
- [A full study on the different git diff algorithms](https://link.springer.com/article/10.1007/s10664-019-09772-z)
