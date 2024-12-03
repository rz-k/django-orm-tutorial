# فصل ۱۳: مدیریت مشکلات سازگاری داده‌ها (Data Integrity)

در این فصل، به مدیریت مشکلات سازگاری داده‌ها در Django ORM پرداخته می‌شود. یکی از اصلی‌ترین جنبه‌های نگهداری پایگاه داده‌های صحیح، استفاده از **محدودیت‌ها (Constraints)** است که به جلوگیری از وارد شدن داده‌های اشتباه به پایگاه داده کمک می‌کند. Django امکانات زیادی برای اضافه کردن این محدودیت‌ها به مدل‌ها فراهم کرده است. در اینجا به بررسی انواع مختلف محدودیت‌ها می‌پردازیم و مثال‌هایی برای استفاده از آن‌ها ارائه می‌دهیم.

---

## ۱۳.۱. محدودیت‌های منحصر به فرد (Unique Constraints)

محدودیت **unique** یکی از مهم‌ترین محدودیت‌هایی است که از تکرار داده‌ها جلوگیری می‌کند. با استفاده از این محدودیت می‌توانیم اطمینان حاصل کنیم که یک فیلد خاص یا ترکیبی از چند فیلد در جدول منحصر به فرد باشد.

### استفاده از UniqueConstraint برای یک فیلد

زمانی که می‌خواهیم فیلدی خاص در جدول منحصر به فرد باشد، می‌توانیم از گزینه `unique=True` استفاده کنیم:

```python
from django.db import models

class User(models.Model):
    username = models.CharField(max_length=100, unique=True)  # این فیلد نباید تکراری باشد
    email = models.EmailField(unique=True)  # این فیلد نباید تکراری باشد
```

در مثال بالا، هر دو فیلد `username` و `email` باید در پایگاه داده منحصر به فرد باشند.

### استفاده از UniqueConstraint برای ترکیب چند فیلد

اگر بخواهیم ترکیب چند فیلد منحصر به فرد باشد، می‌توانیم از **UniqueConstraint** استفاده کنیم:

```python
from django.db import models

class Subscription(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    subscription_type = models.CharField(max_length=100)

    class Meta:
        constraints = [
            models.UniqueConstraint(fields=['user', 'subscription_type'], name='unique_subscription')
        ]
```

در اینجا، ترکیب فیلدهای `user` و `subscription_type` باید منحصر به فرد باشد. این بدان معنی است که یک کاربر نمی‌تواند دو اشتراک از نوع مشابه داشته باشد.

---

## ۱۳.۲. محدودیت‌های بررسی (Check Constraints)

محدودیت **CheckConstraint** به شما این امکان را می‌دهد که یک شرط خاص را برای مقدار یک فیلد در پایگاه داده تعریف کنید. با استفاده از این محدودیت، می‌توانید داده‌ها را قبل از وارد شدن به پایگاه داده اعتبارسنجی کنید.

### استفاده از CheckConstraint

برای مثال، فرض کنید می‌خواهیم اطمینان حاصل کنیم که فقط مقادیر بزرگتر از صفر برای فیلد `price` وارد پایگاه داده شوند:

```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)

    class Meta:
        constraints = [
            models.CheckConstraint(check=models.Q(price__gt=0), name='price_gt_zero')
        ]
```

در این مثال، هرگاه فیلدی برای `price` که مقدار آن صفر یا کمتر از صفر باشد وارد شود، یک خطای پایگاه داده ایجاد خواهد شد.

---

## ۱۳.۳. محدودیت‌های ForeignKey و OneToOneField

Django به‌طور خودکار محدودیت‌های سازگاری داده‌ها را برای فیلدهای **ForeignKey** و **OneToOneField** اعمال می‌کند. به‌عنوان مثال، اگر یک فیلد **ForeignKey** را ایجاد کنید، Django اطمینان حاصل می‌کند که داده‌های وارد شده با داده‌های موجود در جدول مرتبط سازگار باشند.

### مثال:

```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)  # محدودیت ForeignKey
```

در اینجا، هر رکورد در جدول `Book` باید به یک رکورد موجود در جدول `Author` اشاره کند. اگر رکوردی در جدول `Author` حذف شود، تمامی رکوردهای مرتبط در جدول `Book` نیز حذف خواهند شد (این به دلیل پارامتر `on_delete=models.CASCADE` است).

---

## ۱۳.۴. محدودیت‌های عدم پذیرش مقادیر خالی (Null and Blank Constraints)

در بسیاری از مواقع، می‌خواهیم مشخص کنیم که یک فیلد نمی‌تواند `null` باشد یا مقادیر خالی را قبول کند.

### استفاده از null=True و blank=True

- **null=True**: به معنای این است که فیلد می‌تواند `NULL` در پایگاه داده باشد.
- **blank=True**: به معنای این است که فیلد می‌تواند در فرم‌ها خالی باشد.

```python
class Order(models.Model):
    order_number = models.CharField(max_length=100, null=False)  # نمی‌تواند null باشد
    shipping_address = models.CharField(max_length=255, blank=False)  # نمی‌تواند خالی باشد
```

در اینجا، فیلد `order_number` نمی‌تواند `NULL` باشد و فیلد `shipping_address` نمی‌تواند خالی باشد.

---

## ۱۳.۵. محدودیت‌های سفارشی (Custom Constraints)

گاهی اوقات نیاز داریم که محدودیت‌های خاصی اعمال کنیم که به‌طور پیش‌فرض توسط Django قابل دسترسی نیستند. در این صورت می‌توانیم از **Constraints سفارشی** استفاده کنیم. این محدودیت‌ها می‌توانند شامل موارد پیچیده‌تر مانند اعتبارسنجی داده‌ها یا حتی استفاده از توابع SQL سفارشی باشند.

### مثال:

فرض کنید که می‌خواهیم از یک محدودیت سفارشی استفاده کنیم تا اطمینان حاصل کنیم که قیمت یک محصول نباید از مقدار معین (مثلاً 1000) بیشتر باشد:

```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)

    class Meta:
        constraints = [
            models.CheckConstraint(
                check=models.Q(price__lte=1000),
                name='check_price_maximum'
            )
        ]
```

در اینجا، هر زمان که یک محصول با قیمت بیشتر از 1000 وارد پایگاه داده شود، خطای `IntegrityError` ایجاد می‌شود.

---

## ۱۳.۶. مدیریت مشکلات سازگاری داده‌ها در پروژه‌های بزرگ

در پروژه‌های بزرگ، معمولاً مدل‌ها و محدودیت‌ها پیچیده‌تر می‌شوند و نیاز به مدیریت دقیق‌تر دارند. به‌خصوص در تیم‌های بزرگ، **تست سازگاری** مهاجرت‌ها و **مهاجرت‌های پیچیده** اهمیت پیدا می‌کند. 

### بهترین شیوه‌ها برای مدیریت پروژه‌های بزرگ:
1. **تقسیم‌بندی مهاجرت‌ها**: در پروژه‌های بزرگ، مهاجرت‌ها باید به بخش‌های کوچکتر تقسیم شوند تا هم مدیریت بهتری داشته باشیم و هم از بروز مشکلات در پایگاه داده جلوگیری شود.
2. **استفاده از تست‌ها**: استفاده از تست‌های مهاجرت برای اطمینان از سازگاری داده‌ها پس از اعمال مهاجرت‌ها.
3. **استفاده از نسخه‌بندی**: نسخه‌بندی دقیق مهاجرت‌ها و پایگاه‌های داده برای حفظ سازگاری با داده‌ها.

---

## جمع‌بندی

در این فصل، نحوه مدیریت سازگاری داده‌ها در Django ORM و انواع محدودیت‌هایی که می‌توان در پایگاه داده اعمال کرد، مورد بررسی قرار گرفت. استفاده از محدودیت‌ها مانند **UniqueConstraint**، **CheckConstraint**، **ForeignKey Constraints**، و محدودیت‌های سفارشی می‌تواند به بهبود یکپارچگی داده‌ها کمک کند و از مشکلات معمول جلوگیری کند. این مباحث در پروژه‌های بزرگ اهمیت بسیاری دارند، زیرا به حفظ صحت داده‌ها و کاهش خطاها کمک می‌کنند.
