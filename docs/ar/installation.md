# التثبيت

## المتطلّبات

تتطلّب Filament Shortcut Keys ما يلي لتعمل:

- PHP 8.2+
- Laravel v12.0+ أو v13.0+
- Filament v5.0+
- قاعدة بيانات تدعم نوع عمود JSON أصليًا: MySQL 5.7+ أو MariaDB 10.2+ أو PostgreSQL أو SQLite

لا يُدعَم Filament 4. تعمل مجموعة الاختبارات على SQLite وMySQL 8 وPostgreSQL 16.

## تثبيت الحزمة

ثبّت الحزمة باستخدام Composer:

```bash
composer require aristonis/filament-shortcut-keys
```

يكتشف Laravel مزوّد الخدمة (`service provider`) الخاص بالحزمة تلقائيًا.

<a id="running-the-migrations"></a>

## تشغيل الـ `migrations`

تخزّن الإضافة في قاعدة البيانات الإعداداتِ المسبقة المشتركة، وتجاوزات كل مشرف، والخريطة التي اختارها كل مشرف. انشر الـ `migrations` ثم شغّلها:

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-migrations"
php artisan migrate
```

تحتاج إلى هذه الجداول في [وضعَي التخصيص](customization.md#choosing-a-customization-mode) `personal` و`locked`. أمّا في [وضع `config`](customization.md#config-mode)، فلا يُخزَّن شيء ولا تستعلم الإضافة جداولها أبدًا، لذا يمكنك تخطّي هذه الخطوة.

## تسجيل الإضافة

سجّل الإضافة في مزوّد اللوحة (`panel provider`):

```php
use Aristonis\FilamentShortcutKeys\FilamentShortcutKeysPlugin;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->plugin(FilamentShortcutKeysPlugin::make());
}
```

تضيف الإضافة صفحتين إلى اللوحة، **Keyboard shortcuts** و**Manage shortcuts**، وتضيف خريطة المفاتيح وورقة الاختصارات إلى كل صفحة في اللوحة. وفي [وضع `config`](customization.md#config-mode)، لا تضيف إلا صفحة **Keyboard shortcuts**. إذا كان لتطبيقك عدّة لوحات، فسجّل الإضافة على كل لوحة ينبغي أن تحتوي على اختصارات.

تُشتقّ معرّفات روابط الصفحتين (`slug`) من أسماء أصنافهما، لا من عنوانَيهما. استخدمها لإنشاء روابط إلى الصفحتين أو لاختبارهما:

| الصفحة | المعرّف (`slug`) | اسم المسار (`route`) |
| --- | --- | --- |
| Keyboard shortcuts | `shortcut-reference` | `filament.{panel}.pages.shortcut-reference` |
| Manage shortcuts | `manage-shortcuts` | `filament.{panel}.pages.manage-shortcuts` |

`{panel}` هو معرّف اللوحة. وفي وضع `config`، لا يوجد المسار `manage-shortcuts`. فمثلًا، للوحة التي معرّفها `admin`:

```php
$this->assertTrue(Route::has('filament.admin.pages.shortcut-reference'));
```

<a id="publishing-the-assets"></a>

## نشر الملفات الثابتة (`assets`)

تتولّى المفاتيحَ سكربتٌ ينشره Filament مع ملفاته الثابتة الأخرى. بعد تسجيل الإضافة، انشره بالأمر:

```bash
php artisan filament:assets
```

!!! warning
    من دون السكربت المنشور، لا يفعل أي مفتاح شيئًا، ولا يظهر أي خطأ. فخريطة المفاتيح تُعرَض في الصفحة رغم ذلك، فيبدو كل شيء مثبّتًا.

شغّل هذا الأمر مرّة واحدة حتى لو كان تطبيقك يحتوي على خطّاف Composer الموصوف أدناه. فـFilament لا ينشر السكربت إلا للوحة سُجّلت عليها الإضافة. وعند تشغيل `composer require` لا تكون الإضافة على أي لوحة بعد، فلا يجد الخطّاف منها شيئًا لينشره.

التطبيق المُعَدّ باستخدام `php artisan filament:install` يشغّل `filament:upgrade` من خطّاف `post-autoload-dump` في Composer، وهو ينشر الملفات الثابتة لكل الإضافات المسجّلة بعد كل `composer install` و`composer update`. ويُبقي الخطّاف السكربت محدَّثًا عند تحديث الحزمة. تحقّق من أنّ ملف `composer.json` لديك يحتوي على هذا الخطّاف:

```json
"scripts": {
    "post-autoload-dump": [
        "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
        "@php artisan package:discover --ansi",
        "@php artisan filament:upgrade"
    ]
}
```

إن لم يكن يحتوي عليه، فشغّل `php artisan filament:assets` بنفسك بعد كل تحديث.

<a id="publishing-the-configuration"></a>

## نشر ملف الإعدادات

ملف الإعدادات اختياري. لتغيير قيمة افتراضية، انشره:

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-config"
```

يُنشئ هذا الأمر `config/shortcut-keys.php`. كل خيار موصوف في [الإعدادات](configuration.md).

## استخدام سمة مخصّصة

صفحات الإضافة وورقة الاختصارات الخاصة بها تحمل أنماطها الخاصة، ومبنية من مكوّنات Filament نفسها. إذا كانت لوحتك تستخدم سمة (`theme`) مخصّصة، فلا حاجة إلى إضافة الإضافة إليها.

## Laravel Boost

تأتي الحزمة مع إرشادات (`guidelines`) ومهارات (`skills`) لـ [Laravel Boost](https://laravel.com/docs/boost). عندما يسألك `php artisan boost:install` عن حزم الطرف الثالث التي تريد تضمينها، اختر هذه الحزمة. وإذا ثبّتّ Boost بطريقة غير تفاعلية، فأضف الحزمة إلى `boost.json` وشغّل `boost:install` مرة أخرى:

```json
"packages": ["aristonis/filament-shortcut-keys"]
```

## التجربة

افتح أي صفحة في اللوحة:

1. اضغط `G` وارفع إصبعك عنه. تظهر شارة صغيرة بحرف على كل رابط في الشريط الجانبي. اضغط أحد تلك الأحرف خلال ثانية ونصف لفتح تلك الصفحة.
2. اضغط `?`. تسرد ورقة الاختصارات كل اختصار نشط في الصفحة الحالية.
3. في صفحة قائمة، اضغط `J`. يظهر مؤشّر على الصف الأول. استخدم `J` و`K` لتحريكه، و`Enter` لتعديل الصف، و`Esc` لتحرير المؤشّر.

بعد ذلك، اقرأ [الاختصارات](shortcuts.md) لترى ما الاختصارات الموجودة وكيف تُختار أحرفها.
