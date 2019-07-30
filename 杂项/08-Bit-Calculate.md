https://leetcode-cn.com/problems/divide-two-integers/description/?utm_source=LCUS&utm_medium=ip_redirect&utm_campaign=transfer2china

## 加减法

两个数异或运算后，结果即为不考虑进位的结果

两个数与运算后，对结果左移一位，即表示对应位是否有进位

两个结果循环相加，即为加法结果

对于减法，取补码即可

```cpp
int add(int lhs, int rhs)
{
    int nocarry, carry;
    do
    {
        nocarry = lhs ^ rhs;
        carry = (lhs & rhs) << 1;
        lhs = nocarry, rhs = carry;
    } while (nocarry & carry);
    return nocarry | carry;
}
int sub(int lhs, int rhs)
{
  	rhs = -rhs;
    int nocarry, carry;
    do
    {
        nocarry = lhs ^ rhs;
        carry = (lhs & rhs) << 1;
        lhs = nocarry, rhs = carry;
    } while (nocarry & carry);
    return nocarry | carry;
}
```

## 乘法

```cpp
        0101    a
    ×   0110    b
    ----------------
        0000
       0101
      0101
   + 0000
    -------------
     00011110
```

```cpp
int mult(int lhs, int rhs)
{
    int res = 0, sign = (lhs < 0) ^ (rhs < 0);
    if (lhs < 0)
        lhs = -lhs;
    if (rhs < 0)
        rhs = -rhs;
    do
    {
        if (rhs & 1)
            res = add(res, lhs);
        lhs <<= 1;
    } while ((rhs >>= 1) != 0);
    return sign ? -res : res;
}
```

## 除法

```cpp
       10110
    --------
10/   101101
      100000
    --------
        1101
        1000
    --------
         101
         100
    --------
           1
```

```cpp
int divide(int dividend, int divisor)
{
    if (dividend == -INT_MAX - 1 && divisor == -1)
        return INT_MAX;
    if (divisor == 0)
        return 0;
    long long int res = 0;
    int sign = (dividend < 0) ^ (divisor < 0);
    long long int bigDividend = dividend;
    long long int bigDivisor = divisor;
    if (bigDividend < 0)
        bigDividend = -bigDividend;
    if (bigDivisor < 0)
        bigDivisor = -bigDivisor;

    int count = 0;
    for (count = 0; bigDividend >= bigDivisor; bigDivisor <<= 1)
        ++count;
    for (int i = 0; i < count; ++i)
    {
        res <<= 1;
        bigDivisor >>= 1;
        if (bigDividend >= bigDivisor)
        {
            bigDividend -= bigDivisor;
            res |= 1;
        }
    }
    return sign ? -res : res;
}
```



