# فصل ۷: تراکنش‌ها (Transactions) در Django ORM

---

## ۷.۱. معرفی تراکنش‌ها

تراکنش‌ها به ما این امکان را می‌دهند که مجموعه‌ای از عملیات پایگاه داده را به صورت یکجا و اتمیک انجام دهیم. یعنی یا تمامی عملیات موفقیت‌آمیز اجرا می‌شوند، یا در صورت بروز خطا، هیچ‌یک از تغییرات اعمال نخواهد شد. تراکنش‌ها برای مدیریت تغییرات داده‌ها در برنامه‌های پیچیده، جلوگیری از تداخل و تضمین یکپارچگی داده‌ها ضروری هستند.

در Django، تراکنش‌ها از طریق روش‌ها و ابزارهایی مانند `atomic()`, `commit()`, `rollback()`, `savepoint()` و `select_for_update()` مدیریت می‌شوند.

---

## ۷.۲. متدهای مدیریت تراکنش

### ۷.۲.۱. استفاده از `atomic()`

متد `atomic()` برای ایجاد یک بلاک اتمیک استفاده می‌شود که تمامی عملیات داخل آن یا به طور کامل انجام می‌شوند یا در صورت بروز خطا، لغو خواهند شد.

#### مثال:
```
from django.db import transaction

def create_order():
    try:
        with transaction.atomic():
            customer = Customer.objects.create(name="John Doe")
            order = Order.objects.create(customer=customer, total_amount=100)
    except Exception as e:
        print("Error:", e)
```

در این مثال، اگر خطایی در هر یک از عملیات رخ دهد، تمام تغییرات انجام‌شده در بلاک `atomic()` به حالت اولیه بازمی‌گردند.

---

### ۷.۲.۲. استفاده از `select_for_update()`

متد `select_for_update()` رکوردها را برای جلوگیری از تغییر توسط تراکنش‌های دیگر قفل می‌کند. این متد معمولاً در برنامه‌هایی که نیاز به مدیریت همزمانی دارند استفاده می‌شود.

#### مثال:
```
from django.db import transaction

def update_product_stock(product_id, quantity):
    try:
        with transaction.atomic():
            product = Product.objects.select_for_update().get(id=product_id)
            product.stock -= quantity
            product.save()
    except Product.DoesNotExist:
        print("Product not found!")
```

---

### ۷.۲.۳. استفاده از نقاط ذخیره (Savepoints)

با استفاده از نقاط ذخیره، می‌توانید بخش‌هایی از تراکنش را به صورت انتخابی لغو کنید، بدون اینکه کل تراکنش لغو شود.

#### مثال:
```
from django.db import transaction

def update_order():
    try:
        with transaction.atomic():
            order = Order.objects.get(id=1)
            order.status = 'Processing'
            order.save()

            savepoint = transaction.savepoint()
            try:
                product = Product.objects.get(id=1)
                product.stock -= 5
                product.save()
            except Exception:
                transaction.savepoint_rollback(savepoint)
                print("Error updating product stock!")
    except Exception as e:
        print("Error in order update:", e)
```

---

### ۷.۲.۴. مدیریت تراکنش‌های دستی با `commit()` و `rollback()`

#### استفاده از `commit()`

با استفاده از متد `commit()`، می‌توانید تغییرات انجام‌شده را تایید و ذخیره کنید.

#### مثال:
```
from django.db import transaction

def commit_transaction():
    with transaction.atomic():
        product = Product.objects.create(name="Smartphone", price=800)
        transaction.commit()
```

#### استفاده از `rollback()`

متد `rollback()` برای لغو تمامی تغییرات در صورت بروز خطا استفاده می‌شود.

#### مثال:
```
from django.db import transaction

def rollback_transaction():
    try:
        with transaction.atomic():
            product = Product.objects.create(name="Tablet", price=600)
            order = Order.objects.create(product=product, quantity=2)
            transaction.rollback()
    except Exception as e:
        print("Error occurred, rolling back:", e)
```

---

### ۷.۲.۵. استفاده از `set_autocommit()`

به طور پیش‌فرض، Django تراکنش‌ها را به صورت خودکار تایید می‌کند. با استفاده از `set_autocommit(False)` می‌توانید تایید خودکار را غیرفعال کرده و تراکنش‌ها را به صورت دستی مدیریت کنید.

#### مثال:
```
from django.db import transaction

def disable_autocommit():
    transaction.set_autocommit(False)
    try:
        product = Product.objects.create(name="Headphones", price=150)
        order = Order.objects.create(product=product, quantity=1)
        transaction.commit()
    except Exception as e:
        transaction.rollback()
        print("Error occurred:", e)
    finally:
        transaction.set_autocommit(True)
```

---

### ۷.۲.۶. تراکنش‌های تو در تو (Nested Transactions)

تراکنش‌های تو در تو به شما این امکان را می‌دهند که تراکنش‌های کوچک‌تری را در داخل یک تراکنش بزرگتر مدیریت کنید.

#### مثال:
```
from django.db import transaction

def nested_transaction():
    try:
        with transaction.atomic():
            product = Product.objects.create(name="Camera", price=500)
            with transaction.atomic():  # تراکنش داخلی
                order = Order.objects.create(product=product, quantity=3)
    except Exception as e:
        print("Error occurred:", e)
```

---

### ۷.۲.۷. استفاده از Savepoints در تراکنش‌های تو در تو

Django به طور خودکار savepoint‌ها را در تراکنش‌های داخلی مدیریت می‌کند. اگر خطایی در یک بلاک داخلی رخ دهد، تراکنش به حالت قبلی خود بازمی‌گردد.

#### مثال:
```
from django.db import transaction

def nested_with_savepoint():
    try:
        with transaction.atomic():
            product = Product.objects.create(name="Laptop", price=1000)
            savepoint = transaction.savepoint()
            try:
                order = Order.objects.create(product=product, quantity=1)
            except Exception:
                transaction.savepoint_rollback(savepoint)
                print("Inner transaction failed!")
    except Exception as e:
        print("Outer transaction failed:", e)
```

---

## ۷.۳. جمع‌بندی

در این فصل، مفاهیم زیر را بررسی کردیم:

1. **`atomic()`**: بلاک اتمیک برای انجام تراکنش‌های یکپارچه.
2. **`select_for_update()`**: قفل کردن رکوردها برای مدیریت همزمانی.
3. **`savepoint()`**: ایجاد نقاط ذخیره و بازگشت به آن‌ها.
4. **`commit()` و `rollback()`**: تایید یا لغو تراکنش‌ها.
5. **`set_autocommit()`**: غیرفعال کردن تایید خودکار تراکنش‌ها.
6. **Nested Transactions**: مدیریت تراکنش‌های تو در تو.

با استفاده از این ابزارها می‌توانید تراکنش‌های پیچیده و حساس را به راحتی مدیریت کنید و از بروز خطاها و مشکلات همزمانی جلوگیری کنید.
