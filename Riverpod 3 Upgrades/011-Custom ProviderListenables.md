### Custom ProviderListenables

### الوصف

في Riverpod 3.0 بقى ممكن تعمل **ProviderListenables** مخصصة.

الطريقة دي بتديك مرونة تبني الـ logic الخاص بيك بدل ما تلتزم بالـ API الجاهز.

الـ mixin اللي بيسهّل الموضوع هو:

`SyncProviderTransformerMixin`.

---

### مثال

```dart
final class Where<T> with SyncProviderTransformerMixin<T, T> {
  Where(this.source, this.where);
  @override
  final ProviderListenable<T> source;
  final bool Function(T previous, T value) where;

  @override
  ProviderTransformer<T, T> transform(
    ProviderTransformerContext<T, T> context,
  ) {
     return ProviderTransformer(
       initState: (_) => context.sourceState.requireValue,
       listener: (self, previous, next) {
         if (where(previous, next))
           self.state = next;
       },
     );
  }
}

extension<T> on ProviderListenable<T> {
  ProviderListenable<T> where(
    bool Function(T previous, T value) where,
  ) => Where<T>(this, where);
}

```

---

### الشرح

### 1. الكلاس `Where<T>`

- `final class Where<T>`: كلاس جنريك (generic) بيشتغل على أي نوع بيانات `T`.
- `with SyncProviderTransformerMixin<T, T>`: ده اللي بيخلي الكلاس يقدر يحوّل **ProviderListenable** موجود لنسخة جديدة مخصصة.
- `final ProviderListenable<T> source;`
    
    ده المصدر الأصلي اللي بنسمع (listen) منه.
    
- `final bool Function(T previous, T value) where;`
    
    دالة شرطية بتاخد القيمة القديمة (`previous`) والجديدة (`value`)، وتقرر هل التحديث يمرّ ولا لأ.
    
- `transform(...)`: هنا بنحدد إزاي الـ ProviderListenable الجديد هيشتغل.
    - `initState: (_) => context.sourceState.requireValue`
        
        بيحدد الحالة المبدئية (initial state) بالاعتماد على قيمة المصدر.
        
    - `listener: (self, previous, next)`
        
        بيتنادى كل مرة فيه قيمة جديدة جاية من `source`.
        
        - لو الدالة `where` رجعت `true`، بنحدث الحالة (`self.state = next`).
        - لو رجعت `false`، نتجاهل التحديث.

---

### 2. الـ extension

```dart
extension<T> on ProviderListenable<T> {
  ProviderListenable<T> where(
    bool Function(T previous, T value) where,
  ) => Where<T>(this, where);
}

```

- دي بتمكّنك تستخدم `.where(...)` مباشرة على أي `ProviderListenable`.
- يعني بدل ما تكتب كود مطوّل، تقدر تستعملها بنفس سهولة `.select` أو `.map`.

---

### كيفية الاستخدام

```dart
ref.watch(provider.where((previous, value) => value > 0));

```

- هنا بنقول:
    - اسمع (watch) على `provider`.
    - بس متحدثش القيمة غير لو القيمة الجديدة `> 0`.
- النتيجة: هتتجنّب رندر/تحديثات غير لازمة لو القيمة مش مستوفية الشرط.

---
