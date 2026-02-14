# Antigravity Flutter System: Clean Architecture + Bloc + GoRouter

## **1. Core Philosophy & Manifest**

* **Architecture:** Clean Architecture (Data, Domain, Presentation) is non-negotiable.
* **State Management:** `flutter_bloc` is the only state manager.
* **Navigation:** `go_router` is the exclusive routing solution.
* **Documentation:** Complexity demands explanation. Any complex logic **MUST** be documented in a dedicated `.txt` file before implementation details are finalized.
* **Immutability:** Everything is immutable by default. `final` is your best friend.
* **Strict Typing:** `dynamic` is forbidden unless dealing with untyped 3rd-party JSON (and must be cast immediately).

---

## **2. Folder Structure & Organization**

The project structure must be strictly feature-first.

```text
lib/
├── core/                       # Shared logic, utilities, errors, network
│   ├── config/                 # Router, Theme, Env variables
│   │   └── router.dart         # GoRouter configuration
│   ├── error/                  # Generic failures & exceptions
│   ├── usecases/               # Base UseCase interface
│   └── utils/                  # Extensions, constants
├── features/
│   ├── [feature_name]/         # e.g., 'authentication'
│   │   ├── data/
│   │   │   ├── datasources/    # Remote & Local data sources
│   │   │   ├── models/         # DTOs (extends Entity, handles JSON)
│   │   │   └── repositories/   # Implementation of Domain Repositories
│   │   ├── domain/
│   │   │   ├── entities/       # Pure Dart classes (Equatable)
│   │   │   ├── repositories/   # Abstract Interfaces
│   │   │   └── usecases/       # Business Logic Executors
│   │   └── presentation/
│   │       ├── bloc/           # Bloc, Events, States
│   │       ├── pages/          # Scaffold widgets (GoRouter targets)
│   │       └── widgets/        # Reusable UI components for this feature
├── main.dart
└── injection_container.dart    # Dependency Injection (GetIt)

docs/                           # NEW: Mandatory Documentation Directory
├── logic/                      # Complex logic breakdowns
│   └── [feature]_[logic].txt  # e.g. auth_refresh_token_flow.txt
└── architecture/

```

---

## **3. The "Complex Logic" Documentation Rule**

**Rule:** If a specific implementation involves more than 3 conditional branches, multiple interacting streams, or a critical business algorithm (e.g., payment calculation, offline sync, nested navigation guards), you **MUST** create a documentation file **BEFORE** or **DURING** the coding process.

**Protocol:**

1. Create a file in `docs/logic/` named `[feature_name]_[logic_name].txt`.
2. Use the following template:

```text
FILENAME: docs/logic/auth_session_management.txt
DATE: YYYY-MM-DD
AUTHOR: [Name]

1. OBJECTIVE:
   Briefly explain what this complex logic achieves.

2. FLOW DESCRIPTION:
   Step-by-step text description of the flow.
   - If A happens -> Do B
   - If C happens -> Check D

3. EDGE CASES:
   - What happens if network fails?
   - What happens if token expires during the process?

4. PSEUDOCODE / DIAGRAM:
   (Optional but recommended) Simplified code logic.

```

3. **Reference this file in the code comments:**
```dart
// See docs/logic/auth_session_management.txt for logic details
void _onSessionExpired() { ... }

```



---

## **4. Navigation (GoRouter) Rules**

1. **Central Configuration**: All routing logic resides in `lib/core/config/router.dart`.
2. **Type-Safe Routes**: Define route paths as `static const String` constants inside a specific `AppRoutes` class.
3. **ShellRoute for Layouts**: Use `ShellRoute` or `StatefulShellRoute` for persistent bottom navigation bars or drawers.
4. **Guards (Redirects)**:
* Implement top-level redirects for authentication (e.g., if not logged in, send to `/login`).
* Avoid complex logic inside the `redirect` callback. Delegate to an `AuthBloc` state check.


5. **Parameters**:
* **Path Parameters**: Use for required IDs (e.g., `/user/:id`).
* **Query Parameters**: Use for optional filters (e.g., `/search?q=flutter`).
* **Extra Objects**: **Avoid** passing complex objects via `extra`. Pass the ID and fetch the data in the new screen's Bloc. This prevents data desynchronization.


6. **Navigation Layers**:
* **Presentation Layer** triggers navigation.
* **Bloc** should **NOT** depend on `GoRouter`. Bloc emits a state (e.g., `SubmissionSuccess`), and the UI (`BlocListener`) performs the `context.go()`.



---

## **5. Clean Architecture Layers (Strict Mode)**

### **A. Domain Layer (Pure Dart)**

* **Dependencies**: ZERO external dependencies (no Flutter, no HTTP, no JSON).
* **Entities**:
* Must extend `Equatable`.
* `final` fields only.
* Absolutely NO `fromJson`/`toJson`.


* **Repositories**:
* Abstract classes only.
* Methods return `Future<Either<Failure, Type>>` (using `fpdart` or `dartz`).


* **Use Cases**:
* One class per action.
* Name: `VerbSubjectUseCase` (e.g., `GetUserDetailsUseCase`).
* Must have a `call()` method.



### **B. Data Layer (The Interface)**

* **Models**:
* Extend Domain Entities.
* Use `json_annotation` & `json_serializable`.
* Include `toEntity()` and `factory fromEntity()` if shapes differ.


* **Data Sources**:
* Throw Exceptions (not Failures).
* `RemoteDataSource`: Throws `ServerException`.
* `LocalDataSource`: Throws `CacheException`.


* **Repositories**:
* Implement Domain Repository.
* Catch Exceptions and return `Left(Failure)`.
* **NEVER** let an exception bubble up to the Domain.



### **C. Presentation Layer (Flutter + Bloc)**

* **Pages**:
* Responsible for `Scaffold`, `AppBar`, and injecting the `Bloc`.


* **Blocs**:
* **Event-Driven**: Events are "actions" (e.g., `started`, `refreshRequested`), not results.
* **State-Driven**: States are "snapshots" (e.g., `loading`, `success`, `failure`).
* **No Logic**: Blocs strictly delegate to UseCases.
* **Concurrency**: Use `package:bloc_concurrency` (e.g., `droppable`, `restartable`) for high-frequency events like search typing.


* **Widgets**:
* Small, reusable, dumb components.
* Accept data via constructor, return events via callbacks.



---

## **6. Flutter Bloc Rules**

### **Naming Conventions**

1. **Events**: `FeatureEvent` (base), `FeatureActioned` (subclass).
* Examples: `AuthLoginButtonPressed`, `UserProfileLoadRequested`.


2. **States**: `FeatureState`.
* Use `freezed` unions for complex states: `_Initial`, `_Loading`, `_Success`, `_Failure`.
* OR use a single class with `status` enum (`initial`, `loading`, `success`, `failure`).



### **State Management Best Practices**

1. **Cubit vs. Bloc**:
* **Simple Cases:** ALWAYS use **`Cubit`** for simple features (e.g., toggles, counters, simple data fetches).
* **Complex Cases:** Use **`Bloc`** only when event tracing, advanced debouncing, or complex event transformation is required.


2. **State Updates**:
* **Use `copyWith**`: Always implement and use the `copyWith` pattern for state updates. Never mutate state variables directly.
* *Example:* `emit(state.copyWith(status: Status.success, data: newData));`


3. **One Bloc Per Feature**: Don't reuse Blocs across disparate features unless it's a global `UserBloc` or `ThemeBloc`.
4. **BlocProvider**: Provide Blocs at the highest necessary level (usually the Page level, using `GoRouter`'s builder).
5. **BlocConsumer**: Use when you need to both Rebuild UI (Builder) and Perform Actions (Listener, e.g., Show Snackbars/Navigate).
6. **No Context in Bloc**: Never pass `BuildContext` into a Bloc.

---

## **7. Coding Standards & Syntax**

### **Dart**

1. **Variables**:
* `final` everywhere.
* `const` for compile-time constants.
* `late` is a code smell. Avoid unless absolutely necessary (e.g., AnimationControllers).


2. **Functions**:
* Small, focused, single responsibility.
* Type annotate ALL parameters and return types.


3. **Async**:
* Prefer `async/await` over `.then()`.
* Always wrap `await` calls in `try/catch` inside the Data Layer.



### **JSON Handling**

* Use `snake_case` for JSON keys.
* Use `camelCase` for Dart properties.
* Always use `@JsonKey(name: 'server_key_name')` to map explicitly.

### **Error Handling**

* **Domain**: `Either<Failure, Success>`.
* **UI**:
* `Failure` objects should have a `message` property suitable for display (or a key for localization).
* Map Failures to user-friendly messages in the Presentation layer, not the Domain layer.



---

## **8. Testing Protocol**

1. **Unit Tests (Core/Domain/Data)**:
* Mock dependencies using `mocktail`.
* Test every UseCase.
* Test Repository implementations (checking if they catch exceptions and return Failures).


2. **Bloc Tests**:
* Use `bloc_test`.
* Test specific event-to-state emissions.
* Verify that UseCases are called.


3. **Widget Tests**:
* Test the "Happy Path" of the UI.
* Test Error states (does the Snackbar appear?).



---

## **9. Antigravity Implementation Checklist**

Before committing any feature, ensure:

1. [ ] Logic is separated into Data/Domain/Presentation.
2. [ ] Complex logic is documented in `docs/logic/*.txt`.
3. [ ] `GoRouter` path is added to `AppRoutes`.
4. [ ] `Bloc` handles `Loading`, `Success`, and `Failure` states.
5. [ ] Dependency Injection is registered in `injection_container.dart`.
6. [ ] No `print` statements (use a Logger).
7. [ ] No `dynamic` types used implicitly.

---

# Example: Complex Logic Documentation (Template)

**File:** `docs/logic/checkout_payment_flow.txt`

```text
FILENAME: docs/logic/checkout_payment_flow.txt
FEATURE: Checkout
AUTHOR: Senior Dev
DATE: 2024-05-20

1. OBJECTIVE:
   Handle the multi-step payment process involving stock validation, 
   payment gateway intent creation, and final order confirmation.

2. FLOW DESCRIPTION:
   a. User clicks "Pay Now".
   b. App checks local validation (address filled?).
   c. Bloc triggers `ValidateStockUseCase`.
      - IF Fail: Emit StockErrorState.
      - IF Success: Proceed.
   d. Bloc triggers `CreatePaymentIntentUseCase` (Stripe).
      - Returns ClientSecret.
   e. Bloc triggers `ConfirmPaymentUseCase`.
      - IF specific error "3DSecure Required":
        - Emit `RequiresActionState`.
        - UI navigates to 3DS WebView via GoRouter (`/checkout/3ds`).
      - IF Success: Emit `PaymentSuccessState`.
      - IF Fail: Emit `PaymentFailureState`.

3. ROUTING IMPLICATIONS:
   - Success -> context.go('/order-confirmed')
   - 3DS -> context.push('/checkout/3ds')

4. EDGE CASES:
   - Network drops during step 'd'. Implementation: Retry mechanism with exponential backoff (see Data Layer).
   - User closes app during 3DS. Implementation: Webhook listener on backend updates order status eventually.

```
