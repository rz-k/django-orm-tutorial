# فصل ۸: بهینه‌سازی کوئری‌ها و مدیریت کارایی در Django ORM

در این فصل روش‌های مختلف بهینه‌سازی کوئری‌ها، مدیریت عملکرد، و کار با داده‌های سنگین در Django ORM بررسی می‌شود. هدف اصلی این است که از منابع پایگاه داده به بهترین نحو استفاده کنیم و عملکرد برنامه را بهبود دهیم.

---

## ۸.۱. بهینه‌سازی کوئری‌ها

### ۸.۱.۱. استفاده از `select_related()`
برای کاهش تعداد کوئری‌های مربوط به روابط **ForeignKey** یا **OneToOne**، از این متد استفاده می‌شود. این متد با انجام یک کوئری واحد، داده‌های مرتبط را بازیابی می‌کند.

#### مثال:
```python
from myapp.models import Order

# بدون select_related (تعداد زیادی کوئری جداگانه اجرا می‌شود)
orders = Order.objects.all()
for order in orders:
    print(order.customer.name)  # هر بار کوئری جدید به پایگاه داده زده می‌شود

# با select_related (یک کوئری واحد اجرا می‌شود)
orders = Order.objects.select_related('customer').all()
for order in orders:
    print(order.customer.name)  # داده‌ها با یک کوئری بهینه بازیابی می‌شوند
```

---

### ۸.۱.۲. استفاده از `prefetch_related()`
این متد برای بهینه‌سازی روابط ManyToMany یا Reverse ForeignKey استفاده می‌شود. برخلاف `select_related` که از JOIN استفاده می‌کند، `prefetch_related` داده‌های مورد نیاز را در چندین کوئری جداگانه پیش‌بارگذاری می‌کند و سپس آن‌ها را به هم مرتبط می‌کند.

#### مثال:
```python
from myapp.models import Author

# بدون prefetch_related (تعداد زیادی کوئری اضافی)
authors = Author.objects.all()
for author in authors:
    print([book.title for book in author.books.all()])  # یک کوئری جدید برای هر نویسنده اجرا می‌شود

# با prefetch_related (بهینه‌سازی کوئری‌ها)
authors = Author.objects.prefetch_related('books').all()
for author in authors:
    print([book.title for book in author.books.all()])  # داده‌ها از قبل بارگذاری شده‌اند
```

---

### ۸.۱.۳. استفاده از `only()` و `defer()`
این متدها به شما امکان می‌دهند فیلدهای خاصی را از پایگاه داده انتخاب کنید یا برخی از فیلدها را به تأخیر بیندازید.

- **`only()`**: فقط فیلدهای مشخص‌شده را بازیابی می‌کند.
- **`defer()`**: تمام فیلدها به‌جز فیلدهای مشخص‌شده بازیابی می‌شوند.

#### مثال:
```python
from myapp.models import Product

# فقط نام و قیمت را بازیابی کنید
products = Product.objects.only('name', 'price')
for product in products:
    print(product.name, product.price)  # فقط فیلدهای name و price بارگذاری می‌شوند

# همه فیلدها به‌جز description بازیابی می‌شوند
products = Product.objects.defer('description')
for product in products:
    print(product.name, product.price)  # فیلد description هنگام دسترسی بارگذاری می‌شود
```

---

### ۸.۱.۴. استفاده از `annotate()`
برای افزودن فیلدهای محاسباتی به کوئری‌ها، از این متد استفاده می‌شود. این فیلدها بر اساس داده‌های پایگاه داده محاسبه می‌شوند.

#### مثال:
```python
from django.db.models import Count
from myapp.models import Author

# شمارش تعداد کتاب‌های هر نویسنده
authors = Author.objects.annotate(book_count=Count('books'))
for author in authors:
    print(author.name, author.book_count)  # تعداد کتاب‌های هر نویسنده
```

---

### ۸.۱.۵. استفاده از `values()` و `values_list()`
این متدها فقط داده‌های خاصی را بازیابی می‌کنند و خروجی به صورت دیکشنری یا لیست خواهد بود.

#### مثال:
```python
from myapp.models import Product

# استفاده از values()
products = Product.objects.values('name', 'price')
for product in products:
    print(product['name'], product['price'])  # خروجی به صورت دیکشنری

# استفاده از values_list()
products = Product.objects.values_list('name', 'price')
for product in products:
    print(product[0], product[1])  # خروجی به صورت لیست
```

---

## ۸.۲. مدیریت داده‌های سنگین

### ۸.۲.۱. استفاده از `iterator()`
برای کاهش مصرف حافظه هنگام پردازش داده‌های بزرگ، از این متد برای تکرار تدریجی داده‌ها استفاده می‌شود.

#### مثال:
```python
from myapp.models import LargeData

# پردازش تدریجی داده‌ها
for data in LargeData.objects.all().iterator():
    print(data.name)  # داده‌ها به صورت تدریجی از پایگاه داده بازیابی می‌شوند
```

---

### ۸.۲.۲. استفاده از `limit()` و `offset()`
برای بازیابی تعداد محدودی از نتایج یا پرش از تعدادی از رکوردها، از slicing استفاده کنید.

#### مثال:
```python
from myapp.models import Product

# دریافت ۱۰ رکورد اول
products = Product.objects.all()[:10]
for product in products:
    print(product.name)

# پرش از ۱۰ رکورد اول و دریافت ۱۰ رکورد بعدی
products = Product.objects.all()[10:20]
for product in products:
    print(product.name)
```

---

### ۸.۲.۳. بررسی کوئری‌ها با `query`
برای دیباگ کردن کوئری‌های تولیدشده توسط Django ORM، از این ویژگی استفاده کنید.

#### مثال:
```python
from myapp.models import Product

# نمایش کوئری تولیدشده
query = Product.objects.filter(price__gt=100).query
print(query)  # نمایش کوئری SQL
```

---

### نکات مهم:
- از `select_related` و `prefetch_related` برای کاهش کوئری‌های اضافی استفاده کنید.
- با `only` و `defer` داده‌های غیرضروری را از کوئری‌ها حذف کنید.
- برای داده‌های بزرگ از `iterator` و محدودیت‌های `limit` و `offset` استفاده کنید.
- برای بررسی کارایی، از ابزارهایی مثل **django-debug-toolbar** استفاده کنید.
- در پروژه‌های بزرگ، از ابزارهای پروفایلینگ و مانیتورینگ کوئری‌ها بهره ببرید.

---

این فصل به شما کمک می‌کند تا کوئری‌های بهینه‌تر و سریع‌تر در Django بنویسید و از منابع پایگاه داده بهینه استفاده کنید. اگر سوال یا ابهامی دارید، حتما بپرسید!
