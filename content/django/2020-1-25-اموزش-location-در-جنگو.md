---
utid: 20200125191208
date: 2020-01-25 19:12:08
title: اموزش location در جنگو
_index: location-in-django
categories:
  -
tags:
  -
---

یکی از چالش هایی که چند وقت اخیر کلی ذهنم رو درگیر کرد این هست که چطور میتونم location رو به جنگو اضافه کنم و بعد از کلی اینور و اونور کردن تونستم این کار رو انجام بدم و توی این پست قصد دارم اموزش ساخت یه پروژه جنگو مبتنی بر location با شما عزیزان به اشتراک بزارم .



### پیش نیاز :

**( با دقت این قسمت رو بخونین)**

یه توضیحی که در اینجا باید بدم و اون این هست که برای اینکه شما بتونین location رو ذخیره کنین باید به دیتابیس خودتون قابلیت geo رو اضافه کنین . به طور مرسوم من برای این پروژه دیتابیس Postgres رو در نظر گرفتم ٬ پس باید روی Postgres قابلیت geo رو اضافه کنم . برای این کار یک extension برای Postgres هست به اسم PostGis که شما باید این رو روی دیتابیس Postgres خودتون نصب کنید . حالا نکته قضیه اینجاست که این کار خیلی دردسر داشت و در اخرم به هیچ نتیجه ای نتونستم برسم . پس به جای کشیدن این همه دردسر از یه معجزه به نام داکر استفاده کردم ( اگه در مورد داکر اطلاعی ندارین همین جا توصیه میکنم برین و نحوه استفاده از داکر رو یادبگیرین ) . قضیه اینجاست که من از یه image استفاده کردم که روش هم Postgres نصب هست و PostGis هم روش اضافه شده .



### نصب Postgres و PostGis با استفاده از داکر :

خوب من در نظر میگیرم شما قسمت قبل رو خوندین و میدونین چرا قرار از داکر استفاده کنیم و این که داکر الان روی سیستم شما نصب هست . حالا برای نصب  و اجرا image که در بالا در موردش صحبت کردم دستور زیر رو توی ترمینال خودتون وارد کنید :

**(دقت کنید باید فیلتر شکن شما برای نصب image فعال باشه )**

در دستور زیر :

- در جلوی POSTGRES_USER که اسم یوزر دیتابیس هست یک username دلخواه قرار بدین .
- در جلوی POSTGRES_PASSWORD هم یک پسورد انتخابی قرار بدین .
- در جلوی POSTGRES_DB که اسم دیتابیس هست یه اسم دلخواه قرار بدین .

```
docker run --name postgis \
			-e POSTGRES_USER=admin -e POSTGRES_PASSWORD=mysecretpassword \
			-e POSTGRES_DB=location -p 5455:5432 -d mdillon/postgis
```

 حال کمی صبر کنین تا مراحل pull کردن image و ساخت container به اتمام برسه . 

پس اتمام ساخت container شما یک دیتابیس postgres که روش Postgis نصب شده به ادرس 127.0.0.1:5455 دارین . زیبا نیست ؟ :)



### تنظیمات جنگو  :

خوب یه توضیح کلی در مورد پروژه بدم که قرار ما اطلاعات خانه ها رو ذخیره کنیم و بعد مثلا از یه نقطه خاص خانه هایی که ۵ کیلومتری قرار دارن رو نمایش بدیم .

ابتدا 'django.contrib.gis' وارد تنظیمات پروژه میکنیم :

```
INSTALLED_APPS = [
	......
	'django.contrib.gis',
	.....
	]
	
```

حال در همون فایل settings.py قسمت DATABASES رو تغییر میدیم :

به چند نکته دقت کنید :

- در جلوی NAME باید مقداری که در قسمت اول در مقابل POSTGRES_DB برای نصب  داکر Postgres زدین رو وارد کنید .
- در جلوی USER باید مقداری که در قسمت اول در مقابل POSTGRES_USER برای نصب  داکر Postgres زدین رو وارد کنید .
- در جلوی PASSWORD باید مقداری که در قسمت اول در مقابل POSTGRES_PASSWORD برای نصب  داکر Postgres زدین رو وارد کنید .

```
DATABASES = {
    'default': {
        'ENGINE': 'django.contrib.gis.db.backends.postgis',
        'NAME': 'location',
        'USER': 'admin',
        'PASSWORD': 'mysecretpassword',
        'HOST': '127.0.0.1',
        'PORT': '5455'
    }
}
```



### ساخت مدل :

خوب حالا در نظر بگیرین که ما یک مدل داریم به اسم House که یه فیلد position داره و قرار داخلش مختصات جغرافیایی ذخیره بشه .

برای مختصات جغرافیایی ما باید از PointField استفاده کنیم :

```
from django.db import models
from django.contrib.gis.db import models as gm

class House(models.Model):
		...
    position = gm.PointField(null=True, blank=True)
    ...
```



### ساخت یک object :

حالا که مدل رو ساختیم وقت اون رسیده که مختصات داده شده به ما که به صورت lat و long هست رو به عنوان مختصات خونه ذخیره کنیم :

```
from django.contrib.gis.geos import GEOSGeometry
from decimal import Decimal

latitude = Decimal(59.32932)
longitude = Decimal(18.06858)

point = GEOSGeometry("POINT({0} {1})".format(longitde, latitude))

house = House()
house.position = point
house.save()
```

به شکلی که در بالا اشاره شد ما تونستیم مختصات رو ذخیره کنیم . حال در ادامه همراه باشین تا بتونیم بروی مدل query بر اساس فاصله بزنیم .



### نشان دادن خانه های بر اساس فاصله :

خوب حالا در این مرحله میخوایم از یه نقطه خاص بگیم که تمام خانه های ۵ کیلومتری رو نشون بده :

```
from django.contrib.gis.measure import Distance
from decimal import Decimal


latitude = Decimal(LATITUDE_FROM_REQUEST)
longitude = Decimal(LONGITUDE_FROM_REQUEST)


point_of_user = GEOSGeometry("POINT({} {})".format(longitude, latitude))
max_distance = 5


houses_in_range = House.objects.filter(
            position__distance_lte=(
                point_of_user,
                Distance(km=max_distance)
            )
        )
```



قرار نبود که این پست اموزش مو به مو مراحل باشه ٬ در عوض من سعی کردم به صورت خلاصه بگم چطور میشه پروژه location base با جنگو داشته باشیم .

 امیدوارم مفید واقع شده باشه . 

