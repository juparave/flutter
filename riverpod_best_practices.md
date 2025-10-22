# Flutter Riverpod Best Practices Guide - 2025

## Table of Contents
1. [Latest Riverpod Patterns and Best Practices](#1-latest-riverpod-patterns-and-best-practices)
2. [Performance Optimization Techniques](#2-performance-optimization-techniques)
3. [Modern Flutter + Riverpod Architecture Patterns](#3-modern-flutter--riverpod-architecture-patterns)
4. [Common Anti-Patterns to Avoid](#4-common-anti-patterns-to-avoid)
5. [Migration Strategies](#5-migration-strategies)

---

## 1. Latest Riverpod Patterns and Best Practices

### 1.1 Provider Types - 2025 Recommendations

As of 2025, Riverpod focuses on **five core provider types** that cover all common use cases:

#### **Provider** - Read-only values
Use for immutable data or computed values that don't change.

```dart
// Code generation approach (RECOMMENDED)
@riverpod
String example(Ref ref) => 'Hello world';

// Non-codegen approach
final exampleProvider = Provider<String>((ref) => 'Hello world');
```

#### **NotifierProvider** - Synchronous mutable state
Use for simple synchronous state management with methods.

```dart
// Code generation approach (RECOMMENDED)
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state = max(0, state - 1);
}

// Non-codegen approach
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state = max(0, state - 1);
}

final counterProvider = NotifierProvider<Counter, int>(Counter.new);
```

#### **AsyncNotifierProvider** - Asynchronous mutable state
**This is the preferred approach for async state in 2025** - Use instead of FutureProvider/StreamProvider for better consistency and scalability.

```dart
// Code generation approach (RECOMMENDED)
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    // Fetch initial data
    final dio = Dio();
    final response = await dio.get('https://api.example.com/todos');
    return (response.data as List)
      .map((json) => Todo.fromJson(json))
      .toList();
  }

  // Mutations are easy with AsyncNotifier
  Future<void> addTodo(String title) async {
    // Set loading state
    state = const AsyncLoading();

    // Perform the mutation
    state = await AsyncValue.guard(() async {
      final dio = Dio();
      await dio.post('https://api.example.com/todos', data: {'title': title});

      // Refetch or update local state
      final response = await dio.get('https://api.example.com/todos');
      return (response.data as List)
        .map((json) => Todo.fromJson(json))
        .toList();
    });
  }

  // Update is a convenient helper for mutations
  Future<void> toggleTodo(String id) async {
    await update((todos) async {
      final todo = todos.firstWhere((t) => t.id == id);
      final updatedTodo = todo.copyWith(completed: !todo.completed);

      final dio = Dio();
      await dio.put('https://api.example.com/todos/$id',
        data: updatedTodo.toJson());

      return todos.map((t) => t.id == id ? updatedTodo : t).toList();
    });
  }
}

// Non-codegen approach
class TodoList extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    final dio = Dio();
    final response = await dio.get('https://api.example.com/todos');
    return (response.data as List)
      .map((json) => Todo.fromJson(json))
      .toList();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final dio = Dio();
      await dio.post('https://api.example.com/todos', data: {'title': title});
      final response = await dio.get('https://api.example.com/todos');
      return (response.data as List)
        .map((json) => Todo.fromJson(json))
        .toList();
    });
  }
}

final todoListProvider = AsyncNotifierProvider<TodoList, List<Todo>>(TodoList.new);
```

#### **FutureProvider** - Simple one-time async operations (LEGACY)
**Note**: While still supported, prefer AsyncNotifierProvider for better consistency in 2025.

```dart
// Only use for simple, one-time async operations with no mutations
@riverpod
Future<String> simpleData(Ref ref) async {
  final response = await http.get(Uri.parse('https://example.com'));
  return response.body;
}
```

#### **StreamProvider** - Reactive streams (LEGACY)
**Note**: While still supported, prefer AsyncNotifierProvider for better consistency in 2025.

```dart
// Only use when you truly need a stream
@riverpod
Stream<int> timer(Ref ref) {
  return Stream.periodic(const Duration(seconds: 1), (count) => count);
}
```

### 1.2 Code Generation vs Manual Provider Creation

**Code generation is STRONGLY RECOMMENDED in 2025** for all new projects.

#### Benefits of Code Generation:
- Less boilerplate
- Compile-time safety
- Better developer experience
- Automatic provider naming
- Family providers without explicit `.family` modifier
- Auto-dispose by default

#### Setup for Code Generation:

**pubspec.yaml:**
```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^2.4.0
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0
```

**Run the code generator:**
```bash
# One-time generation
dart run build_runner build --delete-conflicting-outputs

# Watch mode (RECOMMENDED during development)
dart run build_runner watch -d
```

**Example with code generation:**
```dart
// my_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'my_provider.g.dart';

@riverpod
int counter(Ref ref) => 0;

@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    return fetchTodos();
  }
}
```

### 1.3 AsyncNotifier vs FutureProvider vs StreamProvider

**2025 Best Practice: Prefer AsyncNotifier for consistency across your codebase.**

| Feature | AsyncNotifier | FutureProvider | StreamProvider |
|---------|---------------|----------------|----------------|
| **One-time data fetch** | ✅ Yes | ✅ Yes | ❌ No |
| **Mutations/Side effects** | ✅ Built-in | ❌ Not designed for this | ❌ Not designed for this |
| **Methods on notifier** | ✅ Yes | ❌ No | ❌ No |
| **State updates** | ✅ Easy with `state` setter | ❌ Difficult | ❌ Difficult |
| **Error handling** | ✅ Comprehensive | ⚠️ Basic | ⚠️ Basic |
| **Scalability** | ✅ Excellent | ⚠️ Limited | ⚠️ Limited |
| **2025 Recommendation** | ✅ **PREFERRED** | ⚠️ Use sparingly | ⚠️ Use only for streams |

**When to use each:**

```dart
// ✅ PREFERRED: AsyncNotifier for async state with mutations
@riverpod
class Products extends _$Products {
  @override
  Future<List<Product>> build() async {
    return fetchProducts();
  }

  Future<void> addProduct(Product product) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      await api.addProduct(product);
      return fetchProducts();
    });
  }
}

// ⚠️ LEGACY: FutureProvider only for simple, one-time fetches
@riverpod
Future<String> config(Ref ref) async {
  return await rootBundle.loadString('assets/config.json');
}

// ⚠️ LEGACY: StreamProvider only when you actually need a stream
@riverpod
Stream<User?> authState(Ref ref) {
  return FirebaseAuth.instance.authStateChanges();
}
```

### 1.4 Family Providers - Parameterized Providers

With code generation, family providers are created automatically when your provider function accepts parameters.

```dart
// Code generation - family is automatic
@riverpod
Future<User> user(Ref ref, String id) async {
  final dio = Dio();
  final response = await dio.get('https://api.example.com/users/$id');
  return User.fromJson(response.data);
}

// Usage
class UserProfile extends ConsumerWidget {
  const UserProfile({required this.userId});
  final String userId;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider(userId));

    return user.when(
      data: (user) => Text(user.name),
      loading: () => const CircularProgressIndicator(),
      error: (err, stack) => Text('Error: $err'),
    );
  }
}

// With AsyncNotifier
@riverpod
class UserNotifier extends _$UserNotifier {
  @override
  Future<User> build(String id) async {
    final dio = Dio();
    final response = await dio.get('https://api.example.com/users/$id');
    return User.fromJson(response.data);
  }

  Future<void> updateName(String newName) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final userId = arg; // Access the parameter
      final dio = Dio();
      await dio.put('https://api.example.com/users/$userId',
        data: {'name': newName});
      final response = await dio.get('https://api.example.com/users/$userId');
      return User.fromJson(response.data);
    });
  }
}
```

### 1.5 AutoDispose Pattern

**In 2025, code generation makes providers auto-dispose by default.**

```dart
// Code generation - autoDispose is DEFAULT
@riverpod
Future<String> data(Ref ref) async {
  return fetchData();
}

// To keep alive, use keepAlive: true
@Riverpod(keepAlive: true)
Future<String> cachedData(Ref ref) async {
  return fetchData();
}

// Conditional keep alive - cache only on success
@riverpod
Future<String> smartCachedData(Ref ref) async {
  final response = await http.get(Uri.parse('https://example.com'));

  // Only keep alive if successful
  if (response.statusCode == 200) {
    ref.keepAlive();
  }

  return response.body;
}
```

---

## 2. Performance Optimization Techniques

### 2.1 Provider Scoping and Rebuilding Optimization

**Key Principle**: Only rebuild widgets that need to change.

```dart
// ❌ BAD: Watching entire provider when only one field is needed
class ProductPrice extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final product = ref.watch(productProvider); // Rebuilds on ANY change
    return Text('\$${product.price}');
  }
}

// ✅ GOOD: Using select to watch only the price field
class ProductPrice extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final price = ref.watch(
      productProvider.select((product) => product.price)
    );
    return Text('\$${price}');
  }
}

// ✅ GOOD: Using selectAsync for async providers
@riverpod
Future<void> processUser(Ref ref) async {
  // Only rebuild when firstName changes
  final firstName = await ref.watch(
    userProvider.selectAsync((user) => user.firstName)
  );

  // Process with firstName
}
```

### 2.2 When to Use select() vs watch() vs read()

#### **ref.watch()** - Subscribe to changes
```dart
// ✅ Use in build methods to rebuild on changes
@override
Widget build(BuildContext context, WidgetRef ref) {
  final todos = ref.watch(todoListProvider);
  return ListView(children: todos.map(TodoTile.new).toList());
}

// ✅ Use in providers to create dependencies
@riverpod
int filteredCount(Ref ref) {
  final todos = ref.watch(todoListProvider);
  final filter = ref.watch(filterProvider);
  return todos.where((t) => t.matches(filter)).length;
}
```

#### **ref.select()** - Subscribe to specific fields only
```dart
// ✅ Use to optimize rebuilds for specific properties
class TodoCount extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Only rebuild when the count changes, not individual todos
    final count = ref.watch(
      todoListProvider.select((todos) => todos.length)
    );
    return Text('$count todos');
  }
}

// ✅ Use with computed properties
class AdultStatus extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isAdult = ref.watch(
      personProvider.select((person) => person.age >= 18)
    );
    return Text(isAdult ? 'Adult' : 'Minor');
  }
}
```

#### **ref.read()** - One-time read (no subscription)
```dart
// ✅ Use in event handlers
class AddTodoButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        // Read once to call a method
        ref.read(todoListProvider.notifier).addTodo('New task');
      },
      child: const Text('Add Todo'),
    );
  }
}

// ❌ DON'T use read() to avoid watch() for "optimization"
// This is an anti-pattern!
@override
Widget build(BuildContext context, WidgetRef ref) {
  final todos = ref.read(todoListProvider); // ❌ Won't rebuild!
  return ListView(children: todos.map(TodoTile.new).toList());
}
```

#### **ref.listen()** - Side effects
```dart
// ✅ Use for side effects (navigation, snackbars, logging)
@override
Widget build(BuildContext context, WidgetRef ref) {
  ref.listen(
    todoListProvider,
    (previous, next) {
      next.whenOrNull(
        error: (error, stack) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Error: $error')),
          );
        },
      );
    },
  );

  final todos = ref.watch(todoListProvider);
  return todos.when(...);
}
```

### 2.3 Family Providers and Caching Strategies

Family providers automatically cache based on parameters.

```dart
@riverpod
Future<User> user(Ref ref, String id) async {
  // Each unique 'id' creates a separate cached provider instance
  return fetchUser(id);
}

// These create separate cached instances
final user1 = ref.watch(userProvider('123')); // Cache entry 1
final user2 = ref.watch(userProvider('456')); // Cache entry 2
final user1Again = ref.watch(userProvider('123')); // Reuses cache entry 1

// ⚠️ IMPORTANT: Parameters must have proper equality
class UserFilter {
  const UserFilter({required this.role, required this.active});
  final String role;
  final bool active;

  // ✅ REQUIRED for proper caching
  @override
  bool operator ==(Object other) =>
    identical(this, other) ||
    other is UserFilter &&
    role == other.role &&
    active == other.active;

  @override
  int get hashCode => Object.hash(role, active);
}

@riverpod
Future<List<User>> filteredUsers(Ref ref, UserFilter filter) async {
  return fetchUsers(filter);
}
```

### 2.4 AutoDispose Patterns and Memory Management

```dart
// ✅ Default behavior (codegen) - auto-dispose when no listeners
@riverpod
Future<String> data(Ref ref) async {
  return fetchData();
}

// ✅ Conditional keep alive - cache successful results
@riverpod
Future<String> cachedData(Ref ref) async {
  final data = await fetchData();

  // Keep alive only if fetch succeeded
  ref.keepAlive();

  return data;
}

// ✅ Timed cache - keep alive for specific duration
@riverpod
Future<String> timedCache(Ref ref) async {
  final data = await fetchData();

  // Keep alive for 5 minutes
  final link = ref.keepAlive();
  Timer(const Duration(minutes: 5), link.close);

  return data;
}

// ✅ Manual disposal - cleanup resources
@riverpod
Stream<int> websocket(Ref ref) {
  final client = WebSocketClient();

  // Dispose when provider is destroyed
  ref.onDispose(() {
    client.close();
  });

  return client.stream;
}
```

### 2.5 Avoiding Unnecessary Rebuilds

```dart
// ❌ BAD: Entire widget rebuilds on any provider change
class TodoListView extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(todoListProvider);
    final filter = ref.watch(filterProvider);
    final sortOrder = ref.watch(sortOrderProvider);

    // All expensive operations run on ANY change
    final filtered = todos.where((t) => t.matches(filter)).toList();
    final sorted = filtered..sort(sortOrder.comparator);

    return ListView.builder(
      itemCount: sorted.length,
      itemBuilder: (context, index) => TodoTile(sorted[index]),
    );
  }
}

// ✅ GOOD: Separate provider for derived state
@riverpod
List<Todo> filteredSortedTodos(Ref ref) {
  final todos = ref.watch(todoListProvider);
  final filter = ref.watch(filterProvider);
  final sortOrder = ref.watch(sortOrderProvider);

  final filtered = todos.where((t) => t.matches(filter)).toList();
  return filtered..sort(sortOrder.comparator);
}

class TodoListView extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(filteredSortedTodosProvider);

    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (context, index) => TodoTile(todos[index]),
    );
  }
}

// ✅ EVEN BETTER: Individual items watch their own data
class TodoTile extends ConsumerWidget {
  const TodoTile({required this.todoId});
  final String todoId;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Only this tile rebuilds when this specific todo changes
    final todo = ref.watch(todoProvider(todoId));
    return ListTile(title: Text(todo.title));
  }
}
```

---

## 3. Modern Flutter + Riverpod Architecture Patterns

### 3.1 Repository Pattern with Riverpod

**Recommended architecture in 2025:**

```dart
// 1. Data layer - Repository
@riverpod
TodoRepository todoRepository(Ref ref) {
  return TodoRepository(
    dio: ref.watch(dioProvider),
  );
}

class TodoRepository {
  TodoRepository({required this.dio});
  final Dio dio;

  Future<List<Todo>> fetchTodos() async {
    final response = await dio.get('/todos');
    return (response.data as List)
      .map((json) => Todo.fromJson(json))
      .toList();
  }

  Future<Todo> createTodo(String title) async {
    final response = await dio.post('/todos', data: {'title': title});
    return Todo.fromJson(response.data);
  }

  Future<void> deleteTodo(String id) async {
    await dio.delete('/todos/$id');
  }
}

// 2. Application layer - State management
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    final repository = ref.watch(todoRepositoryProvider);
    return repository.fetchTodos();
  }

  Future<void> addTodo(String title) async {
    final repository = ref.read(todoRepositoryProvider);

    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      await repository.createTodo(title);
      return repository.fetchTodos();
    });
  }

  Future<void> deleteTodo(String id) async {
    final repository = ref.read(todoRepositoryProvider);

    // Optimistic update
    state = AsyncData(
      state.value!.where((t) => t.id != id).toList(),
    );

    // Perform actual deletion
    try {
      await repository.deleteTodo(id);
    } catch (e) {
      // Rollback on error
      ref.invalidateSelf();
    }
  }
}

// 3. Presentation layer - UI
class TodoListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todoListProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Todos')),
      body: todosAsync.when(
        data: (todos) => ListView.builder(
          itemCount: todos.length,
          itemBuilder: (context, index) => TodoTile(todos[index]),
        ),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(child: Text('Error: $error')),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddDialog(context, ref),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddDialog(BuildContext context, WidgetRef ref) {
    // Implementation
  }
}
```

### 3.2 Dependency Injection Best Practices

```dart
// ✅ Use providers for dependency injection
@riverpod
Dio dio(Ref ref) {
  final dio = Dio(BaseOptions(
    baseUrl: 'https://api.example.com',
  ));

  // Add interceptors
  dio.interceptors.add(LogInterceptor());

  return dio;
}

@riverpod
SharedPreferences sharedPreferences(Ref ref) async {
  return await SharedPreferences.getInstance();
}

@riverpod
AuthService authService(Ref ref) {
  return AuthService(
    dio: ref.watch(dioProvider),
    storage: ref.watch(sharedPreferencesProvider).value!,
  );
}

// ✅ Repositories depend on other providers
@riverpod
UserRepository userRepository(Ref ref) {
  return UserRepository(
    dio: ref.watch(dioProvider),
    authService: ref.watch(authServiceProvider),
  );
}

// ✅ State providers depend on repositories
@riverpod
class CurrentUser extends _$CurrentUser {
  @override
  Future<User?> build() async {
    final authService = ref.watch(authServiceProvider);
    final userId = await authService.getCurrentUserId();

    if (userId == null) return null;

    final repository = ref.watch(userRepositoryProvider);
    return repository.fetchUser(userId);
  }

  Future<void> logout() async {
    final authService = ref.read(authServiceProvider);
    await authService.logout();
    ref.invalidateSelf();
  }
}
```

### 3.3 Testing Strategies

```dart
// ✅ Testing providers with ProviderContainer.test()
void main() {
  test('TodoList fetches todos correctly', () async {
    final container = ProviderContainer.test(
      overrides: [
        todoRepositoryProvider.overrideWithValue(
          MockTodoRepository(),
        ),
      ],
    );

    final todos = await container.read(todoListProvider.future);

    expect(todos.length, 2);
    expect(todos[0].title, 'Test Todo 1');
  });

  test('TodoList adds todo correctly', () async {
    final mockRepo = MockTodoRepository();
    final container = ProviderContainer.test(
      overrides: [
        todoRepositoryProvider.overrideWithValue(mockRepo),
      ],
    );

    await container.read(todoListProvider.notifier).addTodo('New Todo');

    verify(() => mockRepo.createTodo('New Todo')).called(1);
  });
}

// ✅ Widget testing with ProviderScope
void main() {
  testWidgets('TodoListScreen displays todos', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          todoRepositoryProvider.overrideWithValue(
            MockTodoRepository(),
          ),
        ],
        child: const MaterialApp(
          home: TodoListScreen(),
        ),
      ),
    );

    // Wait for async data to load
    await tester.pumpAndSettle();

    expect(find.text('Test Todo 1'), findsOneWidget);
    expect(find.text('Test Todo 2'), findsOneWidget);
  });
}

// ✅ Mock implementation
class MockTodoRepository implements TodoRepository {
  @override
  Future<List<Todo>> fetchTodos() async {
    return [
      Todo(id: '1', title: 'Test Todo 1'),
      Todo(id: '2', title: 'Test Todo 2'),
    ];
  }

  @override
  Future<Todo> createTodo(String title) async {
    return Todo(id: '3', title: title);
  }

  @override
  Future<void> deleteTodo(String id) async {}
}
```

### 3.4 Error Handling Patterns

```dart
// ✅ Comprehensive error handling with AsyncValue
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    try {
      final repository = ref.watch(todoRepositoryProvider);
      return await repository.fetchTodos();
    } on DioException catch (e) {
      if (e.response?.statusCode == 401) {
        ref.read(authServiceProvider).logout();
        throw UnauthorizedException();
      }
      throw NetworkException(e.message);
    } catch (e) {
      throw UnexpectedException(e.toString());
    }
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      final repository = ref.read(todoRepositoryProvider);
      await repository.createTodo(title);
      return repository.fetchTodos();
    });
  }
}

// ✅ Error handling in UI
class TodoListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todoListProvider);

    return todosAsync.when(
      data: (todos) => ListView.builder(
        itemCount: todos.length,
        itemBuilder: (context, index) => TodoTile(todos[index]),
      ),
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, stack) {
        if (error is NetworkException) {
          return ErrorView(
            message: 'Network error. Please check your connection.',
            onRetry: () => ref.invalidate(todoListProvider),
          );
        }
        if (error is UnauthorizedException) {
          return const ErrorView(message: 'Please log in again.');
        }
        return ErrorView(
          message: 'An unexpected error occurred.',
          onRetry: () => ref.invalidate(todoListProvider),
        );
      },
    );
  }
}

// ✅ Listen for errors and show snackbars
@override
Widget build(BuildContext context, WidgetRef ref) {
  ref.listen<AsyncValue<List<Todo>>>(
    todoListProvider,
    (previous, next) {
      next.whenOrNull(
        error: (error, stack) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(error.toString()),
              backgroundColor: Colors.red,
            ),
          );
        },
      );
    },
  );

  final todos = ref.watch(todoListProvider);
  return todos.when(...);
}
```

---

## 4. Common Anti-Patterns to Avoid

### 4.1 Performance Pitfalls

```dart
// ❌ ANTI-PATTERN: Using ref.read() to avoid rebuilds
class BadExample extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // This will NOT rebuild when provider changes!
    final todos = ref.read(todoListProvider);
    return Text('Count: ${todos.length}');
  }
}

// ✅ CORRECT: Use ref.watch() or ref.select()
class GoodExample extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(
      todoListProvider.select((todos) => todos.length)
    );
    return Text('Count: $count');
  }
}

// ❌ ANTI-PATTERN: Watching providers in loops
class BadListView extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todoIds = ref.watch(todoIdsProvider);

    return ListView.builder(
      itemCount: todoIds.length,
      itemBuilder: (context, index) {
        // ❌ Watching in itemBuilder causes performance issues
        final todo = ref.watch(todoProvider(todoIds[index]));
        return ListTile(title: Text(todo.title));
      },
    );
  }
}

// ✅ CORRECT: Separate widget for each item
class TodoItem extends ConsumerWidget {
  const TodoItem({required this.todoId});
  final String todoId;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todo = ref.watch(todoProvider(todoId));
    return ListTile(title: Text(todo.title));
  }
}

class GoodListView extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todoIds = ref.watch(todoIdsProvider);

    return ListView.builder(
      itemCount: todoIds.length,
      itemBuilder: (context, index) => TodoItem(todoId: todoIds[index]),
    );
  }
}
```

### 4.2 Memory Leaks

```dart
// ❌ ANTI-PATTERN: Not disposing resources
@riverpod
Stream<int> badWebsocket(Ref ref) {
  final client = WebSocketClient();
  return client.stream; // ❌ Client never closed!
}

// ✅ CORRECT: Dispose resources
@riverpod
Stream<int> goodWebsocket(Ref ref) {
  final client = WebSocketClient();

  ref.onDispose(() {
    client.close();
  });

  return client.stream;
}

// ❌ ANTI-PATTERN: Keeping everything alive
@Riverpod(keepAlive: true)
class BadCache extends _$BadCache {
  final _data = <String, String>{};

  @override
  Map<String, String> build() => _data; // ❌ Never disposed!
}

// ✅ CORRECT: Use autoDispose with conditional keepAlive
@riverpod
Future<String> goodCache(Ref ref, String key) async {
  final data = await fetchData(key);

  // Only keep alive for successful results
  ref.keepAlive();

  // Or with timed expiration
  final link = ref.keepAlive();
  Timer(const Duration(minutes: 5), link.close);

  return data;
}
```

### 4.3 Incorrect Provider Usage

```dart
// ❌ ANTI-PATTERN: Performing side effects in provider initialization
@riverpod
Future<void> badSubmit(Ref ref) async {
  final formState = ref.watch(formStateProvider);
  // ❌ Providers should be for READ operations, not WRITE
  await http.post('https://api.example.com', body: formState);
}

// ✅ CORRECT: Use notifier methods for side effects
@riverpod
class Form extends _$Form {
  @override
  FormState build() => FormState.empty();

  void updateField(String key, String value) {
    state = state.copyWith(fields: {...state.fields, key: value});
  }

  Future<void> submit() async {
    await http.post('https://api.example.com', body: state.toJson());
  }
}

// ❌ ANTI-PATTERN: Dynamic provider access
class BadWidget extends ConsumerWidget {
  const BadWidget({required this.provider});
  final Provider<int> provider; // ❌ Can't be analyzed statically

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(provider);
    return Text('$value');
  }
}

// ✅ CORRECT: Statically known providers
@riverpod
int value(Ref ref) => 42;

class GoodWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(valueProvider);
    return Text('$value');
  }
}
```

### 4.4 State Synchronization Issues

```dart
// ❌ ANTI-PATTERN: Multiple sources of truth
class BadExample {
  int localCount = 0; // ❌ Local state

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final providerCount = ref.watch(counterProvider); // ❌ Provider state
    // Which one is the source of truth?
    return Text('$localCount vs $providerCount');
  }
}

// ✅ CORRECT: Single source of truth
class GoodExample extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}

// ❌ ANTI-PATTERN: Not invalidating dependent providers
@riverpod
class BadAuth extends _$BadAuth {
  @override
  User? build() => null;

  Future<void> logout() async {
    state = null;
    // ❌ Other providers still have old user data!
  }
}

// ✅ CORRECT: Invalidate dependent providers
@riverpod
class GoodAuth extends _$GoodAuth {
  @override
  User? build() => null;

  Future<void> logout() async {
    state = null;

    // Invalidate all user-related data
    ref.invalidate(userProfileProvider);
    ref.invalidate(userSettingsProvider);
    ref.invalidate(userNotificationsProvider);
  }
}

// ✅ EVEN BETTER: Make dependent providers watch auth
@riverpod
Future<UserProfile> userProfile(Ref ref) async {
  final user = ref.watch(authProvider);
  if (user == null) throw UnauthenticatedException();

  // Automatically refetches when user changes
  return fetchUserProfile(user.id);
}
```

---

## 5. Migration Strategies

### 5.1 From StateNotifierProvider to AsyncNotifierProvider

```dart
// OLD: StateNotifierProvider
class TodoListNotifier extends StateNotifier<AsyncValue<List<Todo>>> {
  TodoListNotifier({required this.ref}) : super(const AsyncLoading()) {
    fetchTodos();
  }

  final Ref ref;

  Future<void> fetchTodos() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final repository = ref.read(todoRepositoryProvider);
      return repository.fetchTodos();
    });
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final repository = ref.read(todoRepositoryProvider);
      await repository.createTodo(title);
      return repository.fetchTodos();
    });
  }
}

final todoListProvider = StateNotifierProvider<TodoListNotifier, AsyncValue<List<Todo>>>(
  (ref) => TodoListNotifier(ref: ref),
);

// NEW: AsyncNotifierProvider with code generation
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    final repository = ref.watch(todoRepositoryProvider);
    return repository.fetchTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final repository = ref.read(todoRepositoryProvider);
      await repository.createTodo(title);
      return repository.fetchTodos();
    });
  }
}

// Migration checklist:
// 1. Remove StateNotifier dependency
// 2. Remove explicit Ref parameter
// 3. Move initialization to build() method
// 4. Use @riverpod annotation
// 5. Update consumer code to use generated provider
```

### 5.2 From FutureProvider to AsyncNotifierProvider

```dart
// OLD: FutureProvider (read-only)
final userProvider = FutureProvider.autoDispose.family<User, String>((ref, id) async {
  final repository = ref.watch(userRepositoryProvider);
  return repository.fetchUser(id);
});

// NEW: AsyncNotifierProvider (with mutations)
@riverpod
class User extends _$User {
  @override
  Future<User> build(String id) async {
    final repository = ref.watch(userRepositoryProvider);
    return repository.fetchUser(id);
  }

  Future<void> updateName(String newName) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final repository = ref.read(userRepositoryProvider);
      await repository.updateUser(id, name: newName);
      return repository.fetchUser(id);
    });
  }
}
```

### 5.3 From Provider to Code Generation

```dart
// OLD: Manual provider
final counterProvider = StateNotifierProvider<Counter, int>((ref) {
  return Counter();
});

class Counter extends StateNotifier<int> {
  Counter() : super(0);

  void increment() => state++;
  void decrement() => state--;
}

// NEW: Code generation
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

// Steps:
// 1. Add riverpod_annotation dependency
// 2. Add riverpod_generator dev dependency
// 3. Create part directive: part 'filename.g.dart';
// 4. Add @riverpod annotation
// 5. Run build_runner
// 6. Update imports and usage
```

### 5.4 Migration Tool

Riverpod provides an automated migration tool:

```bash
# Install the migration tool
dart pub global activate riverpod_cli

# Run migration
dart pub global run riverpod_cli migrate

# This will:
# 1. Analyze your code
# 2. Suggest changes
# 3. Allow you to accept/reject each change
# 4. Update your code automatically
```

### 5.5 Gradual Migration Strategy

You don't need to migrate everything at once. Here's a recommended approach:

1. **Start with new features**: Use code generation for all new code
2. **Migrate leaf providers**: Start with providers that don't depend on others
3. **Migrate repositories**: Update data layer providers
4. **Migrate state providers**: Update business logic layer
5. **Update UI**: Change consumer code last

```dart
// You can mix old and new providers during migration
final oldProvider = Provider<String>((ref) => 'old');

@riverpod
String newProvider(Ref ref) {
  // New provider can watch old providers
  final oldValue = ref.watch(oldProvider);
  return 'new: $oldValue';
}

// Old providers can also watch new providers
final mixedProvider = Provider<String>((ref) {
  final newValue = ref.watch(newProviderProvider);
  return 'mixed: $newValue';
});
```

---

## Quick Reference Cheat Sheet

### Provider Selection Guide

| Use Case | Provider Type | Example |
|----------|--------------|---------|
| Immutable config | `Provider` | API keys, theme data |
| Simple sync state | `NotifierProvider` | Counter, form state |
| Async data (read-only) | `FutureProvider` | Config files (legacy) |
| Async data (with mutations) | `AsyncNotifierProvider` | API data, user state |
| Real-time stream | `StreamProvider` | WebSocket, Firebase |
| Parameterized provider | Add parameters to function | User by ID |

### Ref Methods Quick Guide

| Method | When to Use | Example |
|--------|-------------|---------|
| `ref.watch()` | Subscribe to changes in build | `final todos = ref.watch(todoProvider);` |
| `ref.select()` | Subscribe to specific property | `final name = ref.watch(userProvider.select((u) => u.name));` |
| `ref.read()` | One-time read in callbacks | `ref.read(todoProvider.notifier).add();` |
| `ref.listen()` | Side effects (navigation, snackbars) | `ref.listen(authProvider, (_, next) => navigate());` |
| `ref.invalidate()` | Force provider refresh | `ref.invalidate(todoProvider);` |
| `ref.refresh()` | Invalidate and read | `final new = ref.refresh(todoProvider);` |

### Performance Checklist

- [ ] Use `select()` when watching specific properties
- [ ] Avoid `ref.read()` in build methods
- [ ] Create derived providers for computed state
- [ ] Use separate widgets for list items
- [ ] Implement proper `==` and `hashCode` for family parameters
- [ ] Use `autoDispose` for temporary data
- [ ] Use `keepAlive()` for cached data
- [ ] Dispose resources in `ref.onDispose()`

### Common Patterns

```dart
// 1. Load once and cache
@riverpod
Future<Config> config(Ref ref) async {
  final data = await loadConfig();
  ref.keepAlive(); // Cache forever
  return data;
}

// 2. Optimistic updates
Future<void> deleteTodo(String id) async {
  final old = state.value!;
  state = AsyncData(old.where((t) => t.id != id).toList());

  try {
    await api.delete(id);
  } catch (e) {
    state = AsyncData(old); // Rollback
    rethrow;
  }
}

// 3. Polling
@riverpod
Stream<Data> polledData(Ref ref) async* {
  while (true) {
    yield await fetchData();
    await Future.delayed(const Duration(seconds: 30));
  }
}

// 4. Dependent providers
@riverpod
Future<UserProfile> userProfile(Ref ref) async {
  final userId = ref.watch(currentUserIdProvider);
  return fetchProfile(userId);
}
```

---

## Conclusion

Riverpod in 2025 emphasizes:

1. **Code generation** for reduced boilerplate
2. **AsyncNotifierProvider** as the preferred async pattern
3. **Compile-time safety** and better tooling
4. **Performance optimization** through selective watching
5. **Repository pattern** for clean architecture
6. **Comprehensive testing** with ProviderContainer.test()

By following these patterns and avoiding the anti-patterns, you'll build scalable, maintainable, and performant Flutter applications with Riverpod.
