# C# Advanced - C# Nâng cao

## Mục lục

- [Generics](#generics)
- [Delegates và Events](#delegates-và-events)
- [Lambda Expressions](#lambda-expressions)
- [LINQ](#linq)
- [Async/Await](#asyncawait)
- [Extension Methods](#extension-methods)
- [Attributes](#attributes)
- [Reflection](#reflection)
- [Tuples và Deconstruction](#tuples-và-deconstruction)
- [Pattern Matching Advanced](#pattern-matching-advanced)
- [Span và Memory](#span-và-memory)
- [Nullable Reference Types](#nullable-reference-types)

---

## Generics

Generics cho phép viết code type-safe mà không cần biết trước kiểu dữ liệu cụ thể.

### Generic Class

```csharp
// Generic class
public class Box<T>
{
    private T _content;

    public void Put(T item) => _content = item;
    public T Get() => _content;
}

// Sử dụng
var intBox = new Box<int>();
intBox.Put(42);
int value = intBox.Get();

var stringBox = new Box<string>();
stringBox.Put("Hello");
string text = stringBox.Get();

// Multiple type parameters
public class Pair<TKey, TValue>
{
    public TKey Key { get; set; }
    public TValue Value { get; set; }
}

var pair = new Pair<string, int> { Key = "age", Value = 25 };
```

### Generic Methods

```csharp
public class Utilities
{
    // Generic method
    public T GetDefault<T>() => default(T);

    public void Swap<T>(ref T a, ref T b)
    {
        T temp = a;
        a = b;
        b = temp;
    }

    // Generic với multiple types
    public TOutput Convert<TInput, TOutput>(TInput input, Func<TInput, TOutput> converter)
    {
        return converter(input);
    }
}

// Sử dụng
var utils = new Utilities();
int defaultInt = utils.GetDefault<int>();  // 0
string? defaultStr = utils.GetDefault<string>();  // null

int x = 1, y = 2;
utils.Swap(ref x, ref y);  // x=2, y=1
```

### Generic Constraints

```csharp
// where T : struct - Value type only
public class ValueContainer<T> where T : struct
{
    public T Value { get; set; }
}

// where T : class - Reference type only
public class ReferenceContainer<T> where T : class
{
    public T? Value { get; set; }
}

// where T : new() - Must have parameterless constructor
public T CreateInstance<T>() where T : new()
{
    return new T();
}

// where T : BaseClass - Must inherit from BaseClass
public class AnimalShelter<T> where T : Animal
{
    public List<T> Animals { get; } = new();
}

// where T : IInterface - Must implement interface
public class Repository<T> where T : IEntity
{
    public void Save(T entity) { }
}

// Multiple constraints
public class Service<T> where T : class, IDisposable, new()
{
    public T Create() => new T();
}

// Constraint on multiple type parameters
public class Mapper<TSource, TDest>
    where TSource : class
    where TDest : class, new()
{
    public TDest Map(TSource source) => new TDest();
}
```

### Covariance và Contravariance

```csharp
// Covariance (out) - Có thể dùng derived type thay base type
public interface IProducer<out T>
{
    T Produce();
}

// Contravariance (in) - Có thể dùng base type thay derived type
public interface IConsumer<in T>
{
    void Consume(T item);
}

// Ví dụ
IProducer<Animal> animalProducer = new DogProducer();  // Dog : Animal
IConsumer<Dog> dogConsumer = new AnimalConsumer();    // Animal is base of Dog
```

---

## Delegates và Events

### Delegates

Delegate là type-safe function pointer.

```csharp
// Declare delegate
public delegate int MathOperation(int a, int b);

// Methods matching delegate signature
public static int Add(int a, int b) => a + b;
public static int Multiply(int a, int b) => a * b;

// Sử dụng
MathOperation operation = Add;
int result = operation(5, 3);  // 8

operation = Multiply;
result = operation(5, 3);  // 15

// Multicast delegate
MathOperation combined = Add;
combined += Multiply;  // Cả 2 methods được gọi

// Built-in delegates
Action<string> print = Console.WriteLine;  // void return
Func<int, int, int> add = (a, b) => a + b;  // returns int
Predicate<int> isPositive = x => x > 0;     // returns bool
```

### Events

```csharp
public class Button
{
    // Declare event
    public event EventHandler? Clicked;

    // Custom event args
    public event EventHandler<ButtonClickedEventArgs>? ClickedWithArgs;

    public void Click()
    {
        // Raise event
        Clicked?.Invoke(this, EventArgs.Empty);
        ClickedWithArgs?.Invoke(this, new ButtonClickedEventArgs { X = 10, Y = 20 });
    }
}

public class ButtonClickedEventArgs : EventArgs
{
    public int X { get; set; }
    public int Y { get; set; }
}

// Subscribe to event
var button = new Button();
button.Clicked += (sender, e) => Console.WriteLine("Button clicked!");
button.ClickedWithArgs += (sender, e) => Console.WriteLine($"Clicked at ({e.X}, {e.Y})");

button.Click();

// Unsubscribe
button.Clicked -= HandleClick;
```

---

## Lambda Expressions

```csharp
// Expression lambda
Func<int, int> square = x => x * x;
Func<int, int, int> add = (a, b) => a + b;

// Statement lambda
Func<int, int> factorial = n =>
{
    int result = 1;
    for (int i = 2; i <= n; i++)
        result *= i;
    return result;
};

// Lambda với explicit types
Func<int, string> convert = (int x) => x.ToString();

// Discard parameter
button.Clicked += (_, _) => Console.WriteLine("Clicked");

// Lambda trong LINQ
var numbers = new[] { 1, 2, 3, 4, 5 };
var evens = numbers.Where(x => x % 2 == 0);
var doubled = numbers.Select(x => x * 2);

// Static lambda (C# 9+) - không capture outer variables
Func<int, int> staticSquare = static x => x * x;

// Lambda với attributes (C# 10+)
var parse = [Description("Parses string to int")] (string s) => int.Parse(s);
```

---

## LINQ

LINQ (Language Integrated Query) cho phép query data từ nhiều nguồn khác nhau.

### Query Syntax vs Method Syntax

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Query syntax
var evenQuery = from n in numbers
                where n % 2 == 0
                orderby n descending
                select n;

// Method syntax (phổ biến hơn)
var evenMethod = numbers
    .Where(n => n % 2 == 0)
    .OrderByDescending(n => n);
```

### LINQ Methods

```csharp
var products = new List<Product>
{
    new Product { Id = 1, Name = "Laptop", Price = 1000, Category = "Electronics" },
    new Product { Id = 2, Name = "Phone", Price = 500, Category = "Electronics" },
    new Product { Id = 3, Name = "Shirt", Price = 50, Category = "Clothing" },
    new Product { Id = 4, Name = "Book", Price = 20, Category = "Books" }
};

// Filtering
var expensive = products.Where(p => p.Price > 100);
var electronics = products.Where(p => p.Category == "Electronics");

// Projection
var names = products.Select(p => p.Name);
var projected = products.Select(p => new { p.Name, p.Price });

// Ordering
var byPrice = products.OrderBy(p => p.Price);
var byPriceDesc = products.OrderByDescending(p => p.Price);
var multiSort = products.OrderBy(p => p.Category).ThenBy(p => p.Name);

// Aggregation
int count = products.Count();
int countExpensive = products.Count(p => p.Price > 100);
decimal sum = products.Sum(p => p.Price);
decimal avg = products.Average(p => p.Price);
decimal min = products.Min(p => p.Price);
decimal max = products.Max(p => p.Price);

// Element access
var first = products.First();
var firstOrDefault = products.FirstOrDefault(p => p.Price > 2000);  // null if not found
var single = products.Single(p => p.Id == 1);  // throws if 0 or >1
var last = products.Last();
var elementAt = products.ElementAt(2);

// Quantifiers
bool any = products.Any(p => p.Price > 500);
bool all = products.All(p => p.Price > 0);
bool contains = products.Contains(product);

// Grouping
var byCategory = products.GroupBy(p => p.Category);
foreach (var group in byCategory)
{
    Console.WriteLine($"{group.Key}: {group.Count()} items");
}

// Join
var orders = new List<Order> { /* ... */ };
var joined = products.Join(
    orders,
    p => p.Id,
    o => o.ProductId,
    (p, o) => new { p.Name, o.Quantity }
);

// Set operations
var set1 = new[] { 1, 2, 3 };
var set2 = new[] { 2, 3, 4 };
var union = set1.Union(set2);           // 1, 2, 3, 4
var intersect = set1.Intersect(set2);   // 2, 3
var except = set1.Except(set2);         // 1
var distinct = set1.Concat(set2).Distinct();

// Partitioning
var firstThree = products.Take(3);
var skipTwo = products.Skip(2);
var page = products.Skip(10).Take(10);  // Pagination
var takeWhile = numbers.TakeWhile(n => n < 5);
var skipWhile = numbers.SkipWhile(n => n < 5);

// Conversion
var list = products.ToList();
var array = products.ToArray();
var dict = products.ToDictionary(p => p.Id);
var lookup = products.ToLookup(p => p.Category);
var hashSet = products.ToHashSet();

// Deferred vs Immediate execution
var deferred = products.Where(p => p.Price > 100);  // Chưa execute
var immediate = products.Where(p => p.Price > 100).ToList();  // Execute ngay
```

### LINQ với Anonymous Types

```csharp
var result = products
    .Where(p => p.Price > 50)
    .Select(p => new
    {
        p.Name,
        p.Price,
        Discounted = p.Price * 0.9m,
        Category = p.Category.ToUpper()
    })
    .OrderBy(x => x.Discounted);

foreach (var item in result)
{
    Console.WriteLine($"{item.Name}: {item.Discounted:C}");
}
```

---

## Async/Await

### Basic Async

```csharp
// Async method
public async Task<string> GetDataAsync()
{
    // Simulate async operation
    await Task.Delay(1000);
    return "Data loaded";
}

// Calling async method
public async Task ProcessAsync()
{
    string data = await GetDataAsync();
    Console.WriteLine(data);
}

// Async với HttpClient
public async Task<string> FetchUrlAsync(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);
}

// Multiple async operations
public async Task ProcessMultipleAsync()
{
    // Sequential
    var result1 = await GetDataAsync();
    var result2 = await GetDataAsync();

    // Parallel
    var task1 = GetDataAsync();
    var task2 = GetDataAsync();
    await Task.WhenAll(task1, task2);

    // Get results
    string[] results = await Task.WhenAll(task1, task2);
}
```

### Task Return Types

```csharp
// Task - async void equivalent
public async Task DoWorkAsync()
{
    await Task.Delay(1000);
    Console.WriteLine("Done");
}

// Task<T> - returns value
public async Task<int> CalculateAsync()
{
    await Task.Delay(1000);
    return 42;
}

// ValueTask<T> - better performance for sync completion
public async ValueTask<int> GetCachedValueAsync()
{
    if (_cache.TryGetValue("key", out int value))
        return value;  // No allocation

    return await LoadFromDatabaseAsync();
}

// IAsyncEnumerable<T> - async streams (C# 8+)
public async IAsyncEnumerable<int> GenerateNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}

// Consuming async stream
await foreach (var number in GenerateNumbersAsync())
{
    Console.WriteLine(number);
}
```

### Exception Handling in Async

```csharp
public async Task HandleExceptionsAsync()
{
    try
    {
        await SomeAsyncOperation();
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"HTTP Error: {ex.Message}");
    }
    catch (TaskCanceledException)
    {
        Console.WriteLine("Operation was cancelled");
    }
}

// Handling multiple task exceptions
public async Task HandleMultipleAsync()
{
    var tasks = new[]
    {
        Task.Run(() => throw new InvalidOperationException()),
        Task.Run(() => throw new ArgumentException())
    };

    try
    {
        await Task.WhenAll(tasks);
    }
    catch (Exception)
    {
        foreach (var task in tasks)
        {
            if (task.IsFaulted)
                Console.WriteLine(task.Exception?.InnerException?.Message);
        }
    }
}
```

### Cancellation

```csharp
public async Task LongRunningOperationAsync(CancellationToken cancellationToken)
{
    for (int i = 0; i < 100; i++)
    {
        cancellationToken.ThrowIfCancellationRequested();
        await Task.Delay(100, cancellationToken);
        Console.WriteLine($"Step {i}");
    }
}

// Sử dụng
var cts = new CancellationTokenSource();

// Cancel after 5 seconds
cts.CancelAfter(TimeSpan.FromSeconds(5));

try
{
    await LongRunningOperationAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Operation cancelled");
}

// Manual cancel
cts.Cancel();
```

---

## Extension Methods

Extension methods cho phép "thêm" methods vào existing types mà không modify source code.

```csharp
public static class StringExtensions
{
    // Extension method cho string
    public static bool IsNullOrEmpty(this string? str)
    {
        return string.IsNullOrEmpty(str);
    }

    public static string Truncate(this string str, int maxLength)
    {
        if (str.Length <= maxLength) return str;
        return str.Substring(0, maxLength) + "...";
    }

    public static int WordCount(this string str)
    {
        return str.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
    }
}

public static class EnumerableExtensions
{
    // Extension method cho IEnumerable<T>
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T?> source) where T : class
    {
        return source.Where(x => x != null)!;
    }

    public static void ForEach<T>(this IEnumerable<T> source, Action<T> action)
    {
        foreach (var item in source)
            action(item);
    }
}

// Sử dụng
string text = "Hello World";
bool isEmpty = text.IsNullOrEmpty();  // false
string truncated = text.Truncate(5);  // "Hello..."
int words = text.WordCount();         // 2

var numbers = new[] { 1, 2, 3 };
numbers.ForEach(n => Console.WriteLine(n));
```

---

## Attributes

Attributes cung cấp metadata cho code.

```csharp
// Built-in attributes
[Obsolete("Use NewMethod instead", error: false)]
public void OldMethod() { }

[Serializable]
public class MyClass { }

[Conditional("DEBUG")]
public void DebugOnly() { }

// Custom attribute
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true)]
public class AuthorAttribute : Attribute
{
    public string Name { get; }
    public string Version { get; set; } = "1.0";

    public AuthorAttribute(string name)
    {
        Name = name;
    }
}

// Sử dụng custom attribute
[Author("John Doe", Version = "2.0")]
[Author("Jane Smith")]
public class MyService
{
    [Author("John Doe")]
    public void DoWork() { }
}

// Đọc attributes qua Reflection
var type = typeof(MyService);
var attributes = type.GetCustomAttributes<AuthorAttribute>();
foreach (var attr in attributes)
{
    Console.WriteLine($"Author: {attr.Name}, Version: {attr.Version}");
}
```

---

## Reflection

Reflection cho phép inspect và manipulate types at runtime.

```csharp
// Get type information
Type type = typeof(Person);
Type type2 = person.GetType();
Type type3 = Type.GetType("MyNamespace.Person");

// Get properties
PropertyInfo[] properties = type.GetProperties();
foreach (var prop in properties)
{
    Console.WriteLine($"{prop.Name}: {prop.PropertyType}");
}

// Get methods
MethodInfo[] methods = type.GetMethods();
MethodInfo specificMethod = type.GetMethod("ToString");

// Create instance
object instance = Activator.CreateInstance(type);
object instanceWithArgs = Activator.CreateInstance(type, "John", 25);

// Get/Set property value
PropertyInfo nameProp = type.GetProperty("Name");
nameProp.SetValue(instance, "Alice");
string name = (string)nameProp.GetValue(instance);

// Invoke method
MethodInfo method = type.GetMethod("Introduce");
method.Invoke(instance, null);

// Generic type
Type genericType = typeof(List<>);
Type constructedType = genericType.MakeGenericType(typeof(int));
object list = Activator.CreateInstance(constructedType);

// Check attributes
bool hasAttribute = type.IsDefined(typeof(SerializableAttribute), false);
var attrs = type.GetCustomAttributes<AuthorAttribute>();
```

---

## Tuples và Deconstruction

### Tuples

```csharp
// ValueTuple (C# 7+)
var tuple = (1, "Hello", true);
Console.WriteLine(tuple.Item1);  // 1

// Named tuple elements
var person = (Name: "John", Age: 25);
Console.WriteLine(person.Name);  // John

// Tuple as return type
public (string Name, int Age) GetPerson()
{
    return ("Alice", 30);
}

var result = GetPerson();
Console.WriteLine($"{result.Name}, {result.Age}");

// Tuple với explicit types
(string, int) tuple2 = ("Hello", 42);
(string Name, int Value) tuple3 = ("World", 100);
```

### Deconstruction

```csharp
// Deconstruct tuple
var (name, age) = GetPerson();
Console.WriteLine($"{name} is {age}");

// Discard với _
var (firstName, _) = GetPerson();

// Deconstruct trong foreach
var people = new[] { ("John", 25), ("Alice", 30) };
foreach (var (n, a) in people)
{
    Console.WriteLine($"{n}: {a}");
}

// Custom deconstruction
public class Point
{
    public int X { get; set; }
    public int Y { get; set; }

    public void Deconstruct(out int x, out int y)
    {
        x = X;
        y = Y;
    }
}

var point = new Point { X = 10, Y = 20 };
var (x, y) = point;

// Deconstruct extension method
public static class Extensions
{
    public static void Deconstruct(this KeyValuePair<string, int> kvp,
        out string key, out int value)
    {
        key = kvp.Key;
        value = kvp.Value;
    }
}

foreach (var (key, value) in dictionary)
{
    Console.WriteLine($"{key}: {value}");
}
```

---

## Pattern Matching Advanced

### Switch Expression với Patterns

```csharp
// Type pattern
string Describe(object obj) => obj switch
{
    int i => $"Integer: {i}",
    string s => $"String of length {s.Length}",
    IEnumerable<int> list => $"List with {list.Count()} items",
    null => "Null value",
    _ => "Unknown type"
};

// Property pattern
string GetDiscount(Person person) => person switch
{
    { Age: < 18 } => "Child discount",
    { Age: >= 65 } => "Senior discount",
    { IsMember: true, Age: >= 18 and < 65 } => "Member discount",
    _ => "No discount"
};

// Positional pattern (với Deconstruct)
string Quadrant(Point point) => point switch
{
    (0, 0) => "Origin",
    (> 0, > 0) => "Quadrant 1",
    (< 0, > 0) => "Quadrant 2",
    (< 0, < 0) => "Quadrant 3",
    (> 0, < 0) => "Quadrant 4",
    (_, 0) or (0, _) => "On axis"
};

// Relational pattern
string GetGrade(int score) => score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    >= 60 => "D",
    _ => "F"
};

// Logical patterns (and, or, not)
bool IsLetter(char c) => c is (>= 'a' and <= 'z') or (>= 'A' and <= 'Z');
bool IsNotNull(object? obj) => obj is not null;
```

### List Patterns (C# 11+)

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Exact match
bool isOneTwoThree = numbers is [1, 2, 3, 4, 5];

// Slice pattern
bool startsWithOne = numbers is [1, ..];
bool endsWithFive = numbers is [.., 5];
bool startsOneEndsThree = numbers is [1, .., 5];

// Capture slice
if (numbers is [var first, .. var middle, var last])
{
    Console.WriteLine($"First: {first}, Last: {last}");
    Console.WriteLine($"Middle: {string.Join(", ", middle)}");
}

// In switch
string Describe(int[] arr) => arr switch
{
    [] => "Empty",
    [var single] => $"Single: {single}",
    [var first, var second] => $"Pair: {first}, {second}",
    [var first, .., var last] => $"First: {first}, Last: {last}",
};
```

---

## Span<T> và Memory<T>

Span và Memory cho phép làm việc với memory hiệu quả, không allocation.

```csharp
// Span<T> - stack-only, không thể dùng trong async
int[] array = { 1, 2, 3, 4, 5 };
Span<int> span = array;
Span<int> slice = span.Slice(1, 3);  // [2, 3, 4] - no copy!

// Modify through span
slice[0] = 100;  // array[1] = 100

// ReadOnlySpan
ReadOnlySpan<char> text = "Hello World".AsSpan();
ReadOnlySpan<char> hello = text.Slice(0, 5);

// Span với stackalloc
Span<int> stackArray = stackalloc int[100];

// Memory<T> - có thể dùng trong async
Memory<int> memory = array;
await ProcessAsync(memory);

async Task ProcessAsync(Memory<int> data)
{
    await Task.Delay(100);
    var span = data.Span;
    // Process span
}

// String parsing không allocation
ReadOnlySpan<char> line = "John,25,Engineer".AsSpan();
int firstComma = line.IndexOf(',');
ReadOnlySpan<char> name = line.Slice(0, firstComma);
```

---

## Nullable Reference Types

C# 8+ có nullable reference types để tránh NullReferenceException.

```csharp
// Enable trong .csproj
// <Nullable>enable</Nullable>

// Non-nullable (default)
string name = "John";  // Không thể null
// name = null;  // Warning!

// Nullable reference type
string? nullableName = null;  // OK

// Null checks
if (nullableName != null)
{
    Console.WriteLine(nullableName.Length);  // Safe
}

// Null-forgiving operator
string definitelyNotNull = nullableName!;  // Suppress warning

// Null-conditional
int? length = nullableName?.Length;

// Null-coalescing
string displayName = nullableName ?? "Unknown";

// Pattern matching
if (nullableName is { Length: > 0 } validName)
{
    Console.WriteLine(validName);
}

// Nullable attributes
public class Example
{
    [NotNull]
    public string? GetValue() => "Always returns non-null";

    public void Process([NotNullWhen(true)] string? value, out bool success)
    {
        success = value != null;
    }

    [return: MaybeNull]
    public T Find<T>(Predicate<T> predicate) => default;
}
```

---

## Tóm tắt

| Feature                  | Mô tả                              | Version |
| ------------------------ | ---------------------------------- | ------- |
| Generics                 | Type-safe code với type parameters | C# 2.0  |
| LINQ                     | Query syntax cho collections       | C# 3.0  |
| Async/Await              | Asynchronous programming           | C# 5.0  |
| Null-conditional         | ?. và ?? operators                 | C# 6.0  |
| Pattern Matching         | is, switch expressions             | C# 7.0+ |
| Tuples                   | ValueTuple với named elements      | C# 7.0  |
| Nullable Reference Types | Prevent null reference exceptions  | C# 8.0  |
| Records                  | Immutable reference types          | C# 9.0  |
| List Patterns            | Pattern matching cho arrays        | C# 11.0 |
| Primary Constructors     | Simplified class declarations      | C# 12.0 |
