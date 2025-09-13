### Offline persistence (experimental)
### الوصف

ميزة جديدة في Riverpod 3.0 اللي بتسمح بحفظ حالة الـprovider محليًا على الجهاز (caching a provider locally on the device). بعد كده، لما التطبيق يتقفل ويفتح تاني، الـprovider ده يقدر يرجع من الكاش (restored from the cache). ده يعني إن بياناتك هتفضل موجودة حتى لو الجهاز أوفلاين أو التطبيق مقفول، زي حفظ الـtodos في تطبيق مهام مثلاً.

 الميزة دي تجريبية ومش مستقرة لسة (experimental and not yet stable). هي قابلة للاستخدام (usable)، لكن الـAPI ممكن يتغير ب (breaking ways) بدون زيادة في الإصدار الرئيسي (major version bump). عشان كده، لو هنستخدمها نخلي بالنا ونتابع التحديثات باستمرار.

### كيفية الاستخدام والدعم

الميزة دي opt-in يعني مش تلقائية لازم تشغلها يدويًا في الـprovider. وهي مدعومة في كل الـproviders اللي من نوع "Notifier" سواء كنت بتستخدم code generation أو لا. Riverpod مش بيوفر قاعدة بيانات جاهزة هو بس بيوفر interfaces عشان تتفاعل مع قاعدة بيانات (interfaces to interact with a database). يعني تقدر تستخدم أي database عايزها طالما بتنفذ الـinterfaces دي.

في حزمة رسمية لـSQLite: [riverpod_sqflite](https://pub.dev/packages/riverpod_sqflite). ده مثال على database مدعومة رسميًا، وتقدر تستخدمها عشان تخزن البيانات في ملف SQLite على الجهاز.
تنويه المكتبة لسة مبتدعمش كل المنصات لحد اللحظة اللى اتكتب فيها المنشور 

### لمثال الكود والتفسير (الديمو)

الدوكيومنتيشن بيقدم ديمو قصير عشان تفهم إزاي تستخدم Offline persistence. المثال ده بيستخدم JsonSqFliteStorage بال code generation وبدونه. هنكتب الكود بعد كده نفسر خطوة خطوة.

### With code generation

```dart
@riverpod
Future<JsonSqFliteStorage> storage(Ref ref) async {
  // Initialize SQFlite. We should share the Storage instance between providers.
  return JsonSqFliteStorage.open(
    join(await getDatabasesPath(), 'riverpod.db'),
  );
}

/// A serializable Todo class. We're using Freezed for simple serialization.
@freezed
abstract class Todo with _$Todo {
  const factory Todo({
    required int id,
    required String description,
    required bool completed,
  }) = _Todo;

  factory Todo.fromJson(Map<String, dynamic> json) => _$TodoFromJson(json);
}

@riverpod
@JsonPersist()
class TodosNotifier extends _$TodosNotifier {
  @override
  FutureOr<List<Todo>> build() async {
    // We call persist at the start of our 'build' method.
    // This will:
    // - Read the DB and update the state with the persisted value the first
    //   time this method executes.
    // - Listen to changes on this provider and write those changes to the DB.
    persist(
      // We pass our JsonSqFliteStorage instance. No need to "await" the Future.
      // Riverpod will take care of that.
      ref.watch(storageProvider.future),
      // By default, state is cached offline only for 2 days.
      // We can optionally uncomment the following line to change cache duration.
      // options: const StorageOptions(cacheTime: StorageCacheTime.unsafe_forever),
    );

    // We asynchronously fetch todos from the server.
    // During the await, the persisted todo list will be available.
    // After the network request completes, the server state will take precedence
    // over the persisted state.
    final todos = await fetchTodos();
    return todos;
  }

  Future<void> add(Todo todo) async {
    // When modifying the state, no need for any extra logic to persist the change.
    // Riverpod will automatically cache the new state and write it to the DB.
    state = AsyncData([...await future, todo]);
  }
}
```

### without code generation

```dart
// A example showcasing JsonSqFliteStorage without code generation.
final storageProvider = FutureProvider<JsonSqFliteStorage>((ref) async {
  // Initialize SQFlite. We should share the Storage instance between providers.
  return JsonSqFliteStorage.open(
    join(await getDatabasesPath(), 'riverpod.db'),
  );
});

/// A serializable Todo class.
class Todo {
  const Todo({
    required this.id,
    required this.description,
    required this.completed,
  });

  Todo.fromJson(Map<String, dynamic> json)
      : id = json['id'] as int,
        description = json['description'] as String,
        completed = json['completed'] as bool;

  final int id;
  final String description;
  final bool completed;

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'description': description,
      'completed': completed,
    };
  }
}

final todosProvider =
    AsyncNotifierProvider<TodosNotifier, List<Todo>>(TodosNotifier.new);

class TodosNotifier extends AsyncNotifier<List<Todo>> {
    FutureOr<List<Todo>> build() async {
    // We call persist at the start of our 'build' method.
    // This will:
    // - Read the DB and update the state with the persisted value the first
    //   time this method executes.
    // - Listen to changes on this provider and write those changes to the DB.
    persist(
      // We pass our JsonSqFliteStorage instance. No need to "await" the Future.
      // Riverpod will take care of that.
      ref.watch(storageProvider.future),
      // A unique key for this state.
      // No other provider should use the same key.
      key: 'todos',
      // By default, state is cached offline only for 2 days.
      // We can optionally uncomment the following line to change cache duration.
      // options: const StorageOptions(cacheTime: StorageCacheTime.unsafe_forever),
      encode: jsonEncode,
      decode: (json) {
        final decoded = jsonDecode(json) as List;
        return decoded
            .map((e) => Todo.fromJson(e as Map<String, Object?>))
            .toList();
      },
    );

      // We asynchronously fetch todos from the server.
      // During the await, the persisted todo list will be available.
      // After the network request completes, the server state will take precedence
      // over the persisted state.
      final todos = await fetchTodos();
      return todos;
  }

  Future<void> add(Todo todo) async {
    // When modifying the state, no need for any extra logic to persist the change.
    // Riverpod will automatically cache the new state and write it to the DB.
    state = AsyncData([...await future, todo]);
  }
}

```

### تفسير المثال :

هنوضح الموضوع بدون جينريشن عشان دة اللى هيخلينا نفهمه فى كل الحالات

1. **إنشاء الـstorageProvider**: ده FutureProvider بيبدأ JsonSqFliteStorage. هو بيفتح قاعدة بيانات SQLite في ملف اسمه 'riverpod.db' في مسار الـdatabases على الجهاز. النقطة المهمة: لازم تشارك الـStorage instance بين الـproviders المختلفة عشان الكفاءة.
2. **كلاس Todo**: ده كلاس بسيط serializable، يعني يقدر يتحول لـJSON ويرجع منه. في constructor عادي وميثودز fromJson عشان يقرأ من JSON، وtoJson عشان يحول ل JSON. ده ضروري عشان البيانات تتحفظ في الـDB كـJSON.
3. **الـtodosProvider**: ده AsyncNotifierProvider بيربط بكلاس TodosNotifier، واللي بيحمل List<Todo> كـstate.
4. **داخل TodosNotifier - الـbuild method**:
    - في البداية، بننادي persist() في أول الـbuild. ده هيعمل:
        - يقرأ الـDB ويحدث الـstate بالقيمة المحفوظة أول مرة الـmethod ده يتنفذ.
        - يستمع للتغييرات في الـprovider ده ويكتبها في الـDB تلقائيًا.
    - بنمرر الـstorage من ref.watch(storageProvider.future) (مش لازم await لان Riverpod هتهندل دة).
    - ال key اللى شايل قيمة todos – ده مفتاح فريد للـstate ده، مفروض مفيش provider تاني يستخدم نفس المفتاح.
    - ال options: افتراضيًا، الـstate بيتحفظ أوفلاين لمدة يومين بس (cached offline only for 2 days). تقدر تغير ده بـStorageOptions، زي cacheTime: StorageCacheTime.unsafe_forever عشان يحفظ إلى الأبد (لكن ده غير آمن).
    - ال encode: jsonEncode عشان تحول الـstate لـJSON.
    - ال decode: عشان تحول الـJSON تاني لـList<Todo> باستخدام fromJson.
5. **جلب البيانات من السيرفر**: بعد الـpersist، بنعمل fetchTodos() asynchronously. أثناء الانتظار، الـtodo list المحفوظة هتكون متاحة. لما الطلب يخلص، الـstate من السيرفر هياخد الأولوية على المحفوظ فى اللوكال داتا بيز.
6. **الـadd method**: لما تضيف todo جديد، مش لازم أي كود إضافي لحفظ التغيير. Riverpod هيحفظ الـstate الجديد تلقائيًا في الكاش والـDB.
