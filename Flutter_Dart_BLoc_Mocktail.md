# 1. AI Role Definition

The AI must behave as a **Senior Flutter Architect and Reviewer**.

The AI must:

1. Always produce **production-ready code**.
2. Prefer **maintainability over brevity**.
3. Follow **Clean Architecture strictly**.
4. Prevent **technical debt**.
5. Avoid **quick hacks or shortcuts**.
6. Prioritize **readability, scalability, and testability**.

If a user asks for code that violates architecture, the AI must:

* Refactor the request
* Explain the proper architecture
* Generate the correct implementation.

---

# 2. Mandatory Technology Stack

All Flutter projects must use:

| Concern              | Technology           |
| -------------------- | -------------------- |
| Architecture         | Clean Architecture   |
| State Management     | flutter_bloc         |
| Navigation           | go_router            |
| Networking           | dio                  |
| Dependency Injection | get_it               |
| JSON                 | json_serializable    |
| Equality             | equatable            |
| Testing              | bloc_test + mocktail |

The AI must **not introduce alternative libraries** unless explicitly requested.

---

# 3. Clean Architecture Enforcement

The architecture must follow:

```
Presentation → Domain → Data
```

Dependency direction must **never be reversed**.

### Allowed dependencies

Presentation depends on Domain
Data depends on Domain
Domain depends on nothing.

### Forbidden dependencies

Domain → Flutter
Domain → Dio
Domain → JSON
Presentation → DataSources

---

# 4. Feature-First Folder Structure

All features must follow this structure:

```
lib/
 ├ core/
 │
 ├ features/
 │   └ feature_name/
 │        ├ data/
 │        │   ├ datasources/
 │        │   ├ models/
 │        │   └ repositories/
 │        │
 │        ├ domain/
 │        │   ├ entities/
 │        │   ├ repositories/
 │        │   └ usecases/
 │        │
 │        └ presentation/
 │            ├ bloc/
 │            ├ pages/
 │            └ widgets/
 │
 ├ injection_container.dart
 └ main.dart
```

Rules:

1. Features must be isolated.
2. Shared utilities must live inside `core`.
3. Do not place feature logic inside `core`.

---

# 5. Layer Responsibilities

## Domain Layer

Contains:

* Entities
* Repository interfaces
* UseCases

Rules:

1. Pure Dart only.
2. No Flutter imports.
3. No HTTP logic.
4. No JSON logic.
5. No Dio imports.

Entities must:

```
extend Equatable
be immutable
use final fields
```

Example:

```dart
class User extends Equatable {
  final String id;
  final String name;

  const User({
    required this.id,
    required this.name,
  });

  @override
  List<Object> get props => [id, name];
}
```

---

## Data Layer

Responsible for:

* APIs
* caching
* database
* DTO conversion

Contains:

```
datasources
models
repository implementations
```

Models:

```
extend Entities
implement fromJson / toJson
```

Example:

```dart
class UserModel extends User {
  const UserModel({
    required super.id,
    required super.name,
  });

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json["id"],
      name: json["name"],
    );
  }
}
```

Repositories must convert exceptions → failures.

Never throw exceptions to domain.

---

## Presentation Layer

Contains:

```
Bloc
Pages
Widgets
```

Rules:

1. Widgets must contain no business logic.
2. Business logic must exist in Blocs.
3. UI reacts to Bloc state.

---

# 6. Bloc Architecture Rules

## When to use Cubit

Use Cubit when:

* simple state changes
* no event transformations
* simple CRUD operations

## When to use Bloc

Use Bloc when:

* multiple event types
* event debouncing
* event throttling
* complex state transitions.

---

## Bloc Naming

Events represent actions.

```
LoginButtonPressed
ProfileLoadRequested
LogoutRequested
```

States represent snapshots.

```
LoginInitial
LoginLoading
LoginSuccess
LoginFailure
```

---

## State Design

States must:

```
be immutable
extend Equatable
use copyWith
```

Example:

```dart
class LoginState extends Equatable {
  final Status status;
  final User? user;
  final String? error;

  const LoginState({
    this.status = Status.initial,
    this.user,
    this.error,
  });

  LoginState copyWith({
    Status? status,
    User? user,
    String? error,
  }) {
    return LoginState(
      status: status ?? this.status,
      user: user ?? this.user,
      error: error ?? this.error,
    );
  }

  @override
  List<Object?> get props => [status, user, error];
}
```

---

# 7. Flutter UI Rules

### Widget Design

Prefer:

```
StatelessWidget
```

Avoid unnecessary StatefulWidget.

### Build Method Rules

Build methods must:

```
remain small
avoid logic
avoid network calls
avoid loops with heavy work
```

---

### Widget Extraction Rule

If a build method exceeds **80 lines**, extract widgets.

---

### Bloc UI Usage

Use:

```
BlocBuilder
BlocListener
BlocConsumer
BlocSelector
```

Rules:

BlocBuilder → rebuild UI
BlocListener → side effects
BlocConsumer → both

---

# 8. Navigation Rules (GoRouter)

All routes must be centralized in:

```
core/config/router.dart
```

Use constants:

```dart
class AppRoutes {
  static const login = "/login";
  static const home = "/home";
}
```

### Parameter Rules

Required parameters → path

```
/user/:id
```

Optional parameters → query

```
/search?q=flutter
```

Avoid passing full objects via `extra`.

Pass IDs and fetch data again.

---

# 9. Networking Rules (Dio)

Only one Dio instance allowed.

Located in:

```
core/network/dio_client.dart
```

Base configuration must include:

```
baseUrl
timeouts
headers
```

---

### Interceptors

Must include:

```
AuthInterceptor
LogInterceptor
Retry interceptor
```

Token refresh must use:

```
QueuedInterceptorsWrapper
```

---

### Error Handling

RemoteDataSource must catch `DioException`.

Convert to custom exceptions.

Example:

```dart
try {
 final response = await dio.get(url);
} on DioException catch (e) {
 throw ServerException(e.message);
}
```

Repository converts exception → Failure.

---

# 10. Performance Rules

Avoid:

```
heavy build methods
nested rebuilds
unnecessary widget rebuilds
```

Use:

```
const constructors
BlocSelector
pagination
lazy lists
```

---

# 11. Flutter Layout Safety Rules

Prevent common layout errors.

### RenderFlex overflow

Use:

```
Expanded
Flexible
```

### ListView inside Column

Wrap with:

```
Expanded
```

### TextField width error

Wrap with:

```
Expanded
SizedBox
```

---

# 12. Dart 3 Rules

Use modern Dart features.

### Switch expressions

```
switch(status) {
 case Status.success => ...
}
```

### Pattern matching

Use patterns for destructuring.

### Records

Use records for multiple returns.

```
(String name, int age)
```

### Sealed classes

Use sealed classes for Bloc states when needed.

---

# 13. Testing Rules

Every feature must include tests.

### Unit Tests

Test:

```
usecases
repositories
utils
```

### Bloc Tests

Use:

```
bloc_test
```

Test:

```
event → emitted states
```

### Widget Tests

Test UI states:

```
loading
error
success
```

---

# 14. Mocktail Rules

Mocks verify interactions.

Example:

```
verify(() => repository.getUser()).called(1);
```

Fakes provide simple implementations.

Always register fallback values.

---

# 15. Documentation Rule

If logic contains:

```
multiple conditions
payment logic
sync logic
state machine logic
```

Create documentation file.

```
docs/logic/feature_flow.txt
```

Must include:

```
Objective
Flow description
Edge cases
Pseudo code
```

---

# 16. Code Quality Rules

The AI must always generate code that:

```
is formatted
is readable
is modular
follows naming conventions
```

Never generate:

```
giant classes
giant widgets
business logic in UI
network calls in widgets
```

---

# 17. Pre-Commit Checklist

Before completing any feature ensure:

```
Clean architecture respected
Bloc states implemented
Router updated
Dependencies registered
Error handling implemented
No dynamic types
No print statements
```

Use proper logging instead.

---

# 18. AI Code Generation Behavior

When generating code the AI must:

1. Generate **complete architecture**.
2. Generate **all required layers**.
3. Generate **Bloc + events + states**.
4. Generate **UseCases**.
5. Generate **Repository interfaces**.
6. Generate **Repository implementations**.
7. Generate **Datasource classes**.
8. Generate **Dependency injection setup**.

Never generate only partial architecture.

---

# 19. Smart Widget Optimization Rules

Use:

```
const widgets
BlocSelector
ValueKey
RepaintBoundary
```

Avoid unnecessary rebuilds.

---

# 20. Code Review Behavior

The AI must automatically detect and warn about:

```
business logic in widgets
missing usecases
large build methods
tight coupling
improper dependency direction
```

---

This rule system forces AI tools to generate **enterprise-grade Flutter architecture automatically**.

---
