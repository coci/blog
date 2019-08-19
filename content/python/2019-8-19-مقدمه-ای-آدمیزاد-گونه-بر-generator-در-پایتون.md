---
utid: 20190819075732
date: 2019-08-19 07:57:32
title: مقدمه ای بر generator در پایتون به زبان ادمیزاد
_index: generator-in-python
categories:
  -
tags:
  -
---

# مقدمه ای آدمیزاد گونه بر generator در پایتون

من وقتی اولین بار generator رو توی پایتون دیدم ، دیدم خیلی برام مبهم و به این راحتی ها نمی تونم بفهمم دقیقا چی هست و هیچ اموزش یا تعریف واضح و درست و حسابی براش پیدا نکردم ، البته شاید به اندازه کافی سرچ نکردم :))



# الف - generator چیست ؟

- در اصل همون فانکشن یا متد در پایتونِ
- دقیقا مثل iterator عمل میکنه
- دیتا رو به صدا زننده خودش ( caller ) با yield بر میگردونه به جای return



# ب - یه مثال خیلی ساده برای شروع  :

این فانکشن رو در‌نظر بگیرین :



```python
def generator1():
    yield 1
    yield 2
    yield 3
```



اگه این فانکشن رو مستقیما صدا بزنیم ابجکت generator به ما برگشت داده میشه  :

​        

```python
>>> generator1()
<generator object generator1 at 0x7fac361d8bf8>
```



اما این شکلی میشه ازش استفاده کرد :



```python
>>> next(gen1)
1
>>> next(gen1)
2
>>> next(gen1)
3
>>> next(gen1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>
```



اون stopIteration در اخر به ما برگشت داده  شد به این خاطر که فانکشن ما ۳ تا دیتا در داخلش داشت و دیگه چیزی برای برگشت به ما نبود ( دقیقا مثل iterator ) .



یا مورد استفاده دیگه generator :



```python
>>> gen1 = generator1()
>>> list(gen1)
[1, 2, 3]
>>> list(gen1)
[]
```



اگه توی کد بالا دقت کنید دفعه دوم که صدا زده شد ، ابجکت gen1 کاملا خالی شده و یه لیست خالی به ما برگشت داده شده .



و همچنین میشه ازین روش هم از generator استفاده کرد :



```python
gen1 = generator1()
# this will print out 1,2,3
for i in gen1:
    print(i)
```

# پ- کمی فراتر حرکت کنیم :

مثال های بالا زده شد تا شما با مفهوم yield اشنایی پیدا کنید .

ما میتونیم فراتر بریم و به parameter به generator اضافه کنیم :



```python
# returns a Fibonacci number < fib_max
def fibonacci1(fib_max: int) -> int:
    # initial values
    fib_n_2 = 0
    fib_n_1 = 1
    yield fib_n_2
    yield fib_n_1
    # now general case
    fib_n = fib_n_2 + fib_n_1
    while fib_n <= fib_max:
        yield fib_n
        fib_n_2 = fib_n_1
        fib_n_1 = fib_n
        fib_n = fib_n_2 + fib_n_1
# gives: [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987]
print(list(fibonacci1(1000)))
```



یادتون باشه این یه فانکشن برگشتی ( recursion ) نیست .



خیلی راحت میتونیم کد قبل رو کمی تغییر بدیم و تا بینهایت دیتا بتونیم درست کنیم :



```python
from itertools import islice

# returns an infinite sequence of Fibonacci numbers
def fibonacci2() -> int:
    # initial values
    fib_n_2 = 0
    fib_n_1 = 1
    yield fib_n_2
    yield fib_n_1
    # now general case
    fib_n = fib_n_2 + fib_n_1
    while True:
        yield fib_n
        fib_n_2 = fib_n_1
        fib_n_1 = fib_n
        fib_n = fib_n_2 + fib_n_1
        
# gives: [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987]
print(list(islice(fibonacci2(), 17)))
```



امیدوارم لذت برده باشین ، اگه انتقادی یا پیشنهادی داشتین حتما خوشحال میشم زیر همین پست در بخش کامنت ها عنوان کنید ،



در اخر منبع اصلی این پست :



https://dev.to/dandyvica/a-gentle-introduction-to-python-generators-4g10