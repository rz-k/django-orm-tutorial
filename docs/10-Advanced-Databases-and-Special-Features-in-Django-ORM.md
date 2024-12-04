# فصل ۱۰: **پایگاه داده‌های پیشرفته و ویژگی‌های خاص Django ORM**

در این فصل، به مفاهیمی فراتر از کارهای روزمره می‌پردازیم. این شامل استفاده از ویژگی‌های خاص پایگاه داده‌ها، مدل‌های سفارشی، عملیات سطح پایین، مدیریت پایگاه داده‌های چندگانه، و کار با داده‌های غیربومی در Django ORM می‌شود.

---

## **۱۰.۱. کار با پایگاه داده‌های چندگانه**

گاهی نیاز داریم از چندین پایگاه داده برای اهداف مختلف استفاده کنیم، مثل جداسازی پایگاه داده برای عملیات خواندن و نوشتن یا استفاده از انواع مختلف پایگاه داده (مثلاً PostgreSQL و MySQL) در یک پروژه.

---

### **۱۰.۱.۱. تعریف چند پایگاه داده در تنظیمات**

ابتدا پایگاه داده‌های موردنظر را در فایل `settings.py` تعریف می‌کنیم:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'default_db',
        'USER': 'default_user',
        'PASSWORD': 'default_password',
        'HOST': 'localhost',
        'PORT': '5432',
    },
    'analytics': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'analytics_db',
        'USER': 'analytics_user',
        'PASSWORD': 'analytics_password',
        'HOST': 'localhost',
        'PORT': '3306',
    },
}
```

- **`default`**: پایگاه داده اصلی (برای CRUD عملیات معمولی).
- **`analytics`**: پایگاه داده مخصوص تحلیل داده‌ها یا گزارش‌گیری.

---

### **۱۰.۱.۲. ذخیره و بازیابی داده‌ها از پایگاه داده‌های مختلف**

#### **مثال ۱: ذخیره داده در پایگاه داده خاص**
```python
from myapp.models import Product

# ذخیره یک محصول در پایگاه داده default
product = Product(name="Default Item", price=100)
product.save(using='default')

# ذخیره یک محصول دیگر در پایگاه داده analytics
analytics_product = Product(name="Analytics Item", price=200)
analytics_product.save(using='analytics')
```

#### **مثال ۲: بازیابی داده‌ها از پایگاه داده خاص**
```python
# دریافت محصولات از پایگاه داده default
default_products = Product.objects.using('default').all()
print("Products from Default DB:")
for product in default_products:
    print(product.name, product.price)

# دریافت محصولات از پایگاه داده analytics
analytics_products = Product.objects.using('analytics').all()
print("Products from Analytics DB:")
for product in analytics_products:
    print(product.name, product.price)
```

#### **مثال ۳: انتقال داده بین پایگاه داده‌ها**
```python
# انتقال داده از default به analytics
default_products = Product.objects.using('default').all()

for product in default_products:
    Product.objects.using('analytics').create(
        name=product.name,
        price=product.price
    )
print("Products transferred to analytics DB.")
```

---

### **۱۰.۱.۳. مدیریت تراکنش در پایگاه داده‌های مختلف**

#### **مثال ۱: تراکنش جداگانه در پایگاه داده خاص**
```python
from django.db import transaction

# تراکنش در پایگاه داده analytics
with transaction.atomic(using='analytics'):
    Product.objects.using('analytics').create(name="Analytics Transaction Item", price=300)
    print("Transaction in analytics DB completed.")
```

#### **مثال ۲: تراکنش موازی در چند پایگاه داده**
```python
from django.db import transaction

with transaction.atomic(using='default'):
    Product.objects.using('default').create(name="Default Transaction Item", price=150)
    print("Transaction in default DB completed.")

with transaction.atomic(using='analytics'):
    Product.objects.using('analytics').create(name="Analytics Transaction Item", price=350)
    print("Transaction in analytics DB completed.")
```

---

### **۱۰.۱.۴. تنظیم Router برای انتخاب پایگاه داده**

برای مدیریت بهتر، می‌توانید از یک **Database Router** استفاده کنید تا به صورت خودکار مشخص کنید هر مدل به کدام پایگاه داده متصل شود.

#### **تعریف یک Router سفارشی**
در فایل `db_router.py`:
```python
class MyDatabaseRouter:
    def db_for_read(self, model, **hints):
        """پایگاه داده برای عملیات خواندن"""
        if model._meta.app_label == 'analytics':
            return 'analytics'
        return 'default'

    def db_for_write(self, model, **hints):
        """پایگاه داده برای عملیات نوشتن"""
        if model._meta.app_label == 'analytics':
            return 'analytics'
        return 'default'

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """اجازه مهاجرت مدل‌ها به پایگاه داده مناسب"""
        if app_label == 'analytics':
            return db == 'analytics'
        return db == 'default'
```

#### **فعال کردن Router در تنظیمات**
```python
DATABASE_ROUTERS = ['path.to.db_router.MyDatabaseRouter']
```

---

### **۱۰.۲. کار با داده‌های JSON و انواع داده‌های خاص**

پایگاه داده‌هایی مثل PostgreSQL از انواع داده‌های خاص مثل JSON پشتیبانی می‌کنند. Django ORM به خوبی این قابلیت‌ها را مدیریت می‌کند.

#### **مدل با فیلد JSON**
```python
from django.db import models

class Profile(models.Model):
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE)
    metadata = models.JSONField()  # ذخیره داده‌های JSON
```

#### **ذخیره و بازیابی داده‌های JSON**
```python
# ذخیره داده JSON
profile = Profile.objects.create(
    user=user,
    metadata={'age': 25, 'hobbies': ['coding', 'gaming']}
)
print(profile.metadata)

# فیلتر کردن داده‌های JSON
profiles = Profile.objects.filter(metadata__age=25)
```

---

### **۱۰.۳. عملیات سطح پایین با Raw Queries**

Django اجازه می‌دهد کوئری‌های SQL خام را به کار ببریم.

#### **مثال: اجرای کوئری SELECT خام**
```python
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute("SELECT id, name FROM myapp_product WHERE price > %s", [100])
    for row in cursor.fetchall():
        print(row)
```

---

## **نتیجه‌گیری**

در این فصل:
- روش‌های مدیریت چند پایگاه داده بررسی شد.
- استفاده از داده‌های JSON و انواع داده‌های خاص نمایش داده شد.
- عملیات سطح پایین SQL برای مواقع ضروری ارائه گردید.

این ابزارها به شما کمک می‌کنند پروژه‌های پیچیده و چندبعدی را بهتر مدیریت کنید.
