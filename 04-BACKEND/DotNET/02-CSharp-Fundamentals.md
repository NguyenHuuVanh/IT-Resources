# C# Fundamentals - Cơ bản về C#

## Mục lục

- [Giới thiệu C#](#giới-thiệu-c)
- [Cấu trúc chương trình](#cấu-trúc-chương-trình)
- [Kiểu dữ liệu](#kiểu-dữ-liệu)
- [Biến và Hằng số](#biến-và-hằng-số)
- [Operators](#operators)
- [Control Flow](#control-flow)
- [Arrays và Collections](#arrays-và-collections)
- [Methods](#methods)
- [Classes và Objects](#classes-và-objects)
- [Properties](#properties)
- [Constructors](#constructors)
- [Access Modifiers](#access-modifiers)
- [Static Members](#static-members)
- [Inheritance](#inheritance)
- [Interfaces](#interfaces)
- [Exception Handling](#exception-handling)

---

## Giới thiệu C#

C# (C-Sharp) là ngôn ngữ lập trình **hướng đối tượng**, **type-safe**, được phát triển bởi Microsoft.

### Đặc điểm

- **Strongly Typed**: Kiểm tra kiểu dữ liệu tại compile time
- **Object-Oriented**: Hỗ trợ OOP đầy đủ
- **Modern**: Async/await, LINQ, Pattern matching
- **Memory Safe**: Garbage Collection tự động
- **Cross-platform**: Chạy trên Windows, Linux, macOS

---

## Cấu trúc chương trình

### Traditional (C# 9 trở về trước)

```csharp
using System;

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

### Top-level statements (C# 9+)

```csharp
// Program.cs - Không cần class, namespace
Console.WriteLine("Hello, World!");

// Có thể dùng args
if (args.Length > 0)
{
    Console.WriteLine($"Arguments: {string.Join(", ", args)}");
}
```

### File-scoped namespace (C# 10+)

```csharp
namespace MyApp;  // Không cần braces

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello!");
    }
}
```

---

## Kiểu dữ liệu

### Value Types (Kiểu giá trị)

```csharp
// Integer types
byte b = 255;              // 0 to 255 (8-bit)
sbyte sb = -128;           // -128 to 127
short s = 32767;           // -32,768 to 32,767 (16-bit)
ushort us = 65535;         // 0 to 65,535
int i = 2147483647;        // -2.1B to 2.1B (32-bit) ⭐ Phổ biến
uint ui = 4294967295;      // 0 to 4.2B
long l = 9223372036854775807L;  // 64-bit ⭐ Phổ biến
ulong ul = 18446744073709551615UL;

// Floating-point types
float f = 3.14f;           // 32-bit, ~7 digits precision
double d = 3.14159265359;  // 64-bit, ~15 digits ⭐ Phổ biến
decimal m = 3.14159265359m; // 128-bit, ~28 digits (tiền tệ) ⭐

// Boolean
bool isActive = true;      // true or false

// Character
char c = 'A';              // Single Unicode character

// Struct (custom value type)
DateTime now = DateTime.Now;
TimeSpan duration = TimeSpan.FromHours(2);
Guid id = Guid.NewGuid();
```

### Reference Types (Kiểu tham chiếu)

```csharp
// String
string name = "John Doe";
string empty = string.Empty;
string? nullable = null;   // Nullable reference type

// Object - base type của tất cả
object obj = 42;
object obj2 = "Hello";

// Dynamic - runtime type checking
dynamic dyn = 10;
dyn = "Now I'm a string";

// Arrays
int[] numbers = { 1, 2, 3 };
string[] names = new string[3];

// Class instances
var person = new Person();
```

### Nullable Types

```csharp
// Nullable value types
int? nullableInt = null;
int? anotherInt = 42;

// Check và lấy giá trị
if (nullableInt.HasValue)
{
    int value = nullableInt.Value;
}

// Null-coalescing operator
int result = nullableInt ?? 0;  // 0 nếu null

// Null-coalescing assignment (C# 8+)
nullableInt ??= 10;  // Gán 10 nếu null

// Nullable reference types (C# 8+)
string? nullableName = null;
string nonNullName = "John";  // Không thể null
```

### Type Conversion

```csharp
// Implicit conversion (tự động, an toàn)
int num = 100;
long bigNum = num;      // int → long OK
double d = num;         // int → double OK

// Explicit conversion (casting)
double pi = 3.14159;
int truncated = (int)pi;  // 3 (mất phần thập phân)

// Convert class
string str = "123";
int parsed = Convert.ToInt32(str);
double dbl = Convert.ToDouble("3.14");

// Parse methods
int n = int.Parse("42");
double d2 = double.Parse("3.14");

// TryParse (an toàn, không throw exception)
if (int.TryParse("abc", out int result))
{
    Console.WriteLine(result);
}
else
{
    Console.WriteLine("Invalid number");
}

// as operator (reference types only)
object obj = "Hello";
string? s = obj as string;  // null nếu không convert được

// is operator
if (obj is string text)
{
    Console.WriteLine(text.Length);
}
```

---

## Biến và Hằng số

### Khai báo biến

```csharp
// Explicit type
int age = 25;
string name = "John";

// Implicit type với var (compiler infer type)
var count = 10;           // int
var message = "Hello";    // string
var numbers = new List<int>();  // List<int>

// Multiple declarations
int x = 1, y = 2, z = 3;

// Default values
int defaultInt = default;      // 0
string? defaultStr = default;  // null
bool defaultBool = default;    // false
```

### Hằng số

```csharp
// const - compile-time constant
const double PI = 3.14159;
const string APP_NAME = "MyApp";
const int MAX_SIZE = 100;

// readonly - runtime constant (có thể gán trong constructor)
public class Config
{
    public readonly string ConnectionString;

    public Config(string connStr)
    {
        ConnectionString = connStr;  // OK trong constructor
    }
}

// static readonly - shared across instances
public static readonly DateTime StartTime = DateTime.Now;
```

### String Operations

```csharp
// String declaration
string s1 = "Hello";
string s2 = "World";

// Concatenation
string concat = s1 + " " + s2;           // "Hello World"
string concat2 = string.Concat(s1, s2);  // "HelloWorld"

// String interpolation (C# 6+) ⭐ Recommended
string name = "John";
int age = 25;
string message = $"Name: {name}, Age: {age}";
string formatted = $"Price: {price:C2}";  // Currency format
string padded = $"ID: {id:D5}";           // 00042

// Verbatim string (giữ nguyên format)
string path = @"C:\Users\John\Documents";
string multiLine = @"Line 1
Line 2
Line 3";

// Raw string literals (C# 11+)
string json = """
    {
        "name": "John",
        "age": 25
    }
    """;

// String methods
string text = "  Hello World  ";
text.Trim();                    // "Hello World"
text.ToUpper();                 // "  HELLO WORLD  "
text.ToLower();                 // "  hello world  "
text.Contains("World");         // true
text.StartsWith("  Hello");     // true
text.EndsWith("  ");            // true
text.Replace("World", "C#");    // "  Hello C#  "
text.Split(' ');                // string[]
text.Substring(2, 5);           // "Hello"
text.IndexOf("World");          // 8
text.Length;                    // 15

// String comparison
string.Equals(s1, s2, StringComparison.OrdinalIgnoreCase);
string.Compare(s1, s2);

// StringBuilder (for many concatenations)
var sb = new StringBuilder();
sb.Append("Hello");
sb.Append(" ");
sb.Append("World");
sb.AppendLine("!");
string result = sb.ToString();
```

---

## Operators

### Arithmetic Operators

```csharp
int a = 10, b = 3;

int sum = a + b;        // 13
int diff = a - b;       // 7
int product = a * b;    // 30
int quotient = a / b;   // 3 (integer division)
int remainder = a % b;  // 1 (modulo)

// Với double
double x = 10.0, y = 3.0;
double div = x / y;     // 3.333...

// Increment/Decrement
int i = 5;
i++;    // i = 6 (post-increment)
++i;    // i = 7 (pre-increment)
i--;    // i = 6
--i;    // i = 5

// Compound assignment
i += 5;   // i = i + 5
i -= 3;   // i = i - 3
i *= 2;   // i = i * 2
i /= 2;   // i = i / 2
i %= 3;   // i = i % 3
```

### Comparison Operators

```csharp
int a = 5, b = 10;

bool eq = (a == b);     // false
bool neq = (a != b);    // true
bool lt = (a < b);      // true
bool gt = (a > b);      // false
bool lte = (a <= b);    // true
bool gte = (a >= b);    // false
```

### Logical Operators

```csharp
bool x = true, y = false;

bool and = x && y;      // false (short-circuit AND)
bool or = x || y;       // true (short-circuit OR)
bool not = !x;          // false

// Bitwise operators
int a = 5;  // 0101
int b = 3;  // 0011

int bitwiseAnd = a & b;   // 0001 = 1
int bitwiseOr = a | b;    // 0111 = 7
int bitwiseXor = a ^ b;   // 0110 = 6
int bitwiseNot = ~a;      // 1111...1010
int leftShift = a << 1;   // 1010 = 10
int rightShift = a >> 1;  // 0010 = 2
```

### Null Operators

```csharp
string? name = null;

// Null-coalescing (??)
string displayName = name ?? "Unknown";

// Null-coalescing assignment (??=)
name ??= "Default";

// Null-conditional (?.)
int? length = name?.Length;  // null nếu name null

// Chaining
string? city = person?.Address?.City;

// Null-forgiving (!)
string definitelyNotNull = name!;  // Suppress warning
```

### Ternary Operator

```csharp
int age = 20;
string status = age >= 18 ? "Adult" : "Minor";

// Nested (không khuyến khích)
string grade = score >= 90 ? "A"
             : score >= 80 ? "B"
             : score >= 70 ? "C"
             : "F";
```

---

## Control Flow

### If-Else

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("Excellent");
}
else if (score >= 80)
{
    Console.WriteLine("Good");
}
else if (score >= 70)
{
    Console.WriteLine("Average");
}
else
{
    Console.WriteLine("Need improvement");
}

// Single line (không khuyến khích cho logic phức tạp)
if (score >= 60) Console.WriteLine("Pass");
```

### Switch Statement

```csharp
int day = 3;

switch (day)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    case 3:
    case 4:
    case 5:
        Console.WriteLine("Midweek");
        break;
    case 6:
    case 7:
        Console.WriteLine("Weekend");
        break;
    default:
        Console.WriteLine("Invalid");
        break;
}

// Switch expression (C# 8+) ⭐
string dayName = day switch
{
    1 => "Monday",
    2 => "Tuesday",
    3 => "Wednesday",
    4 => "Thursday",
    5 => "Friday",
    6 or 7 => "Weekend",
    _ => "Invalid"
};
```

### Pattern Matching (C# 7+)

```csharp
object obj = 42;

// Type pattern
if (obj is int number)
{
    Console.WriteLine($"Integer: {number}");
}

// Switch với pattern matching
string Describe(object obj) => obj switch
{
    int i when i > 0 => "Positive integer",
    int i when i < 0 => "Negative integer",
    int => "Zero",
    string s => $"String of length {s.Length}",
    null => "Null",
    _ => "Unknown type"
};

// Property pattern (C# 8+)
bool IsAdult(Person p) => p is { Age: >= 18 };

// Relational pattern (C# 9+)
string GetDiscount(int age) => age switch
{
    < 12 => "Child discount",
    >= 12 and < 18 => "Teen discount",
    >= 65 => "Senior discount",
    _ => "No discount"
};
```

### Loops

```csharp
// for loop
for (int i = 0; i < 10; i++)
{
    Console.WriteLine(i);
}

// foreach loop ⭐
string[] names = { "Alice", "Bob", "Charlie" };
foreach (string name in names)
{
    Console.WriteLine(name);
}

// while loop
int count = 0;
while (count < 5)
{
    Console.WriteLine(count);
    count++;
}

// do-while loop (chạy ít nhất 1 lần)
do
{
    Console.WriteLine(count);
    count--;
} while (count > 0);

// Loop control
for (int i = 0; i < 10; i++)
{
    if (i == 3) continue;  // Skip iteration
    if (i == 7) break;     // Exit loop
    Console.WriteLine(i);
}
```

---

## Arrays và Collections

### Arrays

```csharp
// Single-dimensional array
int[] numbers = new int[5];              // [0, 0, 0, 0, 0]
int[] nums = { 1, 2, 3, 4, 5 };          // Initialized
int[] nums2 = new int[] { 1, 2, 3 };     // Explicit

// Access elements
int first = nums[0];      // 1
int last = nums[^1];      // 5 (C# 8+ index from end)
nums[0] = 10;             // Modify

// Array properties/methods
int length = nums.Length;
Array.Sort(nums);
Array.Reverse(nums);
int index = Array.IndexOf(nums, 3);
bool exists = Array.Exists(nums, x => x > 3);

// Multi-dimensional array
int[,] matrix = new int[3, 3];
int[,] matrix2 = { { 1, 2, 3 }, { 4, 5, 6 }, { 7, 8, 9 } };
int value = matrix2[1, 2];  // 6

// Jagged array (array of arrays)
int[][] jagged = new int[3][];
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5 };
jagged[2] = new int[] { 6 };

// Range và Index (C# 8+)
int[] slice = nums[1..4];    // Elements 1, 2, 3
int[] fromStart = nums[..3]; // First 3 elements
int[] toEnd = nums[2..];     // From index 2 to end
```

### List<T>

```csharp
// Create
List<string> names = new List<string>();
List<int> numbers = new() { 1, 2, 3 };  // Target-typed new

// Add elements
names.Add("Alice");
names.AddRange(new[] { "Bob", "Charlie" });
names.Insert(1, "David");  // Insert at index

// Access
string first = names[0];
string last = names[^1];

// Remove
names.Remove("Bob");       // Remove by value
names.RemoveAt(0);         // Remove by index
names.RemoveAll(n => n.StartsWith("A"));  // Remove matching
names.Clear();             // Remove all

// Query
bool contains = names.Contains("Alice");
int index = names.IndexOf("Bob");
string? found = names.Find(n => n.Length > 5);
List<string> filtered = names.FindAll(n => n.Length > 3);

// Properties
int count = names.Count;
int capacity = names.Capacity;

// Convert
string[] array = names.ToArray();
```

### Dictionary<TKey, TValue>

```csharp
// Create
var dict = new Dictionary<string, int>();
var ages = new Dictionary<string, int>
{
    { "Alice", 25 },
    { "Bob", 30 },
    ["Charlie"] = 35  // Index initializer
};

// Add/Update
dict["key"] = 100;           // Add or update
dict.Add("newKey", 200);     // Add only (throws if exists)
dict.TryAdd("key", 300);     // Add if not exists

// Access
int age = ages["Alice"];     // Throws if not found
if (ages.TryGetValue("Bob", out int bobAge))
{
    Console.WriteLine(bobAge);
}

// Check
bool hasKey = ages.ContainsKey("Alice");
bool hasValue = ages.ContainsValue(25);

// Remove
ages.Remove("Alice");

// Iterate
foreach (KeyValuePair<string, int> kvp in ages)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}

foreach (var (name, age2) in ages)  // Deconstruct
{
    Console.WriteLine($"{name}: {age2}");
}

// Properties
var keys = ages.Keys;
var values = ages.Values;
int count = ages.Count;
```

### Other Collections

```csharp
// HashSet<T> - Unique values, fast lookup
var set = new HashSet<int> { 1, 2, 3 };
set.Add(2);  // Ignored (already exists)
bool contains = set.Contains(2);  // O(1)

// Set operations
set.UnionWith(new[] { 3, 4, 5 });
set.IntersectWith(new[] { 2, 3 });
set.ExceptWith(new[] { 1 });

// Queue<T> - FIFO
var queue = new Queue<string>();
queue.Enqueue("First");
queue.Enqueue("Second");
string first = queue.Dequeue();  // "First"
string peek = queue.Peek();      // "Second" (không remove)

// Stack<T> - LIFO
var stack = new Stack<int>();
stack.Push(1);
stack.Push(2);
int top = stack.Pop();   // 2
int peek2 = stack.Peek(); // 1

// LinkedList<T>
var linked = new LinkedList<int>();
linked.AddFirst(1);
linked.AddLast(3);
linked.AddAfter(linked.First!, 2);

// SortedList<TKey, TValue> - Sorted by key
var sorted = new SortedList<string, int>
{
    { "banana", 2 },
    { "apple", 1 }
};
// Tự động sort: apple, banana

// SortedSet<T> - Sorted unique values
var sortedSet = new SortedSet<int> { 3, 1, 2 };
// Tự động sort: 1, 2, 3
```

---

## Methods

### Method Declaration

```csharp
// Basic method
public int Add(int a, int b)
{
    return a + b;
}

// Expression-bodied method (C# 6+)
public int Multiply(int a, int b) => a * b;

// Void method
public void PrintMessage(string message)
{
    Console.WriteLine(message);
}

// Static method
public static double CalculateArea(double radius)
{
    return Math.PI * radius * radius;
}
```

### Parameters

```csharp
// Optional parameters (phải ở cuối)
public void Greet(string name, string greeting = "Hello")
{
    Console.WriteLine($"{greeting}, {name}!");
}
Greet("John");              // "Hello, John!"
Greet("John", "Hi");        // "Hi, John!"

// Named arguments
Greet(greeting: "Hey", name: "Alice");

// params - Variable number of arguments
public int Sum(params int[] numbers)
{
    return numbers.Sum();
}
Sum(1, 2, 3);           // 6
Sum(1, 2, 3, 4, 5);     // 15

// ref - Pass by reference (must be initialized)
public void Increment(ref int value)
{
    value++;
}
int num = 5;
Increment(ref num);  // num = 6

// out - Output parameter (không cần initialize)
public bool TryParse(string input, out int result)
{
    return int.TryParse(input, out result);
}
if (TryParse("123", out int value))
{
    Console.WriteLine(value);
}

// in - Read-only reference (performance)
public double Distance(in Point p1, in Point p2)
{
    // p1 và p2 không thể modify
    return Math.Sqrt(Math.Pow(p2.X - p1.X, 2) + Math.Pow(p2.Y - p1.Y, 2));
}
```

### Method Overloading

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
    public string Add(string a, string b) => a + b;
}

var calc = new Calculator();
calc.Add(1, 2);           // int version
calc.Add(1.5, 2.5);       // double version
calc.Add(1, 2, 3);        // 3 params version
calc.Add("Hello", "World"); // string version
```

### Local Functions (C# 7+)

```csharp
public int Factorial(int n)
{
    // Local function
    int Calculate(int num)
    {
        if (num <= 1) return 1;
        return num * Calculate(num - 1);
    }

    if (n < 0) throw new ArgumentException("Must be non-negative");
    return Calculate(n);
}

// Static local function (C# 8+) - không capture outer variables
public int Process(int[] numbers)
{
    static int Square(int x) => x * x;
    return numbers.Sum(Square);
}
```

---

## Classes và Objects

### Class Declaration

```csharp
public class Person
{
    // Fields
    private string _name;
    private int _age;

    // Properties
    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }

    // Auto-property
    public string Email { get; set; }

    // Read-only property
    public DateTime CreatedAt { get; } = DateTime.Now;

    // Methods
    public void Introduce()
    {
        Console.WriteLine($"Hi, I'm {Name}");
    }
}

// Create object
Person person = new Person();
person.Name = "John";
person.Introduce();

// Object initializer
var person2 = new Person
{
    Name = "Alice",
    Email = "alice@example.com"
};

// Target-typed new (C# 9+)
Person person3 = new()
{
    Name = "Bob"
};
```

### Records (C# 9+)

```csharp
// Record - immutable reference type với value equality
public record Person(string Name, int Age);

// Sử dụng
var person = new Person("John", 25);
var person2 = new Person("John", 25);

Console.WriteLine(person == person2);  // true (value equality)
Console.WriteLine(person);  // Person { Name = John, Age = 25 }

// With expression (tạo copy với thay đổi)
var older = person with { Age = 26 };

// Record với additional members
public record Employee(string Name, int Age)
{
    public string Department { get; init; }

    public void Work() => Console.WriteLine($"{Name} is working");
}

// Record struct (C# 10+)
public record struct Point(int X, int Y);
```

---

## Properties

```csharp
public class Product
{
    // Full property với backing field
    private decimal _price;
    public decimal Price
    {
        get { return _price; }
        set
        {
            if (value < 0)
                throw new ArgumentException("Price cannot be negative");
            _price = value;
        }
    }

    // Auto-implemented property
    public string Name { get; set; }

    // Read-only auto property
    public int Id { get; }

    // Init-only property (C# 9+)
    public string SKU { get; init; }

    // Computed property
    public decimal PriceWithTax => Price * 1.1m;

    // Property với different access levels
    public string Status { get; private set; }

    // Required property (C# 11+)
    public required string Category { get; set; }
}

// Sử dụng
var product = new Product
{
    Name = "Laptop",
    SKU = "LAP-001",  // init-only, chỉ set được ở đây
    Category = "Electronics"  // required
};
// product.SKU = "NEW";  // Error! init-only
```

---

## Constructors

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Country { get; set; }

    // Default constructor
    public Person()
    {
        Name = "Unknown";
        Age = 0;
        Country = "Unknown";
    }

    // Parameterized constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
        Country = "Unknown";
    }

    // Constructor chaining
    public Person(string name, int age, string country) : this(name, age)
    {
        Country = country;
    }

    // Primary constructor (C# 12+)
    // public class Person(string name, int age);
}

// Static constructor - chạy 1 lần khi class được load
public class Config
{
    public static string ConnectionString { get; }

    static Config()
    {
        ConnectionString = LoadFromFile();
    }
}
```

---

## Access Modifiers

```csharp
public class Example
{
    public int PublicField;           // Accessible everywhere
    private int _privateField;        // Only in this class
    protected int ProtectedField;     // This class + derived classes
    internal int InternalField;       // Same assembly only
    protected internal int ProtectedInternalField;  // Same assembly OR derived
    private protected int PrivateProtectedField;    // Same assembly AND derived
}
```

| Modifier           | Same Class | Derived (Same Assembly) | Same Assembly | Derived (Other Assembly) | Other Assembly |
| ------------------ | ---------- | ----------------------- | ------------- | ------------------------ | -------------- |
| public             | ✓          | ✓                       | ✓             | ✓                        | ✓              |
| private            | ✓          | ✗                       | ✗             | ✗                        | ✗              |
| protected          | ✓          | ✓                       | ✗             | ✓                        | ✗              |
| internal           | ✓          | ✓                       | ✓             | ✗                        | ✗              |
| protected internal | ✓          | ✓                       | ✓             | ✓                        | ✗              |
| private protected  | ✓          | ✓                       | ✗             | ✗                        | ✗              |

---

## Static Members

```csharp
public class MathHelper
{
    // Static field
    public static double PI = 3.14159;

    // Static property
    public static int CalculationCount { get; private set; }

    // Static method
    public static double CircleArea(double radius)
    {
        CalculationCount++;
        return PI * radius * radius;
    }

    // Static constructor
    static MathHelper()
    {
        Console.WriteLine("MathHelper initialized");
    }
}

// Sử dụng - không cần tạo instance
double area = MathHelper.CircleArea(5);
Console.WriteLine(MathHelper.PI);

// Static class - không thể instantiate
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string? str)
    {
        return string.IsNullOrEmpty(str);
    }
}
```

---

## Inheritance

```csharp
// Base class
public class Animal
{
    public string Name { get; set; }

    public virtual void Speak()
    {
        Console.WriteLine("Animal speaks");
    }

    public void Eat()
    {
        Console.WriteLine($"{Name} is eating");
    }
}

// Derived class
public class Dog : Animal
{
    public string Breed { get; set; }

    // Override virtual method
    public override void Speak()
    {
        Console.WriteLine("Woof!");
    }

    // New method specific to Dog
    public void Fetch()
    {
        Console.WriteLine($"{Name} is fetching");
    }
}

// Sealed class - không thể inherit
public sealed class FinalClass { }

// Abstract class - không thể instantiate
public abstract class Shape
{
    public abstract double Area();  // Must be implemented

    public virtual void Draw()      // Can be overridden
    {
        Console.WriteLine("Drawing shape");
    }
}

public class Circle : Shape
{
    public double Radius { get; set; }

    public override double Area() => Math.PI * Radius * Radius;
}
```

---

## Interfaces

```csharp
// Interface declaration
public interface IAnimal
{
    string Name { get; set; }
    void Speak();

    // Default implementation (C# 8+)
    void Sleep() => Console.WriteLine("Zzz...");
}

public interface IFlyable
{
    void Fly();
}

// Implement multiple interfaces
public class Bird : IAnimal, IFlyable
{
    public string Name { get; set; }

    public void Speak() => Console.WriteLine("Tweet!");
    public void Fly() => Console.WriteLine("Flying...");
}

// Interface as type
IAnimal animal = new Bird { Name = "Tweety" };
animal.Speak();

// Check interface
if (animal is IFlyable flyable)
{
    flyable.Fly();
}
```

---

## Exception Handling

```csharp
// Try-catch-finally
try
{
    int result = 10 / 0;
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Cannot divide by zero: {ex.Message}");
}
catch (Exception ex)  // Catch all other exceptions
{
    Console.WriteLine($"Error: {ex.Message}");
}
finally
{
    Console.WriteLine("Cleanup code runs always");
}

// Throw exception
public void ValidateAge(int age)
{
    if (age < 0)
        throw new ArgumentException("Age cannot be negative", nameof(age));

    if (age > 150)
        throw new ArgumentOutOfRangeException(nameof(age), "Age is too high");
}

// Custom exception
public class BusinessException : Exception
{
    public string ErrorCode { get; }

    public BusinessException(string message, string errorCode)
        : base(message)
    {
        ErrorCode = errorCode;
    }
}

// Exception filters (C# 6+)
try
{
    // code
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    Console.WriteLine("Resource not found");
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.Unauthorized)
{
    Console.WriteLine("Unauthorized");
}

// Using statement (auto dispose)
using (var file = new StreamReader("file.txt"))
{
    string content = file.ReadToEnd();
}

// Using declaration (C# 8+)
using var file2 = new StreamReader("file.txt");
string content2 = file2.ReadToEnd();
// Automatically disposed at end of scope
```

---

## Tóm tắt

| Concept         | Mô tả                                       |
| --------------- | ------------------------------------------- |
| Value Types     | int, double, bool, struct - stored on stack |
| Reference Types | class, string, array - stored on heap       |
| var             | Implicit typing, compiler infers type       |
| const           | Compile-time constant                       |
| readonly        | Runtime constant                            |
| Properties      | Encapsulated fields với get/set             |
| Methods         | Functions trong class                       |
| Inheritance     | Kế thừa từ base class                       |
| Interface       | Contract định nghĩa behavior                |
| Abstract        | Không thể instantiate, phải inherit         |
| Sealed          | Không thể inherit                           |
| Static          | Shared across all instances                 |
