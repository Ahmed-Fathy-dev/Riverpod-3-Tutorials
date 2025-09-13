الميزة دي بتضيف دعم تلقائي لإعادة المحاولة (automatic retry support) للـproviders اللي بتفشل أثناء التهيئة (fail during initialization). ده بيحصل مع exponential backoff، يعني الانتظار بيزيد تدريجيًا (مثل 200ms ثم 400ms ثم 800ms، إلخ)، والـprovider هيحاول يعيد المحاولة لحد ما ينجح أو يتdispose (يتدمر). ده بيساعد لو العملية فشلت بسبب مشكلة مؤقتة (temporary issue)، زي انقطاع الشبكة (lack of network connection).

الميزة دي مدعومة في كل الـproviders اللي asynchronous، زي `AsyncNotifier`، `FutureProvider`، `StreamProvider`. وهي تعمل مع riverpod العادي (بدون code generation) ومع riverpod_generator (لو بتستخدم `@Riverpod` لتوليد الكود تلقائيًا).

### السلوك الافتراضي (Default Behavior)

الدوكيومنتيشن بتقول إن السلوك الافتراضي (default behavior) بيعيد المحاولة لأي خطأ (retries any error)، وبيبدأ بتأخير 200ms (200ms delay) اللي بيتضاعف بعد كل محاولة لحد ما يوصل لـ6.4 ثواني (doubles after each retry up to 6.4 seconds). يعني، لو الـprovider فشل أول مرة، هينتظر 200ms ويعيد، لو فشل تاني هينتظر 400ms، ثالث 800ms، رابع 1.6s، خامس 3.2s، سادس 6.4s، وهكذا لحد ما ينجح أو يتdispose.

ده بيحصل تلقائيًا بدون أي إعداد إضافي، سواء كنت بتستخدم riverpod العادي أو riverpod_generator. مثال بسيط (بدون تخصيص):

```dart
final myProvider = FutureProvider<int>((ref) async {
  // لو ده فشل بسبب شبكة، Riverpod هيعيد المحاولة تلقائيًا
  return await fetchDataFromServer();
});

```

### تخصيص الـRetry لكل الـProviders (Global Customization)

الدوكيومنتيشن بتوضح إن تقدر تخصص الـretry لكل الـproviders في التطبيق باستخدام `ProviderContainer` أو `ProviderScope`، عن طريق تمرير parameter اسمه `retry`. الـ`retry` ده دالة بتاخد `retryCount` (عدد المحاولات اللي فاتت) و`error` (الخطأ اللي حصل)، وبترجع `Duration` (التأخير للمحاولة الجاية) أو `null` لو عايز تتوقف عن المحاولات.

**مثال لـProviderScope (في runApp)**:

```dart
void main() {
  runApp(
    ProviderScope(
      // You can customize the retry logic, such as to skip
      // specific errors or add a limit to the number of retries
      // or change the delay
      retry: (retryCount, error) {
        if (error is SomeSpecificError) return null; // تتوقف لو الخطأ ده
        if (retryCount > 5) return null; // تتوقف بعد 5 محاولات
        return Duration(seconds: retryCount * 2); // تأخير يزيد كل مرة (2s, 4s, 6s, إلخ)
      },
      child: MyApp(),
    ),
  );
}

```

- **الشرح**: الـ`retry` هنا بيطبق على كل الـproviders في التطبيق. لو رجعت `null`، الـretry هيتوقف. ده يعمل مع riverpod العادي أو riverpod_generator، لأن `ProviderScope` هو الجذر للـapp.

**مثال لـProviderContainer (في الـtests أو غير الـUI)**:

```dart
final container = ProviderContainer(
  retry: (retryCount, error) {
    if (error is SomeSpecificError) return null;
    if (retryCount > 5) return null;
    return Duration(seconds: retryCount * 2);
  },
);

```

- **الشرح**: نفس المنطق، بس لـ`ProviderContainer` (اللي بيُستخدم خارج الـUI، زي في الـunit tests).

### الجزء الرابع: تخصيص الـRetry لكل Provider لوحده (Per-Provider Customization)

الدوكيومنتيشن بتقول إن تقدر تخصص الـretry لكل provider لوحده عن طريق تمرير `retry` parameter في constructor الـprovider. ده بيطبق بس على ده provider، وهيغلب على الـglobal retry لو موجود.

**مثال لـNotifierProvider (بدون code generation)**:

```dart
final todoListProvider = NotifierProvider<TodoList, List<Todo>>(
  TodoList.new,
  retry: (retryCount, error) {
    if (error is SomeSpecificError) return null; // تتوقف لو الخطأ ده
    if (retryCount > 5) return null; // تتوقف بعد 5 محاولات
    return Duration(seconds: retryCount * 2); // تأخير يزيد كل مرة
  },
);

```

- **الشرح**: هنا الـ`retry` خاص بـ`todoListProvider` بس. لو الـprovider ده فشل، هيستخدم ده المنطق، حتى لو فيه global retry مختلف.

**مع riverpod_generator (code generation)**:
لو بتستخدم `@Riverpod` لتوليد الكود تلقائيًا، الميزة دي تعمل بنفس الطريقة، لأن riverpod_generator بيولد `NotifierProvider` تلقائيًا، وتقدر تضيف الـ`retry` في التعريف إذا احتجت تخصيص يدوي

```dart
Duration retry(int retryCount, Object error) {
  if (error is SomeSpecificError) return null;
  if (retryCount > 5) return null;

  return Duration(seconds: retryCount * 2);
}

@Riverpod(retry: retry)
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];
}
```

- **ملاحظة**: لو عايز تخصيص per-provider مع code generation، كنت هتعمل override للـprovider المولد في `ProviderScope` أو `ProviderContainer`، زي:
    
    ```dart
    ProviderScope(
      overrides: [
        todoListProvider.overrideWith(
          (ref) => TodoList()..retry = (retryCount, error) { /* تخصيص */ },
        ),
      ],
    )
    
    ```
    
    - ده يعمل مع riverpod_generator عشان الـgenerated providers بيورثوا من الـbase classes.

### ملاحظات

- **الفايدة**: الميزة دي بتحسن التعامل مع الفشل المؤقت (زي مشاكل الشبكة)، بدون ما تضطر تكتب كود يدوي للـretry. الـexponential backoff بيقلل الحمل على الـserver.
- **التكامل**: تعمل مع كل أنواع الـproviders الـasynchronous، سواء riverpod العادي أو riverpod_generator. الـdefault behavior بيطبق لو ما حددتش `retry`.
- **تنويه مهم**: الميزة دي جديدة في 3.0، فلو بتعمل migration من إصدار أقدم، تأكد إن الكود بتاعك مايعتمدش على عدم الـretry (زي لو عايز يفشل فورًا، حدد `retry: (retryCount, error) => null;`).
