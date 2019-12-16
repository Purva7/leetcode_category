# Bit Operations

## Basic operations:
* 1. Flip `~x`. WARNING: Python returns negative numbers because of the proceding 1s.

* 2. Get the rightmost **set bit** `x & -x` . Other method is `x & ~(x-1)`. Note that `~(x-1) = -(x-1) - 1 = -x`. Remmber `~x = -x-1`. 


* 3. Check if all bit are set:
DO NOT TRY `~x == 0` because you got extra proceding bits.
Do x & (x + 1) INSTEAD!!!

* 4. Test if `1000..00`, `x&(x-1)==0`

* 5. Remove the last set bit `x&(x-1)`

* 6. Number of set bit `__builtin_popcount(int x)` (C++ CPU specific instruction)

* 7. Remove some bit `A &= ~(1 << bit)`

* 8. Get all 1-bits `~0`

* 9. Set a bit `x |= 1 << bits`, Fill all the postitions by `1`s
```
            x |= x >> 16
            x |= x >> 8
            x |= x >> 4
            x |= x >> 2
            x |= x >> 1
```
* 10. Get the left-most set bit step3 + `x ^ (x >> 1)`

WARNING: Python 3 integers aren't represented using the internal CPU representation, so you have to determine the sign yourself!

**`signbit = int(n < 0)`**

## Templates:

### A really nice summary <https://leetcode.com/problems/sum-of-two-integers/discuss/84278/A-summary:-how-to-use-bit-manipulation-to-solve-problems-easily-and-efficiently>

### Single number problem by XOR
Given an array of integers, every element appears `k (k > 1)` times except for one, which appears `p` times `(p >= 1, p % k != 0)`. Find that single one.

We count the number of occurrence of `1`s on each digit. Say, for a particular digit, the input array contains `w*k + p` ONEs at this digit, then, the number which repeats `p` times has a One at this digit. Otherwise, the input array contains `w*k` ONEs, then, the number which repeats `p` times has a ZERO at this digit.

Note that we do not need to count ZEROs because the input array should has a size mod `k` equal to `p`.

Java Code for the case `k = 3, p = 1`. We need two 32-bits variable to store the occurrence of `1`s on each digit, which ranges from `0b00` to `0b10` (`00->01->10->00`). Note `11` is `00` here.

***LC 137. Single Number II***

```
    x1 = 0
    x2 = 0
    for (int i : nums) {
        x2 ^= x1 & i;
        x1 ^= i;
        mask = ~(x1 & x2);
        x2 &= mask;
        x1 &= mask;
    }
    return x1
```

IF there are two numbers, we can divide the input array into two groups. If we know `x1^x2 > 0`, we know they must be different at one bit (ex. the rightmost set bit of `x1^x2`). Then we go ahead to XOR each group.

***LC 260. Single Number III***

```
    diff = reduce(lambda x,y: x^y, nums)
    diff &= -diff
    x1, x2 = 0, 0
    for n in nums:
        if n & diff:
            x1 ^= n
        else:
            x2 ^= n
    return [x1, x2]
```

IF there are three unique numbers `a`, `b`, `c`, and other number appears twice, let `x` be the XOR of all elements, 
The last set bit of the XOR of the last bit of `x^a` and `x^b` and `x^c` be the M-th bit.
Then we can show only one of `x^a`, `x^b` and `x^c` has a set M-th bit, the other two have unset M-th bit. (Why?)
Then we can identify a list of elements contain one target number, 
and solve the sub-problem of finding the other two among the remaining elements.

See:
http://zhedahht.blog.163.com/blog/static/25411174201283084246412/

### Bit Masks
Mask allows you to have a small subset (no more than 32 elements for intergers) but could be larger using C++ bit_set or larger numbers in Python.

Three convenient operations:
```
Set union A | B
Set intersection A & B
Set subtraction A & ~B
```
First, it allows dynamic programming to know the states quickly. Secondly, the shift of bits usually can be treated as a `O(1)` operation.

**LC 318. Maximum Product of Word Lengths** Find the maximum value of `length(word[i]) * length(word[j])` where the two words do NOT share common letters.
```
    def maxProduct(self, words):
        d = {}
        for w in words:
            mask = 0
            for c in set(w):
                mask |= 1 << (ord(c) - 90)
            d[mask] = max(d.get(mask, 0), len(w))
        return max([d[x] * d[y] for x in d for y in d if not x&y] or [0])
```
**LC 187. Repeated DNA Sequences** Find all the 10-letter-long sequences (substrings) that occur more than once in a DNA.
Given s = `"AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT"`,
Return: `["AAAAACCCCC", "CCCCCAAAAA"]`.

So here we have a **shifting mask** to indicate the sub-sequences.
```
    vector<string> findRepeatedDnaSequences(string s) {
        vector<string> ret;
        map<int, int> d;
        if (s.size() <= 10) return ret;
        int v = 0;
        for (int j = 0; j <= 8; ++j) {
            v <<= 2;
            v |= s[j] == 'A'? 0 : s[j] == 'T'? 1 : s[j] == 'C'? 2 : 3;
        }
        for (int i = 9; i < s.size(); ++i) {
            v = v << 2 & 0xfffff;
            v |= s[i] == 'A'? 0 : s[i] == 'T'? 1 : s[i] == 'C'? 2 : 3;
            if (d[v]++==1) ret.push_back(s.substr(i - 9, 10));
        }
        return ret;
    }
```

**847. Shortest Path Visiting All Nodes** Return shortest path that visits every node in a graph (NP-hard problem), but you can visit node, edges multiple times.
My blog:
<https://leetcode.com/problems/shortest-path-visiting-all-nodes/discuss/141666/10-lines-Python-Recursive-DP>
```
    def shortestPathLength(self, graph):
        df = {}
        def find(state, i):
            if (state, i) in df: return df[(state, i)]
            if state & (state - 1) == 0: return 0
            df[(state, i)] = sys.maxsize
            for j in graph[i]:
                if state & (1<<j):
                    df[(state, i)] = min(df[(state, i)], 1 + min(find(state, j), find(state&~(1<<i), j)))
            return df[(state, i)]
        return min(find(2**len(graph)-1, i) for i in range(len(graph)))
```
