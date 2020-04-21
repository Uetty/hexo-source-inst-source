## 数位DP

数位dp是为了解决类似这样一个问题：不大于n的数（有上界限制），不包含9的数的个数

例如：n = 534时，0-534中有多少个数不包含9



该题使用数位dp的方法解题，思路是从高位开始往下枚举，每一位枚举都不能让枚举的数超过上界534。当百位枚举5，那么十位就只能是0-3，当百位枚举5且十位枚举3，那么个位就只能枚举0-4



按此规则模拟运行一遍数位DP：

构建dp数组，dp数组的第1维表示枚举第几位（从高位开始）。原本只需是一维数组，但由于上界限制的存在，就需要增加一维（第二维）作为状态位，表示是否到达上限，所以dp数组表示为：`dp = new int[3][2]`，值表示符合的枚举有多少个，0表示没限制，1表示有限制

```
dp[0][0] = 5       // 没有到上限情况下0-4没有9
dp[0][1] = 1       // 有到上限情况下5-5没有9

dp[1][0] = ∑(0-8) dp[0][0] + ∑(0-2) dp[0][1] = 48  // 没有到上限的情况下，上一级
dp[1][1] = ∑(3-3) dp[0][1] = 1           // 到上限的情况下，上一级也是到上限的

dp[2][0] = ∑(0-8) dp[1][0] + ∑(0-3) dp[1][1] = 436 
dp[2][1] = ∑(4-4) dp[1][1] = 1

所以从0-534里，没有9的数的个数是436 + 1 = 437
```





题目更改为：0-534中不包含9和52

`dp = new int[3][2][2]`，第三维表示当前数值后缀与52前缀匹配的个数（这里只需考虑0个和1个就能求解，所以长度是2）

```
dp[0][0][0] = 5    // 0-4符合：没有到上限且最后一位不是5
dp[0][0][1] = 0    // 不存在符合：没有到上限且最后一位是5 （0-4和5没有交集）
dp[0][1][0] = 0    // 不存在符合：到上限且最后一位不是5
dp[0][1][1] = 1    // 5符合：到上限且最后一位是5

dp[1][0][0] = ∑(0-4) dp[0][0][0] + ∑(6-8) dp[0][0][0] + ∑(0-1) dp[0][0][1] 
    + ∑(3-4) dp[0][0][1] + ∑(6-8) dp[0][0][1] + ∑(0-2) dp[0][1][0]
    + ∑(0-1) dp[0][1][1] = 42
dp[1][0][1] = ∑(5-5) dp[0][0][0] + ∑(5-5) dp[0][0][1] = 5   (上位是上界本位没有5)
dp[1][1][0] = ∑(3-3) dp[0][1][0] + ∑(3-3) dp[0][1][1] = 1
dp[1][1][1] = 0      (本位是上界，上一位必为上界，上位是上界本位没有5)

dp[2][0][0] = ∑(0-4) dp[1][0][0] + ∑(6-8) dp[1][0][0] + ∑(0-1) dp[1][0][1]
    + ∑(3-4) dp[1][0][1] + ∑(6-8) dp[1][0][1] + ∑(0-3) dp[1][1][0] 
    + ∑(0-1) dp[1][1][1] + ∑(3-3) dp[1][1][1] = 375
dp[2][0][1] = ∑(5-5) dp[1][0][0] + ∑(5-5) dp[1][0][1] + 0 = 47
dp[2][1][0] = ∑(4-4) dp[1][1][0] + ∑(4-4) dp[1][1][1] = 1
dp[2][1][1] = 0
```

编程

```
	private static int calculateByDigestDp(int n) {
        int[] splitNum = splitNum(n);
        
        int[][][] dp = new int[splitNum.length][][];
        dp[0] = new int[2][];
        dp[0][0] = new int[2];
        dp[0][1] = new int[2];

        for (int k = 0; k <= splitNum[0]; k++) {
            if (k == 9) {
                continue;
            }
            int limit = 0;
            int isFive = 0;
            if (k == 5) {
                isFive = 1;
            }
            if (k == splitNum[0]) {
                limit = 1;
            }
            dp[0][limit][isFive]++;
        }
        for (int i = 1; i < splitNum.length; i++) {
            dp[i] = new int[2][];
            dp[i][0] = new int[2];
            dp[i][1] = new int[2];

            // last not limit
            for (int k = 0; k <= 9; k++) {
                if (k == 9) {
                    continue;
                }
                int isFive = 0;
                if (k == 5) {
                    isFive = 1;
                }
                if (k != 2) {
                    dp[i][0][isFive] += dp[i - 1][0][1];
                }
                dp[i][0][isFive] += dp[i - 1][0][0];
            }

            // last limit
            for (int k = 0; k <= splitNum[i]; k++) {
                if (k == 9) {
                    continue;
                }
                int isFive = 0;
                if (k == 5) {
                    isFive = 1;
                }

                if (k == splitNum[i]) {
                    if (k != 2) {
                        dp[i][1][isFive] += dp[i - 1][1][1];
                    }
                    dp[i][1][isFive] += dp[i - 1][1][0];
                } else {
                    if (k != 2) {
                        dp[i][0][isFive] += dp[i - 1][1][1];
                    }
                    dp[i][0][isFive] += dp[i - 1][1][0];
                }
            }
        }

        return dp[splitNum.length - 1][0][0]+ dp[splitNum.length - 1][0][1]
                + dp[splitNum.length - 1][1][0] + dp[splitNum.length - 1][1][1];
    }

    private static int[] splitNum(int n) {
        int count = 0;
        int nn = n;
        while (n > 0) {
            n /= 10;
            count++;
        }
        if (count == 0) {
            count++;
        }
        int[] splitNum = new int[count];
        for (int i = 0; i < count; i++) {
            splitNum[count - 1 - i] = nn % 10;
            nn /= 10;
        }
        return splitNum;
    }
```



