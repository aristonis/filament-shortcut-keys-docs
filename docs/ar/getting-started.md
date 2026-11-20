# البدء

## المتطلبات

- PHP 8.2 أو أحدث
- Laravel 12 أو 13
- Filament 5.x. لا يُدعم Filament 4.
- MySQL 5.7+ أو MariaDB 10.2+ أو PostgreSQL أو SQLite (يستخدم المخطط عمود `json` أصليًا). تعمل حزمة
  الاختبارات على SQLite وMySQL 8 وPostgreSQL 16؛ أمّا الإصدارات الأقدم وMariaDB فلم تُختبر.

## التثبيت

```bash
composer require aristonis/filament-shortcut-keys
```

يُكتشف الـ `service provider` تلقائيًا.

## نشر الـ `migrations` وتشغيلها

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-migrations"
php artisan migrate
```

يُنشئ هذا الجداول `shortcut_maps` و`shortcut_map_entries` و`shortcut_map_selections`. الـ `migrations`
لازمة في كل الأوضاع، بما فيها `locked`: فالخريطة المشتركة وتجاوزاتها واختيار كل مشرف تُحفظ كلها هناك.

## تسجيل الإضافة

```php
use Aristonis\FilamentShortcutKeys\FilamentShortcutKeysPlugin;

public function panel(Panel $panel): Panel
{
    return $panel
        ->plugin(FilamentShortcutKeysPlugin::make());
}
```

هذا هو الإعداد كله. تسجّل الإضافة صفحتين خاصتين بها ("Keyboard shortcuts" و"Manage
shortcuts")، وتحقن خريطة المفاتيح في كل صفحة من صفحات اللوحة، وتشحن السكربت الذي يستمع إلى المفاتيح.
لإسناد مفاتيح إلى إجراءات السجلات في جداولك، أضِف `->rowActions([...])`؛ انظر [إجراءات الصفوف](row-actions.md).

<a id="publish-the-assets"></a>

## نشر الأصول

```bash
php artisan filament:assets
```

تخطّي هذه الخطوة هو السبب الأكثر شيوعًا لعدم حدوث أي شيء عند ضغط مفتاح. فخريطة المفاتيح تُصيَّر في الصفحة
على أي حال، فلا يظهر خطأ تقرؤه؛ كل ما في الأمر أنّ السكربت الذي يستمع إلى المفاتيح غير موجود.

التطبيق المُعَدّ عبر `filament:install` يشغّل أصلًا `@php artisan filament:upgrade` من خطّاف
`post-autoload-dump` في Composer، والذي يعيد نشر أصول كل الإضافات بعد كل `composer install` أو
`composer update`. تحقّق من ملف `composer.json` لديك:

```json
"scripts": {
    "post-autoload-dump": [
        "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
        "@php artisan package:discover --ansi",
        "@php artisan filament:upgrade"
    ]
}
```

من دون هذا الخطّاف، شغّل `filament:assets` يدويًا بعد التثبيت وبعد كل تحديث
للحزمة.

## سمة مخصّصة

إذا كانت لوحتك تستخدم سمة (`theme`) مخصّصة في Filament، فأضِف قوالب العرض الخاصة بالإضافة إلى ملف CSS
الخاص بالسمة، كي تنجو أصنافها من عملية الحذف (`purge`) في Tailwind:

```css
@source '../../../../vendor/aristonis/filament-shortcut-keys/resources/**/*.blade.php';
```

## نشر ملف الإعدادات (اختياري)

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-config"
```

ينسخ هذا الملف `config/shortcut-keys.php` إلى تطبيقك. لا تحتاج إليه إلا لتغيير قيمة افتراضية؛ انظر
[الإعدادات](configuration.md).

## Laravel Boost

تشحن الحزمة إرشادات ومهارات (`skills`) لـ [Laravel Boost](https://github.com/laravel/boost). يسألك
الأمر التفاعلي `php artisan boost:install` عن الحزم الخارجية التي تريد تضمينها؛ اختر هذه الحزمة. أمّا
التثبيت غير التفاعلي (`-n`) فيتخطّى الحزم الخارجية، لذا أضِف الحزمة إلى `boost.json` وشغّل
`boost:install` مرة أخرى:

```json
"packages": ["aristonis/filament-shortcut-keys"]
```

## خطوات أولى للتجربة

افتح أي صفحة في اللوحة ثم:

1. اضغط `G` وارفع إصبعك. أثناء الانتظار تظهر شارة حرف صغيرة على كل رابط في الشريط الجانبي. اضغط أحد
   تلك الأحرف خلال ثانية ونصف، فتنتقل اللوحة إلى هناك.
2. اضغط `?` (`Shift` + `/`). تسرد ورقة الاختصارات كل اختصار فعّال في الصفحة الحالية.
3. في صفحة قائمة، اضغط `J`. يظهر مؤشر الصفوف على الصف الأول. `J` و`K` يحرّكانه، و`Enter`
   يعدّل الصف، و`Esc` يحرّر المؤشر.

أسماء المفاتيح في هذا التوثيق هي أغطية المفاتيح كما هي مطبوعة: `G` تعني مفتاح G مضغوطًا وحده، من دون
`Shift`. بعد ذلك، اقرأ [الاختصارات](shortcuts.md) للاطلاع على جدول المفاتيح كاملًا والقواعد التي تحكمه.
