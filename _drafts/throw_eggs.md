## K个蛋在1-N层的极限不摔碎的层数的投掷策略

确定K个蛋在1-N层哪一层极限不摔碎的投掷策略为：

1. 使用第1个蛋依次在第 `m(1, 1)`、`m(1, 2)`、`m(1, 3)`、...、`m(1, j1)`、...、`m(1, M1)`=`N-1` 层投掷，若蛋在第`j1`次摔碎，则可确定蛋的极限不摔碎点位于`m(1, j1-1)`到`m(1, j1)-1`中的其中一个。（可以看出，第1个蛋最多投掷M1次）
2. 使用第2个蛋依次在第 `m(2, 1)`、`m(2, 2)`、`m(2, 3)`、...、`m(2, j2)`、...、`m(2, M2)` 层投掷，其中`m(2, 1)>m(1, j1-1)`、`m(2, M2)=m(1, j1)-1`，若蛋在第`j2`层摔碎，则可确定蛋的极限不摔碎点位于`m(2, j2-1)`到`m(2, j2)-1`中的其中一个。（可以看出，第2个蛋最多投掷M2次）
3. 第3个蛋开始，与第2个蛋同理类推。
4. 第K个蛋是最后一个蛋，且当前确定极限不摔碎点位于`m(k-1, j-1)`到`m(k-1, j)-1`，则从`m(k-1, j-1)+1`层开始一层层往上投掷，直至摔碎或投掷完`m(k-1,j)-1`层，即可确定极限不摔碎点。

对于第1个蛋，最多投掷M1次（第M1次投掷是第N层），共可产生M1种摔碎状况（当然，若在第N层投掷不摔碎，则可以直接断定极限不摔碎层是N，包含不摔碎的情况实际是M1+1种状况），摔碎情况下为下个蛋确定的区间有M1种，分别为：

```
摔碎层        确定的区间
m(1, 1)      [0, m(1, 1))
m(1, 2)      [m(1, 1), m(1, 2))
m(1, 3)      [m(1, 2), m(1, 3))
             ...
m(1, M1-1)   [m(1, M1-2), m(1, M1-1))
m(1, M1)=N   [m(1, M1-1), N)
N层不摔碎      直接确定极限不摔碎层为N
```

**令g(K, a, b)表示为**：剩余K个蛋的情况下确定区间a到b的极限不摔碎层的最大投掷次数的最优值（最优即最小）。则第1个蛋在第i次投掷摔碎时：`g(K, 0, N) = i + g(K-1, m(1,i-1), m(1, i))`，各种摔碎状况下的g(K, 0, N)如下所示：

```
摔碎层        确定的区间                     投掷总次数(g(K, 0, N))
m(1, 1)      [0, m(1, 1))                 1 + g(K-1, 0, m(1, 1))
m(1, 2)      [m(1, 1), m(1, 2))           2 + g(K-1, m(1, 1), m(1, 2))
m(1, 3)      [m(1, 2), m(1, 3))           3 + g(K-1, m(1, 2), m(1, 3))
             ...                          
m(1, M1-1)   [m(1, M1-2), m(1, M1-1))     M1-1 + g(K-1, m(1, M1-2), m(1, M1-1))
m(1, M1)=N   [m(1, M1-1), N]              M1 + g(K-1, m(M1-1), N)
N层不摔碎      直接确定极限不摔碎层为N          M1
```

为了使最大投掷次数最小，需要确保每种状况下的最大投掷次数尽可能均等分布（除第1个投掷区间最大投掷次数较小外，其他区间最大投掷次数均等）。如果不均等，说明还有优化空间，将最大投掷次数大的区间减小，移到最大投掷次数较小的区间，达到尽可能均等为止，这时候整体最大投掷次数不变或减小。

因为在第M1次（第N层）投掷不摔碎时，投掷次数为M1，所以每种摔碎状况下最大投掷次数也限制为M1。在完全均等分布的情况下（如果不完全均等分布，扩大N的值，也能达到完全均等），就有如下等式：

```
g(K-1, 0, m(1, 1)) = M1-1
g(K-1, m(1, 1), m(1, 2)) = M1-2
g(K-1, m(1, 2), m(1, 3)) = M1-3
...
g(K-1, m(1, M1-2), m(1, M1-1)) = 1
g(K-1, m(M1-1), N) = 0
```

对于`g(K-1, p, q)`，其结果实际只与`q-p`的差值有关，与起点`p`值没有相关性的。所以上述又可表示为

``` 
g(K-1, 0, d(1)) = M1-1            d(1)=m(1, 1) - 0
g(K-1, 0, d(2)) = M1-2            d(2)=m(1, 2) - m(1, 1)
g(K-1, 0, d(3)) = M1-3            d(3)=m(1, 3) - m(1, 2)
...
g(K-1, 0, d(M1-1)) = 1            d(M1-1)=m(1, M1-1) - m(1, M1-2)
g(K-1, 0, d(M1)) = 0              d(M1)=N - m(1, M1-1)

可以看出
d(1) + d(2) + d(3) + ... + d(M1-1) + d(M1) = N
```

令`f( g(K-1, 0, d) ) = d`，即`f(x)`是`g(K-1, 0, d)`关于d的反向函数，上面等式序列等价于

```
f(M1-1) = d(1)
f(M1-2) = d(2)
f(M1-3) = d(3)
...
f(1) = d(M1-1)
f(0) = d(M1)

f(0) + f(1) + ... + f(M1-2) + f(M1-1) = N
```

我们的目标是求最小的M1，即在满足`f(0) + f(1) + ... + f(M1-1) >= N`情况下M1的最小值（`f(0)=0`可以忽略）。为此，`f(1)`、...、`f(M1-1)`都必须是最大值。即K-1个鸡蛋投掷1次、...、M1-1次能覆盖的最大楼层数之和刚好能够达到`N`，那么这个M1便是K个蛋在N层楼最佳投掷策略下的最大投掷数（注意：检测K-1个蛋是检测M1-1次，最终的投掷次数是M1次，能够覆盖的层数是`f(1)+...+f(M1-1)`层）。



编程：

```
	public int superEggDrop(int K, int N) {
        int[][] cache = new int[K-1][]; // cache[i][j]缓存i+2个蛋，N层楼时需要投掷多少次
        for (int i = 2; i <= K; i++) {
            cache[i - 2] = new int[N + 1];
            Arrays.fill(cache[i - 2], -1);
            cache[i - 2][0] = 0;
        }
        return superEggDrop(K, N, cache);
    }

    private int superEggDrop(int K, int N, int[][] cache) {
        if (K == 1) return N;

        if (cache[K-2][N] != -1) {
            return cache[K-2][N];
        }

        int addupFloor = 0;
        int needEggs = 0;

        for (int i = 1; i < cache[K-2].length; i++) {
            int needEggs1 = superEggDrop(K-1, i, cache);
            if (needEggs1 > needEggs) {
                int startFloor = addupFloor + 1;
                addupFloor += i;
                for (int j = startFloor; j <= addupFloor && j < cache[K-2].length; j++) {
                    cache[K-2][j] = needEggs + 1;
                }
                needEggs = needEggs1;
            }
            if (addupFloor >= cache[K-2].length) {
                break;
            }
        }
        for (int j = addupFloor + 1; j < cache[K-2].length; j++) {
            cache[K-2][j] = needEggs + 1;
        }

        return cache[K-2][N];
    }
```


