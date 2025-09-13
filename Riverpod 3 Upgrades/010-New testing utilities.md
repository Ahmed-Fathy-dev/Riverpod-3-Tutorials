## أولا : ProviderContainer.test

```dart
void main() {
  test('My test', () {
    final container = ProviderContainer.test();
    // Use the container
    // ...
    // The container is automatically disposed after the test ends
  });
}

```

**عبارة عن ايه `ProviderContainer.test()`**

دي دالة جديدة في Riverpod 3.0 بتنشئ `ProviderContainer` مخصصة للاختبارات. الفكرة إنك بتختبر providers بمعزل عن التطبيق أو شجرة الـ widgets. ومن ميزاتها إنها **بتتخلص (dispose)** من نفسها في نهاية الاختبار، فمش محتاج تقلق من تنظيف الموارد يدويًا.

**ال `test('My test', () { ... })`**

دي وحدة اختبار عادية من مكتبة `package:test`  جواها بنكتب الكود اللي عايزين نختبره.

**استخدام `final container = ProviderContainer.test();`**

جوا الاختبار، أول خطوة هي إنشاء الـ container. ده هيبقى مكاننا نقرأ منه أو نستمع لـ providers اللي حبّينا نختبرهم.

**تنويه واضح ان “The container is automatically disposed after the test ends”**

الرسالة بتوضح إن ما تقلقش من تنظيف الموارد أو التخلص من `ProviderContainer` بنهاية الاختبار Riverpod بيتكفل بالأمر لما الاختبار يخلص.

**وهنا  You can safely do a global search-and-replace for `createContainer` to `ProviderContainer.test`.”**

 لو عندك كود قديم أو حيل اختبار (helper function) اسمها `createContainer` لاستعمال Riverpod داخل الاختبارات، في Riverpod 3.0 تقدر تغير كل استخداماتها لـ `ProviderContainer.test()` بسهولة، من غير ما تتوه بسبب التغييرات.

---

## ثانيا :  NotifierProvider.overrideWithBuild

الخاصية دي بتتيح لك إنك تعمل (mock) لميثود`build` بس للـ Notifier مش الكائن كله يعني إمّا تبدأ من حالة معيّنة أو ترجع قيمة مبدئية مختلفة، بينما باقي وظائف الـ Notifier زي `increment()` تفضل شغالة الأصلية. ده مفيد للاختبارات (unit tests) لما تحب تبدأ من state معيّن بس مش تحب تبدّل سلوك كل الـ Notifier.

**الكود أول مثال (بدون code generation)**

```dart
class MyNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}

final myProvider = NotifierProvider<MyNotifier, int>(MyNotifier.new);

void main() {
  final container = ProviderContainer.test(
    overrides: [
      myProvider.overrideWithBuild((ref) {
        // Mock the build method to start at 42.
        // The "increment" method is unaffected.
        return 42;
      }),
    ],
  );
}

```

ال `MyNotifier` يمتد من `Notifier<int>`، وبيجهز `build()` يرجّع `0` كحالة أولية افتراضية.

- فيه دالة `increment()` بتعدّل الـ state (زي زيادة العدد).
- بنعرّف `myProvider` كـ `NotifierProvider` مع الـ Notifier ده.
- جوا `main()`, بننشئ `ProviderContainer.test(...)` مع قائمة `overrides` فيها استدعاء لـ `myProvider.overrideWithBuild(...)`.
- الدالة اللي بتمررها لـ `overrideWithBuild((ref) { … })` بترجع `42` — ده يعني إنّ حالة `build()` هتكون `42` بدال `0` في هذا الاختبار.
- مهمة: `increment()` لسه موجودة وتشتغل زي ما هي، مش بتتغيّر.

**الكود الثاني (مع annotation / code generation)**

```dart
@riverpod
class MyNotifier extends _$MyNotifier {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}

void main() {
  final container = ProviderContainer.test(
    overrides: [
      myProvider.overrideWithBuild((ref) {
        // Mock the build method to start at 42.
        // The "increment" method is unaffected.
        return 42;
      }),
    ],
  );
}

```

هنا بنستخدم `@riverpod` annotation فبيولّد الكود اللازم تلقائيًا.

- باقي الكود نفسه: `build()` يرجع `0`، `increment()` بزيادتها في state، override للبناء فقط يبدأ من `42` للاختبار.
- الفائدة: لو عندك Notifier بتعمل logic معين في `build()` أو بتبني الحالة من معاملات أو داتا من مزودات أخرى، تقدر تختبر الحالة الأولى فقط بمساعدة `overrideWithBuild`.

**آثار العملية**

- في الاختبارات، تقدر تضمن الحالة الابتدائية تكون اللي أنت عايزها بسهولة بدون ما تعمل mock كامل أو تستخف بحالة إنك محتاج rest of functionality.
- الكود المنتَج (increment وغيره) تقدر تستخدمة فعليًا للتأكد إن وظائفه تشتغل بعد override.
- لو عندك Notifier فيه side-effects أو dependencies، `overrideWithBuild` بتأثر فقط `build()`، مش تغيّر كل السلوك، فممكن تحتاج تحضر الاختبار بحسب الحالة لو فيه dependencies.

---

### ثالثا : Future/StreamProvider.overrideWithValue

من فترة، كانت الـ `FutureProvider.overrideWithValue` و `StreamProvider.overrideWithValue` متشالين (مش موجودين) مؤقتًا من Riverpod. ودلوقتى رجعو تاني

without generation

```dart
final myFutureProvider = FutureProvider<int>((ref) async {
  return 42;
});

void main() {
  final container = ProviderContainer.test(
    overrides: [
      // Initializes the provider with a value.
      // Changing the override will update the value.
      myFutureProvider.overrideWithValue(AsyncValue.data(42)),
    ],
  );
}

```

riverpod generator

```dart
@riverpod
Future<int> myFutureProvider() async {
  return 42;
}

void main() {
  final container = ProviderContainer.test(
    overrides: [
      // Initializes the provider with a value.
      // Changing the override will update the value.
      myFutureProvider.overrideWithValue(AsyncValue.data(42)),
    ],
  );
}

```

---

## رابعا : **WidgetTester.container**

الخاصية دي **اتضافت في Riverpod 3.0** كطريقة سهلة إنك توصل للـ **ProviderContainer** اللي موجود جوا شجرة الـ widget أثناء اختبارات الـ Flutter.

ببساطة: بدل ما تضطر تعمل `ProviderScope.containerOf(context)` أو تلاقي حلول ملتوية علشان توصل للـ container، دلوقتي تقدر تستخدم **`WidgetTester.container`**بشكل مباشر.

### مثال

```dart
void main() {
  testWidgets('can access a ProviderContainer', (tester) async {
    await tester.pumpWidget(const ProviderScope(child: MyWidget()));
    ProviderContainer container = tester.container();
  });
}

```

### شرح خطوة بخطوة

1. **ال `testWidgets('can access a ProviderContainer', (tester) async { ... })`**
    
    هنا بنكتب اختبار Flutter عادي باستخدام `testWidgets`. جوه الاختبار بنستقبل `tester` اللي  نوع `WidgetTester` وده اللي بيسمحلك تتحكم في الـ widget tree.
    
2. **ال `await tester.pumpWidget(const ProviderScope(child: MyWidget()));`**
    - بنبني الشجرة بتاعة الـ widget اللي عايزين نختبرها.
    - أهم نقطة هنا إننا مغلفين الـ `MyWidget` جوا `ProviderScope`.
    - `ProviderScope` هو اللي بيحتوي على الـ `ProviderContainer` وبيخلي الـ providers متاحة للـ widget tree كلها.
3. **`ProviderContainer container = tester.container();`**
    - هنا بنستخدم الميثود الجديدة `tester.container()`.
    - دي بترجعلك الـ `ProviderContainer` اللي أنشأه الـ `ProviderScope` في الشجرة.
    - بكده بقى عندك وصول مباشر للـ container من غير ما تحتاج تعمل أي خطوات إضافية.

### ليه ده مهم؟

- **سهولة الوصول**: بدل ما تضطر تستخرج الـ container بطرق معقدة، دلوقتي الموضوع مباشر جدًا.
- **اختبار الـ providers**: تقدر تستخدم `container.read`, `container.listen`, أو تعمل overrides للـ providers بسهولة أثناء الاختبار.
- **تنظيف (dispose) أو متابعة الحالة**: أوقات بتحتاج تعرف هل provider اتعمل له dispose ولا لأ — دلوقتي عندك container نفسه تحت إيدك.

---
