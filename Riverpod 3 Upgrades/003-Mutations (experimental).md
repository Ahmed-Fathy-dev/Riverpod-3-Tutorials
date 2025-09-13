### الجزء الأول: الوصف العام والتحذير (Info)

ميزة جديدة في Riverpod 3.0، وهي بتضيف مفهوم اسمه "mutations" عشان تحل مشاكل معينة في التعامل مع الـside-effects.

 الميزة دي تجريبية ومش مستقرة تمامًا (experimental and not yet stable). هي قابلة للاستخدام (usable)، لكن الـAPI ممكن يتغير ب (breaking ways) بدون زيادة في الإصدار الرئيسي (major version bump). عشان كده، لو هنستخدمها نخلي بالنا ونتابع التحديثات باستمرار. زي الـOffline persistence.

### المشاكل اللي بتحلها الميزة

الدوكيومنتيشن بيقول إن الميزة دي بتحل مشكلتين رئيسيتين:

1. **تمكين الـUI من الرد على الـside-effects**: زي لو بنتعامل مع فورم  (form submissions) وبنبعت الداتا بتاعتها ل API ، ضغط الأزرار (button clicks) وغيرها من الحالات المشابهة. ده يسمح للـUI إنه يعرض رسائل تحميل/نجاح/خطأ (enable it to show loading/success/error messages). مثال: "Show a toast when a form is submitted successfully"، يعني عرض توست لما النموذج يتبعت بنجاح. بدل ما الـUI يكون سلبي دلوقتي يقدر يتفاعل مع الـside-effects بشكل ذكي من غير منحتاج نعمل دة داخل البروفايدر او باستخدام listener كعادة ريفربود بتسهل عليك كل العمليات وبتخليك تركز فى اللوجيك اللى بتشتغل عليه.
2. **حل مشكلة الـdisposal أثناء الـside-effect**: ده يتعلق بـonPressed callbacks اللي بتستخدم Ref.read مع Automatic disposal. المشكلة إن الـproviders ممكن تdispose و الـside-effect لسه شغال (could cause providers to be disposed while a side-effect is still in progress). الميزة دي بتحافظ على الـproviders حية لحد ما الـside-effect يخلص.

### كيفية الاستخدام

في أوبجكت جديد اسمه Mutation، وده بيتعلن عنه كـtop-level final variable، زي الـproviders. مثال بسيط:

```dart
final addTodoMutation = Mutation<void>();

```

يعني، ده Mutation من نوع void (مش بيرجع قيمة).

بعد كده، الـUI تقدر تستخدم ref.listen أو ref.watch عشان تستمع لحالة الـmutations دي.  يعني إن الـwidget يقدر يتغير بناءً على حالة الـside-effect، زي عرض زر submit أو spinner أو رسالة خطأ.
طبعا انا مش محتاج اقول الفيتشر دى مريحة ازاى لان كل دة كنا بنعملو احنا وكان بيسبب مشاكل ساعات بسبب ال provider lifecycle

- الـ`Mutation<void>` بيعني إن الـside-effect مش بيرجع قيمة . لو كنت عايز الـside-effect يرجع قيمة (مثلاً `int` أو `String`)، كنت هتعرفها زي `Mutation<int>` أو `Mutation<String>`.
- الـ`Mutation` ده زي كائن بيدير حالة الـside-effect (زي إضافة todo جديد)، ويسمح للـUI إنها تتفاعل مع الحالات المختلفة للـside-effect (idle, pending, success, error).

### مثال بالكود والتفسير

الدوكيومنتيشن بيقدم مثال كود لتوضيح إزاي نستخدم الـMutations. المثال ده بيظهر إزاي ننشئ Mutation لإضافة todo في تطبيق مهام، مع دعم عرض حالة التحميل (loading state) في الـUI. هقدملكم الكود كامل  وبعد كده نفسر كل جزء.

### 2. الـUI بتاع الـWidget

**الكود**:

```dart
class AddTodoButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final addTodo = ref.watch(addTodoMutation);
    return switch (addTodo) {
      ...
    };
  }
}

```

**الشرح**:

- الـ`AddTodoButton` هو `ConsumerWidget`، يعني widget بيقدر يتفاعل مع Riverpod باستخدام `WidgetRef ref`.
- `ref.watch(addTodoMutation)` بيستمع لحالة الـ`addTodoMutation`. الـ`addTodo` هيبقى واحد من أربع حالات:
    - حالة `MutationIdle`: مافيش side-effect شغال.
    - حالة `MutationPending`: الـside-effect شغال دلوقتي.
    - حالة `MutationError`: الـside-effect فشل.
    - حالة `MutationSuccess`: الـside-effect نجح.
- بنستخدم `switch` (من Dart 3) عشان نعرض واجهة مختلفة بناءً على الحالة.

طبعا كمية ال Customization اللى نقدر نعملها عشان التكرار لا حصر لها وكل واحد ليه طريقته زى مكتير مننا بيعمل مع ال asyncValue.when 

### 3. الحالة الأولى: MutationIdle

**الكود**:

```dart
MutationIdle() => ElevatedButton(
      onPressed: () {
        addTodoMutation.run(ref, (tsx) async {
          await tsx.get(todoListProvider.notifier).addTodo('New Todo');
        });
      },
      child: const Text('Submit'),
    ),

```

**الشرح**:

- لو الـ`addTodoMutation` في حالة `MutationIdle` (يعني مافيش side-effect شغال)، بنعرض زر `ElevatedButton` مكتوب عليه "Submit".
- لما الـuser يضغط على الزر (`onPressed`)، بنستدعي `addTodoMutation.run`، ودي الدالة اللي بتبدأ الـside-effect.
- الـ`run` بتاخد:
    - ال `ref`: الـ`WidgetRef` بتاع الـwidget.
    - ال Callback من نوع `(tsx) async { ... }`: الـ`tsx` هنا هو كائن زي `Ref`، بس مخصص للـmutations، وبيستخدم `tsx.get` بدل `Ref.read`.
- جوا الـcallback، بنستخدم `tsx.get(todoListProvider.notifier)` عشان نجيب الـ`Notifier` بتاع `todoListProvider`، وبنستدعي الدالة `addTodo('New Todo')` عليه.
- **ملاحظة مهمة** (من الدوكيومنتيشن): استخدمنا `tsx.get` بدل `Ref.read`. الفرق إن `tsx.get` بيحافظ على الـprovider حي (alive) لحد ما الـmutation تخلّص، عشان نتجنب مشكلة الـdisposal أثناء الـside-effect.

### 4. الحالة التانية: MutationPending

**الكود**:

```dart
MutationPending() => const CircularProgressIndicator(),

```

**الشرح**:

- لو الـ`addTodoMutation` في حالة `MutationPending` (يعني الـside-effect شغال)، بنعرض `CircularProgressIndicator` عشان نوضح إن فيه عملية تحميل.
- دي حالة مؤقتة بتحصل لما الـ`addTodoMutation.run` بيشتغل وإحنا بنستنى الـ`await` (زي جلب بيانات من الـserver).

### 5. الحالة التالتة: MutationError

**الكود**:

```dart
MutationError() => ElevatedButton(
      onPressed: () {
        addTodoMutation.run(ref, (tsx) async {
          await tsx.get(todoListProvider.notifier).addTodo('New Todo');
        });
      },
      child: const Text('Retry'),
    ),

```

**الشرح**:

- لو الـ`addTodoMutation` في حالة `MutationError` (يعني الـside-effect فشل، زي إن الاتصال عمل تايم اوت او الـserver فيه مشكلة)، بنعرض زر `ElevatedButton` مكتوب عليه مثلا "Retry".
- لما الـuser يضغط على الزر، بنستدعي `addTodoMutation.run` تاني بنفس المنطق بتاع الحالة الأولى، عشان نعيد المحاولة.
- زي الحالة الأولى، بنستخدم `tsx.get` عشان نضمن إن الـprovider ما يتdisposeش أثناء المحاولة.

### 6. الحالة الرابعة: MutationSuccess

**الكود**:

```dart
MutationSuccess() => const Text('Todo added!'),

```

**الشرح**:

- لو الـ`addTodoMutation` في حالة `MutationSuccess` (يعني الـside-effect نجح، والـtodo اتضاف بنجاح)، بنعرض نص "Todo added!".
- دي رسالة تأكيد للـuser إن العملية خلّصت بنجاح.

---

### الفوايد والملاحظات

- **الفوايد**:
    - **تفاعل الـUI مع الـside-effects**: الـ`Mutation` بيخلّي الـUI يقدر يتفاعل مع حالات الـside-effect (idle, pending, error, success)، وده بيحسن تجربة المستخدم (زي إظهار spinner أو رسالة نجاح).
    - **حل مشكلة الـdisposal**: باستخدام `tsx.get` بدل `Ref.read`، الـ`Mutation` بيضمن إن الـprovider يفضل حي لحد ما الـside-effect يخلّص، وده بيحل مشكلة الـdisposal المبكر.
    - **مرونة**: تقدر تستخدم الـ`Mutation` مع أي نوع side-effect (مش بس إضافة todos)، زي إرسال نموذج أو تحديث بيانات.
- **ملاحظات**:
    - الميزة دي **تجريبية** (experimental)، يعني ممكن الـAPI يتغير في المستقبل، فاستخدمها بحذر.
    - الدوكيومنتيشن بتشدد على إنك تستخدم `tsx.get` بدل `Ref.read` عشان تضمن إن الـprovider ما يتعملوش dispose أثناء الـside-effect.
    - لو عايز تتأكد إن الكود بتاعك هيفضل شغّال، تابع التحديثات في الـmigration guide (https://riverpod.dev/docs/3.0_migration).
