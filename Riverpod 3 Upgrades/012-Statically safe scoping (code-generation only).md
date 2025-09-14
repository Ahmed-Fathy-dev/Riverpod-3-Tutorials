## Statically safe scoping (code-generation only)

الخاصية دي ظهرت مع **Riverpod + Code Generation** (يعني مع `@Riverpod` و `riverpod_generator`).

الهدف منها:

1. تمنعك من الوقوع في أخطاء وقت التشغيل **runtime errors** لما تستخدم Providers "scoped" (يعني لازم يكون لها **override** أو **dependencies**) من غير ما تضبطها.
2. من خلال **riverpod_lint**، تقدر تكتشف وقت الـ compile إنك ناسي تعمل override أو ناسي تكتب dependency.

---

## المثال

```dart
// A typical "scoped provider"
@Riverpod(dependencies: [])
Future<int> myFutureProvider() => throw UnimplementedError();

```

هنا عندنا **scoped provider**:

- `dependencies: []` → يعني ما فيش dependencies متسجلة لسه.
- الدالة نفسها بترجع `UnimplementedError`، كإشارة إنه مش هيتنفذ غير لما يتعمله override.

---

## إزاي تستخدم الـ provider ده؟

### الخيار الأول: تعمل Override باستخدام ProviderScope

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ProviderScope(
      overrides: [
        myFutureProvider.overrideWithValue(AsyncValue.data(42)),
      ],
      // لازم Consumer عشان تقدر توصل للـ provider
      child: Consumer(
        builder: (context, ref, child) {
          final value = ref.watch(myFutureProvider);
          return Text(value.toString());
        },
      ),
    );
  }
}

```

هنا أنا عرّفت قيمة للـ provider (`42`).

- أي `ref.watch(myFutureProvider)` جوا الـ `Consumer` هيلاقي قيمة بدل ما يقع في error.

---

### ✅ الخيار التاني: تستخدم `@Dependencies`

لو عندك Widget بيعتمد على provider ده، ممكن تعرّف كده:

```dart
@Dependencies([myFutureProvider])
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(myFutureProvider);
    return Text(value.toString());
  }
}

```

باستخدام `@Dependencies([myFutureProvider])` → بتقول صراحة إن `MyWidget` بيعتمد على `myFutureProvider`.

- دلوقتي أي مكان هيستعمل `MyWidget` لازم يوفّر override للـ provider **أو** يضيف dependency زي ما شوفنا.

---

## طيب بعد ما كتبت @Dependencies إيه اللي بيحصل؟

عندك خيارين تانيين لما تستخدم الـ widget ده:

---

### 1. Override باستخدام ProviderScope قبل ما تستعمل `MyWidget`

```dart
void main() {
  runApp(
    ProviderScope(
      overrides: [
        myFutureProvider.overrideWithValue(AsyncValue.data(42)),
      ],
      child: MyWidget(),
    ),
  );
}

```

---

### 2. أو: تضيف @Dependencies للـ widget اللي بيستعمل `MyWidget`

```dart
@Dependencies([myFutureProvider])
class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
     // MyApp indirectly uses scoped providers through MyWidget
     return MyWidget();
  }
}

```

كده `MyApp` بيقول إنه بيعتمد على `myFutureProvider` بشكل غير مباشر (لأنه بيستعمل `MyWidget`).

- بالتالي أي مكان يستخدم `MyApp` لازم يوفر override أو dependency بنفس الطريقة.

---

## الخلاصة

- **ال Statically safe scoping** = نظام يخليك تعرف بدري (عن طريق lint) لو نسيت تعمّل override أو تحدد dependency.
- عندك طريقتين رئيسيتين:
    1. ال `ProviderScope(...overrides: [...])`
    2. ال `@Dependencies([...])` على الـ widgets اللي بتستخدم أو بتستدعي widgets بتعتمد على scoped providers.

ده بيخلي الكود بتاعك **آمن وقت الـ compile** بدل ما يبانلك error مفاجئ في runtime.

---
