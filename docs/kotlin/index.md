# Kotlin Scope Functions: What Problems Do They Actually Solve?

Kotlin gives you a bunch of powerful **scope and utility functions**: `run`, `let`, `apply`, `also`, `with`, `takeIf`, `takeUnless`, and `repeat`.

But **why do they exist**?

In this post, we'll cover **what real problems** these functions solve, show clear examples, and help you understand when to use each.

---

## Problem: I want to execute code in a scope and return a result

### Solution: `run { ... }`

Use the global `run` to execute code in a scope and return the result—perfect for avoiding variable pollution.

```kotlin title="public inline fun <R> run(block: () -> R): R"
val result = run {
    val x = 10
    val y = 20
    x + y
}
println(result) // 30
```

!!! note "Use when"
    You need to evaluate something and return a result, while keeping temporary variables scoped.

---

## Problem: I want to call multiple methods on an object and return a transformed result

### Solution: `T.run { ... }`

Use the extension `run` to execute a lambda with the object as context and return a transformed result.

```kotlin title="public inline fun <T, R> T.run(block: T.() -> R): R"
val person = Person("John", 30)
val description = person.run {
    "Name: $name, Age: $age, IsAdult: ${age >= 18}"
}
println(description) // Name: John, Age: 30, IsAdult: true
```

!!! note "Use when"
    You want to operate on an object and transform it into a different result.

---

## Problem: I want to call multiple methods on an object without repeating its name

### Solution: `with(obj) { ... }`

Use `with` to execute multiple operations on an object without the need to repeat the object name.

```kotlin title="public inline fun <T, R> with(receiver: T, block: T.() -> R): R"
val person = Person("Alice", 25)
val summary = with(person) {
    name = "Alice Smith"
    age = 26
    "Updated: $name is now $age years old"
}
println(summary) // Updated: Alice Smith is now 26 years old
```

!!! note "Use when"
    You want to call multiple methods/properties on a non-null object and return a result.

---

## Problem: I want to configure an object and return the object itself

### Solution: `apply { ... }`

Use `apply` to configure an object (builder pattern) and return the object itself.

```kotlin title="public inline fun <T> T.apply(block: T.() -> Unit): T"
val person = Person("Bob", 40).apply {
    email = "bob@example.com"
    address = "123 Main St"
}
// person now has email and address set
```

!!! note "Use when"
    You want to configure/initialize an object and return the same object (common in builder patterns).

---

## Problem: I want to perform side effects on an object and return the object

### Solution: `also { ... }`

Use `also` to perform additional operations (like logging or validation) while keeping the object in the pipeline.

```kotlin title="public inline fun <T> T.also(block: (T) -> Unit): T"
val numbers = mutableListOf(1, 2, 3)
    .also { println("Original list: $it") }
    .also { it.add(4) }
    .also { println("Modified list: $it") }
// Returns the modified list
```

!!! note "Use when"
    You want to perform side effects (logging, debugging, validation) without changing the object reference.

---

## Problem: I want to transform a nullable object or handle null safely

### Solution: `let { ... }`

Use `let` to work with nullable objects safely and transform them.

```kotlin title="public inline fun <T, R> T.let(block: (T) -> R): R"
val name: String? = "Kotlin"
val length = name?.let {
    println("Name is $it")
    it.length
} ?: 0
println(length) // 6
```

!!! note "Use when"
    You want to execute code only if an object is not null, or to transform an object into another type.

---

## Problem: I want to keep an object only if it meets a condition

### Solution: `takeIf { ... }`

Use `takeIf` to return the object if it satisfies a predicate, otherwise return null.

```kotlin title="public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T?"
val number = 42
val evenNumber = number.takeIf { it % 2 == 0 }
println(evenNumber) // 42

val oddNumber = 41.takeIf { it % 2 == 0 }
println(oddNumber) // null
```

!!! note "Use when"
    You want to filter a single object based on a condition.

---

## Problem: I want to keep an object only if it does NOT meet a condition

### Solution: `takeUnless { ... }`

Use `takeUnless` to return the object if it does NOT satisfy a predicate, otherwise return null.

```kotlin title="public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T?"
val number = 41
val oddNumber = number.takeUnless { it % 2 == 0 }
println(oddNumber) // 41

val evenNumber = 42.takeUnless { it % 2 == 0 }
println(evenNumber) // null
```

!!! note "Use when"
    You want to filter a single object by excluding it when a condition is true (opposite of `takeIf`).

---

## Problem: I want to execute an action multiple times

### Solution: `repeat { ... }`

Use `repeat` to execute a lambda a specified number of times.

```kotlin title="public inline fun repeat(times: Int, action: (Int) -> Unit)"
repeat(3) { index ->
    println("Iteration $index")
}
// Output:
// Iteration 0
// Iteration 1
// Iteration 2
```

!!! note "Use when"
    You need a simple loop that executes a fixed number of times with access to the iteration index.

---

## Quick Reference Table

| Function | Context Object | Return Value | Use Case |
|----------|---------------|--------------|----------|
| `run` | `this` | Lambda result | Execute code in scope, return result |
| `with` | `this` | Lambda result | Operate on non-null object |
| `apply` | `this` | Context object | Configure object (builder pattern) |
| `also` | `it` | Context object | Side effects (logging, validation) |
| `let` | `it` | Lambda result | Null safety, object transformation |
| `takeIf` | `it` | Object or null | Conditional filtering |
| `takeUnless` | `it` | Object or null | Negative conditional filtering |
| `repeat` | N/A | Unit | Execute action N times |

---

## Conclusion

Each Kotlin scope function solves a specific problem. By understanding the problem each one addresses, you'll naturally know which to use in your code.
