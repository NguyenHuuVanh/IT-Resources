# SOLID Principles - Nguyên Tắc Thiết Kế Hướng Đối Tượng

## Giới Thiệu

SOLID là 5 nguyên tắc thiết kế hướng đối tượng được Robert C. Martin đề xuất, giúp code dễ bảo trì, mở rộng và kiểm thử hơn.

- **S** - Single Responsibility Principle (SRP)
- **O** - Open/Closed Principle (OCP)
- **L** - Liskov Substitution Principle (LSP)
- **I** - Interface Segregation Principle (ISP)
- **D** - Dependency Inversion Principle (DIP)

---

## 1. Single Responsibility Principle (SRP)

### Định Nghĩa

Một class chỉ nên có một lý do duy nhất để thay đổi, tức là chỉ đảm nhận một trách nhiệm duy nhất.

### Tại Sao Quan Trọng?

- Code dễ đọc và bảo trì hơn
- Giảm coupling giữa các module
- Dễ dàng test từng chức năng riêng biệt
- Tránh side effects khi sửa đổi

### Ví Dụ Vi Phạm SRP

```javascript
// ❌ BAD: Class này có quá nhiều trách nhiệm
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  // Trách nhiệm 1: Quản lý thông tin user
  getUserInfo() {
    return `${this.name} - ${this.email}`;
  }

  // Trách nhiệm 2: Lưu vào database
  saveToDatabase() {
    console.log("Saving to database...");
    // Database logic
  }

  // Trách nhiệm 3: Gửi email
  sendWelcomeEmail() {
    console.log(`Sending email to ${this.email}`);
    // Email logic
  }

  // Trách nhiệm 4: Validate dữ liệu
  validateEmail() {
    return /\S+@\S+\.\S+/.test(this.email);
  }
}
```

### Ví Dụ Tuân Thủ SRP

```javascript
// ✅ GOOD: Tách thành nhiều class, mỗi class một trách nhiệm

// Class 1: Chỉ quản lý thông tin user
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  getUserInfo() {
    return `${this.name} - ${this.email}`;
  }
}

// Class 2: Chỉ xử lý database
class UserRepository {
  save(user) {
    console.log("Saving to database...");
    // Database logic
  }

  findById(id) {
    // Find logic
  }
}

// Class 3: Chỉ xử lý email
class EmailService {
  sendWelcomeEmail(user) {
    console.log(`Sending email to ${user.email}`);
    // Email logic
  }
}

// Class 4: Chỉ validate
class UserValidator {
  validateEmail(email) {
    return /\S+@\S+\.\S+/.test(email);
  }

  validateName(name) {
    return name && name.length > 0;
  }
}

// Sử dụng
const user = new User("John Doe", "john@example.com");
const validator = new UserValidator();
const repository = new UserRepository();
const emailService = new EmailService();

if (validator.validateEmail(user.email)) {
  repository.save(user);
  emailService.sendWelcomeEmail(user);
}
```

---

## 2. Open/Closed Principle (OCP)

### Định Nghĩa

Software entities (classes, modules, functions) nên mở cho việc mở rộng (open for extension) nhưng đóng cho việc sửa đổi (closed for modification).

### Tại Sao Quan Trọng?

- Thêm tính năng mới mà không làm hỏng code cũ
- Giảm rủi ro khi thay đổi
- Code ổn định hơn

### Ví Dụ Vi Phạm OCP

```javascript
// ❌ BAD: Mỗi lần thêm hình mới phải sửa class
class AreaCalculator {
  calculateArea(shapes) {
    let totalArea = 0;

    for (const shape of shapes) {
      if (shape.type === "circle") {
        totalArea += Math.PI * shape.radius ** 2;
      } else if (shape.type === "rectangle") {
        totalArea += shape.width * shape.height;
      } else if (shape.type === "triangle") {
        totalArea += (shape.base * shape.height) / 2;
      }
      // Thêm hình mới = phải sửa code này!
    }

    return totalArea;
  }
}
```

### Ví Dụ Tuân Thủ OCP

```javascript
// ✅ GOOD: Sử dụng polymorphism, thêm hình mới không cần sửa code cũ

// Base class/interface
class Shape {
  area() {
    throw new Error("Method area() must be implemented");
  }
}

// Các class con implement area()
class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  area() {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

class Triangle extends Shape {
  constructor(base, height) {
    super();
    this.base = base;
    this.height = height;
  }

  area() {
    return (this.base * this.height) / 2;
  }
}

// Calculator không cần sửa khi thêm hình mới
class AreaCalculator {
  calculateArea(shapes) {
    return shapes.reduce((total, shape) => total + shape.area(), 0);
  }
}

// Thêm hình mới - không cần sửa code cũ
class Pentagon extends Shape {
  constructor(side, apothem) {
    super();
    this.side = side;
    this.apothem = apothem;
  }

  area() {
    return (5 * this.side * this.apothem) / 2;
  }
}

// Sử dụng
const shapes = [new Circle(5), new Rectangle(4, 6), new Triangle(3, 4), new Pentagon(5, 3.5)];

const calculator = new AreaCalculator();
console.log(calculator.calculateArea(shapes));
```

---

## 3. Liskov Substitution Principle (LSP)

### Định Nghĩa

Các object của class con phải có thể thay thế object của class cha mà không làm thay đổi tính đúng đắn của chương trình.

### Tại Sao Quan Trọng?

- Đảm bảo tính nhất quán trong inheritance
- Tránh các hành vi bất ngờ
- Code dễ dự đoán hơn

### Ví Dụ Vi Phạm LSP

```javascript
// ❌ BAD: Square vi phạm LSP vì thay đổi hành vi của Rectangle
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width) {
    this.width = width;
    this.height = width; // Vi phạm LSP!
  }

  setHeight(height) {
    this.width = height; // Vi phạm LSP!
    this.height = height;
  }
}

// Test cho thấy vi phạm LSP
function testRectangle(rectangle) {
  rectangle.setWidth(5);
  rectangle.setHeight(4);
  console.log(`Expected area: 20, Got: ${rectangle.getArea()}`);
}

const rect = new Rectangle(0, 0);
testRectangle(rect); // ✅ Output: Expected area: 20, Got: 20

const square = new Square(0, 0);
testRectangle(square); // ❌ Output: Expected area: 20, Got: 16
```

### Ví Dụ Tuân Thủ LSP

```javascript
// ✅ GOOD: Tách riêng, không dùng inheritance sai

class Shape {
  getArea() {
    throw new Error("Method must be implemented");
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(side) {
    super();
    this.side = side;
  }

  setSide(side) {
    this.side = side;
  }

  getArea() {
    return this.side * this.side;
  }
}

// Hoặc sử dụng composition
class Square {
  constructor(side) {
    this.side = side;
  }

  setSide(side) {
    this.side = side;
  }

  getArea() {
    return this.side * this.side;
  }
}
```

### Ví Dụ Khác: Bird

```javascript
// ❌ BAD
class Bird {
  fly() {
    console.log("Flying...");
  }
}

class Penguin extends Bird {
  fly() {
    throw new Error("Penguins cannot fly!"); // Vi phạm LSP
  }
}

// ✅ GOOD
class Bird {
  eat() {
    console.log("Eating...");
  }
}

class FlyingBird extends Bird {
  fly() {
    console.log("Flying...");
  }
}

class Penguin extends Bird {
  swim() {
    console.log("Swimming...");
  }
}

class Eagle extends FlyingBird {
  // Có thể bay
}
```

---

## 4. Interface Segregation Principle (ISP)

### Định Nghĩa

Không nên ép client phải phụ thuộc vào các interface mà chúng không sử dụng. Nên chia interface lớn thành nhiều interface nhỏ, cụ thể hơn.

### Tại Sao Quan Trọng?

- Giảm coupling
- Class chỉ implement những gì cần thiết
- Dễ bảo trì và mở rộng

### Ví Dụ Vi Phạm ISP

```javascript
// ❌ BAD: Interface quá lớn, ép các class implement những method không cần
class Worker {
  work() {
    throw new Error("Must implement");
  }

  eat() {
    throw new Error("Must implement");
  }

  sleep() {
    throw new Error("Must implement");
  }

  getSalary() {
    throw new Error("Must implement");
  }
}

class HumanWorker extends Worker {
  work() {
    console.log("Human working...");
  }

  eat() {
    console.log("Human eating...");
  }

  sleep() {
    console.log("Human sleeping...");
  }

  getSalary() {
    return 5000;
  }
}

class RobotWorker extends Worker {
  work() {
    console.log("Robot working...");
  }

  eat() {
    // Robot không ăn! Vi phạm ISP
    throw new Error("Robot does not eat");
  }

  sleep() {
    // Robot không ngủ! Vi phạm ISP
    throw new Error("Robot does not sleep");
  }

  getSalary() {
    return 0;
  }
}
```

### Ví Dụ Tuân Thủ ISP

```javascript
// ✅ GOOD: Chia thành nhiều interface nhỏ

// Interface nhỏ, cụ thể
class Workable {
  work() {
    throw new Error("Must implement");
  }
}

class Eatable {
  eat() {
    throw new Error("Must implement");
  }
}

class Sleepable {
  sleep() {
    throw new Error("Must implement");
  }
}

class Payable {
  getSalary() {
    throw new Error("Must implement");
  }
}

// Human implement tất cả
class HumanWorker extends Workable {
  constructor() {
    super();
    this.eatable = new HumanEatable();
    this.sleepable = new HumanSleepable();
    this.payable = new HumanPayable();
  }

  work() {
    console.log("Human working...");
  }
}

class HumanEatable extends Eatable {
  eat() {
    console.log("Human eating...");
  }
}

class HumanSleepable extends Sleepable {
  sleep() {
    console.log("Human sleeping...");
  }
}

class HumanPayable extends Payable {
  getSalary() {
    return 5000;
  }
}

// Robot chỉ implement những gì cần
class RobotWorker extends Workable {
  work() {
    console.log("Robot working...");
  }

  charge() {
    console.log("Robot charging...");
  }
}
```

### Ví Dụ Thực Tế: Printer

```javascript
// ❌ BAD
class AllInOnePrinter {
  print() {}
  scan() {}
  fax() {}
  staple() {}
}

class SimplePrinter extends AllInOnePrinter {
  print() {
    console.log("Printing...");
  }

  scan() {
    throw new Error("Not supported"); // Vi phạm ISP
  }

  fax() {
    throw new Error("Not supported"); // Vi phạm ISP
  }

  staple() {
    throw new Error("Not supported"); // Vi phạm ISP
  }
}

// ✅ GOOD
class Printer {
  print() {
    throw new Error("Must implement");
  }
}

class Scanner {
  scan() {
    throw new Error("Must implement");
  }
}

class Fax {
  fax() {
    throw new Error("Must implement");
  }
}

class SimplePrinter extends Printer {
  print() {
    console.log("Printing...");
  }
}

class MultiFunctionPrinter extends Printer {
  constructor() {
    super();
    this.scanner = new ScannerImpl();
    this.fax = new FaxImpl();
  }

  print() {
    console.log("Printing...");
  }
}

class ScannerImpl extends Scanner {
  scan() {
    console.log("Scanning...");
  }
}

class FaxImpl extends Fax {
  fax() {
    console.log("Faxing...");
  }
}
```

---

## 5. Dependency Inversion Principle (DIP)

### Định Nghĩa

- High-level modules không nên phụ thuộc vào low-level modules. Cả hai nên phụ thuộc vào abstractions.
- Abstractions không nên phụ thuộc vào details. Details nên phụ thuộc vào abstractions.

### Tại Sao Quan Trọng?

- Giảm coupling giữa các module
- Dễ dàng thay đổi implementation
- Dễ test với mock/stub
- Code linh hoạt hơn

### Ví Dụ Vi Phạm DIP

```javascript
// ❌ BAD: High-level module phụ thuộc trực tiếp vào low-level module

// Low-level modules
class MySQLDatabase {
  connect() {
    console.log("Connecting to MySQL...");
  }

  query(sql) {
    console.log(`Executing MySQL query: ${sql}`);
  }
}

// High-level module phụ thuộc vào MySQLDatabase cụ thể
class UserService {
  constructor() {
    this.database = new MySQLDatabase(); // Vi phạm DIP!
  }

  getUser(id) {
    this.database.connect();
    this.database.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// Nếu muốn đổi sang MongoDB phải sửa UserService!
```

### Ví Dụ Tuân Thủ DIP

```javascript
// ✅ GOOD: Cả hai phụ thuộc vào abstraction

// Abstraction (Interface)
class Database {
  connect() {
    throw new Error("Must implement");
  }

  query(sql) {
    throw new Error("Must implement");
  }
}

// Low-level modules implement abstraction
class MySQLDatabase extends Database {
  connect() {
    console.log("Connecting to MySQL...");
  }

  query(sql) {
    console.log(`Executing MySQL query: ${sql}`);
  }
}

class MongoDBDatabase extends Database {
  connect() {
    console.log("Connecting to MongoDB...");
  }

  query(query) {
    console.log(`Executing MongoDB query: ${JSON.stringify(query)}`);
  }
}

class PostgreSQLDatabase extends Database {
  connect() {
    console.log("Connecting to PostgreSQL...");
  }

  query(sql) {
    console.log(`Executing PostgreSQL query: ${sql}`);
  }
}

// High-level module phụ thuộc vào abstraction
class UserService {
  constructor(database) {
    this.database = database; // Dependency Injection
  }

  getUser(id) {
    this.database.connect();
    this.database.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// Sử dụng - dễ dàng thay đổi database
const mysqlDb = new MySQLDatabase();
const userService1 = new UserService(mysqlDb);
userService1.getUser(1);

const mongoDb = new MongoDBDatabase();
const userService2 = new UserService(mongoDb);
userService2.getUser(1);

// Dễ dàng test với mock
class MockDatabase extends Database {
  connect() {
    console.log("Mock connect");
  }

  query(sql) {
    console.log("Mock query");
    return { id: 1, name: "Test User" };
  }
}

const mockDb = new MockDatabase();
const userServiceTest = new UserService(mockDb);
```

### Ví Dụ Thực Tế: Payment System

```javascript
// ❌ BAD
class PaymentService {
  processPayment(amount) {
    const stripe = new StripePayment(); // Vi phạm DIP
    stripe.charge(amount);
  }
}

// ✅ GOOD
class PaymentProcessor {
  charge(amount) {
    throw new Error("Must implement");
  }
}

class StripePayment extends PaymentProcessor {
  charge(amount) {
    console.log(`Charging ${amount} via Stripe`);
  }
}

class PayPalPayment extends PaymentProcessor {
  charge(amount) {
    console.log(`Charging ${amount} via PayPal`);
  }
}

class VNPayPayment extends PaymentProcessor {
  charge(amount) {
    console.log(`Charging ${amount} via VNPay`);
  }
}

class PaymentService {
  constructor(paymentProcessor) {
    this.paymentProcessor = paymentProcessor;
  }

  processPayment(amount) {
    this.paymentProcessor.charge(amount);
  }
}

// Sử dụng
const stripeProcessor = new StripePayment();
const paymentService = new PaymentService(stripeProcessor);
paymentService.processPayment(100);

// Dễ dàng đổi sang PayPal
const paypalProcessor = new PayPalPayment();
const paymentService2 = new PaymentService(paypalProcessor);
paymentService2.processPayment(100);
```

---

## Tổng Kết

### So Sánh Nhanh

| Nguyên Tắc | Mục Đích                          | Khi Nào Vi Phạm                           |
| ---------- | --------------------------------- | ----------------------------------------- |
| **SRP**    | Một class một trách nhiệm         | Class làm quá nhiều việc                  |
| **OCP**    | Mở rộng không sửa đổi             | Thêm feature phải sửa code cũ             |
| **LSP**    | Subclass thay thế được superclass | Subclass thay đổi hành vi cha             |
| **ISP**    | Interface nhỏ, cụ thể             | Interface quá lớn, ép implement không cần |
| **DIP**    | Phụ thuộc vào abstraction         | Phụ thuộc trực tiếp vào concrete class    |

### Lợi Ích Khi Áp Dụng SOLID

1. **Maintainability**: Code dễ bảo trì, sửa lỗi
2. **Scalability**: Dễ mở rộng tính năng mới
3. **Testability**: Dễ viết unit test
4. **Flexibility**: Linh hoạt thay đổi implementation
5. **Reusability**: Tái sử dụng code tốt hơn
6. **Readability**: Code dễ đọc, dễ hiểu

### Khi Nào Nên Áp Dụng?

- ✅ Dự án lớn, phức tạp
- ✅ Team nhiều người
- ✅ Dự án dài hạn
- ✅ Cần mở rộng thường xuyên
- ⚠️ Prototype, POC có thể linh hoạt hơn
- ⚠️ Script nhỏ, one-time không cần quá strict

### Tips Thực Hành

1. Không cần áp dụng tất cả ngay từ đầu
2. Refactor dần dần khi code phình to
3. Cân nhắc trade-off giữa SOLID và simplicity
4. Học cách nhận biết code smell
5. Review code thường xuyên
6. Viết test để đảm bảo refactor an toàn

---

## Tài Liệu Tham Khảo

- Clean Code - Robert C. Martin
- Clean Architecture - Robert C. Martin
- Design Patterns - Gang of Four
- SOLID Principles of Object-Oriented Design
