# 1.两数平方和

[633.sum-of-square-numbers](<https://leetcode-cn.com/problems/sum-of-square-numbers/>)

~~~
输入: 5
输出: True
解释: 1 * 1 + 2 * 2 = 5
~~~

题目描述：判断一个数是否为两个数的平方和。

```java
public boolean judgeSquareSum(int c) {
        int j=(int)Math.sqrt(c);
        int i=0;
        int sum;
        while(i<=j){
            sum=i*i+j*j;
            if(sum==c)
                return true;
            else if(sum>c){
                j--;
            }else
                i++;
        }
        return false;
    }
}
```

  