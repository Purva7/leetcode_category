# Dynamic Programming

After you figure out the reduction rule, you should try to optimize the DP:

## Rolling array to save the memory 

You should save memory cost whenever possible. For 2D DP problem, keep asking yourself if the memory cost could be actually 1D.
Try to simulate the DP process in a table and find out the way to do 1D.

**LC 64. Minimum Path Sum** Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.

The basic DP reduction is 2D
```
dp[i,j] = min(dp[i-1,j], dp[i,j-1]) + grid[i][j]
```
But actually we just need 1D vector the store the states because
```
        # A  B  C    This is the old row memo
        #    V  V
        # a->b->c   This is the mew row memo
        #           You update b using a and B
```

Python Code:
```
        if not grid: return 0
        m = len(grid[0])
        memo = [0] + [2**31] * (m-1)
        for row in grid:
            memo[0] = memo[0] + row[0]
            for j in range(m-1):
                memo[j+1] = min(memo[j:j+2]) + row[j+1]
        return memo[-1]
```

**LC 174. Dungeon Game**
A knight gains/loses health entering a room. He dies if health <= 0. Return the knight's minimum initial health to get (N, M) from (0, 0).

So reversively, we compute the min health at `(i,j)` (before gains/lose health) so the knight can reach (N, M). Obviously, `dp[N-1, M-1] = 1 - dungeon[i, j]`. The reduction rule is

```
dp[i, j] = max(1, min(dp[i+1,j],dp[i,j+1]) - dungeon[i, j] )
```

After the simplification, we get
```
        if not dungeon: return 0
        n, m = len(dungeon), len(dungeon[0])
        memo = [2**31] * (m-1) + [1]
        for i in range(n)[::-1]:
            for j in range(m)[::-1]:
                memo[j] = max(1, min(memo[j:j+2])-dungeon[i][j])
        return memo[0]
```
Yes, we can always use standard DP, but actually one row memo would satisfy the needs.

**LC 714. Best Time to Buy and Sell Stock with Transaction Fee** (and the series of stock transaction problems.)

Given the daily prices of a stock, you may buy and sell on each day (only sell after you buy, and buy after you sell).
You may complete as many transactions as you like, but you need to pay the transaction fee for each transaction `fee`.

The max profit at day `i` is
```
        dp[i] = max(dp[i-1], max(p[i] - p[j] - fee + dp[j-1], for all 0 <= j < i) ) 
```
which can be simpilified as
```
        dp[i] = max(dp[i-1], p[i] - fee + max( dp[j-1] - p[j] ) ) 
```
Let `sell = dp[i]` and `buy = max( dp[j-1] - p[j] )`, only two variables are enough for the DP.

Python Code:
```
    def maxProfit(self, prices, fee):

        sell, buy = 0, -float('inf')
        for p in prices:
            buy = max(buy, sell - p)
            sell = max(sell, buy + p - fee)
        return sell
```

## Reduce time complexity by "differentiating" the states

In some cases, `dp[i]` can be related to `dp[i-1]`. So it's better to use `dp[i] - dp[i-1]` as the real dp state.
After we have derived the reduction rule, we should question that **"How are the dp states related to each other"**?


As an extension to **LC 45. Jump Game II**, see **Lintcode climbing-stairs-iii**
<https://www.jiuzhang.com/solution/climbing-stairs-iii/>

The question asks how many ways one can jump to n-th steps from 0th step, assuming at the i-th steps, she can jump `[1, n[i]]` steps.

Suppose there are `x[i]` ways to reach i-th step. Then `x[i] = sum(x[j] for any j < i that j + n[j] >= i)`.

If we use `x[i]` as the DP state directly, the time complexity would be `O(n^2)` because it involves a range operation - for each position `j`, we should add `x[j]` to all the `x[i]`s in the range `j+1 <= i <= j+n[j]`.

But if we differentiate `x[i+1]` and `x[i]`, it is suprisingly simple that `x[i+1]-x[i]` is `x[i]` minus the sum of those `x[k]`s who can reach `i` but not `i+1`. So if we use `dp[i+1] = x[i+1] - x[i]`, we just need to add the current `x[i]` to `dp[i + 1]` and add `-x[i]` to some `i+n[i]+1` - the first position we can't reach from `i`.

Note that `x[i] = dp[i] + dp[i-1] + .. + dp[0]` can by maintained dynamically.

`O(N)` time Python solution:
```
def climb_stairs_iii(self, n):
    N = len(n)
    dp = [1] + [0] * N
    xi = 1
    for i in range(1, N + 1):
        xi = xi + dp[i]
        dp[i + 1] += xi
        dp[i + n[i] + 1] -= xi
    return xi
```

More complicated case, see the [`O(n^2)` solution](https://leetcode.com/problems/guess-number-higher-or-lower-ii/discuss/84826/An-O(n2)-DP-Solution-Quite-Hard.) for **LC 45. Jump Game II**


Problem categories and related videos: (below)
Palindrome Based: LC 516, 5 647
Palindrome Partitioning: LC 132 133
Decode ways: LC 91
Stocks: LC 121 122 309 714
Path : LC 62 63 64
Jump Game : 55 45
Stairs: 70 746
Wildcard: LC 44 10
Word Break: 139 140
Max Sq & Rect: 85 221
Super Egg: LC 887
Coin Change: LC 322, 518, 441
Longest Common substring: 718

Others :
House Robber: LC 213, 198
Paint House : LC 256 265
Subarray : LC 53, 152
Subsequence LIS : LC 300 354
Math: LC 279 343 204

============================================================================================
Jump Game : LC 55, 45
https://www.youtube.com/watch?v=cETfFsSTGJI&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=22



Stair case : LC 70, 746
https://www.youtube.com/watch?v=CFQk7OQO_xM&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=31



Paths ( Unique path and Min Path): LC 62,63,64
https://www.youtube.com/watch?v=lBRtnuxg-gU&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=21
https://www.youtube.com/watch?v=lBRtnuxg-gU&index=21&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr



Stock Based: LC 121,122
https://www.youtube.com/watch?v=oDhu5uGq_ic&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=20



Egg Dropping: LC-887
https://www.youtube.com/watch?v=3hcaVyX00_4&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=13



Wildcard Matching: LC 44 && Regular Expression: LC 10
https://www.youtube.com/watch?v=3ZDZ-N0EPV0&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=24
https://www.youtube.com/watch?v=l3hda49XcDE&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=17





Longest Palindromic Subsequence: LC 516, 5, 647
https://www.youtube.com/watch?v=_nCsPn7_OgI&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=9

Min Edit distance: LC 72
https://www.youtube.com/watch?v=We3YDTzNXEk&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=7



Longest Incresing Subsequence: LC 300, 354
https://www.youtube.com/watch?v=CE2b_-XfVDk&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=8



Coin Change I & II: LC 322, 518, 441
https://www.youtube.com/watch?v=NJuKJ8sasGk&index=10&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr
https://www.youtube.com/watch?v=_fgjrs570YE&index=11&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr



Max Size Rectangle: LC 85, 84
https://www.youtube.com/watch?v=g8bSdXCG-lA&index=18&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr



Max Subsquare in 2D Matrix: LC 221
https://www.youtube.com/watch?v=_Lf1looyJMU&index=28&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr

Palindrome Partition : LC 132, 133
https://www.youtube.com/watch?v=lDYIvtBVmgo&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=26



Interleaving String: LC 97
https://www.youtube.com/watch?v=ih2OZ9-M3OM&index=32&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr



Word Break : LC 139 140 ( But I didn't follow this link)
https://www.youtube.com/watch?v=WepWFGxiwRs&index=19&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr



============================================================================================

Subset Sum:
https://www.youtube.com/watch?v=s6FhG--P7z0&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=4



Longest Common substring:
https://www.youtube.com/watch?v=BysNXJHzCEs&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=16



Max Sum Rectangle Submatrix in Matrix: Similar 85, 363
https://www.youtube.com/watch?v=yCQN096CwWM&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr&index=15

