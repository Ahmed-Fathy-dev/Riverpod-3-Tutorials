
### Pause/Resume support
### الوصف

 يعني "دعم الإيقاف والاستئناف". في Riverpod 2.0 كان فيه بعض الدعم للـpause/resume، لكن كان محدود جدًا (fairly limited). في Riverpod 3.0، كل الـlisteners من `ref.listen` تقدر تتوقف وتستأنف يدويًا على حسب الحاجة (all `ref.listen` listeners can be manually paused/resumed on demand).
 
### الدعم اليدوي للـPause/Resume

 دلوقتى نقدر نوقف ونستأنف الـsubscription من `ref.listen` يدويًا.

المثال الكامل:

```dart
final subscription = ref.listen(
  todoListProvider,
  (previous, next) {
    // Do something with the new value
  },
);
subscription.pause();
subscription.resume();

```
  
 **الشرح**:
    - هنا `ref.listen`: بتعمل subscription لـprovider (هنا `todoListProvider`)، وبتحدد callback بيشتغل لما يحصل تغيير (من previous لـnext).
    - الـ`listen` بترجع `subscription`، وده كائن تقدر تستخدمه عشان تتحكم في الـlistener.
    - هنا `subscription.pause()`: بيوقف الـlistener مؤقتًا، يعني مش هيتلقى إشعارات (notifications) تانية لحد ما تستأنفه.
    - و `subscription.resume()`: بيرجع الـlistener للعمل، وهيرجع يتلقى الإشعارات.

### الـPause التلقائي في حالات معينة

بعد التحديث دة بتتوقف الـproviders تلقائيًا في حالات محددة (Riverpod now pauses providers in various situations):

- **مثلا When a provider is no-longer visible, it is paused (Based off TickerMode)**: يعني، لو الـprovider مش مرئي (no-longer visible)، هيتوقف (paused). ده بناءً على `TickerMode` من Flutter يعني لو الـwidget اللي بيستخدم الـprovider مش نشط (زي لو الـapp في الخلفية أو الـwidget مخفي)، الـprovider هيتوقف لتوفير الموارد.
- **او When a provider rebuilds, its subscriptions are paused until the rebuild completes**: يعني، لما الـprovider يعمل rebuild (يعيد بناء الـstate)، الـsubscriptions (الـlisteners) هتتوقف مؤقتًا لحد ما الـrebuild يخلّص.
- **برده لو When a provider is paused, all of its subscriptions are paused too**: يعني، لو الـprovider اتوقف، كل الـsubscriptions بتاعته هتتوقف كمان.

التحديث دة لازم نركز اوى على قسم تغييرات (life-cycle changes) فى ريفربود عشان فى تفاصيل كتير اتغيرت لازم نفهمها ونراجع عليها وباذن الله هتلاقو شرح شامل للجزئية دى فى السكشن بتاعها

### الفوايد والملاحظات

- **الفوايد**: الدعم اليدوي بيديك تحكم دقيق في الـlisteners، والتلقائي بيحسن الأداء عن طريق توفير الموارد (زي وقف الـanimations أو الـstreams لما مش مطلوبة).
- **ملاحظات**: التغييرات دي ممكن تكون(breaking changes) لو كنت بتعتمد على سلوك Riverpod 2.0، فشوف الـmigration guide (https://riverpod.dev/docs/3.0_migration) عشان تعرف إزاي تتكيف. الميزة دي تعمل مع كل أنواع الـproviders، سواء code generation أو يدوي.
