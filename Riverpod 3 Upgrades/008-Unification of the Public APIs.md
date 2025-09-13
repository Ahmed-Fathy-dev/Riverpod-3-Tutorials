
### Unification of the Public APIs
## الوصف

الهدف من الـ **unification** إن Riverpod 3.0 يبسط الـ API: يوضّح إيه الموصى به وإيه legacy، يلغّي تكرار الواجهات اللي مالهاش لازمة، ويخلي السلوك متسق عبر كل الـ providers. بالتالي تم إجراء شوية تغييرات على الواجهات (interfaces) وطرق الـ imports علشان تبقى أسهل للفهم والاستخدام. 

---

## 1)البروفايدرز `StateProvider`/`StateNotifierProvider` و `ChangeNotifierProvider` — **مُهمّ :Moved to legacy import**

**إيه اللي اتغير؟** الحاجات دي مش محذوفة، لكن أصبحت شبه deprecated و **discouraged** وانتقلت ل import تاني مخصوص للـ legacy بدل المكتبة الرئيسية اللى هى :

```dart
import 'package:riverpod/riverpod.dart';

```

لو عايز تستخدمهم لسبب توافقيّ عشان الكود قديم تستخدم:

```dart
import 'package:riverpod/legacy.dart';

```

دة هيبين للمطوّر إن دي API قديمة وباقية بس للـ backward compatibility، والمطوّرين يفضلوا ينتقلوا لاستخدام الـ Notifier-based APIs الحديثة.

---

## 2) الـ **AutoDispose interfaces** اتشالوا (interfaces unified) — لكن السلوك لسه موجود

دة مش معناه ان feature الـ auto-dispose اتشال اللي اتغيّر إن الـ interfaces المتكررة اتوحّدت: بدل `AutoDisposeNotifier`، `AutoDisposeFamilyNotifier`، `AutoDisposeRef` إلخ — دلوقتي بنستخدم `Notifier` و `Ref` واحد موحد.

ايه التغييرات عالكود بدل:

```dart
class MyNotifier extends AutoDisposeNotifier<int> { ... }

```

تكتب:

```dart
class MyNotifier extends Notifier<int> { ... }
final provider = NotifierProvider.autoDispose<MyNotifier, int>(MyNotifier.new);

```

يعني الـ `autoDispose` لو محتاجه بتتحكم فيها على مستوى الـ provider (مثل `NotifierProvider.autoDispose<...>(...)`) لكن مش عن طريق اسم الكلاس `AutoDisposeNotifier`. 

**ملاحظة مهمة:** عشان يحافظوا على سلامة النوع والتصرف الصحيح، أي تحقّق compile-time اللي كان بيحصل سابقًا اتحوّل لـ lint rule عن طريق `riverpod_lint` بدل تكرار الinterfaces. فشغلك يبقى إنك تفعل الـ lints لو عايز التحذيرات دي. 

---

## 3)ال `FamilyNotifier` و `Notifier` **اتدمجوا (fused)**

**قبل:** كنت تلاقي `FamilyNotifier` كـ interface مختلفة عن `Notifier`، وتعرف `NotifierProvider.family<...>` مع `FamilyNotifier`.

**دلوقتي:** اتوحّدوا — بنستخدم `Notifier` واحد. لو محتاج باراميتر من الـ family، بتاخده في الـ constructor بتاع الـ Notifier. مثال توضيحي (من الدوكمنتيشن):

**قبل:**

```dart
final provider = NotifierProvider.family<CounterNotifier, int, Argument>(MyNotifier.new);

class CounterNotifier extends FamilyNotifier<int, Argument> {
  @override
  int build(Argument arg) => 0;
}

```

**بعد:**

```dart
final provider = NotifierProvider.family<CounterNotifier, int, Argument>(CounterNotifier.new);

class CounterNotifier extends Notifier<int> {
  CounterNotifier(this.arg);
  final Argument arg;

  @override
  int build() => 0;
}

```

**نتيجة العملية:** تخلصنا من تعدد الواجهات (`Notifier`, `FamilyNotifier`, `AutoDisposeNotifier`, `AutoDisposeFamilyNotifier`) — وبنستخدم `Notifier` بطريقة أبسط. 
الكلام ده **ما بيأثرش** على code-generation وبصراحة دى هتفيد كتير فى عملية الميجريشن.

---

## 4) مبدأ One `Ref` to rule them all — توحيد الـ `Ref`

**قبل 2.x:** كان في أنواع متعددة من `Ref` زي `FutureProviderRef`, `StreamProviderRef`, وغيرها، وكل واحد فيه خواص معينة (`state`, `future`, `notifier`...).

**في 3.0:** بقى في `Ref` واحد موحّد. فلو عندك كود معمول ليه جنيريت قبل كده كان يكتب مثلاً:

```dart
Example example(ExampleRef ref) { ... }

```

دلوقتي يكتب:

```dart
Example example(Ref ref) { ... }

```

**ملاحظة مهمة:** ده لا يشمل `WidgetRef` — `WidgetRef` لسه موجود ومختلف عن `Ref`. 

---

## 5)السلوك الخاص ب All `updateShouldNotify` now use `==`

- **إيه المشكلة القديمة؟** في Riverpod 2.0 كان سلوك `updateShouldNotify` متباين: بعض providers استخدموا `==`، بعضهم `identical`، وبعضهم logic أكثر تعقيدًا.
- **التغيير في 3.0:** كل الـ providers بقت تستخدم `==` كطريقة افتراضية لتحديد لو نبلغ الـ listeners أو لا.
- **آثار ممكن تحصل:**
    - بعض الـ providers ممكن **ما يبعتوش notify** في مواقف كانوا بيسجّلو notify فيها قبل كده.
    - بعض الـ listeners ممكن **يتنَبّهوا أكتر** من قبل.
    - لو عندك data class كبيرة وعليها override لـ `==`، ممكن تلاقي تأثيرات أداء طفيفة.
- **لو التغيير أثر عليك:** تقدر تتجاوز  `updateShouldNotify(previous, next)` بنفسك داخل `Notifier` لو محتاج سلوك مخصص من خلال عمل override لل ميثود .

### Without generation

```dart
class TodoList extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  @override
  bool updateShouldNotify(List<Todo> previous, List<Todo> next) {
    // Custom implementation
    return true;
  }
}
```

### With generation

```dart
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];

  @override
  bool updateShouldNotify(List<Todo> previous, List<Todo> next) {
    // Custom implementation
    return true;
  }
}
```

---
