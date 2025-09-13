الوصف 

 الميزة دي بتتعلق بتحسينات في الـcode generation بتاع Riverpod، تحديدًا مع الـgenerics. لما تستخدم الـcode generation، دلوقتي تقدر تعرف type parameters للـproviders اللي بتتولد. الـtype parameters دي بتشتغل زي أي parameter تاني للـprovider، ولازم تمررها لما تعمل `ref.watch` للـprovider.

ده يعني إن الـgenerics (الأنواع العامة) بقت مدعومة في الـproviders المولدة تلقائيًا باستخدام `@Riverpod`، وبتديك مرونة أكتر في كتابة functions أو classes عامة.

### كيفية الاستخدام والمثال

مثال بسيط عشان توضح إزاي تستخدم الـgenerics مع الـcode generation. المثال ده عن function بتاخد type parameter `<T>` وبتستخدمه في عملية حسابية.

**المثال**:

```dart
T multiply<T extends num>(T a, T b) {
  return a * b;
}

// ...
int integer = ref.watch(multiplyProvider<int>(2, 3));
double decimal = ref.watch(multiplyProvider<double>(2.5, 3.5));

```

- **الشرح**:
    - الـfunction `multiply` معرفة بـ`@Riverpod` ( يعني Riverpod هيولد `multiplyProvider` تلقائيًا).
    - هنا `<T extends num>`: ده يعني إن `T` لازم يكون نوع من أنواع الأرقام (num، زي int أو double)، وبتضمن type safety.
    - الـfunction بترجع ضرب `a` في `b` (a * b).
    - لما تعمل `ref.watch`:
        - هنا `multiplyProvider<int>(2, 3)`: هيستخدم `T` كـ`int`، وهيرجع نتيجة الضرب لـ2 و3 (يعني 6).
        - هنا `multiplyProvider<double>(2.5, 3.5)`: هيستخدم `T` كـ`double`، وهيرجع نتيجة الضرب لـ2.5 و3.5 (يعني 8.75).
    - النقطة المهمة: الـtype parameters (`<int>` أو `<double>`) بتتعامل زي أي parameters تانية (زي 2 و3)، ولازم تمررها لما تستدعي الـprovider.

### ليه الميزة دي مفيدة؟

الميزة دي انتظرناها بصراحة كتير اوى واتطلبت من ريمي من مطورين كتير
 وكنا بنعملها احنا بكود زيادة من غير المكتبة عشان نطبق نفس السلوك 

- **مرونة أكبر**: تقدر تعرف providers عامة (generic) بتشتغل مع أنواع بيانات مختلفة (زي num، String، أو كلاسات مخصصة)، بدون ما تعرف provider منفصل لكل نوع.
- **ال Type safety**: الـgenerics بتضمن إن الـDart analyzer يتحقق من الأنواع في وقت الكومبايل (compile-time)، يعني لو استخدمت نوع غلط (مثلاً `multiplyProvider<String>(2, 3)`، لأن String مش extends num)، هيطلع error قبل ما تشغل التطبيق.
- **إعادة استخدام الكود**: لو عندك logic مشترك بين providers، تقدر تستخدم generic provider وتغير بس الـtype parameter حسب الحاجة.
- **تكامل مع code generation**: الميزة دي خاصة بالـcode generation (`@Riverpod`)، فRiverpod هيولد الـprovider تلقائيًا ويدعم الـgenerics بدون مشاكل.

### ملاحظات

الميزة دي خاصة بالـcode generation بس، يعني لازم تستخدم `@Riverpod`  او نوتيشن الـcode generation عشان تستفيد منها. لو بتعرف providers يدوي (بدون generation)، الـgenerics مش هتكون مدعومة بهذه الطريقة.

الدوكيومنتيشن مش بتقدم أمثلة معقدة، لكن الفكرة بسيطة: الـtype parameters بتتعامل زي الـparameters العادية، وبتديك type safety قوي.

الميزة دي مش experimental، يعني هي مستقرة وجاهزة للاستخدام.

لو عايز تستخدم generics في classes بدل functions، تقدر تعمل زي كده (بناءً على المنطق نفسه):

```dart
@Riverpod(keepAlive: true)
class MyGeneric<T extends num> extends _$MyGeneric {
  @override
  T build(T initialValue) {
    return initialValue;
  }
}

```

ثم `ref.watch(myGenericProvider<int>(5))`.
