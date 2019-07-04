用 C++ 实现计算两个日期间的天数：

```c++
#include <iostream>
using namespace std;
bool IsLeapYear(int year)
{
    return ((year % 4 == 0 && year % 100 != 0) || year % 400 == 0);
}
// 计算从0001.01.01到指定日期的天数 
int GetDays(int year, int month, int day)
{
    int monthDays[] = { 0,31,28,31,30,31,30,31,31,30,31,30,31 };
    if (IsLeapYear(year))
        monthDays[2]++;
    int result = 0;
    for (int i = 1; i < year; i++)
    {
        result += 365;
        if (IsLeapYear(i))
            result++;
    }
    for (int i = 1; i < month; i++)
    {
        result += monthDays[i];
    }
    result += day;

    return result;
}
int DayDis(int year1, int month1, int day1,
    int year2, int month2, int day2)
{
    return abs(GetDays(year2, month2, day2) - GetDays(year1, month1, day1));
}

int main(void)
{
    cout << DayDis(2012, 3, 12, 2018, 3, 25) << endl;
}
```

轮子：

```c++
#include <iostream>
#include <ctime>
using namespace std;

int main()
{
    struct tm a = { 0,0,0,12,3,112 }; // 2012.03.12
    struct tm b = { 0,0,0,5,7,118 };  // 2018.07.05
    time_t x = mktime(&a);
    time_t y = mktime(&b);
    int secPerDay = 60 * 60 * 24;
    if (x != (time_t)(-1) && y != (time_t)(-1))
    {
        double difference = difftime(y, x) / secPerDay;
        cout << difference << endl;
    }
}
```

