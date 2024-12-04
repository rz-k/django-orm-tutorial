# فصل ۱۲: سیگنال‌ها در Django ORM

سیگنال‌ها (Signals) یکی از ویژگی‌های قدرتمند Django هستند که به شما امکان می‌دهند که به رویدادهای مختلف در سیستم پاسخ دهید، بدون اینکه مجبور شوید به‌طور مستقیم کدهای مختلف را در مکان‌های مختلف برنامه تغییر دهید. این ویژگی به‌ویژه برای ایجاد لاگ‌ها، اعتبارسنجی، یا انجام عملیات اضافی قبل یا بعد از ذخیره یک مدل کاربرد دارد.

## ۱۲.۱. آشنایی با سیگنال‌ها

سیگنال‌ها در Django معمولاً برای ارتباطات بین اجزای مختلف برنامه استفاده می‌شوند. به‌طور خاص، سیگنال‌ها به شما این امکان را می‌دهند که کد خود را از یک "سیگنال" فعال کنید که به‌طور خودکار در صورت وقوع رویدادهایی مانند ذخیره (save)، حذف (delete) یا تغییر یک مدل، اجرا می‌شود.

## ۱۲.۲. متداول‌ترین سیگنال‌ها در Django

### ۱۲.۲.۱. سیگنال‌های مربوط به ذخیره‌سازی

- **pre_save**: قبل از ذخیره‌سازی یک شیء مدل.
- **post_save**: بعد از ذخیره‌سازی یک شیء مدل.

### ۱۲.۲.۲. سیگنال‌های مربوط به حذف

- **pre_delete**: قبل از حذف یک شیء مدل.
- **post_delete**: بعد از حذف یک شیء مدل.

### ۱۲.۲.۳. سیگنال‌های مربوط به مایگریشن‌ها

- **m2m_changed**: وقتی یک رابطه ManyToMany تغییر می‌کند، این سیگنال فعال می‌شود.

## ۱۲.۳. ساختار سیگنال‌ها

برای استفاده از سیگنال‌ها باید از ماژول `django.db.models.signals` استفاده کنید. سیگنال‌ها معمولاً به یک تابع یا سیگنال گوش می‌دهند و در صورت وقوع رویداد، آن تابع اجرا می‌شود.

## ۱۲.۴. نحوه استفاده از سیگنال‌ها در Django

برای استفاده از سیگنال‌ها در Django، باید مراحل زیر را طی کنید:

1. **تعریف سیگنال‌ها**: ایجاد سیگنال‌ها و اتصال آن‌ها به مدل‌های خاص.
2. **اتصال سیگنال‌ها**: سیگنال‌ها باید به رویدادهای خاص متصل شوند.
3. **استفاده از گیرنده‌ها**: برای هر سیگنال، یک یا چند تابع گیرنده (receiver) ایجاد کنید که به سیگنال پاسخ دهند.

## ۱۲.۵. مثال‌ها

### ۱۲.۵.۱. سیگنال `pre_save`

این سیگنال قبل از ذخیره‌سازی مدل اجرا می‌شود. به‌عنوان مثال، فرض کنید بخواهیم هر بار که یک شیء از مدل `Product` ذخیره می‌شود، یک تاریخ‌چاپ خاص به آن اضافه کنیم.

```python
from django.db.models.signals import pre_save
from django.dispatch import receiver
from .models import Product

@receiver(pre_save, sender=Product)
def add_timestamp_to_product(sender, instance, **kwargs):
    instance.updated_at = timezone.now()
```

در این مثال، هر بار که یک شیء `Product` ذخیره می‌شود، تاریخ و زمان آن به‌طور خودکار در فیلد `updated_at` ثبت می‌شود.

### ۱۲.۵.۲. سیگنال `post_save`

این سیگنال بعد از ذخیره‌سازی یک شیء مدل اجرا می‌شود. مثلاً فرض کنید بعد از ذخیره یک شیء `Order` بخواهیم یک ایمیل تأیید برای کاربر ارسال کنیم.

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.core.mail import send_mail
from .models import Order

@receiver(post_save, sender=Order)
def send_order_confirmation(sender, instance, created, **kwargs):
    if created:
        send_mail(
            'Order Confirmation',
            f'Your order {instance.id} has been placed successfully.',
            'from@example.com',
            [instance.customer.email],
            fail_silently=False,
        )
```

در این مثال، هر بار که یک سفارش جدید از نوع `Order` ذخیره می‌شود، یک ایمیل تأیید به مشتری ارسال می‌شود.

### ۱۲.۵.۳. سیگنال `pre_delete`

این سیگنال قبل از حذف یک شیء مدل اجرا می‌شود. به‌عنوان مثال، فرض کنید قبل از حذف یک `Product` بخواهیم از آن یک بک‌آپ تهیه کنیم.

```python
from django.db.models.signals import pre_delete
from django.dispatch import receiver
from .models import Product

@receiver(pre_delete, sender=Product)
def backup_product_before_delete(sender, instance, **kwargs):
    # ذخیره بک‌آپ از محصول در سیستم فایل یا پایگاه داده
    with open(f'backup_{instance.id}.txt', 'w') as f:
        f.write(str(instance))
```

در اینجا، قبل از اینکه محصول حذف شود، اطلاعات آن در یک فایل بک‌آپ ذخیره می‌شود.

### ۱۲.۵.۴. سیگنال `post_delete`

این سیگنال بعد از حذف یک شیء مدل اجرا می‌شود. به‌عنوان مثال، می‌خواهیم پس از حذف یک `Customer`، رکوردهای مربوط به آن را در سیستم‌های دیگر نیز حذف کنیم.

```python
from django.db.models.signals import post_delete
from django.dispatch import receiver
from .models import Customer

@receiver(post_delete, sender=Customer)
def delete_related_data(sender, instance, **kwargs):
    # حذف داده‌های مرتبط با مشتری از سیستم‌های دیگر
    print(f'Deleting related data for customer {instance.id}')
```

در این مثال، پس از حذف یک مشتری، داده‌های مرتبط نیز پاک می‌شوند.

### ۱۲.۵.۵. سیگنال `m2m_changed`

این سیگنال زمانی فعال می‌شود که رابطه‌ی ManyToMany میان دو مدل تغییر کند. مثلاً فرض کنید می‌خواهید هر بار که یک محصول به یک سبد خرید اضافه می‌شود، یک لاگ از این تغییر ذخیره کنید.

```python
from django.db.models.signals import m2m_changed
from django.dispatch import receiver
from .models import Cart, Product

@receiver(m2m_changed, sender=Cart.products.through)
def log_product_added(sender, instance, action, **kwargs):
    if action == 'post_add':
        print(f'Product added to cart: {instance.id}')
```

در این مثال، هر بار که محصولی به سبد خرید اضافه می‌شود، این تغییر در لاگ ثبت می‌شود.

## ۱۲.۶. مدیریت سیگنال‌ها

### ۱۲.۶.۱. قطع سیگنال‌ها

گاهی اوقات ممکن است بخواهید سیگنال‌ها را به‌طور موقت غیرفعال کنید. این کار می‌تواند با استفاده از `disconnect` انجام شود.

```python
pre_save.disconnect(add_timestamp_to_product, sender=Product)
```

### ۱۲.۶.۲. ارسال سیگنال‌ها دستی

در برخی موارد ممکن است بخواهید سیگنال‌ها را به‌طور دستی ارسال کنید. این کار را می‌توان با استفاده از متد `send` انجام داد.

```python
from django.db.models.signals import post_save
from django.dispatch import Signal

my_signal = Signal()

# ارسال سیگنال
my_signal.send(sender=Product, instance=my_product_instance)
```

## ۱۲.۷. جمع‌بندی

سیگنال‌ها در Django ابزار قدرتمندی برای مدیریت رویدادها و ارتباطات میان اجزای مختلف برنامه هستند. با استفاده از سیگنال‌ها، می‌توانید به‌طور اتوماتیک به تغییرات در مدل‌ها و داده‌ها پاسخ دهید، و از آن‌ها برای انجام کارهای مختلف مثل ارسال ایمیل، لاگ‌گیری، و انجام اقدامات اضافی استفاده کنید.

در این فصل، سیگنال‌های پرکاربرد Django را به همراه مثال‌هایی کاربردی بررسی کردیم. این تکنیک‌ها به شما کمک خواهند کرد که برنامه‌های پیچیده‌تر و مقیاس‌پذیرتری بسازید.
