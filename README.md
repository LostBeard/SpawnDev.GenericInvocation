# SpawnDev.GenericInvocation



A high-performance reflection utility for **.NET 8, 9, and 10** that bridges the gap between runtime `Type` variables and compile-time generic (`<T>`) method signatures.

By utilizing **one-time compiled Expression Trees** and **cached non-boxing generic bridges**, `SpawnDev.GenericInvocation` completely avoids the heavy runtime performance penalties of `MethodInfo.Invoke` and the memory churn of the C# `dynamic` keyword call-site binder.

## 🚀 Why Use It?

In advanced architecture patterns (like high-throughput Marshallers, Serializers, or JS-Interop pipelines), you often encounter situations where you know an object's destination type *only at runtime*, but you need to feed it into a heavily optimized, strongly-typed method blueprint:

```csharp
// You have this at runtime:
Type runtimeType = typeof(double);

// And you need to invoke this without causing a massive bottleneck:
async ValueTask<T> WriteTypedValue<T>() { ... }
```

### The Old Ways vs. SpawnDev

1. **`MethodInfo.Invoke`**: Extremely slow. Forces the runtime to run heavy security validation checks and **box value types** (like primitives or `ValueTask`) into heap-allocated `object` structures on *every single call*.
2. **`dynamic` Dispatch**: Faster on warm paths due to DLR caching, but introduces silent heap allocations, state-machine tracking overhead, and forces struct-boxing on value-type tasks like `ValueTask<T>`.
3. **`SpawnDev.GenericInvocation`**: Inspects metadata and compiles a native execution lambda expression **exactly once** per type combination. Subsequent hot-path invocations execute at raw, near-handwritten speed with **zero allocation overhead**.

---

## ⚡ Performance Benchmarks

*Target Frameworks: .NET 8, 9, and 10 | 1,000,000 Hot-Path Loop Iterations calling `ValueTask<T>`*

| Implementation Strategy | Cost Per Call | Total Elapsed Time |
| :--- | :--- | :--- |
| Hardcoded Direct Native Call *(Baseline Limit)* | **~0.44 μs** | 447,260 μs |
| **SpawnDev.GenericInvocation** | **~2.41 μs** | **2,414,099 μs** |
| Dynamic Keyword (`dynamic`) Unwrapping | ~4.36 μs | 4,418,700 μs |

*By eliminating call-site binder churn and preventing value-type unboxing, `SpawnDev.GenericInvocation` delivers a **~38% raw performance leap** over standard dynamic async unwrapping strategies.*

---

## 💻 Quick Start & Examples

### 1. Single Type Parameter Invocations
```csharp
using SpawnDev.GenericInvocation;

// Create a delegate targeting your generic method signature
var myMethod = ((Delegate)MyAction<object>).InvokeGenericAsync(typeof(double));

async ValueTask<T> MyAction<T>() 
{
    Console.WriteLine(\$"Executed natively with type: {typeof(T).Name}");
    return default;
}
```

### 2. Multi-Type Parameter Fallbacks
The library intelligently handles single and multi-type cached dimensions dynamically without forcing array mutations on the heap on cache hits:

```csharp
Type[] runtimeTypes = [typeof(string), typeof(int)];

// Dispatches seamlessly into multi-generic signatures
await myMethodGroup.InvokeGenericAsync(runtimeTypes, "Hello World", 42);
```

### 3. Native Task & ValueTask Auto-Unwrapping
Whether your generic target returns a `Task<T>`, `ValueTask<T>`, or synchronous primitives, the under-the-hood expression compiler automatically extracts the inner value seamlessly:

```csharp
// Returns the actual unwrapped type without executing a slow reflection lookup on Task.Result
object? result = await myDelegate.InvokeGenericAsync(typeof(int));
```

---

## ⚙️ Features Under the Hood

* **Zero Array Heap Allocations**: Leverages .NET 10 `Span<object?>` parameter layouts combined with conditional key-cloning to ensure that checking cache dictionaries is entirely garbage-collection silent.
* **Auto-Target Resolution**: Native support for instance methods, static methods, and complex captured local functions alike without triggering delegate targeting binding failures (`Arg_DlgtTargMeth`).
* **Built for the Future**: Designed specifically with lightweight runtime features optimized for high-throughput UI wrappers, framework components, and Blazor WebAssembly environments.

---

## 📄 License
This project is licensed under the [MIT License](LICENSE).
