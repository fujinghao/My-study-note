# 一、函数

## 1.查找

---

### 1. 实现lower_bound

```cpp
//lower_bound() 函数用于在指定区域内查找不小于目标值的第一个元素
int MyLower_bound(vector<int>& nums, int target){
    int left = 0, right = nums.size() - 1;
    //找到比target小的第一个数（left）
    //把数组里的所有元素想象成相等的更好理解
    while (left <= right) {
        int mid = (left + right) / 2;
        if (nums[mid] >= target) {
            right = mid - 1;
        } 
        else {
            left = mid + 1;
        }
    }
    // return right + 1;
    return left;
}
```

### 2. 实现upper_bound

```cpp
//用于在指定范围内查找大于目标值的第一个元素
int MyUpper_bound(vector<int>& nums, int target){
    int left = 0, right = nums.size() - 1;
    while (left <= right) {
        int mid = (left + right) / 2;
        if (nums[mid] <= target) {
            left = mid + 1;
        } 
        else {
            right = mid - 1;
        }
    }
    return left;
}
```

## 2. 字符串处理

---

### 1. 实现split()

```cpp
vector<string> split(string str, char c){
    vector<string> ret;
    string cur;
    for (auto i : str){
        if (i == c){
            //防止把空字符串添加到开始
            if (!cur.empty()){
                ret.push_back(move(cur));
            }
        }
        else {
            cur += i;
        }
    }
    //防止把空字符串添加到jie'we
    if (!cur.empty()){
        ret.push_back(cur);
    }
    return ret;
}
```

### 2. 字母转化大小写

```cpp
//大写字符与其对应的小写字符的 ASCII 的差为 32,异或运算的不进位加法特点——即如果原数二进制第5位如果是0则变成1（加32），如果是1则变成0（减32）
alpha ^= 32;
//或者
alpha ^= (1 << 5);

```

### 3.字符串查找元素

```cpp
//如果查找成功，find()函数返回子串或字符在字符串中第一次出现的位置；否则，返回一个特殊值string::npos，表示查找失败。
size_t find (const string& str, size_t pos = 0) const;
size_t find (const char* s, size_t pos = 0) const;
size_t find (const char* s, size_t pos, size_t n) const;
size_t find (char c, size_t pos = 0) const;
#include <iostream>
#include <string>
 
int main()
{
    std::string str = "Hello, world!";
    std::cout << str.find("world") << '\n'; // 输出7
    std::cout << str.find('w') << '\n'; // 输出7
    std::cout << str.find("abc") << '\n'; // 输出18446744073709551615（即string::npos）
}
//上面的代码会在控制台输出“7\n7\n18446744073709551615”。
```

## 3 .数论

---

### 1.判断质数

```cpp

    bool isprime(int num){
        //1不是质数，2是
        if(num < 2)
            return false;
        for(int i = 2; i * i <= num; i++){
                if(num % i == 0){
                    return false;
                }
            }
        return true;
    }
//时间复杂度：O(n^(1/2))
 


```

### 2. 一个无符号整数其二进制中数字位数为 '1' 的个数（也被称为汉明重量）

```cpp
    int hammingWeight(uint32_t n) {
        int ret = 0;
        while (n) {
            //n &= n - 1;其运算结果恰为把 n 的二进制位中的最低位的 1 变为 0 之后的结果。
            n &= n - 1;
            ret++;
        }
        return ret;
    }
//时间复杂度：O(log⁡n)。循环次数等于 n的二进制位中 1 的个数，最坏情况下 n的二进制位全部为 1。我们需要循环 log⁡n 次。
```

### 3.给定整数 n，返回 所有小于非负整数 n 的质数的数量

```c++
//埃氏筛
    int countPrimes(int n) {
        //初始默认所有数为质数
        vector<int> isPrime(n, 1);
        int ans = 0;
        for (int i = 2; i < n; ++i) {
            if (isPrime[i]) {
                ans += 1;
                //如果 x 是质数，那么大于 x 的 x 的倍数 2x,3x,… 一定不是质数，因此我们可以从这里入手。
                for (int j = i + i; j < n; j += i) {
                    ////排除不是质数的数
                    isPrime[j] = 0;
                }
            }
        }
        return ans;
    }

```

### 4.最大公约数和最小公倍数

对于任意两个正整数 n，m的最小公倍数为` n×m / gcd⁡(n,m)·`.其中 gcd⁡(n,m) 为 n 和 m 的最大公约数。

```c++
//最大公约数,两个
__gcd(m,n)
//c++17
gcd(m, n)
//辗转相除法又叫做欧几里得算法，是指用于计算两个正整数a,b的最大公约数。
//思想：用较大的数除以较小的数得到余数，再用较小的数除以余数以此循环，直到最后余数为零，最后除的数即为最大公约数，代码为表示为
 gcd(a,b)=gcd(b,a mod b);
//递归实现
int gcd(int a,int b)
{
	if(b>a) return gcd(b,a);
	int remind=a%b;
	if(remind==0) return b;
	else return gcd(b,remind);
}

//循环实现
int gcd(int a,int b)
{
	if(b>a) return gcd(b,a);
	int remind=1;
	while(remind)
	{
		remind=a%b;
		a=b;
		b=remind;
	}
	return a;
}


//最小公倍数
lcm(m,n)
```

## 4. 栈

---

### 1. 验证栈序列

```cpp
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        stack<int> st;
        int j = 0;
        for (auto i : pushed){
            st.push(i);
            //当st为空时不能取top()
            while (!st.empty() && st.top() == popped[j]){
                j++;
                st.pop();
            }
        }
        return st.empty();
    }
```

## 5. 杂

### 1. 求区间AB相交的范围

```c++
int range = min(A_right, B_right) - max(A_left) 
```
