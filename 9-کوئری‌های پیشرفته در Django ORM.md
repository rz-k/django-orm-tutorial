# فصل ۹: کوئری‌های پیشرفته در Django ORM

در این فصل به مباحث پیشرفته‌تر در کوئری‌های Django ORM پرداخته می‌شود. هدف این است که توانایی شما برای نوشتن کوئری‌های پیچیده و بهینه افزایش یابد.

---

## ۹.۱. فیلترهای پیچیده

### ۹.۱.۱. استفاده از `Q`
برای ایجاد کوئری‌های شرطی پیچیده و ترکیبی، از شیء `Q` استفاده می‌شود. با استفاده از `Q` می‌توان شرط‌های AND، OR، و NOT را در کوئری‌ها اعمال کرد.

#### مثال ۱: استفاده از OR
```python
from django.db.models import Q
from myapp.models import Product

# دریافت محصولاتی که نام آن‌ها شامل "Book" یا قیمت آن‌ها کمتر از 50 است
products = Product.objects.filter(Q(name__icontains='Book') | Q(price__lt=50))
for product in products:
    print(product.name, product.price)
```

#### مثال ۲: استفاده از OR با شروط ترکیبی
```python
# دریافت محصولاتی که نام آن‌ها شامل "Laptop" یا (قیمت کمتر از ۲۰ و موجودی بیشتر از ۱۰)
products = Product.objects.filter(Q(name__icontains='Laptop') | (Q(price__lt=20) & Q(stock__gt=10)))
for product in products:
    print(product.name, product.price, product.stock)
```

#### مثال ۳: استفاده از NOT
```python
from django.db.models import Q

# دریافت محصولاتی که نام آن‌ها شامل "Phone" نیست
products = Product.objects.filter(~Q(name__icontains='Phone'))
for product in products:
    print(product.name)
```

#### مثال ۴: ترکیب NOT با AND
```python
# دریافت محصولاتی که قیمت آن‌ها کمتر از ۵۰ است اما شامل "Book" نیستند
products = Product.objects.filter(Q(price__lt=50) & ~Q(name__icontains='Book'))
for product in products:
    print(product.name, product.price)
```

---

### ۹.۱.۲. استفاده از `F`
برای مقایسه مقادیر فیلدهای مختلف در یک مدل، از شیء `F` استفاده می‌شود.

#### مثال ۱: مقایسه دو فیلد
```python
from django.db.models import F

# دریافت محصولاتی که قیمت آن‌ها از قیمت تخفیف‌خورده بیشتر است
products = Product.objects.filter(price__gt=F('discounted_price'))
for product in products:
    print(product.name, product.price, product.discounted_price)
```

#### مثال ۲: محاسبه و به‌روزرسانی مقادیر فیلدها
```python
from django.db.models import F

# افزایش قیمت همه محصولات به اندازه ۱۰ واحد
Product.objects.update(price=F('price') + 10)

# کاهش موجودی محصولات به اندازه ۵ واحد
Product.objects.update(stock=F('stock') - 5)
```

#### مثال ۳: استفاده از F با فیلتر ترکیبی
```python
# دریافت محصولاتی که قیمت آن‌ها دو برابر موجودی است
products = Product.objects.filter(price__gte=F('stock') * 2)
for product in products:
    print(product.name, product.price, product.stock)
```

---

## ۹.۲. Subquery و Expressions

### ۹.۲.۱. استفاده از Subquery
برای انجام کوئری‌های پیچیده‌تر که نیاز به زیرکوئری دارند، از `Subquery` استفاده می‌شود.

#### مثال ۱: دریافت آخرین تاریخ سفارش برای هر مشتری
```python
from django.db.models import Subquery, OuterRef
from myapp.models import Order, Customer

# زیرکوئری برای یافتن آخرین سفارش هر مشتری
latest_order = Order.objects.filter(customer=OuterRef('pk')).order_by('-order_date').values('order_date')[:1]

# اضافه کردن فیلد زیرکوئری به کوئری اصلی
customers = Customer.objects.annotate(latest_order_date=Subquery(latest_order))
for customer in customers:
    print(customer.name, customer.latest_order_date)
```

#### مثال ۲: جمع مبلغ کل آخرین سفارش هر مشتری
```python
from django.db.models import Subquery, OuterRef, Sum

# زیرکوئری برای یافتن مجموع مبلغ سفارش
latest_order_total = Order.objects.filter(customer=OuterRef('pk')).order_by('-order_date').values('total_price')[:1]

# اضافه کردن مجموع قیمت آخرین سفارش
customers = Customer.objects.annotate(latest_order_total=Subquery(latest_order_total))
for customer in customers:
    print(customer.name, customer.latest_order_total)
```

---

### ۹.۲.۲. استفاده از Expressions مانند `Case` و `When`
این ابزارها برای ایجاد شرط‌های پویا در کوئری‌ها استفاده می‌شوند.

#### مثال ۱: افزودن وضعیت به محصولات بر اساس قیمت
```python
from django.db.models import Case, When, Value, CharField

# افزودن فیلدی به کوئری که نشان دهد آیا محصول گران است یا ارزان
products = Product.objects.annotate(
    price_status=Case(
        When(price__gte=100, then=Value('Expensive')),
        When(price__lt=100, then=Value('Cheap')),
        output_field=CharField(),
    )
)
for product in products:
    print(product.name, product.price, product.price_status)
```

#### مثال ۲: افزودن وضعیت بر اساس تعداد موجودی
```python
# افزودن فیلدی به کوئری که نشان دهد محصول در انبار موجود است یا تمام شده است
products = Product.objects.annotate(
    stock_status=Case(
        When(stock__gt=0, then=Value('In Stock')),
        default=Value('Out of Stock'),
        output_field=CharField(),
    )
)
for product in products:
    print(product.name, product.stock, product.stock_status)
```

---

## ۹.۳. Aggregate و Group By

### ۹.۳.۱. استفاده از Aggregation
#### مثال ۱: میانگین قیمت محصولات
```python
from django.db.models import Avg

# محاسبه میانگین قیمت
average_price = Product.objects.aggregate(Avg('price'))
print("Average Price:", average_price['price__avg'])
```

#### مثال ۲: مجموع موجودی تمام محصولات
```python
from django.db.models import Sum

# محاسبه مجموع موجودی
total_stock = Product.objects.aggregate(Sum('stock'))
print("Total Stock:", total_stock['stock__sum'])
```

### ۹.۳.۲. استفاده از Group By
#### مثال ۱: شمارش تعداد سفارش‌ها برای هر مشتری
```python
from django.db.models import Count
from myapp.models import Customer

# شمارش تعداد سفارش‌های هر مشتری
customers = Customer.objects.annotate(order_count=Count('order'))
for customer in customers:
    print(customer.name, customer.order_count)
```

#### مثال ۲: محاسبه مجموع مبلغ سفارش‌ها برای هر مشتری
```python
from django.db.models import Sum

# محاسبه مجموع مبلغ سفارش‌ها
customers = Customer.objects.annotate(total_spent=Sum('order__total_price'))
for customer in customers:
    print(customer.name, customer.total_spent)
```

---

## ۹.۴. Raw SQL Queries

### ۹.۴.۱. اجرای کوئری‌های خام
#### مثال ۱: اجرای یک کوئری SELECT
```python
from django.db import connection

# اجرای کوئری خام
with connection.cursor() as cursor:
    cursor.execute("SELECT name, price FROM myapp_product WHERE price > %s", [100])
    for row in cursor.fetchall():
        print(row)
```

#### مثال ۲: اجرای کوئری INSERT
```python
from django.db import connection

# افزودن یک محصول جدید
with connection.cursor() as cursor:
    cursor.execute("INSERT INTO myapp_product (name, price) VALUES (%s, %s)", ['New Product', 150])
print("Product added successfully!")
```

---

## نتیجه‌گیری
در این فصل، به بررسی کوئری‌های پیشرفته Django ORM پرداختیم. ابزارهای معرفی‌شده در این فصل به شما اجازه می‌دهند:
1. از شرط‌های پیچیده استفاده کنید.
2. کوئری‌های انعطاف‌پذیر و پیشرفته بنویسید.
3. با استفاده از Subquery و Expressions داده‌های پیشرفته‌تر را مدیریت کنید.
