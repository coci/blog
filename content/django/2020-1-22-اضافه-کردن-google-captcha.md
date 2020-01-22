---
utid: 20200122103534
date: 2020-01-22 10:35:34
title: اضافه کردن google captcha 
_index: add-google-captcha-django
categories:
  -
tags:
  -
---

### اضافه کردن google captcha :

امروزتوی پست قصد دارم اموزش اضافه کردن google captcha رو با شما به اشتراک بزارم .

گوگل captcha یکی از معروف ترین راه حل ها برای جلوگیری از bot و spam هست و پیاده سازی خیلی راحتی داره .



#### ثبت سایت در گوگل :

 در اولین گام نیازمند اضافه کردن دامنه خودمون توی سایت google هستیم . برای این کار وارد لینک زیر بشین :

[captcha google](https://www.google.com/recaptcha/admin)

و پس از وارد شدن با حساب google خودتون باید صفحه مانند زیر رو ببینید :

![block](/django/images/captcha1.jpg)

دکمه + بالای صفحه رو بزنین و اطلاعات رو مانند عکس زیر پر کنید :

- به جای label که من نوشتم test شما یک اسم دلخواه وارد کنید .
- به جای قسمت domains شما باید ادرس دامنه خودتون رو وارد کنید .

![block](/django/images/captcha2.jpg)

بعد از اتمام مراحل بالا سایت گوگل یه کدی رو به شما میده که باید اون رو وارد جنگو کنیم . حالا وارد settings.py پروژه بشین و در پایین صفحه کد زیر رو وارد کنید :

( دقت کنید شما در اینجا به جای 6LdRSRYUAAAAAOnk5yom باید کد خودتون رو وارد کنید )

```
GOOGLE_RECAPTCHA_SECRET_KEY = '6LdRSRYUAAAAAOnk5yom'
```



 #### پیاده سازی :

حالا وقتش رسیده google captcha رو به template مورد نظر وارد کنیم . فرض کنید من در این صفحه یک فرم دارم که میخوام هربار کاربر بخواد این فرم رو پر کنه باید google captcha رو تایید کنه .

- در کد پایین در جلوی data-sitekey شما باید کد که از گوگل دریافت کردید و داخل settings.py وارد کردید رو در این قسمت هم وارد کنید

```
 #myapp/templates/index.html
 
 <form method="post">
    {% csrf_token %}
    {{ form.as_p }}

    <script src='https://www.google.com/recaptcha/api.js'></script>
    <div class="g-recaptcha" data-sitekey="6LdRSRYUAAAAAFCqQ1aZnYfRGJIlAUMX3qkUWlcF"></div>

    <button type="submit" class="btn btn-primary">Post</button>
  </form>
```

حالا در views.py مربوط به همین template کدهای زیر رو وارد کنید :

```
import urllib
import urllib2
import json

from django.shortcuts import render, redirect
from django.conf import settings
from django.contrib import messages

from .models import Comment
from .forms import CommentForm


def comments(request):
    comments_list = Comment.objects.order_by('-created_at')

    if request.method == 'POST':
        form = CommentForm(request.POST)
        if form.is_valid():

            ''' Begin reCAPTCHA validation '''
            recaptcha_response = request.POST.get('g-recaptcha-response')
            url = 'https://www.google.com/recaptcha/api/siteverify'
            values = {
                'secret': settings.GOOGLE_RECAPTCHA_SECRET_KEY,
                'response': recaptcha_response
            }
            data = urllib.urlencode(values)
            req = urllib2.Request(url, data)
            response = urllib2.urlopen(req)
            result = json.load(response)
            ''' End reCAPTCHA validation '''

            if result['success']:
                form.save()
                messages.success(request, 'New comment added with success!')
            else:
                messages.error(request, 'Invalid reCAPTCHA. Please try again.')

            return redirect('comments')
    else:
        form = CommentForm()

    return render(request, 'core/comments.html', {'comments': comments_list, 'form': form})
```

 کار تمام است . اما یه توضیحی بدم که روند کار به چه شکل هست . در function بالا در مرحله اول اطلاعات رو برای google میفرستیم اگه captcha توسط کاربر به درستی زده شده باشه یه True و اگه اشتباه باشه False به ما برگشت داده میشه . حالا ما با دستور :

```
if result['success']:
```

میام چک میکنیم که ایا True هست و در صورت True بودن response از سمت گوگل میایم دیتا رو وارد دیتابیس میکنیم و در غیر این صورت به کاربر پیامی مبنی در بر اینکه google captcha رو نزده یا اشتباه وارد کرده میدیم .

موفق باشید .