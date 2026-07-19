# Câu hỏi: Hãy trình bày 4 tính chất của lập trình hướng đối tượng (OOP)

## Trả lời

Lập trình hướng đối tượng (Object-Oriented Programming - OOP) là phương pháp lập trình tổ chức chương trình thành các **đối tượng (Object)**. Mỗi đối tượng bao gồm **thuộc tính (Attribute/Property)** và **phương thức (Method)** để mô tả dữ liệu cũng như hành vi của đối tượng đó.

OOP có 4 tính chất cốt lõi:

* Encapsulation (Đóng gói)
* Inheritance (Kế thừa)
* Polymorphism (Đa hình)
* Abstraction (Trừu tượng)

---

# 1. Encapsulation (Đóng gói)

## Khái niệm

Đóng gói là việc **gom dữ liệu và các phương thức thao tác với dữ liệu vào cùng một class**, đồng thời **ẩn những dữ liệu quan trọng** và chỉ cho phép truy cập thông qua các phương thức được cung cấp.

Nói đơn giản:

> Không cho phép bên ngoài sửa dữ liệu một cách tùy ý.

Ví dụ:

Một tài khoản ngân hàng có số dư.

Ta không muốn người khác viết:

```ts
account.balance = -100000;
```

vì điều này làm dữ liệu sai.

Thay vào đó:

```ts
class BankAccount {
  private balance = 0;

  deposit(amount: number) {
    if (amount > 0) {
      this.balance += amount;
    }
  }

  withdraw(amount: number) {
    if (amount <= this.balance) {
      this.balance -= amount;
    }
  }

  getBalance() {
    return this.balance;
  }
}
```

Người dùng chỉ có thể:

```ts
account.deposit(1000);
account.withdraw(500);

console.log(account.getBalance());
```

không thể sửa trực tiếp `balance`.

---

## Mục đích

Đóng gói giúp:

* Bảo vệ dữ liệu
* Kiểm soát dữ liệu đầu vào
* Tránh trạng thái không hợp lệ
* Giảm bug
* Dễ bảo trì

---

## Trong TypeScript

Thường sử dụng:

* private
* protected
* public

Ví dụ

```ts
private balance
```

chỉ class đó mới truy cập được.

---

## Ví dụ thực tế

Facebook không cho phép bạn sửa trực tiếp số lượt thích của bài viết.

Bạn chỉ có thể nhấn Like.

Bên trong hệ thống sẽ:

* kiểm tra đã like chưa
* tăng số lượng
* lưu database

Đây chính là Encapsulation.

---

# 2. Inheritance (Kế thừa)

## Khái niệm

Kế thừa là việc một class có thể sử dụng lại các thuộc tính và phương thức của class khác.

Mục tiêu:

> Tái sử dụng code.

Ví dụ:

```ts
class Animal {

  eat(){
      console.log("Eating")
  }
}

class Dog extends Animal{

}
```

Dog không cần viết lại:

```ts
eat()
```

vẫn sử dụng được.

---

## Có thể mở rộng

```ts
class Dog extends Animal{

    bark(){
        console.log("Woof")
    }

}
```

Dog vừa có:

* eat()

vừa có

* bark()

---

## Có thể ghi đè (Override)

```ts
class Animal{

    makeSound(){

        console.log("Animal")

    }

}

class Dog extends Animal{

    makeSound(){

        console.log("Woof")

    }

}
```

Đây là Override.

---

## Ưu điểm

* Giảm code lặp
* Dễ mở rộng
* Dễ bảo trì

---

## Nhược điểm

Nếu lạm dụng inheritance sẽ làm:

* Class cha quá lớn
* Quan hệ phụ thuộc phức tạp
* Khó bảo trì

Ngày nay người ta thường ưu tiên:

> Composition over Inheritance.

---

## Ví dụ thực tế

Hệ thống nhân viên.

```
Employee

|

|------ Developer

|

|------ Tester

|

|------ Manager
```

Tất cả đều có:

* name
* age
* salary

Nhưng:

Developer có:

* code()

Tester có:

* testing()

Manager có:

* approve()

---

# 3. Polymorphism (Đa hình)

## Khái niệm

Đa hình nghĩa là:

> Cùng một hành động nhưng các đối tượng khác nhau sẽ thực hiện theo cách khác nhau.

Ví dụ:

```ts
class Animal{

    makeSound(){}

}
```

Dog:

```
Woof
```

Cat

```
Meow
```

Cow

```
Moo
```

Đều gọi:

```ts
makeSound()
```

nhưng kết quả khác nhau.

---

Ví dụ:

```ts
const animals = [

 new Dog(),

 new Cat(),

 new Cow()

]

animals.forEach(a=>a.makeSound())
```

Không cần biết object là gì.

Chỉ cần gọi:

```ts
makeSound()
```

Đây chính là đa hình.

---

## Lợi ích

Code linh hoạt.

Không cần:

```ts
if(type=="dog")
```

hay

```ts
switch(type)
```

Liên tục.

---

## Ví dụ thực tế

Thanh toán.

```
Payment
```

có:

```ts
pay()
```

Các lớp con:

```
MomoPayment

VNPayPayment

PaypalPayment

StripePayment
```

Đều có:

```ts
pay()
```

Nhưng xử lý khác nhau.

---

# 4. Abstraction (Trừu tượng)

## Khái niệm

Trừu tượng là:

> Chỉ cho người dùng thấy những gì cần thiết, còn cách hệ thống hoạt động bên trong sẽ được che giấu.

Người dùng không cần biết:

* thuật toán
* database
* API
* mã hóa

Chỉ cần biết:

```ts
login()
```

là đăng nhập.

---

Ví dụ:

```ts
abstract class Payment{

    abstract pay(amount:number):void

}
```

Các lớp:

```
Momo

Visa

Paypal
```

đều phải cài đặt:

```ts
pay()
```

nhưng cách xử lý hoàn toàn khác nhau.

---

## Ví dụ đời sống

Bạn lái ô tô.

Bạn chỉ cần:

* ga
* phanh
* vô lăng

Bạn không cần biết:

* ECU
* Kim phun
* Hộp số
* Động cơ hoạt động ra sao

Đây chính là Abstraction.

---

# Phân biệt Encapsulation và Abstraction

Đây là câu follow-up rất hay gặp.

### Encapsulation

Mục tiêu:

> Bảo vệ dữ liệu.

Quan tâm:

> Ai được phép truy cập.

Ví dụ:

```ts
private balance
```

---

### Abstraction

Mục tiêu:

> Che giấu sự phức tạp.

Quan tâm:

> Người dùng chỉ cần biết cách sử dụng.

Ví dụ:

```ts
payment.pay()
```

không cần biết bên trong xử lý thế nào.

---

# Tóm tắt

| Tính chất     | Mục đích                             | Ví dụ                         |
| ------------- | ------------------------------------ | ----------------------------- |
| Encapsulation | Bảo vệ dữ liệu                       | `private`, getter/setter      |
| Inheritance   | Tái sử dụng code                     | `extends`                     |
| Polymorphism  | Một interface, nhiều cách triển khai | Override `makeSound()`        |
| Abstraction   | Ẩn chi tiết triển khai               | `abstract class`, `interface` |

# Cách trả lời khi phỏng vấn

> OOP có 4 tính chất cốt lõi. **Đóng gói (Encapsulation)** giúp bảo vệ dữ liệu và kiểm soát cách dữ liệu được truy cập thông qua các phương thức. **Kế thừa (Inheritance)** cho phép lớp con tái sử dụng thuộc tính và hành vi của lớp cha, giúp giảm lặp code. **Đa hình (Polymorphism)** cho phép cùng một phương thức hoặc interface nhưng mỗi đối tượng có cách triển khai khác nhau, giúp hệ thống linh hoạt và dễ mở rộng. **Trừu tượng (Abstraction)** giúp che giấu các chi tiết triển khai phức tạp, chỉ cung cấp cho người dùng những chức năng cần thiết thông qua interface hoặc abstract class. Bốn tính chất này kết hợp với nhau giúp mã nguồn dễ bảo trì, tái sử dụng và mở rộng hơn.
