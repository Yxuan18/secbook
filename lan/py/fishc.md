---
description: 此选项为小甲鱼《零基础入门学习python（第二版）》对应内容，所有涉及脚本将会写在这里
---

# fishc

##  第一个小游戏代码

{% tabs %}
{% tab title="Python" %}


{% code title="\# 2-1.py" %}
```python
temp = input("不妨猜一下小甲鱼现在心里想的是哪个数字：")
guess = int(temp)
if guess == 8:
    print("你是小甲鱼心里的蛔虫吗")
    print("哼，猜中了也没有奖励")
else:
    print("猜错了，我心里想的数字是8！")
print("游戏结束")
```
{% endcode %}

{% code title="\# 3-1.py" %}
```python
temp = input("不妨猜一下小甲鱼现在心里想的是哪个数字：")
guess = int(temp)
while guess != 8:
    if guess > 8:
        print("大了")
    else:
        print("小了")
    temp = input("再试试：")
    guess = int(temp)
print("你是小甲鱼心里的蛔虫吗")
```
{% endcode %}



{% code title="\# 3-2.py" %}
```python
import random

secret = random.randint(1,10)
temp = input("不妨猜一下小甲鱼现在心里想的是哪个数字：")
guess = int(temp)
times = 1
while (guess != secret) and (times < 3):
    if guess > secret:
        print("大了")
    else:
        print("小了")

    temp = input("再试试：")
    guess = int(temp)
    times = times + 1

if (times <= 3) and (guess == secret):
    print("你是小甲鱼心里的蛔虫吗")
    print("哼，猜中了也没有奖励")
else:
    print("三次都错了，不和你玩了")
```
{% endcode %}
{% endtab %}
{% endtabs %}



##  分支与循环

```python
# 4-1.py

score = int(input("请输入一个分数："))

if 100 >= score >= 90:
    print('A')
if 90 >= score >= 80:
    print('B')
if 80 >= score >= 70:
    print('C')
if 70 >= score >= 60:
    print('D')
if score < 0 or score > 100:
    print("输入错误！")
```

```python
# 4-2.py

score = int(input("请输入一个分数："))

if 100 >= score >= 90:
    print('A')
else:
    if 90 >= score >= 80:
        print('B')
    else:
        if 80 >= score >= 70:
            print('C')
        else:
            if 70 >= score >= 60:
                print('D')
            else:
                if score < 0 or score > 100:
                    print("输入错误！")

```

```python
# 4-3.py

score = int(input("请输入一个分数："))

if 100 >= score >= 90:
    print('A')
elif 90 >= score >= 80:
    print('B')
elif 80 >= score >= 70:
    print('C')
elif 70 >= score >= 60:
    print('D')
elif score < 0 or score > 100:
    print("输入错误！")
```

```c
# 4-4.c

##include <stdio.h>
int main(void)
{
    int age = 20;
    char sore = 'A';

    if (age>18)
    if (sore == 'A')
    printf("恭喜获得一等奖");
    else
    printf("你年龄不对");
        return 0;
}
```

```python
# 4-5.py

age = 20
score = 'A'

if age < 18:
    if score = 'A':
        print("恭喜，一等奖")
else:
    printf("你年龄不对");
```

```python
# 4-6.py

score = int(input("请输入一个分数："))

if 100 >= score >= 90:
    level = 'A'
elif 90 >= score >= 80:
    level ='B'
elif 80 >= score >= 70:
    level ='C'
elif 70 >= score >= 60:
    level ='D'
else:
    print("输入错误！")

print(level)
```

```text
# 4-7.py

score = int(input("请输入一个分数："))

level = 'A' if 100 >= score >= 90 else 'B' if 90 > score >= 80 else 'C' if 80 >= score >= 70 else 'D' if 70 > score >= 60 else print("输入错误！")

print(level)
```

```python
# 4-8.py

i = 1
sum = 0
while i <= 100:
    sum += i
    i += 1
print(sum)
```

```text
# 4-9.py(10)

sum = 0
for i in range(101):
    sum += i
print(sum)
```

```text
# 4-11.py


```

```text
# 4-12.py


```

```text
# 4-13.py


```

```text
# 6-2.py

def discount(price,rate):
    final_price = price * rate
    return  final_price

old_price = float(input("请输入原价"))
rate = float(input("请输入折扣"))
new_price = discount(old_price,rate)

print("打折后： %.2f" % new_price)
```

```text

```

```text

```

