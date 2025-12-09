# Service Layer Pattern

## 1. Khái niệm

**Service Layer Pattern** là pattern tạo một lớp trung gian chứa business logic giữa Controller và Data Access Layer (Repository/Model). Service đóng vai trò điều phối và xử lý nghiệp vụ.

### Tại sao cần Service Layer?

**Vấn đề: Fat Controller**

```javascript
// ❌ Bad: Controller chứa quá nhiều logic
class OrderController {
  async createOrder(req, res) {
    const { userId, items } = req.body;

    // Validation
    if (!items || items.length === 0) {
      return res.status(400).json({ error: "Items required" });
    }

    // Check user
    const user = await User.findById(userId);
    if (!user) {
      return res.status(404).json({ error: "User not found" });
    }

    // Check stock cho từng item
    for (const item of items) {
      const product = await Product.findById(item.productId);
      if (!product) {
        return res.status(404).json({ error: `Product ${item.productId} not found` });
      }
      if (product.stock < item.quantity) {
        return res.status(400).json({ error: `${product.name} hết hàng` });
      }
    }

    // Tính tổng tiền
    let total = 0;
    for (const item of items) {
      const product = await Product.findById(item.productId);
      total += product.price * item.quantity;
    }

    // Áp dụng discount
    if (user.membershipLevel === "gold") {
      total *= 0.9; // Giảm 10%
    } else if (user.membershipLevel === "silver") {
      total *= 0.95; // Giảm 5%
    }

    // Xử lý thanh toán
    const paymentResult = await PaymentGateway.charge(user.paymentMethod, total);
    if (!paymentResult.success) {
      return res.status(400).json({ error: "Thanh toán thất bại" });
    }

    // Tạo order
    const order = await Order.create({
      userId,
      items,
      total,
      paymentId: paymentResult.id,
      status: "confirmed",
    });

    // Cập nhật stock
    for (const item of items) {
      await Product.findByIdAndUpdate(item.productId, {
        $inc: { stock: -item.quantity },
      });
    }

    // Gửi email xác nhận
    await EmailService.send(user.email, "Xác nhận đơn hàng", { order });

    res.status(201).json({ data: order });
  }
}
```

**Vấn đề với Fat Controller:**

- Controller quá dài, khó đọc
- Business logic lẫn với HTTP handling
- Khó unit test (phải mock req, res)
- Không thể reuse logic ở nơi khác
- Khó maintain khi logic phức tạp

**Service Layer giải quyết:**

- Tách business logic ra Service
- Controller chỉ handle HTTP
- Service dễ unit test
- Logic có thể reuse

## 2. Cấu trúc

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌──────────┐
│ Controller  │────►│   Service   │────►│ Repository  │────►│ Database │
│   (HTTP)    │     │ (Business)  │     │   (Data)    │     │          │
└─────────────┘     └─────────────┘     └─────────────┘     └──────────┘
       │                   │
       │                   ├────► External Services
       │                   │      (Payment, Email...)
       │                   │
       ▼                   ▼
   Validate           Business
   Request            Logic
```

## 3. Triển khai

### Service Layer

```javascript
// services/OrderService.js
class OrderService {
  constructor(orderRepository, userRepository, productRepository, paymentService, emailService) {
    this.orderRepository = orderRepository;
    this.userRepository = userRepository;
    this.productRepository = productRepository;
    this.paymentService = paymentService;
    this.emailService = emailService;
  }

  /**
   * Tạo đơn hàng mới
   * @param {string} userId - ID người dùng
   * @param {Array} items - Danh sách sản phẩm [{productId, quantity}]
   * @returns {Order} - Đơn hàng đã tạo
   */
  async createOrder(userId, items) {
    // 1. Validate input
    this.validateItems(items);

    // 2. Lấy thông tin user
    const user = await this.getUser(userId);

    // 3. Validate stock và lấy products
    const products = await this.validateStockAndGetProducts(items);

    // 4. Tính tổng tiền (có discount)
    const total = this.calculateTotal(items, products, user);

    // 5. Xử lý thanh toán
    const paymentResult = await this.processPayment(user, total);

    // 6. Tạo order
    const order = await this.orderRepository.create({
      userId,
      items,
      total,
      paymentId: paymentResult.id,
      status: "confirmed",
      createdAt: new Date(),
    });

    // 7. Cập nhật stock
    await this.updateStock(items);

    // 8. Gửi email (async, không chờ)
    this.sendConfirmationEmail(user, order);

    return order;
  }

  /**
   * Validate danh sách items
   */
  validateItems(items) {
    if (!items || !Array.isArray(items) || items.length === 0) {
      throw new ValidationError("Danh sách sản phẩm không được trống");
    }

    for (const item of items) {
      if (!item.productId) {
        throw new ValidationError("productId là bắt buộc");
      }
      if (!item.quantity || item.quantity < 1) {
        throw new ValidationError("quantity phải >= 1");
      }
    }
  }

  /**
   * Lấy thông tin user
   */
  async getUser(userId) {
    const user = await this.userRepository.findById(userId);
    if (!user) {
      throw new NotFoundError("Không tìm thấy người dùng");
    }
    return user;
  }

  /**
   * Validate stock và lấy danh sách products
   */
  async validateStockAndGetProducts(items) {
    const products = [];

    for (const item of items) {
      const product = await this.productRepository.findById(item.productId);

      if (!product) {
        throw new NotFoundError(`Không tìm thấy sản phẩm: ${item.productId}`);
      }

      if (product.stock < item.quantity) {
        throw new ValidationError(`Sản phẩm "${product.name}" chỉ còn ${product.stock} trong kho`);
      }

      products.push(product);
    }

    return products;
  }

  /**
   * Tính tổng tiền với discount theo membership
   */
  calculateTotal(items, products, user) {
    let total = 0;

    for (let i = 0; i < items.length; i++) {
      const product = products[i];
      const quantity = items[i].quantity;
      total += product.price * quantity;
    }

    // Áp dụng discount theo membership
    const discountRate = this.getDiscountRate(user.membershipLevel);
    total = total * (1 - discountRate);

    // Làm tròn 2 chữ số
    return Math.round(total * 100) / 100;
  }

  /**
   * Lấy tỷ lệ discount theo membership level
   */
  getDiscountRate(membershipLevel) {
    const discounts = {
      gold: 0.1, // 10%
      silver: 0.05, // 5%
      bronze: 0.02, // 2%
      normal: 0, // 0%
    };
    return discounts[membershipLevel] || 0;
  }

  /**
   * Xử lý thanh toán
   */
  async processPayment(user, total) {
    const result = await this.paymentService.charge(user.paymentMethod, total, { userId: user.id });

    if (!result.success) {
      throw new PaymentError("Thanh toán thất bại: " + result.message);
    }

    return result;
  }

  /**
   * Cập nhật stock sau khi đặt hàng
   */
  async updateStock(items) {
    for (const item of items) {
      await this.productRepository.decrementStock(item.productId, item.quantity);
    }
  }

  /**
   * Gửi email xác nhận (async)
   */
  sendConfirmationEmail(user, order) {
    this.emailService.sendOrderConfirmation(user.email, order).catch((err) => {
      console.error("Gửi email thất bại:", err);
      // Không throw error, order vẫn thành công
    });
  }

  /**
   * Lấy order theo ID
   */
  async getOrderById(orderId) {
    const order = await this.orderRepository.findById(orderId);
    if (!order) {
      throw new NotFoundError("Không tìm thấy đơn hàng");
    }
    return order;
  }

  /**
   * Lấy danh sách orders của user
   */
  async getOrdersByUser(userId) {
    return this.orderRepository.findByUserId(userId);
  }

  /**
   * Hủy đơn hàng
   */
  async cancelOrder(orderId, userId) {
    const order = await this.getOrderById(orderId);

    // Check quyền
    if (order.userId !== userId) {
      throw new ForbiddenError("Bạn không có quyền hủy đơn hàng này");
    }

    // Check trạng thái
    if (order.status !== "confirmed") {
      throw new ValidationError("Chỉ có thể hủy đơn hàng đang chờ xử lý");
    }

    // Hoàn tiền
    await this.paymentService.refund(order.paymentId);

    // Hoàn stock
    for (const item of order.items) {
      await this.productRepository.incrementStock(item.productId, item.quantity);
    }

    // Cập nhật trạng thái
    return this.orderRepository.update(orderId, {
      status: "cancelled",
      cancelledAt: new Date(),
    });
  }
}

module.exports = OrderService;
```

### Controller (Thin)

```javascript
// controllers/OrderController.js
class OrderController {
  constructor(orderService) {
    this.orderService = orderService;
  }

  /**
   * POST /orders
   * Tạo đơn hàng mới
   */
  async create(req, res, next) {
    try {
      const userId = req.user.id; // Từ auth middleware
      const { items } = req.body;

      const order = await this.orderService.createOrder(userId, items);

      res.status(201).json({
        success: true,
        message: "Đặt hàng thành công",
        data: order,
      });
    } catch (error) {
      next(error);
    }
  }

  /**
   * GET /orders/:id
   * Lấy chi tiết đơn hàng
   */
  async getById(req, res, next) {
    try {
      const order = await this.orderService.getOrderById(req.params.id);

      res.json({
        success: true,
        data: order,
      });
    } catch (error) {
      next(error);
    }
  }

  /**
   * GET /orders
   * Lấy danh sách đơn hàng của user
   */
  async getMyOrders(req, res, next) {
    try {
      const orders = await this.orderService.getOrdersByUser(req.user.id);

      res.json({
        success: true,
        data: orders,
      });
    } catch (error) {
      next(error);
    }
  }

  /**
   * POST /orders/:id/cancel
   * Hủy đơn hàng
   */
  async cancel(req, res, next) {
    try {
      const order = await this.orderService.cancelOrder(req.params.id, req.user.id);

      res.json({
        success: true,
        message: "Đã hủy đơn hàng",
        data: order,
      });
    } catch (error) {
      next(error);
    }
  }
}

module.exports = OrderController;
```

## 4. Custom Errors

```javascript
// errors/AppErrors.js

class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message) {
    super(message, 400);
  }
}

class NotFoundError extends AppError {
  constructor(message) {
    super(message, 404);
  }
}

class ForbiddenError extends AppError {
  constructor(message) {
    super(message, 403);
  }
}

class PaymentError extends AppError {
  constructor(message) {
    super(message, 402);
  }
}

module.exports = {
  AppError,
  ValidationError,
  NotFoundError,
  ForbiddenError,
  PaymentError,
};
```

## 5. Error Handler Middleware

```javascript
// middleware/errorHandler.js

function errorHandler(err, req, res, next) {
  console.error("Error:", err);

  // Operational errors (expected)
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      success: false,
      error: err.message,
    });
  }

  // Programming errors (unexpected)
  res.status(500).json({
    success: false,
    error: "Đã xảy ra lỗi, vui lòng thử lại sau",
  });
}

module.exports = errorHandler;
```

## 6. Dependency Injection

```javascript
// container.js
const OrderRepository = require("./repositories/OrderRepository");
const UserRepository = require("./repositories/UserRepository");
const ProductRepository = require("./repositories/ProductRepository");
const PaymentService = require("./services/PaymentService");
const EmailService = require("./services/EmailService");
const OrderService = require("./services/OrderService");
const OrderController = require("./controllers/OrderController");

// Repositories
const orderRepository = new OrderRepository();
const userRepository = new UserRepository();
const productRepository = new ProductRepository();

// External Services
const paymentService = new PaymentService(process.env.STRIPE_KEY);
const emailService = new EmailService(process.env.SMTP_CONFIG);

// Business Services
const orderService = new OrderService(orderRepository, userRepository, productRepository, paymentService, emailService);

// Controllers
const orderController = new OrderController(orderService);

module.exports = { orderController };
```

## 7. Routes

```javascript
// routes/orders.js
const express = require("express");
const router = express.Router();
const { orderController } = require("../container");
const { authenticate } = require("../middleware/auth");

// Tất cả routes cần authentication
router.use(authenticate);

// Bind methods để giữ context
router.post("/", (req, res, next) => orderController.create(req, res, next));
router.get("/", (req, res, next) => orderController.getMyOrders(req, res, next));
router.get("/:id", (req, res, next) => orderController.getById(req, res, next));
router.post("/:id/cancel", (req, res, next) => orderController.cancel(req, res, next));

module.exports = router;
```

## 8. Unit Testing Service

```javascript
// __tests__/OrderService.test.js
describe("OrderService", () => {
  let orderService;
  let mockOrderRepo, mockUserRepo, mockProductRepo;
  let mockPaymentService, mockEmailService;

  beforeEach(() => {
    // Mock repositories
    mockOrderRepo = {
      create: jest.fn(),
      findById: jest.fn(),
      findByUserId: jest.fn(),
      update: jest.fn(),
    };
    mockUserRepo = { findById: jest.fn() };
    mockProductRepo = {
      findById: jest.fn(),
      decrementStock: jest.fn(),
      incrementStock: jest.fn(),
    };

    // Mock services
    mockPaymentService = {
      charge: jest.fn(),
      refund: jest.fn(),
    };
    mockEmailService = {
      sendOrderConfirmation: jest.fn().mockResolvedValue(),
    };

    // Create service with mocks
    orderService = new OrderService(mockOrderRepo, mockUserRepo, mockProductRepo, mockPaymentService, mockEmailService);
  });

  describe("createOrder", () => {
    const userId = "user-1";
    const items = [{ productId: "prod-1", quantity: 2 }];
    const user = { id: userId, membershipLevel: "gold", paymentMethod: "card" };
    const product = { id: "prod-1", name: "iPhone", price: 1000, stock: 10 };

    it("should create order successfully", async () => {
      // Arrange
      mockUserRepo.findById.mockResolvedValue(user);
      mockProductRepo.findById.mockResolvedValue(product);
      mockPaymentService.charge.mockResolvedValue({ success: true, id: "pay-1" });
      mockOrderRepo.create.mockResolvedValue({
        id: "order-1",
        userId,
        items,
        total: 1800, // 2000 * 0.9 (gold discount)
        status: "confirmed",
      });

      // Act
      const result = await orderService.createOrder(userId, items);

      // Assert
      expect(result.id).toBe("order-1");
      expect(result.total).toBe(1800);
      expect(mockPaymentService.charge).toHaveBeenCalledWith("card", 1800, { userId });
      expect(mockProductRepo.decrementStock).toHaveBeenCalledWith("prod-1", 2);
    });

    it("should throw ValidationError when items empty", async () => {
      await expect(orderService.createOrder(userId, [])).rejects.toThrow(ValidationError);
    });

    it("should throw NotFoundError when user not found", async () => {
      mockUserRepo.findById.mockResolvedValue(null);

      await expect(orderService.createOrder(userId, items)).rejects.toThrow(NotFoundError);
    });

    it("should throw ValidationError when product out of stock", async () => {
      mockUserRepo.findById.mockResolvedValue(user);
      mockProductRepo.findById.mockResolvedValue({ ...product, stock: 1 });

      await expect(orderService.createOrder(userId, items)).rejects.toThrow(ValidationError);
    });

    it("should throw PaymentError when payment fails", async () => {
      mockUserRepo.findById.mockResolvedValue(user);
      mockProductRepo.findById.mockResolvedValue(product);
      mockPaymentService.charge.mockResolvedValue({
        success: false,
        message: "Card declined",
      });

      await expect(orderService.createOrder(userId, items)).rejects.toThrow(PaymentError);
    });
  });

  describe("calculateTotal", () => {
    it("should apply gold discount (10%)", () => {
      const items = [{ productId: "1", quantity: 2 }];
      const products = [{ price: 100 }];
      const user = { membershipLevel: "gold" };

      const total = orderService.calculateTotal(items, products, user);

      expect(total).toBe(180); // 200 * 0.9
    });

    it("should apply silver discount (5%)", () => {
      const items = [{ productId: "1", quantity: 2 }];
      const products = [{ price: 100 }];
      const user = { membershipLevel: "silver" };

      const total = orderService.calculateTotal(items, products, user);

      expect(total).toBe(190); // 200 * 0.95
    });

    it("should not apply discount for normal user", () => {
      const items = [{ productId: "1", quantity: 2 }];
      const products = [{ price: 100 }];
      const user = { membershipLevel: "normal" };

      const total = orderService.calculateTotal(items, products, user);

      expect(total).toBe(200);
    });
  });

  describe("cancelOrder", () => {
    it("should cancel order and refund", async () => {
      const order = {
        id: "order-1",
        userId: "user-1",
        status: "confirmed",
        paymentId: "pay-1",
        items: [{ productId: "prod-1", quantity: 2 }],
      };

      mockOrderRepo.findById.mockResolvedValue(order);
      mockPaymentService.refund.mockResolvedValue({ success: true });
      mockOrderRepo.update.mockResolvedValue({ ...order, status: "cancelled" });

      const result = await orderService.cancelOrder("order-1", "user-1");

      expect(result.status).toBe("cancelled");
      expect(mockPaymentService.refund).toHaveBeenCalledWith("pay-1");
      expect(mockProductRepo.incrementStock).toHaveBeenCalledWith("prod-1", 2);
    });

    it("should throw ForbiddenError when user not owner", async () => {
      mockOrderRepo.findById.mockResolvedValue({
        id: "order-1",
        userId: "other-user",
      });

      await expect(orderService.cancelOrder("order-1", "user-1")).rejects.toThrow(ForbiddenError);
    });
  });
});
```

## 9. Cấu trúc thư mục

```
src/
├── controllers/           # HTTP handling
│   ├── OrderController.js
│   ├── UserController.js
│   └── ProductController.js
├── services/              # Business logic
│   ├── OrderService.js
│   ├── UserService.js
│   ├── PaymentService.js
│   └── EmailService.js
├── repositories/          # Data access
│   ├── OrderRepository.js
│   ├── UserRepository.js
│   └── ProductRepository.js
├── models/                # Database models
│   ├── Order.js
│   ├── User.js
│   └── Product.js
├── errors/                # Custom errors
│   └── AppErrors.js
├── middleware/
│   ├── auth.js
│   ├── validate.js
│   └── errorHandler.js
├── routes/
│   ├── index.js
│   ├── orders.js
│   └── users.js
├── container.js           # Dependency Injection
└── app.js
```

## 10. Ưu điểm

| Ưu điểm                   | Giải thích                            |
| ------------------------- | ------------------------------------- |
| **Thin Controller**       | Controller chỉ handle HTTP, dễ đọc    |
| **Testable**              | Service dễ unit test với mock         |
| **Reusable**              | Logic có thể dùng ở nhiều controllers |
| **Maintainable**          | Business logic tập trung              |
| **Single Responsibility** | Mỗi layer có nhiệm vụ riêng           |

## 11. Nhược điểm

| Nhược điểm        | Giải thích                   |
| ----------------- | ---------------------------- |
| **More files**    | Thêm layer = thêm files      |
| **Overkill**      | Quá phức tạp cho simple CRUD |
| **Service bloat** | Service có thể phình to      |

## 12. Khi nào dùng?

### ✅ Phù hợp khi:

- Controller có > 50 lines logic
- Business logic cần reuse
- Cần unit test business logic
- Multiple entry points (API, CLI, Queue)
- Team lớn, cần clear boundaries

### ❌ Không phù hợp khi:

- Simple CRUD operations
- Logic chỉ dùng 1 chỗ
- Prototype nhanh

## 13. Tổng kết

**Service Layer là gì?** Layer chứa business logic giữa Controller và Repository.

**Tại sao dùng?** Thin controller, dễ test, reusable logic.

**Khi nào dùng?** Khi controller phình to, logic phức tạp.

**Nhớ:**

- Controller: HTTP handling only
- Service: Business logic
- Repository: Data access
- Errors: Custom error classes
- DI: Inject dependencies
