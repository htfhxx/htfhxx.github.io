##### 题目描述
You are given coins of different denominations and a total amount of money. Write a function to compute the number of combinations that make up that amount. You may assume that you have infinite number of each kind of coin.

Example 1:
```
Input: amount = 5, coins = [1, 2, 5]
Output: 4
Explanation: there are four ways to make up the amount:
5=5
5=2+2+1
5=2+1+1+1
5=1+1+1+1+1
```
Example 2:
```
Input: amount = 3, coins = [2]
Output: 0
Explanation: the amount of 3 cannot be made up just with coins of 2.
```
Example 3:
```
Input: amount = 10, coins = [10] 
Output: 1
 
```
Note:
```
You can assume that

0 <= amount <= 5000
1 <= coin <= 5000
the number of coins is less than 500
the answer is guaranteed to fit into signed 32-bit integer
```
##### 提交链接

https://leetcode.com/problems/coin-change-2/


##### 代码思路
coins=[1, 2, 5]   amount=5 dp[0]=1

dp[1]-dp[5]:
0 0 0 0 0 
1 1 1 1 1 
1 2 2 3 3 
1 2 2 3 4 


dp[amount]可以定义为累积值为amount的方法数

整个过程可以看做是：
每次对coins的迭代相当于加入了一种新硬币
每次对amount的迭代就是加入新硬币之后对累积值为amount的方法数的更新

更新过程为：
目前累积值dp[amount]，只使用了前i-1种硬币
累积值amount-coins[i]向累积值amount变化的过程中加入了新硬币，添加了一种新方法
amount-coins[i]  -> amount    方法数： 1
0  ->      amount-coins[i]    方法数：dp[amount-coins[i]]
因此：dp[j]=dp[j]+dp[j-coins[i]]


##### 代码实现

```
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        if(amount==0)
            return 1;
        if(coins.empty()==true || amount<0)  
            return 0;
        vector<int> dp(amount+1,0);
        dp[0]=1;  //重要
        for(int i=0;i<coins.size();i++){
            for(int j=coins[i];j<=amount;j++){
                dp[j]=dp[j]+dp[j-coins[i]];
            }
        }
        return dp[amount];
    }
};





```
