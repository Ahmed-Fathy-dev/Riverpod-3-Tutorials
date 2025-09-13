دي من أهم التغييرات اللي حصلت في Riverpod 3.0 وتعتبر **fundamental changes** في طريقة شغل الـ providers.
 التغييرات دي لازم تتفهم كويس جدا لان تأثيراتها كتير على الـ app بتاعك عشان كده لازم تكون حذر في الـ upgrade.

## مقدمة عن الـ Life-cycle Changes

الـ **Provider life-cycle** يقصد بيه دورة حياة الـ provider من لحظة إنشاؤه لحد ما يتم dispose. التغييرات اللي حصلت في النسخة 3.0 كانت ضرورية عشان تدعم الـ features الجديدة زي `Ref.mounted` والـ pause/resume functionality .

 هنقسم الشرح لعدة أقسام رئيسية عشان نتناول كل تغيير بالتفصيل:

## أولا: ProviderException Wrapping

في Riverpod 2.0، لو الـ provider رمى exception، ريفربود كان بيعيد رمي الـ error مباشرة:

**مثال بـ riverpod:**

```dart
dartfinal exampleProvider = FutureProvider<int>((ref) async {
  throw StateError('Error');
});

*// ...*
ElevatedButton(
  onPressed: () async {
    *// This will rethrow the StateError*
    ref.read(exampleProvider).requireValue;
    
    *// This also rethrows the StateError*
    await ref.read(exampleProvider.future);
  },
  child: Text('Click me'),
);
```

**مثال بـ riverpod_generator:**

```dart
dart@riverpod
Future<int> example(Ref ref) async {
  throw StateError('Error');
}

*// ...*
ElevatedButton(
  onPressed: () async {
    *// This will rethrow the StateError*
    ref.read(exampleProvider).requireValue;
    
    *// This also rethrows the StateError*
    await ref.read(exampleProvider.future);
  },
  child: Text('Click me'),
);
```

## التغيير في 3.0

في النسخة 3.0، الـ error دلوقتي بيتلف في **ProviderException** اللي بيحتوي على الـ original error والـ stack trace .

## الفوائد من التغيير ده

1. **تحسين الـ Debugging** - دلوقتي عندنا stack trace أفضل بكتير
2. **التمييز بين الـ Errors** - دلوقتي نقدر نعرف لو الـ provider فشل مباشرة ولا بسبب dependency هو اللى عمل الخطأ

## مثال عملي على الاستخدام

```dart
dartclass MyObserver extends ProviderObserver {
  @override
  void providerDidFail(ProviderObserverContext context, Object error, StackTrace stackTrace) {
    if (error is ProviderException) {
      *// The provider didn't fail directly, but instead depends on a failed provider.// The error was therefore already logged.*
      return;
    }
    
    *// Log the error*
    print('Provider failed: $error');
  }
}
```

## الاستخدام في Automatic Retry

الـ **ProviderException** بيستخدم داخلياً في آلية الـ automatic retry. الـ default retry behavior بيتجاهل الـ `ProviderException`:

```dart
dartProviderContainer(
  *// Example of the default retry behavior*
  retry: (retryCount, error) {
    if (error is ProviderException) return null;
    *// ...*
  },
);
```

## ثانيا: Listeners inside widgets that are not visible are now paused

لما يكون عندك مستمعين (listeners) داخل Widgets مش ظاهرة حاليًا (غير مرئية للـ user).
 ريفربود دلوقتي بوقفهم تلقائيًا (“pause”) 
 بدل ما يكونو شغالين ومستهلكين موارد أو بتسبب إعادة بناء أو استماعات ملهاش لازمة.

- ريفربود بتعتمد علي Flutter’s `TickerMode` للتأكد من الـ visibility بتاع الـ widget. يعني لو الـ widget مش ظاهر على الشاشة (visibility false)، الاستماعات (listeners) اللي جواها بتتبطل مؤقتًا (`paused`).
- ده بيحصل لأي listener اللي اتعمل من خلال `ref.listen` أو `ref.watch` جوّه `Consumer` داخل widget مش ظاهرة.
- الـ provider اللي بيكون listener بيتأثر — لو كان مش ظاهر بالكامل، أي subscription من المستمعين بتتوقف. ده توفير في الأداء، وبيقلل استهلاك الموارد.

## مثال مع الشرح

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return TickerMode(
      enabled: false, // لما تخلّي القيمة false → المستمعين يُوقفوا
      child: Consumer(
        builder: (context, ref, child) {
          // هذا الـ watch هيتوقف مؤقتًا
          final value = ref.watch(myProvider);
          return Text(value.toString());
        },
      ),
    );
  }
}

```

في الكود ده، `myProvider` عنده `ref.watch` داخل `Consumer`، لكن لأن الـ widget مربوط بـ `TickerMode(enabled: false)`، الـ listener يتوقف (`paused`) حتى `TickerMode.enabled` يبقى true. 

---

## شرح اعمق

- لو عندك أجزاء من الـ UI مش مرئية حاليًا (مثلاً صفحة مخفية ورا صفحة أخرى، أو widget داخل `Offstage` أو `Visibility(visible: false)` أو `TickerMode(enabled: false)`)، الـ listeners جواهم مش هتكون فعّالة وقتها  مش هيتابعوا تغييرات الـ provider لحد الرجوع للعرض. ده توفير موارد وبيمنع نشاط غير ضروري.
- لو الكود بتاعك بيعتمد على `ref.watch` + side-effects من التغيرات، يبقى المكون اللي مستمع يبقى مستعد إنه ميبقاش فيه تغيرات أو مش هيشوفها وقت ما يكون مخفي. لازم تفكّر في الحالة دي.
- تراك شغلك ممكن تحتاج تختبر إنه لو الـ widget أصبح مرئي بعد ما كان مخفي، الاستماعات ترجع تشتغل من تاني.

---

## حاجات مُهم تراعيها / حدود التغيير

- التغيير ده مش بيؤثر على كل المستمعين  المستمع يكون جوّه widget مش مرئي يستخدم طرق الـ listen/watch — لكن لو listener مستخدم في مكان غير widget (مثلاً داخل provider أو logic مش مرئي)، ممكن ميتأثرش بنفس الطريقة. الدوكمنتشن بتقول “inside widgets that are not visible”.
- استخدام `TickerMode` مهم جدًا: لازم تكون عارف أمتى الـ widget مش مرئية لتستخدم أو تفهم السلوك الجديد.
- لو عندك timer أو stream أو WebSocket بيشتغل في الخلفية لكنه متعلق widget مش ظاهر، ممكن يكون أفضل تنقله لمكان تاني لو عايز يفضل شغّال حتى لو مش ظاهر.

---

# ثالثا : **If a provider is only used by paused providers, it is paused too**

### إيه المُشكلة اللي كانت موجودة قبل التحديث؟

- في Riverpod 2.0، لو عندك provider (نسميه `exampleProvider`) مفيش مستمع مباشر (listener) من الـ UI أو من مكان ظاهر، لكن بيُستخدم *ضمنيًا* عن طريق provider تاني (`anotherProvider`) اللي هو نفسه **مُوقَّف (paused)** — provider التاني مش ظاهر أو مستمع ليه، فكان `exampleProvider` بيفضل شغال رغم إنه فعليًا مش محتاج يكون شغال.

الكود اللي في الدوكمنتيشن بيوضح السيناريو ده:

```dart
final exampleProvider = Provider<int>((ref) {
  ref.onCancel(() => print('paused'));
  ref.onResume(() => print('resumed'));
  return 0;
});

final anotherProvider = Provider<int>((ref) {
  return ref.watch(exampleProvider);
});

class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Button(
      onPressed: () {
        ref.read(anotherProvider);
      },
      child: Text('Click me'),
    );
  }
}

```

في الـ version القديم، لو تضغط الزر مرّة، `anotherProvider` يبدأ يستخدم `exampleProvider`، لكن لو تبتدي تركّز إن `anotherProvider` بيتوقف أو يُوقَّف، `exampleProvider` كان هيفضل شغال بالرغم إن كل اللي بيستخدمه providers اللي موقوفة.

---

### ✅ التغيير اللي اتعمل في Riverpod 3.0

- في النسخة الجديدة، Riverpod بقى عنده قاعدة: **لو provider ما بيُستخدمش إلا من قِبل providers موقوفة (paused)، يبقى هو كمان يتوقَّف (paused).**
- يعني الـ `exampleProvider` في المثال الفوق ده  بعد الضغط  لو `anotherProvider` اتوقَّف `exampleProvider` كمان هيتوقَّف تلقائي. مش هيكون شغال في الخلفية بدون داعي.

---


### مثال باستخدام **riverpod**:

```dart
final exampleProvider = Provider<int>((ref) {
  ref.onCancel(() => print('paused'));
  ref.onResume(() => print('resumed'));
  return 0;
});

final anotherProvider = Provider<int>((ref) {
  return ref.watch(exampleProvider);
});

class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        ref.read(anotherProvider);
      },
      child: const Text('Click me'),
    );
  }
}

```

لما تقدر زرار الـ `ElevatedButton` تتضغط، `anotherProvider` يبدأ يقرأ `exampleProvider`.

لو بعد كده `anotherProvider` بيُوقَّف أو مفيش مستمع ظاهر ليه، Riverpod 3.0 هيوقّفه. ومن ثم `exampleProvider` هيُوقّف كمان لأنه ما بيُستخدمش إلا من `anotherProvider` اللي هو موقوف. 

### مثال باستخدام **riverpod_generator**:

```dart
@riverpod
int example(Ref ref) {
  ref.onCancel(() => print('paused'));
  ref.onResume(() => print('resumed'));
  return 0;
}

@riverpod
int another(Ref ref) {
  return ref.watch(exampleProvider);
}

class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        ref.read(anotherProvider);
      },
      child: const Text('Click me'),
    );
  }
}

```

نفس المنطق هنا: exampleProvider يُعرّف مع callbacks `onCancel` و `onResume`. anotherProvider بـ `ref.watch(exampleProvider)` يستخدمه.

- ريفربود 3.0 بتراقب إن `anotherProvider` موقوف  في الحالة دي `exampleProvider` كمان يتوقَّف لأنه "مُستخدم فقط بواسطة providers موقوفة".

---

### فوائد التغيير ده

- تقليل استهلاك الموارد (resources)  ما فيش provider شغال بدون داعي.
- الأداء هيتحسن لما يكون عندك tree من الـ providers مترابطة، لكن بعضها مش ظاهر أو مش مستخدم من الـ UI.
- السلوك داخليًا يصبح *أكثر اتساقًا* مع التوقع: لو مفيش مستمع مباشر، ولا مستخدم فعال، يبقى مش لازم يعمل work.

---

### أمور محتاج تراعيها

- لو عندك كود بيعتمد بشكل ضمني على provider "مشتغل" حتى لو مفيش مستمع ظاهر، ممكن يبقى فيه تغيّر في السلوك وهنا فايدة التست.
- لو عندك side-effects أو logic داخل provider بيعتمد إنه دايمًا شغال، لازم تتأكد إن الحالة دي مش بتتغيّر لأن provider ممكن يُوقَّف تلقائيًا.
- استخدام `onResume` و `onCancel` ممكن يساعد تختبر السلوك الجديد إن الـ providers فعلاً بتوقف وتعود للعمل وقت ما يكون مطلوب.

---

## رابعا :When a provider rebuilds, its previous subscriptions now are kept until the rebuild completes

### 1) المشكلة في Riverpod 2.0

لما يكون عندك provider async (مثلاً `FutureProvider`) + provider تاني autoDispose (مثلاً `StreamProvider.autoDispose`)،
 وجوا الـ async provider تعمل `await null` أو أي await ( *async gap*)
 وبعدها تستخدم `ref.watch` للـ autoDispose provider، ممكن الـ autoDispose provider يعمل (disposed) خلال الـ gap ده.

ده بيأدي لعدة مشاكل: يكون فيه مستمع (listener) مطلوب إنه يستمر، لكن تم إزالته فجأة بدون إن الـ rebuild الجديد يبدأ.

---

طبعا اللى كان بيستخدم ريفربود في لوجيك معقد ومرتبط باكتر من بروفايدر اكيد كان بيلاحظ المشكلة دى وخلينا نشوف دة بمثال اوضح
**riverpod** بدون generator

```dart
final autoDisposeProvider = StreamProvider.autoDispose<int>((ref) {
  ref.onDispose(() => print('disposed'));
  ref.onCancel(() => print('paused'));
  ref.onResume(() => print('resumed'));
  // A stream that emits a value every second
  return Stream.periodic(Duration(seconds: 1), (i) => i);
});
final asynchronousExampleProvider = FutureProvider<int>((ref) async {
  print('Before async gap');
  // An async gap inside a provider ; typically an API call.
  // This will dispose the "autoDispose" provider
  // before the async operation is completed
  await null;

  print('after async gap');
  // We listen to our auto-dispose provider
  // after the async operation
  return ref.watch(autoDisposeProvider.future);
});
void main() {
  final container = ProviderContainer();
  // This will print 'disposed' every second,
  // and will constantly print 0
  container.listen(asynchronousExampleProvider, (_, value) {
    if (value is AsyncData) print('${value.value}\n----');
  });
}

```

### **riverpod_generator** مع الـ annotation

```dart
@riverpod
Stream<int> autoDispose(Ref ref) {
  ref.onDispose(() => print('disposed'));
  ref.onCancel(() => print('paused'));
  ref.onResume(() => print('resumed'));
  // A stream that emits a value every second
  return Stream.periodic(const Duration(seconds: 1), (i) => i);
}
@riverpod
Future<int> asynchronousExample(Ref ref) async {
  print('Before async gap');
  // An async gap inside a provider ; typically an API call.
  // This will dispose the "autoDispose" provider
  // before the async operation is completed
  await null;

  print('after async gap');
  // We listen to our auto-dispose provider
  // after the async operation
  return ref.watch(autoDisposeProvider.future);
}
void main() {
  final container = ProviderContainer();
  // This will print 'disposed' every second,
  // and will constantly print 0
  container.listen(asynchronousExampleProvider, (_, value) {
    if (value is AsyncData) print('${value.value}\n----');
  });
}

```

---

### 3) السلوك في Riverpod 2.0

الكود يطبع:

```
Before async gap
after async gap
0
----
paused
Before async gap
disposed // الـ autoDispose provider تم disposed خلال الـ async gap
after async gap
0
----
paused
Before async gap
disposed
after async gap
0
----
...

```

الفكرة إن كل ما `asynchronousExampleProvider` يعيد بناء، بسبب الـ await null، الـ autoDispose provider يتخلص، بعدين لما الـ watch يحصل يبقى لازم يبدأ من جديد.

النتيجة: البرنامج يطبع `0` كل ثانية، لكن كل ثانية بيكون فيه disposal غير متوقع ل الليسينرز.

---

### 4) التعديل في Riverpod 3.0

التغيير إنه بدل ما dispose يستدعي فورًا (immediately) عند الـ rebuild، الآن الـ listeners القديمة **مُوقَفة مؤقتًا (paused)** لحد لما الـ rebuild يخلص.

وبكده، ما يحصلش dispose مفاجئ خلال الـ async gap.

نفس الكود بعد التحديث هيطبع:

```
Before async gap
after async gap
0
----
paused
Before async gap
after async gap
resumed
1
----
paused
Before async gap
after async gap
resumed
2
----
... // وهكذا كل ثانية

```

يعني بدل ما القيمة تبقى دايمًا `0`، هترتفع بعد ما يكون الـ provider resumed (يستعيد الـ listener) ويبدأ يستلم القيم المستمرة من الستريم.

---

## تأثير التغييرات دى

- لو الكود بتاعك بيعتمد على `ref.watch(...)` بعد `await`, خصوصًا مع `autoDispose`, لازم تبقى عارف السلوك الجديد ده عشان متتفاجئش إن الـ provider مش اتخلص فجأة.
- ممكن تتأكد إن أي `ref.watch(...)` مهم يكون قبل أي `await` لو عايز تستفيد من الاستمرارية من البداية، لكن دلوقتي الموضوع أكثر استقرار بفضل التحديث ده.
- مهم تختبر التطبيق بعد التحديث عشان تتأكد إن ما فيش سلوك غير متوقع خصوصًا في الحالات اللي فيها async latency كبيرة أو استجابات متأخرة.

---

## خامسا : **Exceptions in providers are rethrown as a `ProviderException`**

 الهدف إن يكون فيه فرق واضح بين:

• إن الـ provider نفسه فشل يبني القيمة بتاعته (failed)

• وإنه فشل لأنه بيعتمد على provider تاني فشل.

 عشان كده، Riverpod 3.0 عمل راب لل **exceptions (الأخطاء)** اللي بتحصل داخل الـ provider في كائن اسمه `ProviderException`، والكائن ده بيحتوي على الخطأ الأصلي والـ stack trace بتاعه. 

**ازاى الكود كان بيستخدم try/catch قبل التغيير**

```dart
try {
  await ref.read(myProvider.future);
} on NotFoundException {
  // Handle NotFoundException
}

```

 هنا بيتوقع أن `ref.read(...).future` ممكن يرمي `NotFoundException` مباشرةً، فيتم الإمساك بيه.

**ودلوقتى الكود لازم يتغير بعد التحديث**

```dart
try {
  await ref.read(myProvider.future);
} on ProviderException catch (e) {
  if (e.exception is NotFoundException) {
    // Handle NotFoundException
  }
}

```

دلوقتى الـ error بيترمى داخل `ProviderException`، فـ الـ `catch` لازم يمسك `ProviderException`، ومن جواه يبص على الحقل `exception` (اللي بيكون الخطأ الأصلي) لو عايز يعمل معالجة خاصة حسب نوعه. 

**إيه الحاجات اللي *مش* تأثرت بالتغيير**

 ال`AsyncValue.error`: لما تستخدم `.watch(...)` وترجعك `AsyncValue`, لو فيه خطأ، القيمة هتحتوي الخطأ نفسه، مش ملفوف في `ProviderException` دائماً. 

ال `ref.listen(..., onError: ...)`: لو مستمع للـ provider باستخدام `ref.listen` مع المعامل `onError`, الخطأ بيوصل بنفسه. 

 ال`ProviderObserver`: لو بتراقب providers باستخدام observer خاص، الخطأ الأصلي هيوصلهم (مش ملفوف) كمان.

## التأثير العملي

- لو عندك كود بيعتمد على `try on SomeSpecificException` بدون `ProviderException` — هتفاجئ إن الكود مش بيمسك الخطأ لأنه بقى ملفوف.
- اللي عايز يعالج أنواع أخطاء معينة (مثلاً `NotFoundException`, `TimeoutException`, إلخ) — لازم تعدّل try/catch بتاعك يمسك `ProviderException` ويبص على `e.exception`.
- أما لو بتستخدم `AsyncValue` أو `watch` أو `listen` أو Observer غالبًا مش محتاج تغيير في الأماكن دي بخصوص هذا التغيير.

---
