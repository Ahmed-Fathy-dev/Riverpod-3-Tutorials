### أخيرا وبعد طول انتظار تحديث Riverpod 3 

فى الفايل دة هيكون في مقدمة للتحديث و keynotes لأهم التغييرات اللى تمت 
والشرح الشامل بالتفاصيل هيكون فى ملف خاص بكل ميزة باذن الله

فى البداية لازم نأكد ان المحتوى دة هو شرح من الدوكمنتيشن الجديدة واللى برشح جدا انكم تبصو عليها برده طبعا هى بقت منظمة كتير عن الفيرجن اللى فات وبتضم كل حاجة تخص المكتبة 
رابط التحديثات الجديدة 
 [What's new in Riverpod 3.0](https://riverpod.dev/docs/whats_new)

الدوكيومنتيشن بيؤكد إن الإصدار ده مش بس تحديث عادي، ده بيحتوي على ميزات كتيرة كانت متأخرة (long-due features)، إصلاحات للأخطاء (bug fixes)، وتبسيطات في الـAPI (simplifications of the API). يعني، الفريق ورا Riverpod عمل شغل كبير عشان يخلي الإطار أفضل وأسهل في الاستخدام.

النقطة المهمة هنا: الإصدار ده مش نهاية الطريق، ده "transition period"، يعني فترة انتقالية نحو Riverpod أبسط وموحد أكتر (toward a simpler, unified Riverpod). يعني، Riverpod 3.0 بيمهد الطريق لإصدارات مستقبلية هتكون أكتر تماسكًا، وده يعني إن المطورين هيستفيدوا من تغييرات تساعد في كتابة كود أنظف وأقوى.

### تنويهات مهمة

 الـcaution اللي موجودة في الدوكيومنتيشن 

"This version contains a few life-cycle changes. Those could break your app in subtle ways. Upgrade carefully."

 يعني، الإصدار ده بيحتوي على بعض التغييرات في دورة حياة الـproviders (life-cycle changes)، ودي ممكن تعمل مشاكل لو حدثنا علطول. عشان كده نصيحة  Upgrade carefully  يعني لما نيجي نحدث لازم نعمل دة بحذر لو عندك تطبيق موجود بيستخدم Riverpod إصدار أقدم، مفروض تعمل التحديث دة بوعى ان هيحصل مشاكل ولازم تعمل دة على مستوي بسيط وخطوة خطوة مش مرة واحدة على مستوى التطبيق كله، عشان التغييرات دي ممكن تؤثر على سلوك الـproviders أو الـlisteners زي مهنتكلم في الشرح اللى جاي، والمشاكل ممكن تكون لوجيكال ايرور مش كومبايل او رن تايم.

### الميزات التجريبية

كل ما هو داخل تنويه ميزة تجريبية يعني الميزة دي تجريبية ومش مستقرة تمامًا (experimental and not yet stable). هي قابلة للاستخدام (usable)، لكن الـAPI ممكن يتغير ب (breaking ways) بدون زيادة في الإصدار الرئيسي (major version bump) عشان كده لو هتستخدمها حاول تاخد بالك وتتابع عشان ممكن تحتاج تعديلات في المستقبل.

### ال Migration Guide

دة رابط [migration page](https://www.notion.so/docs/3.0_migration)
مهم نقراه لو عايز تعرف إزاي تحول كودك من إصدار قديم لـRiverpod 3.0 بدون مشاكل ده دليل مفصل هيساعدك في التعامل مع التغييرات، زي إزالة بعض الـAPIs أو تغيير طريقة الاستخدام.


### **ال keynotes**  لتحديثات Riverpod 3

---

### [**Offline persistence (experimental)**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/002-Offline%20persistence%20(experimental).md)

ميزة تجريبية جديدة فى ريفربود 3 وهى انك دلوقتى تقدر تحفظ حالة البروفايدر لوكال علي الجهاز  

لما التطبيق يتقفل ويفتح تاني الـprovider ده يقدر يسترجع اخر حالة مخزنة ليه 
لو انت كنت من مستخدمين مكتبة بلوك فاكيد تعرف hydrated bloc نفس الفكرة بس الفرق ان بلوك بتعتمد على hive فى التخزين اما ريفربود فى مكتبة رسمية لـSQLite [riverpod_sqflite](https://pub.dev/packages/riverpod_sqflite) database مدعومة رسميًا تقدر تستخدمها عشان تخزن البيانات في ملف SQLite على الجهاز

---

### [**Mutations (experimental)**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/003-Mutations%20(experimental).md)

ميزة تجريبية جديدة فى ريفربود 3 وهي بتضيف مفهوم اسمه "mutations" عشان تحل مشكلتين رئيسيتين 
* **تمكين الـUI من الرد على الـside-effects** زي (form submissions) - ضغط الأزرار (button clicks) والحالات المشابهة. 
* **حل مشكلة الـdisposal أثناء الـside-effect** ده يتعلق بـonPressed callbacks اللي بتستخدم Ref.read مع Automatic disposal.

---

### [**Automatic retry**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/004-Automatic%20retry.md)

ميزة جديدة فى ريفربود 3 وهى ان تم إضافة parameter اسمها `retry` في الـ`build` method بتاعة الـAsyncNotifier (أو أي provider asynchronous تاني)
الـ`retry` كـparameter. لو عندنا `fetchData` من API مثلا فشلت (زي مشكلة شبكة أو server error) Riverpod هيحاول يعيد تنفيذها تلقائيًا بناءً على الـRetryPolicy

---

### [**Ref.mounted**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/005-Ref.mounted.md)

ميزة جديدة فى ريفربود 3 الفكرة منها إنك تقدر تشيك إذا كان الـprovider لسه موجود (mounted) ولا لأ زى `context.mounted`  الموجودة فى اطار العمل بس الفرق انك هنا بتتشيك على البروفايدر نفسه 
****


### [**Generic support (code-generation)**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/006-Generic%20support%20(code-generation).md)

**تحسينات فى** code-generation اخيرا وبعد طول انتظار دعم رسمي لل **Generic  فى code-generation** ده هيخلي الكود أكتر مرونة وأمان من ناحية الـtype safety

---

### [**Pause/Resume support**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/007-Pause-Resume%20support.md)

يعني دعم الإيقاف والاستئناف الميزة دي بتتيح للـproviders إنها تتوقف مؤقتًا (pause) وتستأنف (resume) بناءً على حالة الـlisteners بتوعها

`ref.pause()`: بيوقف الـprovider مؤقتًا، يعني هيوقف أي عمليات زي تحديثات الـstate أو استدعاء الـbuild method

`ref.resume()`: بيرجع الـprovider للحالة النشطة، فيرجع يشتغل زي الأول ويستدعي الـbuild method لو احتاج.

---

### [**Unification of the Public APIs**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/008-Unification%20of%20the%20Public%20APIs.md)

- StateProvider/StateNotifierProvider and ChangeNotifierProvider are discouraged and moved to a different import.
دلوقتي بقوا غير مستحب استخدامهم (discouraged)، وتم نقلهم من المكتبة الرئيسية `flutter_riverpod` إلى مكتبة فرعية اسمها `flutter_riverpod/legacy`
- AutoDispose interfaces are removed.
ـinterfaces الخاصة بالـ`AutoDispose` (زي `AutoDisposeProvider`, `AutoDisposeStateNotifierProvider`  إلخ) تم إزالتها من الـAPI العام لكن الميزة نفسها موجودة لسة
دلوقتي، الـ`AutoDispose` بقى جزء مدمج في الـproviders العادية. يعني، بدل ما تستخدم `AutoDisposeProvider`، بتستخدم `Provider` مع modifier
- "FamilyNotifier" and "Notifier" are fused.
الـ`FamilyNotifier` والـ`Notifier` اتدمجوا مع بعض في Riverpod 3.0. في الإصدارات القديمة، لو كنت عايز تستخدم provider بياخد parameter (زي `FamilyProvider` أو `FamilyNotifierProvider`)، كنت بتحتاج تعرف `FamilyNotifier` منفصل. دلوقتي، الـ`Notifier` نفسه بقى بيدعم الـparameters، فمفيش داعي للـ`FamilyNotifier` ككيان منفصل.
- One Ref to rule them all.
 يعني، "Ref واحد يتحكم في الكل". الفكرة هنا إن Riverpod 3.0 بيوحد طريقة استخدام الـ`Ref` عبر كل أنواع الـproviders. في الإصدارات القديمة، كان فيه أنواع مختلفة من `Ref` (زي `ProviderRef`, `StateNotifierProviderRef`, إلخ) حسب نوع الـprovider. دلوقتي، كل الـproviders بتستخدم نفس الـ`Ref` interface، وده بيبسط الكود جدًا.
- All updateShouldNotify now use ==.
يعني، "كل الـupdateShouldNotify دلوقتي بتستخدم العامل ==." ده يعني إن طريقة تحديد إذا كان الـprovider محتاج يبلغ الـlisteners بتغيير في الـstate بقت تعتمد على مقارنة الـ`==` بدل الطرق القديمة

---

### [**Provider life-cycle changes**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/009-Provider%20life-cycle%20changes.md)

تغييرات في دورة حياة الـproviders

- When reading a provider results in an exception, the error is now wrapped in a ProviderException.
يعني، لما قراءة الـprovider (زي `ref.read` أو `ref.watch`) بتسبب exception، الخطأ دلوقتي بيتغلف (wrapped) في كائن من نوع `ProviderException`.
- Listeners inside widgets that are not visible are now paused.
يعني، "الـlisteners اللي جوا widgets مش ظاهرة دلوقتي بتتوقف مؤقتًا (paused).
في Riverpod 3.0، لو الـwidget مش ظاهر (not visible)، الـlisteners اللي جواه بتتوقف مؤقتًا (paused). يعني، الـprovider مش هيبعت إشعارات (notifications) للـwidget ده لحد ما يرجع يظهر تاني.
- If a provider is only used by paused providers, it is paused too.
عني، "لو provider مستخدم بس من providers متوقفة مؤقتًا (paused)، هيتوقف مؤقتًا برضه." دي تغييرة في سلوك الـlifecycle بتاع الـproviders في Riverpod 3.0، وبتهدف إنها توفر موارد التطبيق بشكل أكبر.
- When a provider rebuilds, its previous subscriptions now are kept until the rebuild completes.
يعني، "لما provider يعمل rebuild، الـsubscriptions القديمة بتاعتو بتفضل محفوظة لحد ما الـrebuild يكتمل." دي تغييرة في سلوك الـlifecycle بتاع الـproviders في Riverpod 3.0، وبتهدف إنها تحافظ على استقرار الـlisteners أثناء تحديث الـprovider.
- Exceptions in providers are rethrown as a ProviderException.
يعني، "الـexceptions اللي بتحصل في الـproviders دلوقتي بتترمي تاني (rethrown) كـProviderException." دي تغييرة في إزاي Riverpod 3.0 بيتعامل مع الأخطاء اللي بتحصل جوا الـproviders.

---

### [**New testing utilities**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/010-New%20testing%20utilities.md)

- ProviderContainer.test.
دي extension method على الـ`ProviderContainer`، بتخليك تنشئ container مؤقت للاختبارات مع إعدادات مخصصة. الـ`ProviderContainer` هو الكائن اللي بيدير الـproviders في Riverpod، وعادةً بيُستخدم في الـunit tests.
- NotifierProvider.overrideWithBuild.
دي method بتسمحلك تعمل override للـ`NotifierProvider` في الاختبارات، بس بدل ما تعمل override بكائن `Notifier` كامل، بتعمل override بس للـ`build` method.
- Future/StreamProvider.overrideWithValue.
دي method بتسمحلك تعمل override للـ`FutureProvider` أو `StreamProvider` في الاختبارات، بحيث إنك تحدد قيمة مباشرة (value) بدل ما تستخدم `Future` أو `Stream` حقيقي.
- WidgetTester.container.
دي extension method على `WidgetTester` (الكائن اللي بتستخدمه في Flutter للـwidget tests)، بتخليك تنشئ `ProviderContainer` مؤقت للاختبارات اللي بتتضمن widgets.

---

### [**Custom ProviderListenables**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/011-Custom%20ProviderListenables.md)

دي طريقة تخليك تعرف كائنات تقدر تُستمع ليها (listened to) زي الـproviders العادية، بس بتتحكم فيها بنفسك. الـ`ProviderListenable` هو interface يسمح لأي كائن إنه يبقى قابل للاستماع من الـ`ref.listen` أو `ref.watch` في Riverpod.

---

### [**Statically safe scoping (code-generation only)**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/012-Statically%20safe%20scoping%20(code-generation%20only).md)

ال scoping آمن statically في الـ code generation.
دعم طريقة جديدة للـscoping بتضمن الـtype safety في وقت الكومبايل (compile-time) لما تستخدم الـcode generation. الـscoping هنا بيعني إنك تحدد نطاق الـprovider (يعني إزاي وفين يبقى متاح) بطريقة آمنة وواضحة.

---

### [**Other changes**](https://github.com/Ahmed-Fathy-dev/Riverpod-3-Tutorials/blob/main/Riverpod%203%20Upgrades/013-Other%20changes.md)

- Ref.invalidate now returns a Future
"الـ`ref.invalidate` دلوقتي بيرجع `Future<void>`، وده بيعني إن عملية الـinvalidation بقت asynchronous، وبتقدر تنتظر اكتمالها باستخدام `await`.
دي تغييرة في سلوك الـ`ref.invalidate`، وهي method بتستخدم عشان تعيد تهيئة (reset) الـprovider لحالته الأولية.
- Watching a provider during a rebuild no longer throw
الـ`watch` لprovider أثناء الـrebuild ماعدش بيرمي exception." دي تحسينة في سلوك الـ`ref.watch` لما بيحصل rebuild للـprovider.
بقى بيتعامل مع الحالة دي بشكل أفضل، والـ`ref.watch` ماعدش بيرمي exception لو حصل أثناء الـrebuild. بدل كده، Riverpod بيضمن إن الـdependency يتم التعامل معاها بشكل آمن.
- StateNotifier now implements Listenable
"الـ`StateNotifier` دلوقتي بيطبق الـ`Listenable` interface." دي تغييرة بتخلي الـ`StateNotifier` يدعم الاستماع للتغييرات بطريقة مباشرة زي أي كائن `Listenable` في Flutter.
- AsyncValue.when supports skipLoadingOnRefresh/skipError
يعني، "الـ`AsyncValue.when` دلوقتي بيدعم `skipLoadingOnRefresh` و`skipError`." دي تحسينة في طريقة التعامل مع الـ`AsyncValue` في الـUI، خصوصًا لما بتتعامل مع حالات الـloading والـerror.
- All Ref listeners now return a way to remove the listener.
الـ`ref` listeners (زي `ref.onDispose`, `ref.onCancel`, إلخ) دلوقتي بيرجعوا دالة تقدر تستخدمها عشان تلغي الـlistener.
- Weak listeners - listen to a provider without preventing auto-dispose.
في الإصدارات القديمة، لو عملت `ref.listen` على provider، الـprovider كان بيفضل "حي" (مش بيتعمل dispose) طالما الـlistener موجود.
في Riverpod 3.0 لما تستخدم `ref.listen`، تقدر تضيف `weak: true` عشان تخلي الـlistener ما يمنعش الـauto-dispose للـprovider.
